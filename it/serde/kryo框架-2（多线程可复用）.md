# kryo 多线程可复用

## 线程安全

Kryo实例不是线程安全的对象，参见官方文档：https://github.com/EsotericSoftware/kryo#threading

文档：

----------

**Kryo is not thread safe. Each thread should have its own Kryo, Input, and Output instances. Also, the byte[] Input uses may be modified and then returned to its original state during deserialization, so the same byte[] "should not be used concurrently in separate threads.**

----------

每个线程需要独占一个Kryo实例，我们可以使用ThreadLocal获取Kryo，从而达到Kryo线程安全。

```java

/**
 * 线程安全
 *
 * @author mojunbin
 * @since 2014-12-29
 */
public class ThreadKryoFactory {

    private static ThreadLocal<Kryo> kryos = new ThreadLocal<Kryo>() {

        protected Kryo initialValue() {

            Kryo kryo = new Kryo();
            return kryo;
        }
    };

    public static Kryo create() {

        return kryos.get();
    }

    public static void remote() {

        kryos.remove();
    }
}

```

当线程需要kryo实例时，使用ThreadKryoFactory#create创建kryo。
当线程销毁时，使用ThreadKryoFactory#remove销毁kryo。

开发过程中使用ThreadKryoFactory可以保证线程安全，但仍有不足。

# kryo 对象池

Kryo对象是**可复用**的，详情参见kryo源码。

假设有线程T1和T2，Kryo实例k1。当T1使用完k1后，T2可继续使用k1。但T1,T2不能同时使用k1，Kryo对象**非线程安全**。


可以利用对象池技术，预先创建一批Kryo对象。


```java

/**
 * 对象池
 * 线程安全
 *
 * @author mjb
 * @since 2015-01-05
 */
public abstract class SimplePool<T> {

    private Queue<T> queue;

    public SimplePool() {

        queue = new ConcurrentLinkedQueue<T>();
    }

    public T borrow() {

        T res;
        if ((res = queue.poll()) != null) {
            return res;
        }

        return newInstance();
    }

    public void release(T t) {

        queue.offer(t);
    }

    protected abstract T newInstance();
}

/**
 * 简单Kryo对象池
 */
class SimpleKryoPool extends SimplePool<Kryo> {
    @Override
    protected Kryo newInstance() {

        return new Kryo();
    }
}

```
SimpleKryoPool可以复用Kryo对象，且SimpleKryoPool是线程安全的，多线程可以直接使用SimpleKryoPool#borrow得到Kryo对象，使用完毕后通过SimpleKryoPool#release返还到对象池，以供其它线程使用。


# Kryo 3.0 自带对象池

SimpleKryoPool功能上没问题，但性能上有缺陷：
当系统遇到高并发时，会创建大量Kryo对象；当系统闲置时，先前创建的大量Kryo闲置在Queue中，造成资源浪费。


要控制对象池的资源利用率，可以使用专业的开源框架**commons-pool**，可以参照Redis客户端Jedis的JedisPool实现。
Commons Pool官网：http://commons.apache.org/proper/commons-pool/

Kryo 3.0 起，作者实现了简单的KryoPool。资源的释放通过**软引用（SoftReference）**实现，当JVM发生GC且内存不足时，软引用对象会被销毁，从而达到释放资源目的。

Kryo官方KryoPool：https://github.com/EsotericSoftware/kryo#pooling-kryo-instances

使用示例：

```java
/**
 * @author mjb
 * @since 2015-01-05
 */
public class KryoPoolDemo {

    private static KryoPool kryoPool = new KryoPool.Builder(new KryoFactory() {
        @Override
        public Kryo create() {

            return new Kryo();
        }
    }).build();


    public static void main(String[] args) {

        demo1();
        demo2();

    }

    private static void demo1() {

        Kryo kryo = null;

        try {
            kryo = kryoPool.borrow();
            //使用实例
        } finally {
            kryoPool.release(kryo);
        }
    }

    /**
     * 自动获取kryo，自动释放kryo
     */
    private static void demo2() {

        byte[] buf = null;
        buf = kryoPool.run(new KryoCallback<byte[]>() {
            @Override
            public byte[] execute(Kryo kryo) {

                ByteArrayOutputStream bos = new ByteArrayOutputStream();
                Output output = new Output(bos);
                Object obj = new Object(); // 替换成实例对象
                kryo.writeObject(output, obj);
                output.close();
                return bos.toByteArray();
            }
        });

    }

}

```

demo1和demo2分别是KryoPool两种使用方式。

KryoPool是线程安全的。

KryoPool可通过软引用管理资源，使用new KryoPool.Builder(KryoFactory)..softReferences().build()即可。
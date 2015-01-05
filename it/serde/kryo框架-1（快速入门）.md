# Kryo快速入门


# Kryo是什么

kryo是一套针对java平台的序列化框架，它的性能相当好，基准测试请看：https://github.com/eishay/jvm-serializers/wiki

通过maven使用：

```xml
<dependency>
    <groupId>com.esotericsoftware.kryo</groupId>
    <artifactId>kryo</artifactId>
    <version>2.24.0</version>
</dependency>
```

## 快速入门

实体LoginLog：

```java
public class LoginLog {

    private long loginLogId;// 主键
    private long userId;    // 登录用户
    private Date loginAt;   // 登录时间
    private int loginType;  // 登录类型

    //无参构造函数
    public LoginLog() {

    }

    //省略方法：geter,seter,toString,equals,hashCode
}
```

使用Kryo序列化LoginLog对象：

```java
Log.TRACE();//日志级别

Kryo kryo = new Kryo();

LoginLog loginLog = new LoginLog();
loginLog.setLoginLogId(123456);
loginLog.setUserId(789);
loginLog.setLoginAt(new Date());

Output output = new Output(new FileOutputStream("login.bin"));
kryo.writeObject(output,loginLog); //序列化
output.close();

Input input = new Input(new FileInputStream("login.bin"));
LoginLog loginLog1 = kryo.readObject(input, LoginLog.class);
input.close();

System.out.println("loginLog = " + loginLog);
System.out.println("loginLog1 = " + loginLog1);
System.out.println("[loginLog == loginLog2] = " + (loginLog == loginLog1));
System.out.println("[loginLog.equals(loginLog1)] = " + (loginLog.equals(loginLog1)));
```

输出结果（手动空行格式化不同输出）：

----------

00:00 TRACE: [kryo] Register class ID 0: int (com.esotericsoftware.kryo.serializers.DefaultSerializers$IntSerializer)
00:00 TRACE: [kryo] Register class ID 1: String (com.esotericsoftware.kryo.serializers.DefaultSerializers$StringSerializer)
00:00 TRACE: [kryo] Register class ID 2: float (com.esotericsoftware.kryo.serializers.DefaultSerializers$FloatSerializer)
00:00 TRACE: [kryo] Register class ID 3: boolean (com.esotericsoftware.kryo.serializers.DefaultSerializers$BooleanSerializer)
00:00 TRACE: [kryo] Register class ID 4: byte (com.esotericsoftware.kryo.serializers.DefaultSerializers$ByteSerializer)
00:00 TRACE: [kryo] Register class ID 5: char (com.esotericsoftware.kryo.serializers.DefaultSerializers$CharSerializer)
00:00 TRACE: [kryo] Register class ID 6: short (com.esotericsoftware.kryo.serializers.DefaultSerializers$ShortSerializer)
00:00 TRACE: [kryo] Register class ID 7: long (com.esotericsoftware.kryo.serializers.DefaultSerializers$LongSerializer)
00:00 TRACE: [kryo] Register class ID 8: double (com.esotericsoftware.kryo.serializers.DefaultSerializers$DoubleSerializer)
00:00 TRACE: [kryo] Register class ID 9: void (com.esotericsoftware.kryo.serializers.DefaultSerializers$VoidSerializer)


00:00 TRACE: [kryo] Write initial object reference 0: LoginLog{loginLogId=123456, userId=789, loginAt=Mon Jan 05 10:45:55 CST 2015}
00:00 DEBUG: [kryo] Write: LoginLog{loginLogId=123456, userId=789, loginAt=Mon Jan 05 10:45:55 CST 2015}
00:00 TRACE: [kryo] Optimize ints: true
00:00 TRACE: [kryo] Field loginLogId: long
00:00 TRACE: [kryo] Field userId: long
00:00 TRACE: [kryo] Field loginAt: class java.util.Date
00:00 TRACE: [kryo] Field loginType: int
00:00 TRACE: [kryo] Register class name: code.entity.LoginLog (com.esotericsoftware.kryo.serializers.FieldSerializer)
00:00 TRACE: [kryo] FieldSerializer.write fields of class: code.entity.LoginLog
00:00 TRACE: [kryo] Write field: loginAt (code.entity.LoginLog) pos=1
00:00 TRACE: [kryo] Register class name: java.util.Date (com.esotericsoftware.kryo.serializers.DefaultSerializers$DateSerializer)
00:00 TRACE: [kryo] Write class name: java.util.Date
00:00 TRACE: [kryo] Write initial object reference 1: Mon Jan 05 10:45:55 CST 2015
00:00 DEBUG: [kryo] Write: Mon Jan 05 10:45:55 CST 2015
00:00 TRACE: [kryo] Object graph complete.


00:00 TRACE: [kryo] Read initial object reference 0: code.entity.LoginLog
00:00 TRACE: [kryo] Read field: loginAt (code.entity.LoginLog) pos=1
00:00 TRACE: [kryo] Read class name: java.util.Date
00:00 TRACE: [kryo] Read initial object reference 1: java.util.Date
00:00 DEBUG: [kryo] Read: Mon Jan 05 10:45:55 CST 2015
00:00 DEBUG: [kryo] Read: LoginLog{loginLogId=123456, userId=789, loginAt=Mon Jan 05 10:45:55 CST 2015}
00:00 TRACE: [kryo] Object graph complete.


loginLog = LoginLog{loginLogId=123456, userId=789, loginAt=Mon Jan 05 10:45:55 CST 2015}
loginLog1 = LoginLog{loginLogId=123456, userId=789, loginAt=Mon Jan 05 10:45:55 CST 2015}
[loginLog == loginLog2] = false
[loginLog.equals(loginLog1)] = true

----------

从输出结果看，loginLog1和loginLog2**值**相同，但**引用地址**不同。


## 类注册

kryo有ID和NAME两种类注册方式，通过NAME注册的类，其ID为-1。

类通过ID注册到kryo，性能更佳。


### 通过ID注册类

Registration代表一个注册实例，注册一个类到Kryo，至少需要提供Class。

可以使用Kryo#register(Class type, Serializer serializer)注册一个类，提供Class和Serializer，类ID由方法Kryo#getNextRegistrationId自动产生。

Kryo已注册类可以在Kryo构造方法Kryo(ClassResolver, ReferenceResolver, StreamFactory)中看到:

```java

// Primitives and string. Primitive wrappers automatically use the same registration as primitives.
register(int.class, new IntSerializer());
register(String.class, new StringSerializer());
register(float.class, new FloatSerializer());
register(boolean.class, new BooleanSerializer());
register(byte.class, new ByteSerializer());
register(char.class, new CharSerializer());
register(short.class, new ShortSerializer());
register(long.class, new LongSerializer());
register(double.class, new DoubleSerializer());
register(void.class, new VoidSerializer());

```

注册一个类，关键Serializer，DefaultSerializers实现常见类的Serializer（如Date），可以直接使用。

kryo项目[kryo-serializers](https://github.com/magro/kryo-serializers "kryo-serializers")也实现了许多Serializer。


已注册类的作用域仅限于某个kryo实例，如果有实例kryo1和kryo2，那么kryo1和kryo2两者已注册类不共享。


### 通过NAME注册类

当一个类没有通过ID注册到kryo，默认会使用NAME（全限定类名,ID=-1）注册，默认使用的Serializer是FieldSerializer。

----------

Register class name: code.entity.LoginLog (com.esotericsoftware.kryo.serializers.FieldSerializer)

----------

输出内容代表LoginLog通过NAME注册到kryo，其ID=-1,使用FieldSerializer作为Serializer，更多信息参见源码。






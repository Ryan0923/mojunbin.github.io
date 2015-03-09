# netty启动分析

io.netty.eventLoopThreads设定bossGroup线程数，默认是CPU核数*2。

遍历ByteBuf

```java
for (int i = 0; i < buf.readableBytes(); i++) {
    System.out.print(buf.getByte(i)+" ");
}
```


buf的readIndex不会改变。

```java
while (buf.isReadable()){
    System.out.print(buf.readByte()+" ");
}
```

buf的readIndex会改变。


客户端与服务端通讯，要定义好消息对象。

当客户端和服务端TCP链路建立成功之后,Netty的NIO线程会调用channelActive方法。

DefaultAttributeMap用到了JDK5的原子同步类，值得研究。



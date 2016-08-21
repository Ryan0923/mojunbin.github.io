# jprofiler 远程监控

jprofiler是最常见的java性能分析工具，功能强大，可以监控本地和远程JVM运行情况。

## jprofiler 远程监控

**jprofiler license**:jprofiler是一款收费工具，使用需要license，但只是使用jprofiler GUI 才需要license，其他功能无需license。


工具环境：

客户端：jprofiler.8_1_4；
服务端：jprofiler.8_1_4；
服务端JDK：至少1.4，建议1.6以上；
服务端操作系统：centos 64位;

**客户端与服务端jprofiler版本必须一致**


### 关键参数

关键参数**agentpath**,【java -help】查看详情：

-agentpath:<pathname>[=<选项>]
    按完整路径名加载本机代理库

参数示例：
- linux
    -agentpath:/opt/jprofiler/bin/linux-x64/libjprofilerti.so=port=8849 
- windows
    D:\dev\jprofiler_windows-x64_8_1_1\bin\windows-x64\jprofilerti.dll=port=8849

参数说明：

- port: jprofiler服务端监听端口

### 参考配置

普通java应用,按照下面方式启动：

```shell
java -agentpath:/opt/jprofiler/bin/linux-x64/libjprofilerti.so=port=8849 
```

resin服务器，在resin.conf添加配置：
```shell
<jvm-arg>-agentpath:D:\dev\jprofiler_windows-x64_8_1_1\bin\windows-x64\jprofilerti.dll=port=8849</jvm-arg>

```

其他tomcat、jetty服务器配置类似，关键是把**agentpath**参数传递给JVM。


# SSH
- [jprofile ssh](http://blog.ej-technologies.com/2015/11/remote-profiling-through-ssh-tunnel.html)

http://blog.ej-technologies.com/2011/09/inspections-in-heap-walker.html
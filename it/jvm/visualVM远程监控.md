
# VisualVM远程监控


java_home=/usr/java/jdk1.6.0_32/

**请把${java_home}**用实际目录替代。


## 配置jstatd服务

- 配置权限

创建${java_home}/jstatd.all.policy，jstatd.all.policy内容如下：

```xml
grant codebase "file:${java_home}/lib/tools.jar" {
   permission java.security.AllPermission;
};
```

- 启动jstatd服务

```shell
nohup ${java_home}/bin/jstatd -J-Djava.security.policy=${java_home}/jstatd.all.policy >& /dev/null &
```

参数**java.rmi.server.logCalls=true**可开启日志功能，可以使用以下命令启动jststd：

```shell
nohup ${java_home}/bin/jstatd -J-Djava.security.policy=${java_home}/jstatd.all.policy -J-Djava.rmi.server.logCalls=true >& /dev/null &
```

## 配置JMX


以resin为例，在resin_home/resin.conf.xml添加JVM配置:

```xml
      <jvm-arg>-Dcom.sun.management.jmxremote</jvm-arg>
      <jvm-arg>-Djava.rmi.server.hostname=192.168.74.210</jvm-arg>
      <jvm-arg>-Dcom.sun.management.jmxremote.port=9910</jvm-arg>
      <jvm-arg>-Dcom.sun.management.jmxremote.ssl=false</jvm-arg> 
      <jvm-arg>-Dcom.sun.management.jmxremote.authenticate=false</jvm-arg>
```

**java.rmi.server.hostname**:服务器IP

**com.sun.management.jmxremote.port**：jmx端口

**com.sun.management.jmxremote.ssl 和com.sun.management.jmxremote.authenticate**：鉴权配置，设为false较方便


## 启动VisualVM连接

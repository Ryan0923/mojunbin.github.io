
# jetty-maven-plugin 概述

jetty-maven-plugin是基于jetty的一个maven插件，利用jetty的热部署功能，开发过程中可以从减少编译、部署、重启的繁琐工作。


这里主要讲解jetty-maven-plugin（7、8），采用版本：**8.1.16.v20140903**，pom.xml片段配置如下：

```xml
                      
        <!--jetty-maven-plugin插件，可选-->
        <dependency>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>${jetty_version}</version>
            <scope>provided</scope>
        </dependency>
        
        <!--jetty-maven-plugin插件，必选-->
        <plugin>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>${jetty_version}</version>
        </plugin>

        <properties>
            <jetty_version>8.1.16.v20140903</jetty_version>
        </properties>

```

pom.xml片段中，配置jetty-maven-plugin的dependency，设置scope=provided，方便在配置jetty-maven-plugin参数时，查看配置项有哪些可配置参数。

**scope=provided**这句必须有，否则相当于scope=compile，这会导致jetty:run启动项目过程中，出现异常。



# jetty:run

jetty:run是jetty-maven-plugin的一个goal，它能在jetty上运行maven项目。

jetty:run常见pom.xml片段配置如下：

```Xml

        <!--jetty-maven-plugin插件，可选-->
        <dependency>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>${jetty_version}</version>
            <scope>provided</scope>
        </dependency>

        <plugin>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>${jetty_version}</version>
            <configuration>
                <stopKey>base</stopKey>
                <stopPort>9999</stopPort>
                <jettyXml>${basedir}/src/main/config/jetty.xml</jettyXml>
                <scanIntervalSeconds>2</scanIntervalSeconds>
                <connectors>
                    <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
                        <port>8003</port>
                        <maxIdleTime>60000</maxIdleTime>
                    </connector>
                </connectors>
                <webApp>
                    <contextPath>/</contextPath>
                </webApp>
            </configuration>
        </plugin>

        <properties>
            <jetty_version>8.1.16.v20140903</jetty_version>
        </properties>

```

* stopPort： 停止jetty的端口，jetty在该端口监听stop command。
* stopKey: 字符串值，发送到stopPort端口，校验stop command。


stopKey和stopPort一般结合jetty:stop使用，用来停止jetty服务器。

* scanIntervalSeconds： jetty扫描变更的时间，单位：秒。
* connectors： 可以配置port和host，参见SelectChannelConnector#setter方法。
* webApp： web相关配置，其实现是org.eclipse.jetty.webapp.WebAppContext，这里比较重要属性是contextPath，用于配置web上下文。


运行**jetty:run**，通过**localhost:8003/**可以访问web应用。


# jetty:stop

停止jetty服务器，需要配置**stopPort**和**stopKey**。


# jetty:run debug模式

jetty:run运行下，不支持在pom.xml配置jvmArgs传递JVM参数，如**-Xdebug**，这导致没法开启JVM调试模式。

```xml
           <plugin>
                <groupId>org.mortbay.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>${jetty_version}</version>
                <configuration>
                    <stopKey>base</stopKey>
                    <stopPort>9999</stopPort>
                    <jettyXml>${basedir}/src/main/config/jetty.xml</jettyXml>
                    <scanIntervalSeconds>2</scanIntervalSeconds>
                    <connectors>
                        <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
                            <port>8003</port>
                            <maxIdleTime>60000</maxIdleTime>
                        </connector>
                    </connectors>
                    <webApp>
                        <contextPath>/</contextPath>
                    </webApp>
                    <!--jetty:run运行下，下面jvmArgs无法传递给JVM-->
                    <jvmArgs>
                        -Xdebug
                        -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005
                    </jvmArgs>
                </configuration>
            </plugin>

```

pom.xml无法传递jvmArgs，但可以通过配置MAVEN_OPTS环境变量传递jvmArgs：


```shell

export MAVEN_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"

```
重新执行jetty:run，热部署+调试已经配置完毕。


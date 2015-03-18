## 服务端


注解，标识接口是远程接口


业务系统：服务接口+实现


RpcServer：
1、结合RpcService标注，对这个类做处理。
2、netty服务端处理，编码解码


RpcDecoder：从实现上看，只支持String
RpcHandler:处理RpcDecoder解码的数据。
RpcEncoder:处理RpcResponse数据

#服务端处理逻辑

高性能要求：
netty高性能要点：
1. 耗时操作不要在IO线程
2. 异步处理
3. 外部线程池
4. 服务端缓存反射，何时嵌入K-V处理？适配不同方式，JDK自带，cglib
5. 通用反射处理

1. 接受请求
2. 解码
3. 线程池处理
4. 处理回调
5. 写数据
6. 编码

# 协议处理
1、netty解码
2、业务解码（线程池）


# 客户端处理

## 如何定位一个服务？

服务接口名->tcpclient

1. 接口名
2. 注册中心(接口名->服务IP：端口)
3. 通过netty连接
4. 发送数据

Rpcclient管理所有服务







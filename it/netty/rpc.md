## 服务端


注解，标识接口是远程接口


业务系统：服务接口+实现


RpcServer：
1、结合RpcService标注，对这个类做处理。
2、netty服务端处理，编码解码


RpcDecoder：从实现上看，只支持String
RpcHandler:处理RpcDecoder解码的数据。
RpcEncoder:处理RpcResponse数据


# RPC

RPC（Remote Procedure Call）—远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。

- 客户端存根(Client Stub)：存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端。
- 服务端存根(Server Stub)：接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理。

## 流程

- 客户端通过本地调用的方式调用服务。
- 客户端存根(Client Stub)接收到调用请求后负责将方法、入参等信息序列化(组装)成能够进行网络传输的消息体。
- 客户端存根(Client Stub)找到远程的服务地址，并且将消息通过网络发送给服务端。
- 服务端存根(Server Stub)收到消息后进行解码(反序列化操作)。
- 服务端存根(Server Stub)根据解码结果调用本地的服务进行相关处理
- 服务端(Server)本地服务业务处理。
- 处理结果返回给服务端存根(Server Stub)。
- 服务端存根(Server Stub)序列化结果。
- 服务端存根(Server Stub)将结果通过网络发送至消费方。
- 客户端存根(Client Stub)接收到消息，并进行反序列化。
- 客户端得到最终结果。

![](http://emall-t.oss-cn-hangzhou.aliyuncs.com/blog/2020-07-06-045334.jpg)



## feign

基于Http Restful的RPC框架。

通过一系列的封装和处理，将以JAVA注解的方式定义的远程调用API接口，最终转换成HTTP的请求形式，然后将HTTP的请求的响应结果，解码成JAVA Bean，放回给调用者。

Feign通过处理注解，将请求模板化，当实际调用的时候，传入参数，根据参数再应用到请求上，进而转化成真正的 Request 请求。通过Feign以及JAVA的动态代理机制，可以不用通过HTTP框架去封装HTTP请求报文的方式，完成远程服务的HTTP调用。

###  流程

1. 基于面向接口的动态代理方式生成实现类

    - 将目标服务方法作为接口。

    - 代理类加入了配置类信息、接口类型、目标服务名称、url地址

2. 根据接口类的注解声明规则，解析出底层 Methodhandler

3. 基于 RequestBean，动态生成 Request

4. Encoder将Bean包装成 Http 报文请求

5. 拦截器对请求和响应进行装饰处理

6. 日志记录

7. 基于 HTTP 发送请求

![](https://upload-images.jianshu.io/upload_images/19816137-512fc8f62746eac7?imageMogr2/auto-orient/strip|imageView2/2/w/756)





## dubbo

集成了负载均衡，限流等功能。

- 使用**自定义的 RPC 协议**，更适合小数据高并发场景，性能更好。
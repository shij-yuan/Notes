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

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gizdpudef2j30i30c274o.jpg)



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

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghtnnkgx9yj30l00llacs.jpg)





## dubbo

集成了负载均衡，限流等功能。

默认的协议采用单一长连接和 NIO 异步通讯，适合于小数据量高并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况。不适合传送大数据量的服务。



### 流程

1. 通过`register`将服务提供者的url注册到`Registry`注册中心中。
2. 客户端`Consumer`从注册中心获取被调用服务端注册信息，如：接口名称，URL地址等信息。
3. 将获取的url地址返回到`Consumer`客户端，客户端通过获取的URL地址支持`invoke`反射机制获取服务的实现。



![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghtnnq1m6bj30p00iqqow.jpg)

Dubbo框架设计一共划分了10个层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI；最上面的 Service 层是留给实际想要使用 Dubbo 开发分布式服务的开发者实现业务逻辑的接口层；图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口， 位于中轴线上的为双方都用到的接口。

1. 服务接口层（Service）：该层是与实际业务逻辑相关的，根据服务提供方和服务消费方的业务设计对应的接口和实现。
2. 配置层（Config）：对外配置接口，以 ServiceConfig 和 ReferenceConfig 为中心，可以直接 new 配置类，也可以通过 spring 解析配置生成配置类。
3. 服务代理层（Proxy）：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton，以 ServiceProxy 为中心，扩展接口为 ProxyFactory。
4. 服务注册层（Registry）：封装服务地址的注册与发现，以服务URL为中心，扩展接口为 RegistryFactory、Registry 和 RegistryService。可能没有服务注册中心，此时服务提供方直接暴露服务。
5. 集群层（Cluster）：封装多个提供者的路由及负载均衡，并桥接注册中心，以Invoker为中心，扩展接口为 Cluster、Directory、Router 和 LoadBalance。将多个服务提供方组合为一个服务提供方，实现对服务消费方来透明，只需要与一个服务提供方进行交互。
6. 监控层（Monitor）：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory、Monitor 和 MonitorService。
7. 远程调用层（Protocol）：封将 RPC 调用，以 Invocation 和 Result 为中心，扩展接口为 Protocol、Invoker 和 Exporter。Protocol 是服务域，它是 Invoker 暴露和引用的主功能入口，它负责 Invoker 的生命周期管理。Invoker 是实体域，它是 Dubbo 的核心模型，其它模型都向它靠扰，或转换成它，它代表一个可执行体，可向它发起 invoke 调用，它有可能是一个本地的实现，也可能是一个远程的实现，也可能一个集群实现。
8. 信息交换层（Exchange）：封装请求响应模式，同步转异步，以 Request 和 Response 为中心，扩展接口为 Exchanger、ExchangeChannel、ExchangeClient 和 ExchangeServer。
9. 网络传输层（Transport）：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel、Transporter、Client、Server 和 Codec。
10. 数据序列化层（Serialize）：可复用的一些工具，扩展接口为 Serialization、 ObjectInput、ObjectOutput和ThreadPool。
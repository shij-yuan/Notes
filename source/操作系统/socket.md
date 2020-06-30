# socket

服务器端先初始化Socket，然后与端口绑定(bind)，对端口进行监听(listen)，调用accept阻塞，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器(connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接

![](https://images.cnblogs.com/cnblogs_com/goodcandle/socket3.jpg)



## 三次握手

1. 当客户端调用connect时，触发了连接请求，向服务器发送了SYN J包，此时connect进入阻塞状态
2. 服务器监听到连接请求，即收到SYN J包，调用accept函数接收请求向客户端发送SYN K ，ACK J+1，这时accept进入阻塞状态
3. 客户端收到服务器的SYN K ，ACK J+1之后，这时connect返回，并对SYN K进行确认，发送 ACK K+1
4. 服务器收到ACK K+1时，accept返回



## 四次挥手

1. 主动关闭的首先调用close主动关闭连接，这时TCP发送一个FIN M

2. 接收到FIN M之后，被动关闭的一方执行被动关闭，对FIN进行确认。

    FIN也作为文件结束符传递给应用进程，因为FIN的接收意味着应用进程不会再接受到数据

3. 接收到文件结束符的应用进程调用 close 关闭它的socket，然后它的TCP也发送一个FIN N

4. 主动关闭的一方收到 FIN N, 并发送ACK N+1 确认
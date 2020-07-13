# RabbitMQ

## 应用场景

1. 异步处理
2. 应用解耦
    订阅消息队列，无需修改程序
3. 流量控制（削峰）
    防止阻塞

## 概念

- Message
    消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key (路由键)、priority (相对于其他消息的优先权)、delivery-mode (指出该消息可能需要持久性存储)等。

- Publisher
    消息的生产者，也是一个向交换器发布消息的客户端应用程序。
- Consumer
    消息的消费者，表示个从消息队列中取得消息的客户端应用程序。

- Exchange
    交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
    Exchange有4种类型: direct(默认)， fanout，topic，和 headers，不同类型的Exchange转发消息的策略有所区别
    - Direct: 消息中的路由键( routing key )与 Binding key 一致， 交换器就将消息发送到对应的队列。完全匹配，单播
    - fanout：每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。
    - topic：交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一一个模式上。它将路 由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符:符号`#`和符号 `*` 。# 匹配个或多个单词，* 匹配一个单词。

- Queue
    消息队列，用来保存消息直到发送给消费者。，它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息直在队列里面，等待消费者连接到这个队列将其取走。

- Binding
    绑定，用于**消息队列和交换器之间的关联**。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

    Exchange和Queue的绑定可以是多对多的关系。

- Connection
    网络连接，比如一个TCP连接。长连接

- Channel
    信道，**多路复用连接中的一条独立的双向数据流通道**。信道是建立在真实的TCP连接内的虚拟连接，AMQP命令都是通过信道发出去的。不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概态，以复用一条TCP连接。

- Virtual Host
    虚拟主机，表示批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是AMQP概念的基础，必须在连接时指定，RabbitMQ默认的vhost是/。

- Broker
    表示消息队列代理服务器

![image-20200608204105257](https://tva1.sinaimg.cn/large/007S8ZIlly1gfl5z1top7j30v4090juv.jpg)



### Docker 安装

```
docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
```



### Test

1. 使用 AmqpAdmin 创建交换机、队列、绑定

```java
@Test
    void createExchange() {
        // 创建交换机
        DirectExchange directExchange = new DirectExchange("test-java-exchange", true, false);
        amqpAdmin.declareExchange(directExchange);

    }

    @Test
    void createQueue(){
        Queue queue = new Queue("test-java-queue", true, false, false);
        amqpAdmin.declareQueue(queue);

    }

    @Test
    void createBinding(){
        Binding binding = new Binding("test-java-binding",
                Binding.DestinationType.QUEUE,
                "test-java-exchange",
                "hello.java",
                null);
        amqpAdmin.declareBinding(binding);
    }
```



2. 使用 RabbitTemplate 发送消息

```java
@Test
    void sendMessage(){
        // 交换机、路由键、消息
        rabbitTemplate.convertAndSend("test-java-exchange", "hello.java", "hello rabbitmq");
    }
```

若发送地消息是对象，对象必须实现序列化。 可注入配置类

```java
@Configuration
public class MyRabbitConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```



3. RabbitListener, RabbitHandler

`@RabbitListener`： 可以标注在类+方法上（监听队列即可）

`@RabbitHandler` ： 标注在方法上 （重载、区分不同类型地消息）


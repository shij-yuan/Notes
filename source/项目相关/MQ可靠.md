# 消息可靠



## 确认机制

- Publisher : confirmCallback
- Publisher : returnCallback
- Consumer : ack

![image-20200626170455442](https://tva1.sinaimg.cn/large/007S8ZIlly1gg5svnxserj314o07owjn.jpg)

### confirmCallback

- 在创建 connectionFactory的时候设置 PublisherConfirms(true)选项, 开启 confirmcallback。
- Correlation Data:用来表示当前消息唯一性。
- 消息只要被 broker接收到就会执行 confirmCallback,如果是 cluster 模式,需要所有 broker接收到才会调用 confirm Callback
- 被 broker接收到只能表示 message已经到达服务器,并不能保证消肖息定会被投递到目标 queue里。所以需要用到接下来的 return Callback。

### returnCallback

- confrim模式只能保证消息到达 broker,不能保证消息准确投递到目标queue里。在有些业务场景下,我们需要保证消息一定要投递到目标queue里,此时就需要用到 return 退回模式。
- 这样如果未能投递到目标 queue里将调用 returnCallback,可以记录详细到投递数据,定期的巡检或者自动纠错都需要这些数据。

```java
# 开启发送端消息抵达队列的确认
spring.rabbitmq.publisher-returns=true
# 只要抵达队列，就以异步发送优先回调 returnConfirm
spring.rabbitmq.template.mandatory=true
```



### ack

默认情况下会发生消息丢失。需要进行手动确认

`spring.rabbitmq.listener.direct.acknowledge-mode=manual`

- 消费者获取到消息,成功处理,可以回复Ack给 Broker
    - basic.ack 用于肯定确认; broker将移除此消息
    - basic.nack 用于否定确认;可以指定 broker是否丢弃此消息,可以批量
    - basic.reject 用于否定确认;同上,但不能批量
- 默认, 消息被消费者收到, 就会从 broker 的 queue中移除
- queue无消费者, 消息依然会被存储,直到消肖费者消费
- 消费者收到消息,默认会自动ack。但是如果无法确定此消息是否被处理完成,或者成功处理。我们可以开启手动ack模式
    - 消息处理成功,ack( ),接受下一个消息,此消息 broker就会移除
    - 消息处理失败, nack( ) / reject( ) , 重新发送给其他人进行处理,或者容错处理后ack
    - 消息一直没有调用 ack/nack 方法, broker认为此消息正在被处理,不会投递给别人, 此时客户端断开,消息不会被 broker移除,会投递给别人



## 解决方案
### 消息丢失

#### 情况一：消息发送出去，由于网络问题没有抵达服务器

- 做好容错方法(try-catch) ，发送消息可能会网络失败，失败后要有重试机制，可**记录到数据库，采用定期扫描重发的方式**
- 做好日志记录，每个消息状态是否都被服务器收到都应该记录

#### 情况二：消息抵达Broker, Broker要将消息写入磁盘 (持久化)才算成功。此时Broker尚未持久化完成，宕机

- publisher 也必须加入确认回调机制，确认成功的消息，修改数据库消息状态。

#### 情况三：自动ACK的状态下。消费者收到消息，但没来得及消息然后宕机

- 关闭自动ACK，开启手动ACK,消费成功才移除，失败或者没来得及处理就noAck并重新入队

```java
		// 确认回调
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            /**
             * @param correlationData 消息唯一ID
             * @param b 是否成功收到
             * @param s 失败原因
             */
            @Override
            public void confirm(CorrelationData correlationData, boolean b, String s) {
                System.out.println("confirm msg");
            }
        });
```



### 消息重复

- 保证幂等性
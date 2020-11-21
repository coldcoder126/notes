# RabbitMQ简介

**RabbitMQ**是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。

## 什么是消息中间件

**消息（Message）**是指在应用间传送的数据

**消息队列中间件（Message Queue Middleware，简称MQ）**是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并基于数据通信来进行分布式系统的集成。通过提供消息传递和消息排队模型，它可以在分布式环境下扩展进程间的通信。

**消息队列的传递模式**

点对点(P2P)模式：消息生产者发送消息到队列，消息消费者从队列中接收消息

发布/订阅(Pub/Sub)模式：消息发布者将消息发布到主题，消息订阅者从主题中订阅消息

## 重要概念

Virtual Host：虚拟主机是共享相同的身份认证和加密环境的独立服务器域。（参考数据库中的库理解）

Exchange：生产者将消息发送到Exchange，由Exchange将消息路由到一个或多个Queue中（或者丢弃），是否需要交换机取决于选择消息模型

Queue：是RabbitMQ的内部对象，用于存储消息

Connection：是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑

Channel：信道，多路复用连接中的一条独立的双向数据流通道。为会话提供物理传输介质。

消息模型：

- 直连：生产者的消息直接放到队列中，消费者直接从队列中取



 ## 消息中间件的作用

解耦：

冗余（存储）：消息中间件可以将数据进行持久化直到它们已经被完全处理。
扩展性：提高消息消息入队和处理的效率是很容易的，只需要另外增加处理过程即可，不需要改变代码

削峰：突然增加的大量访问请求不需要一下处理掉，可以以最大的能力慢慢处理



# Java使用RabbitMQ

##使用基本方法进行点对点的消息队列连接

## 第一种模型：直连

一个生产者直接将消息放入队列，一个消费者直接从队列中消费消息

1. 在项目中引入依赖(版本需对应)

```xml 
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.8.0</version>
</dependency>
```

2. 在网页客户端新建一个Virtual Host ，命名为`/ems`
3. 在网页客户端新建一个用户，并允许其访问`/ems`

```java
 public void testSendMsg() throws IOException, TimeoutException {
        //创建连接MQ的连接工厂对象
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //设置连接MQ的主机
        connectionFactory.setHost("47.92.2.46");
        //设置连接的端口号
        connectionFactory.setPort(8672);
        //设置连接哪个虚拟主机
        connectionFactory.setVirtualHost("/ems");
        //设置访问虚拟主机的用户名和密码
        connectionFactory.setUsername("user1");
        connectionFactory.setPassword("123");

        //获取连接对象
        Connection connection = connectionFactory.newConnection();
        //获取连接通道
        Channel channel = connection.createChannel();
        //通道绑定对应的消息队列，
        //参数1：对列名，如果不存在则自动创建;
        //参数2:用来定义队列特性是否要持久化，false重启队列将消失，true重启队列持久化，消息清除；该参数需要和消费者对应
        //参数3：是否独占队列
        //参数4：是否在消费完成后自动删除队列
        // 参数5：额外参数
        channel.queueDeclare("hello",false,false,false,null);

        //发布消息
        // 参数1：交换机名称
        // 参数2：队列名称
        // 参数3：传递消息额外设置，(MessageProperties.PERSISTENT_TEXT_PLAIN,设置消息持久化)
        // 参数4：消息的具体内容
        channel.basicPublish("","hello",null,"hello rabbitMQ".getBytes());

        //关闭连接，关闭通道
        channel.close();
        connection.close();
    }
/**-----------------------------------------------------------------------------*/
		//接收channel.basicPublic以上代码相同,生产者消费者对应参数需相同
        //消费消息
        // 参数1：要消费的队列
        // 参数2：开启消息的自动确认机制
        // 参数3：消费时的回调接口
        channel.basicConsume("hello",true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
            }
        });
```

## 第二种模型：工作队列(work queue)

第一种模型中可能生产消息的速度回远远大于消费消息的速度，，导致消息堆积过多。所以可以将多个消费者绑定到一个队列，共同消费队列中的消息。消息被消费一次就会消失。

> 该模式在默认情况下，RabbitMQ将按顺序将每个消息发送给下一各使用者，平均而言，每个消费者都会收到相同数量的消息，这种分发消息的方式称为循环。
>
> 此种情况下消费慢的消费者会拖垮队列。

###消息确认机制和能者多劳的实现

```java
        //消费消息
		//一次只消费一个
		chanel.basicQos(1);
        // 参数1：要消费的队列
        // 参数2：开启消息的自动确认机制；若为true，消费者自动向rabbitmq确认消费
        // 参数3：消费时的回调接口
        channel.basicConsume("hello",true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
            }
        });
```

自动确认会在消息发送给消费者后立即确认，但存在丢失消息的可能(如消费者未处理完就宕机)。

可在业务执行完成后进行一条消息消费的手动确认，保证消费完成再通知队列删除

```java
//参数1：确认队列中那个具体消息
//参数2：是否开启多个消息同时确认
channel.baxicAck(envelope.getDeliveryTag(),false);
```

##第三种模型:扇出/广播（fanout）

生产者发送的消息被多个消费者同时消费

- 可以有多个消费者
- 每个消费者都有自己的queue
- 每个队列都要绑定到Exchange
- 生产者发送的消息只能发送到Exchange
- 生产者发送的消息，只能发送到交换机，交换机来决定发给哪个队列，生产者无法决定
- 交换机把消息发送给绑定过的所有队列
- 队列的消费者都能拿到消息，实现一条消息被多个消费者消费

##第四种模型：路由(Routing)

### Routing之订阅模型-Direct(直连)

在fanout模式中，一条消息会被所有订阅的队列都消费。但是某些场景下我们希望不同的消息被不同的队列消费

在Direct模型下：

- 队列与交换机的绑定不是任意绑定，而是要指定一个RoutingKey（路由key）
- 消息的发送方在向Exchange发送消息时，也必须指定消息的RoutingKey。
- Exchange不再吧消息交给每一个绑定的队列，而是根据消息的RoutingKey进行判断，只有队列的RoutingKey与消息的RoutingKey完全一致才会接收到消息

![流程](C:\Users\DX\Desktop\python-four.webp)

##Topic模型

Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列绑定RoutingKey的时候使用通配符。这种模型的RoutingKey一般是由一个或多个单词组成，多个单词之间以"."分隔，如item.insert

`*`：匹配一个单词

`#`：匹配多个单词和`.`



![](C:\Users\DX\Desktop\python-five.webp)

# SpringBoot整合RabbitMQ

1. 引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

2. 配置配置文件

```yaml
spring: 
  rabbitmq:
    host: 47.92.2.46
    port: 8672
    username: user1
    password: 123
    virtual-host: /ems
```

3.  使用`RabbitTemplate`来简化操作，使用的时候直接在项目注入即可

```java

```


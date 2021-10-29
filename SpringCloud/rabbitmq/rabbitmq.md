# rabbitmq

## 各个名词介绍

![](..\..\pics\SpringCloud\MQ\消息中间件RabbitMQ.jpg)

`Broker`：接收和分发消息的应用，RabbitMQ Server 就是 Message Broker
`Virtual host`：出于多租户和安全因素设计的，把 AMQP 的基本组件划分到一个虚拟的分组中，类似于网络中的 namespace 概念。当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等
`Connection`：publisher／consumer 和 broker 之间的 TCP 连接
`Channel`：如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCP Connection 的开销将是巨大的，效率也较低。Channel 是在 connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和 message broker 识别 channel，所以 channel 之间是完全隔离的。**Channel 作为轻量级的Connection 极大减少了操作系统建立 TCP connection 的开销**
`Exchange`：message 到达 broker 的第一站，根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (multicast)
`Queue`：消息最终被送到这里等待 consumer 取走
`Binding`：exchange 和queue 之间的虚拟连接，binding 中可以包含 routing key，Binding 信息被保存到 exchange 中的查询表中，用于 message 的分发依据

## 基本代码

生产者声明队列

```
public class Producer {
private final static String QUEUE_NAME = "hello";
public static void main(String[] args) throws Exception {
//创建一个连接工厂
ConnectionFactory factory = new ConnectionFactory(); factory.setHost("182.92.234.71");

factory.setUsername("admin"); factory.setPassword("123");

//channel 实现了自动 close 接口 自动关闭 不需要显示关闭
try(Connection connection = factory.newConnection();Channel connection.createChannel()) {
/**
* 生成一个队列
* 1.队列名称
* 2.队列里面的消息是否持久化 默认消息存储在内存中
* 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
* 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
* 5.其他参数
*/ channel.queueDeclare(QUEUE_NAME,false,false,false,null); String message="hello world";
/**
* 发送一个消息
* 1.发送到那个交换机
* 2.路由的 key 是哪个
* 3.其他的参数信息
* 4.发送消息的消息体
*/ channel.basicPublish("",QUEUE_NAME,null,message.getBytes()); System.out.println("消息发送完毕");
}
}
}

```





```
public class Consumer {
private final static String QUEUE_NAME = "hello";
public static void main(String[] args) throws Exception { ConnectionFactory factory = new ConnectionFactory(); factory.setHost("182.92.234.71"); factory.setUsername("admin"); factory.setPassword("123");
Connection connection = factory.newConnection(); Channel channel = connection.createChannel(); System.out.println("等待接收消息 ......... ");
//推送的消息如何进行消费的接口回调
DeliverCallback deliverCallback=(consumerTag,delivery)->{ 
String message= new String(delivery.getBody());
System.out.println(message);
};
//取消消费的一个回调接口 如在消费的时候队列被删除掉了
CancelCallback cancelCallback=(consumerTag)->{ System.out.println("消息消费被中断");
};
/**
* 消费者消费消息
* 1.消费哪个队列
* 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
* 3.消费者未成功消费的回调
*/
channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
}
```





自动应答：消息发送后立即被认为已经传送成功，**消费者那边出现连接或者 channel 关闭，那么消息就丢失了,**

取消自动应答。消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。



发布确认：消息保存磁盘成功后跟生产者说一下,需要开启功能。

异步确认，监听器



## 交换机：

`本来`：没有交换机，队列发消息只能被一个消费者消费。消费完会销毁队列的消息

`发布订阅模式`：交换机发同一条消息到二条队列，两个消费者消费到了





类型：直接(direct), 主题(topic) ,标题(headers) , 扇出(fanout)

绑定：可以ui手动绑定的,看routingkey

(**广播**)Fanout：它是将接收到的所有消息广播到它知道的所有队列中。系统中默认有些 exchange 类型（routingkey一样）



（**直接**）Direct：routingkey不一样

```
channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
```



（**主题**）Topics：按照主题

**当一个队列绑定键是#,那么这个队列将接收所有数据，就有点像 fanout 了**

**如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是 direct 了**



模板

消费者

1. 信道
2. 声明交换机

3. 声明队列
4. 绑定队列，交换机及routingkey
5. 接受（回调）



生产者

1. 信道
2. 声明交换机
3. 往交换机发送消息





不关闭防火墙的话,需要开放15672和5672两个端口,一个是连接控制台,一个是连接服务
fireawll-cmd --zone=public --add-port=15672/tcp --permanent
fireawll-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --reload
如果是云服务器的话,还需要给这两个端口配置安全组



##  死信

先从概念解释上搞清楚这个定义，死信，顾名思义就是无法被消费的消息，字面意思可以这样理解，一般来说，producer 将消息投递到 broker 或者直接到queue 里了，consumer 从 queue 取出消息进行消费，但某些时候由于特定的原因导致 queue 中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。
应用场景:**为了保证订单业务的消息数据不丢失，需要使用到 RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中.还有比如说: 用户在商城下单成功并点击去支付后在指定时间未支付时自动失效**



死信的来源

1. 消息 TTL 过期
2. 队列达到最大长度(队列满了，无法再添加数据到 mq 中) 
3. 消息被拒绝(basic.reject 或 basic.nack)并且 requeue=false.



消息 TTL 过期

生产者：要设置TTL

```java
try (Channel channel = RabbitMqUtils.getChannel()) { channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
//设置消息的 TTL 时间
AMQP.BasicProperties properties =
new AMQP.BasicProperties().builder().expiration("10000").build();
//该信息是用作演示队列个数限制
for (int i = 1; i <11 ; i++) {
    String message="info"+i;
    channel.basicPublish(NORMAL_EXCHANGE, "zhangsan",properties,message.getBytes());
    System.out.println("生产者发送消息:"+message);
}
```



消费者1：声明各种交换机，绑定

```java
public class Consumer01 {
//普通交换机名称
private static final String NORMAL_EXCHANGE = "normal_exchange";
//死信交换机名称
private static final String DEAD_EXCHANGE = "dead_exchange"; 
public static void main(String[] argv) throws Exception {
        Channel channel = RabbitUtils.getChannel();
        //声明死信和普通交换机 类型为 direct 
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT); 
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
        //声明死信队列
        String deadQueue = "dead-queue";
    	channel.queueDeclare(deadQueue, false, false, false, null);
        //死信队列绑定死信交换机与 routingkey
        channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi");
    
        //正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        //正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");
        String normalQueue = "normal-queue"; 
    	channel.queueDeclare(normalQueue, false, false, false, params);
 		channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");
    
        System.out.println("等待接收消息 ........... ");
        DeliverCallback deliverCallback = (consumerTag, delivery) -> { 
            String message = new String(delivery.getBody(), "UTF-8"); 		
            System.out.println("Consumer01 接收到消息"+message);
        };
        channel.basicConsume(normalQueue, true, deliverCallback, consumerTag -> {
        });
	}
}
```

消费者2；只管接受死信交换机

```java
public class Consumer02 {
private static final String DEAD_EXCHANGE = "dead_exchange";
public static void main(String[] argv) throws Exception {
	Channel channel = RabbitUtils.getChannel();
    channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT); 
    String deadQueue = "dead-queue";
	channel.queueDeclare(deadQueue, false, false, false, null); 
    channel.queueBind(deadQueue, DEAD_EXCHANGE, "lisi"); 
    System.out.println("等待接收死信队列消息 ........... ");
    DeliverCallback deliverCallback = (consumerTag, delivery) -> { 
        String message = new String(delivery.getBody(), "UTF-8"); 
        System.out.println("Consumer02 接收死信队列的消息" + message);
    };
	channel.basicConsume(deadQueue, true, deliverCallback, consumerTag -> {
	});
}
```

### 队列达到最大长度

生产者去掉TTL属性

消费者：

```
 //正常队列绑定死信队列信息
        Map<String, Object> params = new HashMap<>();
        //正常队列设置死信交换机 参数 key 是固定值
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        //正常队列设置死信 routing-key 参数 key 是固定值
        params.put("x-dead-letter-routing-key", "lisi");
         //正常队列设置长度
         params.put("x-mac-length", "lisi");
        String normalQueue = "normal-queue"; 
    	channel.queueDeclare(normalQueue, false, false, false, params);
 		channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");
```

改变属性，原先有队列记得删除

### 消息被拒

1. 消息生产者代码同上生产者一致

消费者改变一下

```java
//正常队列绑定死信队列信息
Map<String, Object> params = new HashMap<>();
//正常队列设置死信交换机 参数 key 是固定值
params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
//正常队列设置死信 routing-key 参数 key 是固定值
params.put("x-dead-letter-routing-key", "lisi"); String normalQueue = "normal-queue";
channel.queueDeclare(normalQueue, false, false, false, params); channel.queueBind(normalQueue, NORMAL_EXCHANGE, "zhangsan");
//上面一样
System.out.println("等待接收消息 ........... ");
DeliverCallback deliverCallback = (consumerTag, delivery) -> { 
    String message = new String(delivery.getBody(), "UTF-8");
if(message.equals("info5")){
    System.out.println("Consumer01 接收到消息" + message + "并拒绝签收该消息");
    //requeue 设置为 false 代表拒绝重新入队 该队列如果配置了死信交换机将发送到死信队列中
    channel.basicReject(delivery.getEnvelope().getDeliveryTag(), false);
}else {
    System.out.println("Consumer01 接收到消息"+message); 
    //下面的false是批量的意思
    channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
}
};
    boolean autoAck = false;
    channel.basicConsume(normalQueue, autoAck, deliverCallback, consumerTag -> {
    });
```



## 延迟队列

基于死信队列，

结合springboot

```java
@Bean("queueA")
public Queue queueA(){
Map<String, Object> args = new HashMap<>(3);
//声明当前队列绑定的死信交换机
args.put("x-dead-letter-exchange", Y_DEAD_LETTER_EXCHANGE);
//声明当前队列的死信路由 key
args.put("x-dead-letter-routing-key", "YD");
//声明队列的 TTL
args.put("x-message-ttl", 10000);
return QueueBuilder.durable(QUEUE_A).withArguments(args).build();
}
// 声明队列 A 绑定 X 交换机
@Bean
public Binding queueaBindingX(@Qualifier("queueA") Queue queueA,@Qualifier("xExchange") DirectExchange xExchange){
    
    return BindingBuilder.bind(queueA).to(xExchange).with("XA");
}
```

**声明队列时绑定了y**

**后面又绑定x**

X=>q=>Y

消费者，生产者略



注意：

看起来似乎没什么问题，但是在最开始的时候，就介绍过如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时“死亡“，因为 **RabbitMQ 只会检查第一个消息是否过期**，如果过期则丢到死信队列， **如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行**。



rabbitmq_delayed_message_exchange 插件：延迟到了交换机



## 发布确认高级



有可能交换机坏了，或者队列，进行回调。



## 幂等性







### 消息重复消费

消费者在消费 MQ 中的消息时，**MQ 已把消息发送给消费者，消费者在给MQ 返回 ack 时网络中断， 故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者**，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。

### 解决思路

**MQ 消费者的幂等性的解决一般使用全局 ID 或者写个唯一标识比如时间戳** 或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，**每次消费消息时用该 id 先判断该消息是否已消费过。**

### 消费端的幂等性保障

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性， 这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。业界主流的幂等性有两种操作:a. 唯一 ID+指纹码机制,利用数据库主键去重, b.**利用 redis 的原子性去实现**

### 唯一ID+指纹码机制

指纹码:我们的一些规则或者**时间戳加别的服务给到的唯一信息码**,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中,优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，**如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。**

### Redis 原子性

利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费

## 优先级队列

对一些大客户

## 惰性队列

惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。



## 惰性队列

惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。

### 回调接口

```
@Component
@Slf4j
public class MyCallBack implementsRabbitTemplate.ConfirmCallback {
    /**
     * 1. CorrelationData 保存回调消息的ID及相关信息
     * 1.2 交换机收到消息  ack=true
     * 1.3  cause  null
      *2. 发消息  交换机接收失败了 回调
      *2.1 correlation  保存回调消息的ID及相关信息
      *2.2  交换机收到消息  ack = false
      * cause 失败的原因
       */
    @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        String id=correlationData!=null?correlationData.getId():"";
        if(ack){
            log.info("交换机已经收到 id 为:{}的消息",id);
        }else{
            log.info("交换机还未收到 id 为:{}消息,由于原因:{}",id,cause);
        }
    }
}
```



# **回调函数**

1、对普通函数的调用：调用程序发出对普通函数的调用后，程序执行立即转向被调用函数执行，直到被调用函数执行完毕后，
再返回调用程序继续执行。从发出调用的程序的角度看，这个过程为“调用-->等待被调用函数执行完毕-->继续执行”。

2、对回调函数调用：调用程序发出对回调函数的调用后，不等函数执行完毕，立即返回并继续执行。这样，调用程序执和被调用函数同时在执行。
当被调函数执行完毕后，被调函数会反过来调用某个事先指定函数，以通知调用程序：函数调用结束。这个过程称为回调（Callback），这正是回调函数名称的由来。



callback 是一种特殊的函数，这个函数被作为参数传给另一个函数去调用。这样的函数就是回调函数。


主函数需要调用回调函数

中间函数登记回调函数

触发回调函数事件

调用回调函数

响应回调事件



所谓回调，就是客户程序C调用服务程序S中的某个方法a，然后S又在某个时候反过来调用C中的某个方法b，对于C来说，这个b便叫做回调函数。
一般说来，C不会自己调用b，C提供b的目的就是让S来调用它，而且是C不得不提供。由于S并不知道C提供的b叫甚名谁，
所以S会约定b的接口规范（函数原型），然后由C提前通过S的一个函数r告诉S自己将要使用b函数，这个过程称为回调函数的注册，r称为注册函数。
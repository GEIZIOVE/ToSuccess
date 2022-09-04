# [RabbitMQ 之死信队列](https://www.iocoder.cn/RabbitMQ/dead-letter-queue/)

## DLX

### 1.定义

`DLX, Dead-Letter-Exchange`。利用DLX, 当消息在一个队列中变成死信（dead message）之后，它能被重新publish到另一个**Exchange**，这个**Exchange**就是DLX。消息变成死信一向有一下几种情况：

- 消息被拒绝（basic.reject/ basic.nack）并且requeue=false
- 消息TTL过期（参考：[RabbitMQ之TTL（Time-To-Live 过期时间）](http://blog.csdn.net/u013256816/article/details/54916011)）
- 队列达到最大长度

### 2.解释

**DLX也是一个正常的Exchange，和一般的Exchange没有区别**，它能在任何的队列上被指定，实际上就是设置某个队列的属性，当这个队列中有死信时，RabbitMQ就会自动的将这个消息重新发布到设置的Exchange上去，进而被路由到另一个队列，可以监听这个队列中消息做相应的处理，这个特性可以弥补RabbitMQ 3.0以前支持的immediate参数（可以参考[RabbitMQ之mandatory和immediate](http://blog.csdn.net/u013256816/article/details/54914525)）的功能。

### 3.使用

核心代码实现：通过在`queueDeclare`方法中加入“x-dead-letter-exchange”实现。

```java
channel.exchangeDeclare("some.exchange.name", "direct");

Map<String, Object> args = new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "some.exchange.name");
channel.queueDeclare("myqueue", false, false, false, args);
```



你也可以为这个DLX指定routing key，如果没有特殊指定，则使用原队列的routing key

```
args.put("x-dead-letter-routing-key", "some-routing-key");
```



还可以使用policy来配置：

```
rabbitmqctl set_policy DLX ".*" '{"dead-letter-exchange":"my-dlx"}' --apply-to queues
```



修改[RabbitMQ之TTL（Time-To-Live 过期时间）](http://blog.csdn.net/u013256816/article/details/54916011)中的例子：



```java
public static void createQueue(){
    try {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(ip);
        factory.setPort(port);
        factory.setUsername(username);
        factory.setPassword(password);

        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        Map<String, Object>  argss = new HashMap<String, Object>();
        argss.put("vhost", "/");
        argss.put("username","root");
        argss.put("password", "root");
        argss.put("x-message-ttl",6000);
        argss.put("x-dead-letter-exchange","exchange.dlx.test");
 argss.put("x-dead-letter-routing-key","queue.dlx.test");
        channel.queueDeclare("queue.dlx.test", durable, exclusive, autoDelete, argss);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (TimeoutException e) {
        e.printStackTrace();
    }
}
```



通过RabbitMQ的管理界面可以看到：![这里写图片描述](E:\Development\Typora\images\96387d6eccc26f1a4226a70cbe2bdb6e.png)queue.dlx.test这个queue中有个“DLX"和“DLK”的标记. DLX关联的是exchangeName, DLK关联的是routingKey.

### **详细说明**： 

​	在RabbitMQ中有两个**exchange**: `exchange.dlx.self`和`exchange.dlx.test`，

​											两个**queue**：`queue.dlx.test`和`%DLX%queue.dlx.test`     

`exchange.dlx.self`是正常情况下，生产者发送消息到此exchange中，绑定关系如图：![这里写图片描述](E:\Development\Typora\images\01375ad70d4769baee262dc4adb00e6b.png)

`exchang.dlx.test`是产生死信之后，原queue[queue.dlx.test]的死信发送到此exchange中，绑定关系如图：![这里写图片描述](E:\Development\Typora\images\4f00b88b98fb2168ab9d08df91425fd7.png)

数据首先发送到 **exchange[exchange.dlx.self]**，根据routingkey[dlx]路由到queue.dlx.test,如果正常情况下，消费者可以消费queue.dlx.test的内容。但是如果queue.dlx.test中有消息变成了**dead message**即死信了，那么这个死信则会通过exchangeName=exchange.dlx.test, routingKey="queue.dlx.test"路由到死信队列%DLX%queue.dlx.test中，如果要消费这个dead message, 此时消费者必须消费%DLX%queue.dlx.test中的内容而不是queue.dlx.test中的内容。

> 如果不指定x-dead-letter-routing-key参数，则使用原来的routingkey
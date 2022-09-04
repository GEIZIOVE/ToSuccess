# 1. 概述

如果胖友还没了解过分布式消息队列 [RabbitMQ](https://www.rabbitmq.com/) ，建议先阅读下艿艿写的 [《芋道 RabbitMQ 极简入门》](http://www.iocoder.cn/RabbitMQ/install/?self) 文章。虽然这篇文章标题是安装部署，实际可以理解成《一文带你快速入门 RabbitMQ》，哈哈哈。

考虑这是 RabbitMQ 如何在 Spring Boot 整合与使用的文章，所以还是简单介绍下 RabbitMQ 是什么？

> FROM [《AMQP 消息服务器 RabbitMQ》](https://www.oschina.net/p/rabbitmq)
>
> RabbitMQ 是由 LShift 提供的一个 Advanced Message Queuing Protocol (AMQP) 的开源实现，由以高性能、健壮以及可伸缩性出名的 ErFlang 写成，因此也是继承了这些优点。
>
> ![img](E:\Development\Typora\images\991c599424d61e17aea8eb76571715c9.png)
>
> - AMQP 里主要要说两个组件：Exchange 和 Queue ，绿色的 X 就是 Exchange ，红色的是 Queue ，这两者都在 Server 端，又称作 Broker ，这部分是 RabbitMQ 实现的。
> - 而蓝色的则是客户端，通常有 Producer 和 Consumer 两种类型（角色）。

在本文中，我们会比 [《芋道 RabbitMQ 极简入门》](http://www.iocoder.cn/RabbitMQ/install/?self) 提供更多的生产者 Producer 和消费者 Consumer 的使用示例。例如说：

- 四种类型的交换机( Exchange )
- Producer 发送**顺序**消息，Consumer **顺序**消费消息。
- Producer 发送**定时**消息。
- Producer **批量**发送消息。
- Producer 发送**事务**消息。
- Consumer **广播**和**集群**消费消息。
- Consumer 批量消费消息。

# 2. Spring-AMQP

在 Spring 生态中，提供了 [Spring-AMQP](https://spring.io/projects/spring-amqp) 项目，让我们更简便的使用 AMQP 。其官网介绍如下：

> The Spring AMQP project applies core Spring concepts to the development of AMQP-based messaging solutions. Spring-AMQP 项目将 Spring 核心概念应用于基于 AMQP 的消息传递解决方案的开发。
>
> It provides a "template" as a high-level abstraction for sending and receiving messages. 它提供了一个“模板”作为发送消息的高级抽象。
>
> It also provides support for Message-driven POJOs with a "listener container". 它还通过“侦听器容器”为消息驱动的 POJO 提供支持。
>
> These libraries facilitate management of AMQP resources while promoting the use of dependency injection and declarative configuration. 这些库促进 AMQP 资源的管理，同时促进使用依赖注入和声明性配置。
>
> In all of these cases, you will see similarities to the JMS support in the Spring Framework. 在所有这些情况下，您将看到与 Spring 框架中的 JMS 支持的相似之处。
>
> The project consists of two parts; spring-amqp is the base abstraction, and spring-rabbit is the RabbitMQ implementation. 该项目包括两个部分：
>
> - [`spring-amqp`](https://mvnrepository.com/artifact/org.springframework.amqp/spring-amqp) 是 AMQP 的基础抽象。
> - [`spring-rabbit`](https://mvnrepository.com/artifact/org.springframework.amqp/spring-rabbit) 是基于 RabbitMQ 对 AMQP 的具体实现。

- 哈哈哈，艿艿蹩脚的英语，很塑料。😈 其实重点是最后一段内容，相信胖友能够明白意思的。

> Features(功能特性)
>
> - Listener container for asynchronous processing of inbound messages 监听器容器：异步处理接收到的消息
> - [RabbitTemplate](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/RabbitTemplate.java) for sending and receiving messages RabbitTemplate：发送和接收消息
> - [RabbitAdmin](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/RabbitAdmin.java) for automatically declaring queues, exchanges and bindings RabbitAdmin：自动创建队列，交换器，绑定器。

在 [Spring-Boot](https://spring.io/projects/spring-boot) 项目中，提供了 AMQP 和 RabbitMQ 的自动化配置，所以我们仅需引入 [`spring-boot-starter-amqp`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-amqp) 依赖，即可愉快的使用。

# 3. 快速入门

> 示例代码对应仓库：[lab-04-rabbitmq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo) 。

在 AMQP 中，Producer 将消息发送到 Exchange ，再由 Exchange 将消息路由到一个或多个 Queue 中（或者丢弃）。

> 概念的讲解，引用自 [《RabbitMQ 基础概念详解》](http://www.iocoder.cn/RabbitMQ/Detailed-understanding-of-the-basic-concepts-of-RabbitMQ/?self) 文章。

Exchange 根据 Routing Key 和 Binding Key 将消息路由到 Queue 。目前提供了 Direct、Topic、Fanout、Headers 四种类型。

## 3.1 Direct Exchange

Direct 类型的 Exchange 路由规则比较简单，它会把消息路由到那些 binding key 与 routing key 完全匹配的 Queue 中。以下图的配置为例：

![Direct Exchange](E:\Development\Typora\images\aeb33c91bbf83726c24ba1dae9dc4e00.png)

- 我们以 `routingKey="error"` 发送消息到 Exchange ，则消息会路由到 Queue1(`amqp.gen-S9b…`) 。
- 我们以 `routingKey="info"` 或 `routingKey="warning"` 来发送消息，则消息只会路由到 Queue2(`amqp.gen-Agl…`) 。
- 如果我们以其它 routingKey 发送消息，则消息不会路由到这两个 Queue 中。
- 总结来说，指定 Exchange + routing key ，有且仅会路由到至多一个 Queue 中。😈 极端情况下，如果没有匹配，消息就发送到“空气”中，不会进入任何 Queue 中。

> 注：Queue 名字 `amqp.gen-S9b…` 和 `amqp.gen-Agl…` 自动生成的。

下面，我们来创建一个 Direct Exchange 的使用示例，对应 [lab-04-rabbitmq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo) 项目。

### 3.1.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-04-rabbitmq-demo</artifactId>

    <dependencies>
        <!-- 实现对 RabbitMQ 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <!-- 方便等会写单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```



具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。

### 3.1.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
```



- 在 `spring.rabbitmq` 配置项，设置 RabbitMQ 的配置，对应 [RabbitProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java) 配置类。这里咱暂时最小化添加，更多的配置项，我们在下文的示例中，一点点抽丝剥茧。
- Spring Boot 提供的 [RabbitAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitAutoConfiguration.java) 自动化配置类，实现 RabbitMQ 的自动配置，创建相应的 Producer 和 Consumer 。

### 3.1.3 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```
// Application.java

@SpringBootApplication
@EnableAsync // 开启异步
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



- 我们额外添加了 `@EnableAsync` 注解，因为我们稍后要使用 Spring 提供的异步调用的功能。不了解这块的胖友，可以看看艿艿写的 [《芋道 Spring Boot 异步任务入门》](http://www.iocoder.cn/Spring-Boot/Async-Job/?self) 文章。

### 3.1.4 Demo01Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo01Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo01Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo01Message.java

public class Demo01Message implements Serializable {

    public static final String QUEUE = "QUEUE_DEMO_01";

    public static final String EXCHANGE = "EXCHANGE_DEMO_01";

    public static final String ROUTING_KEY = "ROUTING_KEY_01";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 注意，要实现 Java Serializable 序列化接口。因为 RabbitTemplate 默认使用 Java 自带的序列化方式，进行序列化 POJO 类型的消息。
- 在消息类里，我们枚举了 Exchange、Queue、RoutingKey 的名字。

### 3.1.5 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 Direct Exchange 示例相关的 Exchange、Queue、Binding 的配置。代码如下：



```
// RabbitConfig.java

import org.springframework.amqp.core.Queue;

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo01Queue() {
            return new Queue(Demo01Message.QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo01Exchange() {
            return new DirectExchange(Demo01Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo01Message.EXCHANGE
        // Routing key：Demo01Message.ROUTING_KEY
        // Queue：Demo01Message.QUEUE
        @Bean
        public Binding demo01Binding() {
            return BindingBuilder.bind(demo01Queue()).to(demo01Exchange()).with(Demo01Message.ROUTING_KEY);
        }

    }

}
```



- 在 DirectExchangeDemoConfiguration 内部静态类中，我们创建了 Exchange、Queue、Binding 三个 Bean ，后续 [RabbitAdmin](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/RabbitAdmin.java) 会自动创建交换器、队列、绑定器。感兴趣的胖友，可以看看 [`RabbitAdmin#initialize()`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/RabbitAdmin.java#L555-L612) 方法，或 [《RabbitMQ 自动创建队列/交换器/绑定》](https://my.oschina.net/huaxian8812/blog/782300) 文章。

### 3.1.6 Demo01Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo01Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo01Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// Demo01Producer.java

@Component
public class Demo01Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo01Message 消息
        Demo01Message message = new Demo01Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo01Message.EXCHANGE, Demo01Message.ROUTING_KEY, message);
    }

    public void syncSendDefault(Integer id) {
        // 创建 Demo01Message 消息
        Demo01Message message = new Demo01Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo01Message.QUEUE, message);
    }

    @Async
    public ListenableFuture<Void> asyncSend(Integer id) {
        try {
            // 发送消息
            this.syncSend(id);
            // 返回成功的 Future
            return AsyncResult.forValue(null);
        } catch (Throwable ex) {
            // 返回异常的 Future
            return AsyncResult.forExecutionException(ex);
        }
    }

}
```



- RabbitTemplate 是 [AmqpTemplate](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/AmqpTemplate.java) 接口的实现类，所以此时使用 AmqpTemplate 亦可。不过又因为 RabbitTemplate 还实现了其它接口，所以操作会更为丰富。因此，这里我们选择了注入 RabbitTemplate 属性。

- `#syncSend(Integer id)` 方法，调用 RabbitTemplate 的同步发送消息方法。方法定义如下：

  ```
  // AmqpTemplate.java
  
  void convertAndSend(String exchange, String routingKey, Object message) throws AmqpException;
  ```

  

  - 指定 Exchange + RoutingKey ，从而路由到一个 Queue 中。

- `#syncSendDefault(Integer id)` 方法，也调用 RabbitTemplate 的同步发送消息方法。方法定义如下：

  ```
  // AmqpTemplate.java
  
  void convertAndSend(String routingKey, Object message) throws AmqpException;
  ```

  

  - 是不是觉得有点奇怪，这里我们传入的 RoutingKey 为队列名？！因为 RabbitMQ 有一条默认的 [Exchange: (AMQP default)](https://emacsist.github.io/2015/12/06/rabbitmq中的基本概念/#default-exchange-默认交换机) 规则：`The default exchange is implicitly bound to every queue, with a routing key equal to the queue name. It is not possible to explicitly bind to, or unbind from the default exchange. It also cannot be deleted` 。
  - 翻译过来的意思：默认交换器，隐式地绑定到每个队列，路由键等于队列名称。
  - 所以，此处即使我们传入的 RoutingKey 为队列名，一样可以发到对应队列。

- `#asyncSend(Integer id)` 方法，通过 `@Async` 注解，声明异步调用该方法，从而实现异步消息到 RabbitMQ 中。因为 RabbitTemplate 并未像 [KafkaTemplate](https://github.com/spring-projects/spring-kafka/blob/master/spring-kafka/src/main/java/org/springframework/kafka/core/KafkaTemplate.java) 或 [RocketMQTemplate](https://github.com/apache/rocketmq-spring/blob/master/rocketmq-spring-boot/src/main/java/org/apache/rocketmq/spring/core/RocketMQTemplate.java) 直接提供了异步发送消息的方法，所以我们需要结合 Spring 异步调用来实现。

  > 在写完本文之后，发现还存在 [AsyncRabbitTemplate](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/AsyncRabbitTemplate.java) 类，提供一部的 RabbitMQ 操作。😈

### 3.1.7 Demo01Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo01Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo01Consumer.java) 类，消费消息。代码如下：



```
// Demo01Consumer.java

@Component
@RabbitListener(queues = Demo01Message.QUEUE)
public class Demo01Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo01Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

//    @RabbitHandler(isDefault = true)
//    public void onMessage(org.springframework.amqp.core.Message message) {
//        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
//    }

}
```



- 在类上，添加了 [`@RabbitListener`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/annotation/RabbitListener.java) 注解，声明了消费的队列是 `"QUEUE_DEMO_01"` 。
- 在方法上，添加了 [`@RabbitHandler`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/annotation/RabbitHandler.java) 注解，申明了处理消息的方法。同时，方法入参为消息的类型。这里，我们设置了[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 。
- 如果我们想要获得消费消息的更多信息，例如说，RoutingKey、创建时间等等信息，则可以考虑使用艿艿**注释掉的那段代码**，通过方法入参为 [`org.springframework.amqp.core.Message`](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/Message.java) 类型。不过绝大多数情况下，我们并不需要这么做。

### 3.1.8 简单测试

创建 [Demo01ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo01ProducerTest.java) 测试类，编写三个单元测试方法，调用 Demo01Producer 三个发送消息的方式。代码如下：



```
// Demo01ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo01ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo01Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void tesSyncSendDefault() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSendDefault(id);
        logger.info("[tesSyncSendDefault][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testAsyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.asyncSend(id).addCallback(new ListenableFutureCallback<Void>() {

            @Override
            public void onFailure(Throwable e) {
                logger.info("[testASyncSend][发送编号：[{}] 发送异常]]", id, e);
            }

            @Override
            public void onSuccess(Void aVoid) {
                logger.info("[testASyncSend][发送编号：[{}] 发送成功，发送成功]", id);
            }

        });
        logger.info("[testASyncSend][发送编号：[{}] 调用完成]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- 比较简单，胖友自己看下三个单元测试方法。

我们来执行 `#testSyncSend()` 方法，测试同步发送消息。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 00:19:18.736  INFO 87164 --- [           main] c.i.s.l.r.producer.Demo01ProducerTest    : [testSyncSend][发送编号：[1575908358] 发送成功]

# Demo01Consumer 成功消费了该消息
2019-12-15 00:19:18.751  INFO 87164 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo01Consumer        : [onMessage][线程编号:17 消息内容：Demo01Message{id=1575908358}]
```



- 同步发送的消息，成功被消费。

我们再来执行 `#tesSyncSendDefault()` 方法，测试另一个同步发送消息。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 00:20:50.226  INFO 87515 --- [           main] c.i.s.l.r.producer.Demo01ProducerTest    : [tesSyncSendDefault][发送编号：[1575908450] 发送成功]

# Demo01Consumer 成功消费了该消息
2019-12-15 00:20:50.240  INFO 87515 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo01Consumer        : [onMessage][线程编号:17 消息内容：Demo01Message{id=1575908450}]
```



- 同步发送的消息，成功也被消费。

我们最后来执行 `#testAsyncSend()` 方法，测试异步发送消息。控制台输出如下：



```
# Producer 异步发送消息的调用完成。
2019-12-15 00:22:48.891  INFO 88018 --- [           main] c.i.s.l.r.producer.Demo01ProducerTest    : [testASyncSend][发送编号：[1575908568] 调用完成]

# Producer 异步发送消息成功。【回调】
2019-12-15 00:22:48.905  INFO 88018 --- [         task-1] c.i.s.l.r.producer.Demo01ProducerTest    : [testASyncSend][发送编号：[1575908568] 发送成功，发送成功]

# Demo01Consumer 成功消费了该消息
2019-12-15 00:22:48.918  INFO 88018 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo01Consumer        : [onMessage][线程编号:17 消息内容：Demo01Message{id=1575908568}]
```



- 异步发送的消息，成功也被消费。

## 3.2 Topic Exchange

前面讲到 Direct Exchange路由规则，是完全匹配 binding key 与routing key，但这种严格的匹配方式在很多情况下不能满足实际业务需求。

Topic Exchange 在匹配规则上进行了扩展，它与 Direct 类型的Exchange **相似**，也是将消息路由到 binding key 与 routing key 相匹配的 Queue 中，但这里的匹配规则有些不同，它约定：

- routing key 为一个句点号 `"."` 分隔的字符串。我们将被句点号`"."`分隔开的每一段独立的字符串称为一个单词，例如 "stock.usd.nyse"、"nyse.vmw"、"quick.orange.rabbit"
- binding key 与 routing key 一样也是句点号 `"."` 分隔的字符串。
- binding key 中可以存在两种特殊字符 `"*"` 与 `"#"`，用于做模糊匹配。其中 `"*"` 用于匹配一个单词，`"#"` 用于匹配多个单词（可以是零个）。

以下图中的配置为例：![Topic Exchange](E:\Development\Typora\images\d343228b9d7606ac673ccd0028d4e424.png)

- `routingKey="quick.orange.rabbit"` 的消息会同时路由到 Q1 与 Q2 。
- `routingKey="lazy.orange.fox"` 的消息会路由到 Q1 。
- `routingKey="lazy.brown.fox"` 的消息会路由到 Q2 。
- `routingKey="lazy.pink.rabbit"` 的消息会路由到Q2（只会投递给 Q2 一次，虽然这个 routingKey 与 Q2 的两个 bindingKey 都匹配）。
- `routingKey="quick.brown.fox"`、`routingKey="orange"`、`routingKey="quick.orange.male.rabbit"` 的消息将会被丢弃，因为它们没有匹配任何 bindingKey 。

下面，我们来创建一个 Topic Exchange 的使用示例，继续在 [lab-04-rabbitmq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo) 项目。

### 3.2.1 Demo02Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo02Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo02Message.java) 消息类，提供给当前示例使用。代码如下：



```java
// Demo02Message.java

public class Demo02Message implements Serializable {

    public static final String QUEUE = "QUEUE_DEMO_02";

    public static final String EXCHANGE = "EXCHANGE_DEMO_02";

    public static final String ROUTING_KEY = "#.yu.nai";

    /**
     * 编号     
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 在消息类里，我们枚举了 Exchange、Queue、RoutingKey 的名字。
- 重点看我们新定义的路由键 `ROUTING_KEY = "#.yu.nai"` ，我们要匹配以 `"yu.nai"` 结尾，开头可以是任意个单词的。

### 3.2.2 RabbitConfig

修改 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 Topic Exchange 示例相关的 Exchange、Queue、Binding 的配置。代码如下：



```java
// RabbitConfig.java

/**
 * Topic Exchange 示例的配置类
 */
public static class TopicExchangeDemoConfiguration {

    // 创建 Queue
    @Bean
    public Queue demo02Queue() {
        return new Queue(Demo02Message.QUEUE, // Queue 名字
                true, // durable: 是否持久化
                false, // exclusive: 是否排它
                false); // autoDelete: 是否自动删除
    }

    // 创建 Topic Exchange
    @Bean
    public TopicExchange demo02Exchange() {
        return new TopicExchange(Demo02Message.EXCHANGE,
                true,  // durable: 是否持久化
                false);  // exclusive: 是否排它
    }

    // 创建 Binding
    // Exchange：Demo02Message.EXCHANGE
    // Routing key：Demo02Message.ROUTING_KEY
    // Queue：Demo02Message.QUEUE
    @Bean
    public Binding demo02Binding() {
        return BindingBuilder.bind(demo02Queue()).to(demo02Exchange()).with(Demo02Message.ROUTING_KEY);
    }

}
```



- 在 TopicExchangeDemoConfiguration 内部静态类中，我们**也是**创建了 Exchange、Queue、Binding 三个 Bean 。有差异点的是，这次我们创建的是 [TopicExchange](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/TopicExchange.java) 。

### 3.2.3 Demo02Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo02Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo02Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```java
// Demo02Producer.java

@Component
public class Demo02Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id, String routingKey) {
        // 创建 Demo02Message 消息
        Demo02Message message = new Demo02Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo02Message.EXCHANGE, routingKey, message);
    }

}
```



- 和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的 `#syncSend(Integer id)` 方法大体相似，差异点在于新增了方法参数 `routingKey` ，方便我们传入不同的路由键。

### 3.2.4 Demo02Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo02Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo02Consumer.java) 类，消费消息。

和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_02"` 。

### 3.2.5 简单测试

创建 [Demo02ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo02ProducerTest.java) 测试类，编写两个单元测试方法，调用 Demo02Producer 发送消息的方法。代码如下：



```java
// Demo02ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo02ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo02Producer producer;

    @Test
    public void testSyncSendSuccess() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id, "da.yu.nai");
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testSyncSendFailure() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id, "yu.nai.shuai");
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- `#testSyncSendSuccess()` 方法，发送消息的 RoutingKey 是 `"da.yu.nai"` ，可以匹配到 `"DEMO_QUEUE_02"` 。
- `#testSyncSendFailure()` 方法，发送消息的 RoutingKey 是 `"yu.nai.shuai"` ，无法匹配到 `"DEMO_QUEUE_02"` 。

我们先来执行 `#testSyncSendSuccess()` 方法，可以匹配到 `"DEMO_QUEUE_02"` 的情况。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 09:35:54.924  INFO 6894 --- [           main] c.i.s.l.r.producer.Demo02ProducerTest    : [testSyncSend][发送编号：[1575941754] 发送成功]

# Demo02Consumer 成功消费了该消息
2019-12-15 09:35:54.941  INFO 6894 --- [ntContainer#1-1] c.i.s.l.r.consumer.Demo02Consumer        : [onMessage][线程编号:19 消息内容：Demo02Message{id=1575941754}]
```



- 符合预期。

我们再来执行 `#testSyncSendFailure()` 方法，无法匹配到 `"DEMO_QUEUE_02"` 的情况。控制台输出如下：



```
// Producer 同步发送消息成功。
2019-12-15 09:37:11.353  INFO 7186 --- [           main] c.i.s.l.r.producer.Demo02ProducerTest    : [testSyncSend][发送编号：[1575941831] 发送成功]
```



- 符合预期。因为 无法匹配到 `"DEMO_QUEUE_02"` ，自然 Demo02Consumer 无法进行消费。

## 3.3 Fanout Exchange

Fanout Exchange 路由规则非常简单，它会把所有发送到该 Exchange 的消息路由到所有与它绑定的 Queue 中。如下图：

![Fanout Exchange](E:\Development\Typora\images\203b64e17bd9e398cf619acb5df98e6b.png)

- 生产者（P）发送到 Exchange（X）的所有消息都会路由到图中的两个 Queue，并最终被两个消费者（C1 与 C2）消费。
- 总结来说，指定 Exchange ，会路由到**多个**绑定的 Queue 中。

下面，我们来创建一个 Fanout Exchange 的使用示例，继续在 [lab-04-rabbitmq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo) 项目。

### 3.3.1 Demo03Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo03Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo03Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo03Message.java

public class Demo03Message implements Serializable {

    public static final String QUEUE_A = "QUEUE_DEMO_03_A";
    public static final String QUEUE_B = "QUEUE_DEMO_03_B";

    public static final String EXCHANGE = "EXCHANGE_DEMO_03";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 我们**未**定意思 RoutingKey 的名字。因为，Fanout Exchange 仅需要 Exchange 即可。
- 我们定义**两个** Queue 的名字。因为，我们要测试 Fanout Exchange 投递到多个 Queue 的效果。

### 3.3.2 RabbitConfig

修改 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 Fanout Exchange 示例相关的 Exchange、Queue、Binding 的配置。代码如下：



```
// RabbitConfig.java

/**
 * Fanout Exchange 示例的配置类
 */
public static class FanoutExchangeDemoConfiguration {

    // 创建 Queue A
    @Bean
    public Queue demo03QueueA() {
        return new Queue(Demo03Message.QUEUE_A, // Queue 名字
                true, // durable: 是否持久化
                false, // exclusive: 是否排它
                false); // autoDelete: 是否自动删除
    }

    // 创建 Queue B
    @Bean
    public Queue demo03QueueB() {
        return new Queue(Demo03Message.QUEUE_B, // Queue 名字
                true, // durable: 是否持久化
                false, // exclusive: 是否排它
                false); // autoDelete: 是否自动删除
    }

    // 创建 Fanout Exchange
    @Bean
    public FanoutExchange demo03Exchange() {
        return new FanoutExchange(Demo03Message.EXCHANGE,
                true,  // durable: 是否持久化
                false);  // exclusive: 是否排它
    }

    // 创建 Binding A
    // Exchange：Demo03Message.EXCHANGE
    // Queue：Demo03Message.QUEUE_A
    @Bean
    public Binding demo03BindingA() {
        return BindingBuilder.bind(demo03QueueA()).to(demo03Exchange());
    }

    // 创建 Binding B
    // Exchange：Demo03Message.EXCHANGE
    // Queue：Demo03Message.QUEUE_B
    @Bean
    public Binding demo03BindingB() {
        return BindingBuilder.bind(demo03QueueB()).to(demo03Exchange());
    }

}
```



- 在 FanoutExchangeDemoConfiguration 内部静态类中，我们**也是**创建了 Exchange、Queue、Binding 三个 Bean 。有差异点的是，这次我们创建的是 [FanoutExchange](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/FanoutExchange.java) 。
- 同时，因为我们要投递到两个 Queue 中，所以我们创建了两个 Binding 。

### 3.3.3 Demo03Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo03Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo03Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// Demo03Producer.java

@Component
public class Demo03Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo03Message 消息
        Demo03Message message = new Demo03Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo03Message.EXCHANGE, null, message);
    }

}
```



- 和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的 `#syncSend(Integer id)` 方法大体相似，差异点在于传入 `routingKey = null` ，因为不需要。

### 3.3.4 Demo03Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo03ConsumerA](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo03ConsumerA.java) 和 [Demo03ConsumerB](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo03ConsumerB.java) **两个**类，消费消息。代码如下：



```
// Demo03ConsumerA.java
@Component
@RabbitListener(queues = Demo03Message.QUEUE_A)
public class Demo03ConsumerA {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo03Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}

// Demo03ConsumerB.java
@Component
@RabbitListener(queues = Demo03Message.QUEUE_B)
public class Demo03ConsumerB {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo03Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 两个消费者，分别消费 `"QUEUE_DEMO_03_A"`、`"QUEUE_DEMO_03_B"` 队列。

### 3.3.5 简单测试

创建 [Demo03ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo03ProducerTest.java) 测试类，编写一个单元测试方法，调用 Demo03Producer 发送消息的方法。代码如下：



```
// Demo03ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo03ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo03Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



我们先来执行 `#testSyncSend()` 方法，确认下 Fanout Exchange 的效果。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 13:42:51.794  INFO 50616 --- [           main] c.i.s.l.r.producer.Demo03ProducerTest    : [testSyncSend][发送编号：[1575956571] 发送成功]

# Demo03ConsumerA 成功消费了该消息
2019-12-15 13:42:51.811  INFO 50616 --- [ntContainer#2-1] c.i.s.l.r.consumer.Demo03ConsumerA       : [onMessage][线程编号:22 消息内容：Demo03Message{id=1575956571}]

# Demo03ConsumerB 也成功消费了该消息
2019-12-15 13:42:51.811  INFO 50616 --- [ntContainer#3-1] c.i.s.l.r.consumer.Demo03ConsumerB       : [onMessage][线程编号:24 消息内容：Demo03Message{id=1575956571}]
```



- 符合预期。
- 发送的消息，成功投递到了两个队列中，所以被两个消费者都消费到了。

## 3.4 Headers Exchange

Headers Exchange 不依赖于 routing key 与 binding key 的匹配规则来路由消息，而是根据发送的消息内容中的 headers 属性进行匹配。

- 在绑定 Queue 与 Exchange 时指定一组 headers 键值对。
- 当消息发送到 Exchange 时，RabbitMQ 会取到该消息的 headers（也是一个键值对的形式），对比其中的键值对是否完全匹配 Queue 与 Exchange 绑定时指定的键值对；如果完全匹配则消息会路由到该 Queue ，否则不会路由到该 Queue 。

不过艿艿在查询资料的时候，有资料说 Headers Exchange 性能很差，实际场景也使用比较少。所以本小节的内容，胖友可以选择性看。

下面，我们来创建一个 Headers Exchange 的使用示例，继续在 [lab-04-rabbitmq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo) 项目。

### 3.4.1 Demo04Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo04Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo04Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo03Message.java

public class Demo04Message implements Serializable {

    public static final String QUEUE = "QUEUE_DEMO_04_A";

    public static final String EXCHANGE = "EXCHANGE_DEMO_04";

    public static final String HEADER_KEY = "color";
    public static final String HEADER_VALUE = "red";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 我们**未**定意思 RoutingKey 的名字。因为，Headers Exchange 是通过 Exchange + Headers 的组合。
- 我们定义**一个** Headers 键值对，`color = red` 。

### 3.4.2 RabbitConfig

修改 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 Headers Exchange 示例相关的 Exchange、Queue、Binding 的配置。代码如下：



```
// RabbitConfig.java

/**
 * Headers Exchange 示例的配置类
 */
public static class HeadersExchangeDemoConfiguration {

    // 创建 Queue
    @Bean
    public Queue demo04Queue() {
        return new Queue(Demo04Message.QUEUE, // Queue 名字
                true, // durable: 是否持久化
                false, // exclusive: 是否排它
                false); // autoDelete: 是否自动删除
    }

    // 创建 Headers Exchange
    @Bean
    public HeadersExchange demo04Exchange() {
        return new HeadersExchange(Demo04Message.EXCHANGE,
                true,  // durable: 是否持久化
                false);  // exclusive: 是否排它
    }

    // 创建 Binding
    // Exchange：Demo04Message.EXCHANGE
    // Queue：Demo04Message.QUEUE
    // Headers: Demo04Message.HEADER_KEY + Demo04Message.HEADER_VALUE
    @Bean
    public Binding demo4Binding() {
        return BindingBuilder.bind(demo04Queue()).to(demo04Exchange())
                .where(Demo04Message.HEADER_KEY).matches(Demo04Message.HEADER_VALUE); // 配置 Headers 匹配
    }

}
```



- 在 TopicExchangeDemoConfiguration 内部静态类中，我们**也是**创建了 Exchange、Queue、Binding 三个 Bean 。有差异点的是，这次我们创建的是 [HeadersExchange](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/HeadersExchange.java) 。
- 同时，我们创建的 Binding 是基于 Headers 匹配。

### 3.4.3 Demo04Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo04Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo04Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// Demo02Producer.java

@Component
public class Demo04Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id, String headerValue) {
        // 创建 MessageProperties 属性
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setHeader(Demo04Message.HEADER_KEY, headerValue); // 设置 header
        // 创建 Message 消息
        Message message = rabbitTemplate.getMessageConverter().toMessage(
                new Demo04Message().setId(id), messageProperties);
        // 同步发送消息
        rabbitTemplate.send(Demo04Message.EXCHANGE, null, message);
    }

}
```



- 和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的 `#syncSend(Integer id)` 方法大体相似，差异点在于新增了方法参数 `headerValue` ，方便我们传入不同的 Headers 值。
- 因为 RabbitTemplate 会提供很方便的传递 Headers 的 API 方法，所以我们只好自己构建，当然也比较简单哈。

### 3.4.4 Demo04Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo04Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo04Consumer.java) 类，消费消息。

和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_04"` 。

### 3.4.5 简单测试

创建 [Demo04ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo04ProducerTest.java) 测试类，编写两个单元测试方法，调用 Demo04Producer 发送消息的方法。代码如下：



```
// Demo04ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo04ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo04Producer producer;

    @Test
    public void testSyncSendSuccess() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id, Demo04Message.HEADER_VALUE);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testSyncSendFailure() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id, "error");
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- `#testSyncSendSuccess()` 方法，发送消息的 Headers 的值 `"red"` ，可以匹配到 `"DEMO_QUEUE_04"` 。
- `#testSyncSendFailure()` 方法，发送消息的 Headers 的值 `"error"` ，无法匹配到 `"DEMO_QUEUE_04"` 。

我们先来执行 `#testSyncSendSuccess()` 方法，可以匹配到 `"DEMO_QUEUE_04"` 的情况。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 14:30:05.872  INFO 61498 --- [           main] c.i.s.l.r.producer.Demo04ProducerTest    : [testSyncSend][发送编号：[1575959405] 发送成功]

# Demo04Consumer 成功消费了该消息
2019-12-15 14:30:05.888  INFO 61498 --- [ntContainer#4-1] c.i.s.l.r.consumer.Demo04Consumer        : [onMessage][线程编号:25 消息内容：Demo04Message{id=1575959405}]
```



- 符合预期。

我们再来执行 `#testSyncSendFailure()` 方法，无法匹配到 `"DEMO_QUEUE_04"` 的情况。控制台输出如下：



```
// Producer 同步发送消息成功。
2019-12-15 14:30:47.090  INFO 61664 --- [           main] c.i.s.l.r.producer.Demo04ProducerTest    : [testSyncSend][发送编号：[1575959447] 发送成功]
```



- 符合预期。因为 无法匹配到 `"DEMO_QUEUE_04"` ，自然 Demo04Consumer 无法进行消费。

# 4. 批量发送消息

> 示例代码对应仓库：[lab-04-rabbitmq-demo-batch](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch) 。

在一些业务场景下，我们希望使用 Producer 批量发送消息，提高发送性能。不同于我们在[《芋道 Spring Boot 消息队列 RocketMQ 入门》](http://www.iocoder.cn/Spring-Boot/RocketMQ/?self) 的[「4. 批量发送消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 功能，RocketMQ 是提供了一个可以批量发送多条消息的 API 。而 Spring-AMQP 提供的批量发送消息，它提供了一个 [MessageBatch](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/batch/MessageBatch.java) 消息收集器，将发送给**相同 Exchange + RoutingKey 的消息们**，“**偷偷**”收集在一起，当满足条件时候，一次性批量发送提交给 RabbitMQ Broker 。

Spring-AMQP 通过 [BatchingRabbitTemplate](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/BatchingRabbitTemplate.java) 提供批量发送消息的功能。如下是三个条件，满足**任一**即会批量发送：

- 【数量】`batchSize` ：超过收集的消息数量的最大条数。
- 【空间】`bufferLimit` ：超过收集的消息占用的最大内存。
- 【时间】`timeout` ：超过收集的时间的最大等待时长，单位：毫秒。😈 不过要注意，这里的超时开始计时的时间，是**以最后一次发送时间为起点**。也就说，每调用一次发送消息，都以当前时刻开始计时，重新到达 `timeout` 毫秒才算超时。

另外，BatchingRabbitTemplate 提供的批量发送消息的能力**比较弱**。对于同一个 BatchingRabbitTemplate 对象来说，**同一时刻只能有一个批次(保证 Exchange + RoutingKey 相同)**，否则会报错。

下面，我们来实现一个 Producer 批量发送消息的示例。考虑到不污染[「3. 快速入门」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 的示例，我们新建一个 [lab-04-rabbitmq-demo-batch](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch) 项目。

## 4.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/pom.xml) 文件。

## 4.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/resources/application.yaml) 文件。

## 4.3 Demo05Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo05Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo05Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 4.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 BatchingRabbitTemplate 的配置。代码如下：



```java
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo05Queue() {
            return new Queue(Demo05Message.QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo05Exchange() {
            return new DirectExchange(Demo05Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo05Message.EXCHANGE
        // Routing key：Demo05Message.ROUTING_KEY
        // Queue：Demo05Message.QUEUE
        @Bean
        public Binding demo05Binding() {
            return BindingBuilder.bind(demo05Queue()).to(demo05Exchange()).with(Demo05Message.ROUTING_KEY);
        }

    }

    @Bean
    public BatchingRabbitTemplate batchRabbitTemplate(ConnectionFactory connectionFactory) {
        // 创建 BatchingStrategy 对象，代表批量策略
        int batchSize = 16384; // 超过收集的消息数量的最大条数。
        int bufferLimit = 33554432; // 每次批量发送消息的最大内存
        int timeout = 30000; // 超过收集的时间的最大等待时长，单位：毫秒
        BatchingStrategy batchingStrategy = new SimpleBatchingStrategy(batchSize, bufferLimit, timeout);

        // 创建 TaskScheduler 对象，用于实现超时发送的定时器
        TaskScheduler taskScheduler = new ConcurrentTaskScheduler();

        // 创建 BatchingRabbitTemplate 对象
        BatchingRabbitTemplate batchTemplate = new BatchingRabbitTemplate(batchingStrategy, taskScheduler);
        batchTemplate.setConnectionFactory(connectionFactory); // 设置 RabbitMQ 连接工厂
        return batchTemplate;
    }

}
```



- DirectExchangeDemoConfiguration 配置类，用于定义 Queue、Exchange、Binding 的配置。

- ```java
  #batchRabbitTemplate(ConnectionFactory connectionFactory)
  ```

   

  方法，创建 BatchingRabbitTemplate Bean 对象。

  - 具体的 `batchSize`、`bufferLimit`、`timeout` 数值配置多少，根据自己的应用来。这里，我们故意将 `timeout` 配置成了 30 秒，主要为了演示之用。
  - 创建 BatchingRabbitTemplate 对象的代码，艿艿已经添加注释，可以自己阅读理解下噢。

## 4.5 Demo05Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo05Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo05Producer.java) 类，它会使用 Spring-AMQP 封装提供的 BatchingRabbitTemplate ，实现批量发送消息。代码如下：



```java
// Demo05Producer.java

@Component
public class Demo05Producer {

    @Autowired
    private BatchingRabbitTemplate batchingRabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo05Message 消息
        Demo05Message message = new Demo05Message();
        message.setId(id);
        // 同步发送消息
        batchingRabbitTemplate.convertAndSend(Demo05Message.EXCHANGE, Demo05Message.ROUTING_KEY, message);
    }

}
```



- 看起来和我们在[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)提供的发送消息的方法，除了换成了 BatchingRabbitTemplate 来发送消息，其它都是一致的。😈 对的，这也是为什么艿艿在上文说到，Spring-AMQP 是“**偷偷**”收集来实现批量发送，对于我们使用发送消息的方法，还是一致的。

BatchingRabbitTemplate 通过重写 [`#send(String exchange, String routingKey, Message message, CorrelationData correlationData)`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/BatchingRabbitTemplate.java#L76-L99) **核心**方法，实现批量发送的功能。感兴趣的胖友，可以自己去研究下源码，不复杂哈~

## 4.6 Demo05Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo05Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo05Consumer.java) 类，消费消息。代码如下：



```java
// Demo05Consumer.java

@Component
@RabbitListener(queues = Demo05Message.QUEUE)
public class Demo05Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo05Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_02"` 。

## 4.7 简单测试

创建 [Demo05ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo05ProducerTest.java) 测试类，编写单元测试方法，测试 Producer 批量发送消息的效果。代码如下：



```java
// Demo05ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo05ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo05Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            // 同步发送消息
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);

            // 故意每条消息之间，隔离 10 秒
            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
            Thread.sleep(10 * 1000L);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- 同步发送三条消息，每次发送消息之间，都故意 sleep 10 秒。😈 目的是，恰好满足我们配置的 `timeout` 最大等待时长。

我们来执行 `#testASyncSend()` 方法，测试批量发送消息。控制台输出如下：



```bash
// Producer 成功同步发送了 3 条消息，每条间隔 10 秒。
2019-12-15 16:50:15.419  INFO 94085 --- [           main] c.i.s.l.r.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1575967815] 发送成功]
2019-12-15 16:50:25.426  INFO 94085 --- [           main] c.i.s.l.r.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1575967825] 发送成功]
2019-12-15 16:50:35.427  INFO 94085 --- [           main] c.i.s.l.r.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1575967835] 发送成功]

// Demo05Consumer 在最后一条消息发送成功后果的 30 秒，消费到这 3 条消息。
2019-12-15 16:51:05.449  INFO 94085 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo05Consumer        : [onMessage][线程编号:17 消息内容：Demo05Message{id=1575967815}]
2019-12-15 16:51:05.450  INFO 94085 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo05Consumer        : [onMessage][线程编号:17 消息内容：Demo05Message{id=1575967825}]
2019-12-15 16:51:05.450  INFO 94085 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo05Consumer        : [onMessage][线程编号:17 消息内容：Demo05Message{id=1575967835}]
```



- 因为使用 `BatchingRabbitTemplate` 批量发送消息，所以在 Producer 成功发送完第一条消息后，Consumer 并未消费到这条消息。
- 又因为 `BatchingRabbitTemplate` 是按照每次发送后，都重新计时，所以在最后一条消息成功发送后的 30 秒，Consumer 才消费到批量发送的 3 条消息。

# 5. 批量消费消息

> 示例代码对应仓库：[lab-04-rabbitmq-demo-batch-consume](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume)

在[「4. 批量发送消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节，我们已经实现批量发送消息到 RabbitMQ Broker 中。那么，我们来思考一个问题，这批消息在 RabbitMQ Broker 到底是存储**一条**消息，还是**多条**消息？

- 如果胖友使用过 Kafka、RocketMQ 这两个消息队列，那么判断肯定会是**多条**消息。
- 从[「4.6 Demo05Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中，我们可以看到**逐条**消息的消费，也会认为是**多条**消息。

😭 实际上，RabbitMQ Broker 存储的是**一条**消息。又或者说，**RabbitMQ 并没有提供批量接收消息的 API 接口**。

那么，为什么我们在[「4. 批量发送消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)能够实现呢？答案是批量发送消息是 Spring-AMQP 的 [SimpleBatchingStrategy](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/batch/SimpleBatchingStrategy.java) 所封装提供：

- 在 Producer 最终批量发送消息时，SimpleBatchingStrategy 会通过 [`#assembleMessage()`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/batch/SimpleBatchingStrategy.java#L141-L156) 方法，将批量发送的**多条**消息**组装**成一条“批量”消息，然后进行发送。
- 在 Consumer 拉取到消息时，会根据[`#canDebatch(MessageProperties properties)`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/batch/SimpleBatchingStrategy.java#L158-L163) 方法，判断该消息是否为一条“批量”消息？如果是，则调用[`# deBatch(Message message, Consumer fragmentConsumer)`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/batch/SimpleBatchingStrategy.java#L165-L194) 方法，将一条“批量”消息**拆开**，变成**多条**消息。

> 这个操作，是不是略微有点骚气？！艿艿在这里卡了很久！！！莫名其妙的~一直以为，RabbitMQ 提供了批量发送消息的 API 接口啊。
>
> OK ，虽然很悲伤，但是我们还是回到这个小节的主题。

在一些业务场景下，我们希望使用 Consumer 批量消费消息，提高消费速度。在 Spring-AMQP 中，提供了两种批量消费消息的方式。本小节，我们先来看第一种，它需要基于[「4. 批量发送消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)之上实现。

在 SimpleBatchingStrategy 将一条“批量”消息拆开，变成多条消息后，直接**批量**交给 Consumer 进行消费处理。

下面，我们来实现一个 Consumer 批量消费消息的示例。考虑到不污染[「4. 批量发送消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 的示例，我们在 [lab-04-rabbitmq-demo-batch](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch) 项目的基础上，复制出一个 [lab-04-rabbitmq-demo-batch-consume](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume) 项目。😈 酱紫，我们也能少写点代码，哈哈哈~

## 5.1 RabbitConfig

修改 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加自定义的 [SimpleRabbitListenerContainerFactory](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/config/SimpleRabbitListenerContainerFactory.java) Bean ，支持用于创建**支持批量消费**的 [SimpleRabbitListenerContainer](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/listener/SimpleMessageListenerContainer.java) 。代码如下：



```java
// RabbitConfig.java

@Bean(name = "consumerBatchContainerFactory")
public SimpleRabbitListenerContainerFactory consumerBatchContainerFactory(
        SimpleRabbitListenerContainerFactoryConfigurer configurer, ConnectionFactory connectionFactory) {
    // 创建 SimpleRabbitListenerContainerFactory 对象
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    configurer.configure(factory, connectionFactory);
    // <X> 额外添加批量消费的属性
    factory.setBatchListener(true);
    return factory;
}
```



- 在 [RabbitAnnotationDrivenConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitAnnotationDrivenConfiguration.java) 自动化配置类中，它会默认创建一个名字为 `"rabbitListenerContainerFactory"` 的 SimpleRabbitListenerContainerFactory Bean ，可用于消费者的监听器是**单个**消费消费的。
- 我们自定义创建的一个名字为`"consumerBatchContainerFactory"` 的 SimpleRabbitListenerContainerFactory Bean ，可用于消费者的监听器是**批量**消费消费的。重点是 `<X>` 处，配置消费者的监听器是**批量**消费消息的类型，其它的可以暂时不用理解。

## 5.2 Demo05Consumer

修改 [Demo05Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo05Consumer.java) 类，**批量**消费消息。代码如下：



```
// Demo05Consumer.java

@Component
@RabbitListener(queues = Demo05Message.QUEUE,
    containerFactory = "consumerBatchContainerFactory")
public class Demo05Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(List<Demo05Message> messages) {
        logger.info("[onMessage][线程编号:{} 消息数量：{}]", Thread.currentThread().getId(), messages.size());
    }

}
```



- 在类上的 `@@RabbitListener` 注解的 `containerFactory` 属性，设置了我们在[「5.1 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)创建的 SimpleRabbitListenerContainerFactory Bean ，表示它要批量消费消息。
- 在 `#onMessage(...)` 消费方法上，修改方法入参的类型为 List 数组。

## 5.3 简单测试

和 [「4.7 简单测试」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [Demo05ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo05ProducerTest.java) 单元测试类。

我们来执行 `#testASyncSend()` 方法，测试批量消费消息。控制台输出如下：



```
// Producer 成功同步发送了 3 条消息，每条间隔 10 秒。
2019-12-15 22:42:08.755  INFO 60216 --- [           main] c.i.s.l.r.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1575988928] 发送成功]
2019-12-15 22:42:18.757  INFO 60216 --- [           main] c.i.s.l.r.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1575988938] 发送成功]
2019-12-15 22:42:28.758  INFO 60216 --- [           main] c.i.s.l.r.producer.Demo05ProducerTest    : [testSyncSend][发送编号：[1575988948] 发送成功]

// Demo05Consumer 在最后一条消息发送成功后果的 30 秒，一次性批量消费了这 3 条消息。
2019-12-15 22:42:58.775  INFO 60216 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo05Consumer        : [onMessage][线程编号:17 消息数量：3]
```



- 符合预期，Demo05Consumer 批量消费了 3 条消息。

# 6. 批量消费消息（第二弹）

> 示例代码对应仓库：[lab-04-rabbitmq-demo-batch-consume-02](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02)

在[「5. 批量消费消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节，我们已经学习了一种批量消费消息的方式。因为其依赖[「4. 批量发送消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的功能，有点过于苛刻。所以，Spring-AMQP 提供了第二种批量消费消息的方式。

其实现方式是，阻塞等待最多 `receiveTimeout` 秒，拉取 `batchSize` 条消息，进行批量消费。

- 如果在 `receiveTimeout` 秒内已经成功拉取到 `batchSize` 条消息，则直接进行批量消费消息。
- 如果在 `receiveTimeout` 秒还没拉取到 `batchSize` 条消息，不再等待，而是进行批量消费消息。

不过 Spring-AMQP 的阻塞等待时长 `receiveTimeout` 的设计有点“神奇”。

- 它代表的是，每次拉取一条消息，最多阻塞等待 `receiveTimeout` 时长。如果等待不到下一条消息，则进入已获取到的消息的批量消费。😈 也就是说，极端情况下，可能等待 `receiveTimeout * batchSize` 时长，才会进行批量消费。
- 感兴趣的朋友，可以点击 [`SimpleMessageListenerContainer#doReceiveAndExecute(BlockingQueueConsumer consumer)`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/listener/SimpleMessageListenerContainer.java#L922) 方法，简单阅读源码，即可快速理解。

下面，我们来实现一个 Consumer 批量消费消息的示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-batch-consume-02](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02) 项目。

## 6.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/pom.xml) 文件。

## 6.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/main/resources/application.yaml) 文件。

## 6.3 Demo06Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/) 包下，创建 [Demo06Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo06Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 6.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加自定义的 [SimpleRabbitListenerContainerFactory](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/config/SimpleRabbitListenerContainerFactory.java) Bean ，支持用于创建**支持批量消费**的 [SimpleRabbitListenerContainer](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/listener/SimpleMessageListenerContainer.java) 。代码如下：



```
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo06Queue() {
            return new Queue(Demo06Message.QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo06Exchange() {
            return new DirectExchange(Demo06Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo06Message.EXCHANGE
        // Routing key：Demo06Message.ROUTING_KEY
        // Queue：Demo06Message.QUEUE
        @Bean
        public Binding demo06Binding() {
            return BindingBuilder.bind(demo06Queue()).to(demo06Exchange()).with(Demo06Message.ROUTING_KEY);
        }

    }

    @Bean(name = "consumerBatchContainerFactory")
    public SimpleRabbitListenerContainerFactory consumerBatchContainerFactory(
            SimpleRabbitListenerContainerFactoryConfigurer configurer, ConnectionFactory connectionFactory) {
        // 创建 SimpleRabbitListenerContainerFactory 对象
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        configurer.configure(factory, connectionFactory);
        // 额外添加批量消费的属性
        factory.setBatchListener(true);
        // <X>
        factory.setBatchSize(10);
        factory.setReceiveTimeout(30 * 1000L);
        factory.setConsumerBatchEnabled(true);
        return factory;
    }

}
```



- DirectExchangeDemoConfiguration 配置类，用于定义 Queue、Exchange、Binding 的配置。
- 相比[「5.1 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，额外增加了 `batchSize = 10`、`receiveTimeout = 30 * 1000L`、`consumerBatchEnabled = 30 * 1000L` 属性。😈 严格意义上来说，**本小节才是真正意义上的批量消费消息**。

## 6.5 Demo06Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo06Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo06Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。

和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、RoutingKey 名字不同。

## 6.6 Demo06Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo05Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo05Consumer.java) 类，**批量**消费消息。代码如下：



```
// Demo06Consumer.java

@Component
@RabbitListener(queues = Demo06Message.QUEUE,
    containerFactory = "consumerBatchContainerFactory")
public class Demo06Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(List<Demo06Message> messages) {
        logger.info("[onMessage][线程编号:{} 消息数量：{}]", Thread.currentThread().getId(), messages.size());
    }

}
```



- 和[「5.2 Demo05Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只差在消费不同的队列。

## 6.7 简单测试

创建 [Demo06ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo06ProducerTest.java) 测试类，编写单元测试方法，测试 Consumer 批量消费消息的效果。代码如下：



```
// Demo06ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo06ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo06Producer producer;

    @Test
    public void testSyncSend01() throws InterruptedException {
        // 发送 3 条消息
        this.testSyncSend(3);
    }

    @Test
    public void testSyncSen02() throws InterruptedException {
        // 发送 10 条消息
        this.testSyncSend(10);
    }

    private void testSyncSend(int n) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            // 同步发送消息
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);
            logger.info("[testSyncSendMore][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- `#testSyncSend01()` 方法，发送 3 条消息，测试 Demo06Consumer 获取数量为 `batchSize = 10` 消息，**超时**情况下的批量消费。
- `#testSyncSend02()` 方法，发送 10 条消息，测试 Demo06Consumer 获取数量为 `batchSize = 10` 消息，**未超时**情况下的批量消费。

我们来执行 `#testSyncSend01()` 方法，**超时**情况下的批量消费。控制台输出如下：



```
// Producer 成功同步发送了 3 条消息
2019-12-15 00:01:18.097  INFO 78389 --- [           main] c.i.s.l.r.producer.Demo06ProducerTest    : [testSyncSendMore][发送编号：[1575993678] 发送成功]
2019-12-15 00:01:18.099  INFO 78389 --- [           main] c.i.s.l.r.producer.Demo06ProducerTest    : [testSyncSendMore][发送编号：[1575993678] 发送成功]
2019-12-15 00:01:18.099  INFO 78389 --- [           main] c.i.s.l.r.producer.Demo06ProducerTest    : [testSyncSendMore][发送编号：[1575993678] 发送成功]

// Consumer 30 秒超时等待后，批量消费到 3 条消息
2019-12-15 00:01:48.116  INFO 78389 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo06Consumer        : [onMessage][线程编号:17 消息数量：3]
```



- 符合预期。具体胖友看下日志上的注释说明。

我们来执行 `#testSyncSend02()` 方法，**未超时**情况下的批量消费。控制台输出如下：



```
// Producer 成功同步发送了 10 条消息
2019-12-15 00:03:50.406  INFO 78997 --- [           main] c.i.s.l.r.producer.Demo06ProducerTest    : [testSyncSendMore][发送编号：[1575993830] 发送成功]
// ... 省略 8 条消息
2019-12-15 00:03:50.410  INFO 78997 --- [           main] c.i.s.l.r.producer.Demo06ProducerTest    : [testSyncSendMore][发送编号：[1575993830] 发送成功]

// Consumer 拉取到 10 条消息后，立即批量消费到 10 条消息
2019-12-15 00:03:50.429  INFO 78997 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo06Consumer        : [onMessage][线程编号:17 消息数量：10
```



- 符合预期。具体胖友看下日志上的注释说明。

😈 至此，我们已经完成了两种 Spring-AMQP 的批量消费消费的方法。更多的内容，可以看看 [《Spring-AMQP 官方文档 —— @RabbitListener with Batching》](https://docs.spring.io/spring-amqp/docs/current/reference/html/#receiving-batch) 文档。

# 7. 消费重试

> 示例代码对应仓库：[lab-04-rabbitmq-consume-retry](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry)

在开始本小节之前，胖友首先要对 RabbitMQ 的[死信队列](https://www.rabbitmq.com/dlx.html)的机制，有一定的了解。不了解的胖友，可以看看[《RabbitMQ 之死信队列》](http://www.iocoder.cn/RabbitMQ/dead-letter-queue/?self)文章。

在消息**消费失败**的时候，Spring-AMQP 会通过**消费重试**机制，重新投递该消息给 Consumer ，让 Consumer 有机会重新消费消息，实现消费成功。

当然，Spring-AMQP 并不会无限重新投递消息给 Consumer 重新消费，而是在默认情况下，达到 N 次重试次数时，Consumer 还是消费失败时，该消息就会进入到**死信队列**。后续，我们可以通过对死信队列中的消息进行重发，来使得消费者实例再次进行消费。

- 在[《芋道 Spring Boot 消息队列 RocketMQ 入门》](http://www.iocoder.cn/Spring-Boot/RocketMQ/?self)的[「6. 消费重试」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节中，我们可以看到，消费重试和死信队列，是 RocketMQ 自带的功能。
- 而在 RabbitMQ 中，消费重试是由 Spring-AMQP 所封装提供的，死信队列是 RabbitMQ 自带的功能。

那么消费失败到达最大次数的消息，是怎么进入到死信队列的呢？Spring-AMQP 在消息到达最大消费次数的时候，会将该消息进行否定(`basic.nack`)，并且 `requeue=false` ，这样后续就可以利用 RabbitMQ 的[死信队列](https://www.rabbitmq.com/dlx.html)的机制，将该消息转发到死信队列。

另外，每条消息的失败重试，是可以配置一定的**间隔时间**。具体，我们在示例的代码中，来进行具体的解释。

下面，我们来实现一个 Consumer 消费重试的示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-consume-retry](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry) 项目。

## 7.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/pom.xml) 文件。

## 7.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
    listener:
      simple:
        # 对应 RabbitProperties.ListenerRetry 类
        retry:
          enabled: true # 开启消费重试机制
          max-attempts: 3 # 最大重试次数。默认为 3 。
          initial-interval: 1000 # 重试间隔，单位为毫秒。默认为 1000 。
```



- 相比[「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，我们通过**新增** `spring.rabbitmq.simple.retry.enable=true` 配置项，来开启 Spring-AMQP 的消费重试的功能。同时，通过**新增** `max-attempts` 和 `initial-interval` 配置项，设置重试次数和间隔。

  > `max-attempts` 配置项要注意，是一条消息一共尝试消费总共 `max-attempts` 次，包括首次的正常消费。

- 另外，胖友可以通过添加 `spring.rabbitmq.listener.simple.retry.multiplier` 配置项来实现**递乘**的时间间隔，添加 `spring.rabbitmq.listener.simple.retry.max-interval` 配置项来实现**最大**的时间间隔。

在 Spring-AMQP 的消费重试机制中，在消费失败到达最大次数后，会**自动**抛出 [AmqpRejectAndDontRequeueException](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/AmqpRejectAndDontRequeueException.java) 异常，从而结束该消息的消费重试。这意味着什么呢？如果我们在消费消息的逻辑中，**主动**抛出 AmqpRejectAndDontRequeueException 异常，也能结束该消息的消费重试。😈 结束的方式，Spring-AMQP 是通过我们在上文中提到的 `basic.nack` + `requeue=false` ，从而实现转发该消息到死信队列中。

另外，默认情况下，`spring.rabbitmq.simple.retry.enable=false` ，关闭 Spring-AMQP 的消费重试功能。但是实际上，消费发生异常的消息，还是会一直**重新消费**。这是为什么呢？Spring-AMQP 会将该消息通过 `basic.nack` + `requeue=true` ，重新投递回**原队列的尾巴**。如此，我们便会不断拉取到该消息，不断“重试”消费该消息。当然在这种情况下，我们一样可以**主动**抛出 AmqpRejectAndDontRequeueException 异常，也能结束该消息的消费重试。😈 结束的方式，Spring-AMQP 也是通过我们在上文中提到的 `basic.nack` + `requeue=false` ，从而实现转发该消息到死信队列中。

这里，我们再来简单说说 Spring-AMQP 是怎么提供**消费重试**的功能的。

- Spring-AMQP 基于 [spring-retry](https://github.com/spring-projects/spring-retry) 项目提供的 [RetryTemplate](https://github.com/spring-projects/spring-retry/blob/master/src/main/java/org/springframework/retry/support/RetryTemplate.java) ，实现重试功能。Spring-AMQP 在获取到消息时，会交给 RetryTemplate 来调用消费者 Consumer 的监听器 Listener(就是我们实现的)，实现该消息的**多次**消费重试。

- 在该消息的**每次消费失败**后，RetryTemplate 会通过 [BackOffPolicy](https://github.com/spring-projects/spring-retry/blob/master/src/main/java/org/springframework/retry/backoff/BackOffPolicy.java) 来进行计算，该消息的**下一次重新消费的时间**，通过 `Thread#sleep(...)` 方法，实现重新消费的时间间隔。到达时间间隔后，RetryTemplate 又会调用消费者 Consumer 的监听器 Listener 来消费该消息。

- 当该消息的重试消费到达**上限**后，RetryTemplate 会调用 [MethodInvocationRecoverer](https://github.com/spring-projects/spring-retry/blob/master/src/main/java/org/springframework/retry/interceptor/MethodInvocationRecoverer.java) 回调来实现恢复。而 Spring-AMQP 自定义实现了 [RejectAndDontRequeueRecoverer](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/retry/RejectAndDontRequeueRecoverer.java) 来**自动**抛出 AmqpRejectAndDontRequeueException 异常，从而结束该消息的消费重试。😈 结束的方式，Spring-AMQP 是通过我们在上文中提到的 `basic.nack` + `requeue=false` ，从而实现转发该消息到死信队列中。

- 有一点需要注意，Spring-AMQP 提供的消费重试的**计数**是**客户端**级别的，重启 JVM 应用后，计数是会丢失的。所以，如果想要计数进行持久化，需要自己重新实现下。

  > 😈 RocketMQ 提供的消费重试的计数，目前是**服务端**级别，已经进行持久化。

> 😜 瞎哔哔了好长一段，涉及到的信息量可能比较大，如果艿艿有解释不清晰或者错误的地方，又或者哪里不了解，可以给艿艿留言，乐意之至为胖友解答。
>
> 同时，也可以调试下整个过程涉及到的源码，更加具象下。「源码之前，了无秘密」。

## 7.3 Demo07Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/) 包下，创建 [Demo07Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo07Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo07Message.java


    public static final String QUEUE = "QUEUE_DEMO_07"; // 正常队列
    public static final String DEAD_QUEUE = "DEAD_QUEUE_DEMO_07"; // 死信队列

    public static final String EXCHANGE = "EXCHANGE_DEMO_07";

    public static final String ROUTING_KEY = "ROUTING_KEY_07"; // 正常路由键
    public static final String DEAD_ROUTING_KEY = "DEAD_ROUTING_KEY_07"; // 死信路由键


    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 相比[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，额外增加了死信队列会用到的 Queue 和 RoutingKey ，而 Exchange 我们先复用 `EXCHANGE = "EXCHANGE_DEMO_07"` 。

## 7.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，额外添加**死信队列**的配置。代码如下：



```
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo07Queue() {
            return QueueBuilder.durable(Demo07Message.QUEUE) // durable: 是否持久化
                    .exclusive() // exclusive: 是否排它
                    .autoDelete() // autoDelete: 是否自动删除
                    .deadLetterExchange(Demo07Message.EXCHANGE)
                    .deadLetterRoutingKey(Demo07Message.DEAD_ROUTING_KEY)
                    .build();
        }

        // 创建 Dead Queue
        @Bean
        public Queue demo07DeadQueue() {
            return new Queue(Demo07Message.DEAD_QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo07Exchange() {
            return new DirectExchange(Demo07Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo07Message.EXCHANGE
        // Routing key：Demo07Message.ROUTING_KEY
        // Queue：Demo07Message.QUEUE
        @Bean
        public Binding demo07Binding() {
            return BindingBuilder.bind(demo07Queue()).to(demo07Exchange()).with(Demo07Message.ROUTING_KEY);
        }

        // 创建 Dead Binding
        // Exchange：Demo07Message.EXCHANGE
        // Routing key：Demo07Message.DEAD_ROUTING_KEY
        // Queue：Demo07Message.DEAD_QUEUE
        @Bean
        public Binding demo07DeadBinding() {
            return BindingBuilder.bind(demo07DeadQueue()).to(demo07Exchange()).with(Demo07Message.DEAD_ROUTING_KEY);
        }

    }

}
```



- 相比[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，主要有**两个**差异点。
- 第一点，创建的正常 Queue 额外设置了，当消息成为死信时，RabbitMQ 自动转发到 Exchange 为 `Demo07Message.EXCHANGE`，RoutingKey 为 `Demo07Message.DEAD_ROUTING_KEY` 的死信队列中。
- 第二点，通过 `#demo07DeadQueue()` 方法来创建死信队列的 Queue ，通过 `#demo07DeadBinding()` 方法来创建死信队列的 Binding 。😈 因为我们重用了 Exchange 为 `Demo07Message.EXCHANGE` ，所以无需创建。当然，胖友也可以根据自己的需要，创建死信队列的 Exchange 。

## 7.5 Demo07Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo07Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-batch-consume-02/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo07Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。

和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、RoutingKey 名字不同。

## 7.6 Demo07Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/) 包下，创建 [Demo07Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo07Consumer.java) 类，消费消息。代码如下：



```
// Demo07Consumer.java

@Component
@RabbitListener(queues = Demo07Message.QUEUE)
public class Demo07Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo07Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // <X> 注意，此处抛出一个 RuntimeException 异常，模拟消费失败
        throw new RuntimeException("我就是故意抛出一个异常");
    }

}
```



- 在 `<X>` 处，我们在消费消息时候，抛出一个 RuntimeException 异常，模拟消费失败。

## 7.7 Demo07DeadConsumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/) 包下，创建 [Demo07DeadConsumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo07DeadConsumer.java) 类，消费**死信队列**的消息。代码如下：



```
// Demo07DeadConsumer.java

@Component
@RabbitListener(queues = Demo07Message.DEAD_QUEUE)
public class Demo07DeadConsumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo07Message message) {
        logger.info("[onMessage][【死信队列】线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 在类上，添加了 [`@RabbitListener`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/annotation/RabbitListener.java) 注解，声明了消费的队列是 `"DEAD_QUEUE_DEMO_07"` 这个**死信队列**。

这里的消费逻辑，仅仅是示例，实现逻辑胖友根据自己的需要，自己来具体实现，嘿嘿。

## 7.8 简单测试

和 [「3.1.8 简单测试」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 大体一致，见 [Demo05ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-consume-retry/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo07ProducerTest.java) 单元测试类。

我们来执行 `#testSyncSend()` 方法，测试 Consumer 消费重试的效果。控制台输出如下：



```
# Producer 成功同步发送了 1 条消息
2019-12-15 14:21:40.424  INFO 66569 --- [           main] c.i.s.l.r.producer.Demo07ProducerTest    : [testSyncSend][发送编号：[1576045300] 发送成功]

# Demo07Consumer 第 1 次消费
2019-12-15 14:21:40.442  INFO 66569 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo07Consumer        : [onMessage][线程编号:17 消息内容：Demo07Message{id=1576045300}]
# 一秒后，Consumer 第 2 次消费
2019-12-15 14:21:41.446  INFO 66569 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo07Consumer        : [onMessage][线程编号:17 消息内容：Demo07Message{id=1576045300}]
# 一秒后，Consumer 第 3 次消费
2019-12-15 14:21:42.450  INFO 66569 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo07Consumer        : [onMessage][线程编号:17 消息内容：Demo07Message{id=1576045300}]

# RejectAndDontRequeueRecoverer 打印该消息消费重试到达上限，同时打印异常堆栈
2019-12-15 14:21:42.457  WARN 66569 --- [ntContainer#0-1] o.s.a.r.r.RejectAndDontRequeueRecoverer  : Retries exhausted for message (Body:'[B@514e3b1c(byte[187])' MessageProperties [headers={}, contentType=application/x-java-serialized-object, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=EXCHANGE_DEMO_07, receivedRoutingKey=ROUTING_KEY_07, deliveryTag=1, consumerTag=amq.ctag-UpkeXbl-7TYRNt_LuYDZJQ, consumerQueue=QUEUE_DEMO_07])
// ... 省略一大堆异常堆栈

# Demo07DeadConsumer 消费死信队列的该条消息
2019-12-15 14:21:42.463  INFO 66569 --- [ntContainer#1-1] c.i.s.l.r.consumer.Demo07DeadConsumer    : [onMessage][【死信队列】线程编号:19 消息内容：Demo07Message{id=1576045300}]
```



- Demo07Consumer 重试消费消息 3 次，每次间隔 1 秒，全部都失败，最终该消息转发到死信队列中。
- Demo07DeadConsumer 消费死信队列中的该消息。

## 7.9 发送重试

在 Spring-AMQP 也提供了消息发送失败时的重试机制，也是基于 [spring-retry](https://github.com/spring-projects/spring-retry) 项目提供的 [RetryTemplate](https://github.com/spring-projects/spring-retry/blob/master/src/main/java/org/springframework/retry/support/RetryTemplate.java) 来实现。在 `application.yaml` 配置如下即可：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
    template:
      # 对应 RabbitProperties.Retry 类
      retry:
        enabled: true # 开启发送机制
        max-attempts: 3 # 最大重试次数。默认为 3 。
        initial-interval: 1000 # 重试间隔，单位为毫秒。默认为 1000 。
```



- `spring.rabbitmq.template.enable=true` 配置项，来开启 Spring-AMQP 的发送重试的功能。同时，通过**新增** `max-attempts` 和 `initial-interval` 配置项，设置重试次数和间隔。

  > `max-attempts` 配置项要注意，是一条消息一共尝试消费总共 `max-attempts` 次，包括首次的正常消费。

- 另外，胖友可以通过添加 `spring.rabbitmq.template.retry.multiplier` 配置项来实现**递乘**的时间间隔，添加 `spring.rabbitmq.template.retry.max-interval` 配置项来实现**最大**的时间间隔。

这里艿艿就暂时不拓展开来讲，胖友可以自己尝试下哈。

# 8. 定时消息

> 示例代码对应仓库：[lab-04-rabbitmq-demo-delay](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay)

在[「7. 消费重试」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节中，我们看到 Spring-AMQP 基于 RabbitMQ 提供的**死信队列**，通过 `basic.nack` + `requeue=false` 的方式，将重试消费到达上限次数的消息，投递到死信队列中。

本小节，我们还是基于 RabbitMQ 的**死信队列**，实现**定时消息**的功能。RabbitMQ 提供了过期时间 [TTL](https://www.rabbitmq.com/ttl.html) 机制，可以设置消息在队列中的存活时长。在消息到达过期时间时，会从当前队列中删除，并被 RabbitMQ 自动转发到对应的死信队列中。

那么，如果我们创建消费者 Consumer ，来消费该死信队列，是不是就实现了**延迟队列**的效果。😈 如此，我们便实现了定时消息的功能。

下面，我们来实现一个定时消息的示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-delay](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay) 项目。

## 8.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/pom.xml) 文件。

## 8.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/resources/application.yaml) 文件。

## 8.3 Demo08Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo08Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo08Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo08Message.java

public class Demo08Message implements Serializable {

    public static final String QUEUE = "QUEUE_DEMO_08"; // 正常队列
    public static final String DELAY_QUEUE = "DELAY_QUEUE_DEMO_08"; // 延迟队列（死信队列）

    public static final String EXCHANGE = "EXCHANGE_DEMO_08";

    public static final String ROUTING_KEY = "ROUTING_KEY_08"; // 正常路由键
    public static final String DELAY_ROUTING_KEY = "DELAY_ROUTING_KEY_08"; // 延迟路由键（死信路由键）

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 相比[「8.3 Demo07Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，基本一致，只是换了下命名，将 `DEAD` 改成 `DELAY` 来方便胖友理解。

## 8.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，额外添加**延迟队列**（死信队列）的配置。代码如下：



```
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo08Queue() {
            return QueueBuilder.durable(Demo08Message.QUEUE) // durable: 是否持久化
                    .exclusive() // exclusive: 是否排它
                    .autoDelete() // autoDelete: 是否自动删除
                    .ttl(10 * 1000) // 设置队列里的默认过期时间为 10 秒
                    .deadLetterExchange(Demo08Message.EXCHANGE)
                    .deadLetterRoutingKey(Demo08Message.DELAY_ROUTING_KEY)
                    .build();
        }

        // 创建 Delay Queue
        @Bean
        public Queue demo08DelayQueue() {
            return new Queue(Demo08Message.DELAY_QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo08Exchange() {
            return new DirectExchange(Demo08Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo08Message.EXCHANGE
        // Routing key：Demo08Message.ROUTING_KEY
        // Queue：Demo08Message.QUEUE
        @Bean
        public Binding demo08Binding() {
            return BindingBuilder.bind(demo08Queue()).to(demo08Exchange()).with(Demo08Message.ROUTING_KEY);
        }

        // 创建 Delay Binding
        // Exchange：Demo08Message.EXCHANGE
        // Routing key：Demo08Message.DELAY_ROUTING_KEY
        // Queue：Demo08Message.DELAY_QUEUE
        @Bean
        public Binding demo08DelayBinding() {
            return BindingBuilder.bind(demo08DelayQueue()).to(demo08Exchange()).with(Demo08Message.DELAY_ROUTING_KEY);
        }

    }

}
```



- 相比[「7.4 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，主要有**一个**差异点。在 `#demo08Queue()` 方法来创建的 Queue ，我们设置了该队列的消息的默认过期时间为 10 秒。

## 8.5 Demo08Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo08Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo08Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// Demo08Producer.java

@Component
public class Demo08Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id, Integer delay) {
        // 创建 Demo07Message 消息
        Demo08Message message = new Demo08Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo08Message.EXCHANGE, Demo08Message.ROUTING_KEY, message, new MessagePostProcessor() {

            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                // 设置消息的 TTL 过期时间
                if (delay != null && delay > 0) {
                    message.getMessageProperties().setExpiration(String.valueOf(delay)); // Spring-AMQP API 设计有问题，所以传入了 String = =
                }
                return message;
            }

        });
    }

}
```



- 调用 `#syncSend(Integer id, Integer delay)` 方法来发送消息时，如果传递了方法参数 `delay` ，则我们会设置消息的 TTL 过期时间。

## 8.6 Demo08Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo08Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo08Consumer.java) 类，消费**延迟队列**（死信队列）的消息。代码如下：



```
@Component
@RabbitListener(queues = Demo08Message.DELAY_QUEUE)
public class Demo08Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo08Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 在类上，添加了 [`@RabbitListener`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/annotation/RabbitListener.java) 注解，声明了消费的队列是 `"DELAY_QUEUE_DEMO_08"` 这个**延迟队列（死信队列）**
- 在消费逻辑中，我们正常消费该消息即可，实现自己需要的业务逻辑。

## 8.7 简单测试

创建 [Demo08ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-delay/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo08ProducerTest.java) 测试类，编写单元测试方法，测试**定时消息**的效果。代码如下：



```
// Demo08ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo08ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo08Producer producer;

    @Test
    public void testSyncSend01() throws InterruptedException {
        // 不设置消息的过期时间，使用队列默认的消息过期时间
        this.testSyncSendDelay(null);
    }

    @Test
    public void testSyncSend02() throws InterruptedException {
        // 设置发送消息的过期时间为 5000 毫秒
        this.testSyncSendDelay(5000);
    }

    private void testSyncSendDelay(Integer delay) throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id, delay);
        logger.info("[testSyncSendDelay][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- `#testSyncSend01()` 方法，不设置消息的过期时间，使用队列**默认的消息过期**时间。
- `#testSyncSend02()` 方法，发送消息的**过期时间为 5000 毫秒**。

我们先来执行 `#testSyncSend01()` 方法，不设置消息的过期时间，使用队列**默认的消息过期**时间。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 15:44:34.571  INFO 85481 --- [           main] c.i.s.l.r.producer.Demo08ProducerTest    : [testSyncSendDelay][发送编号：[1576050274] 发送成功]

# Consumer 10 秒后，消费到该消息
2019-12-15 15:44:44.588  INFO 85481 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo08Consumer        : [onMessage][线程编号:17 消息内容：Demo08Message{id=1576050274}]
```



- 符合预期。

我们再来执行 `#testSyncSend02()` 方法，发送消息的**过期时间为 5000 毫秒**。控制台输出如下：



```
# Producer 同步发送消息成功。
2019-12-15 15:45:41.076  INFO 85735 --- [           main] c.i.s.l.r.producer.Demo08ProducerTest    : [testSyncSendDelay][发送编号：[1576050341] 发送成功]

# Consumer 5 秒后，消费到该消息
2019-12-15 15:45:46.090  INFO 85735 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo08Consumer        : [onMessage][线程编号:17 消息内容：Demo08Message{id=1576050341}]
```



- 符合预期。

## 8.8 RabbitMQ Delayed Message Plugin

RabbitMQ 目前提供了 [RabbitMQ Delayed Message Plugin](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange) 插件，提供更加**通用**的定时消息的功能。

使用起来比较简单，艿艿这里先暂时不提供示例。感兴趣的胖友，可以看看 [《Spring Boot RabbitMQ 延迟消息实现完整版》](http://www.iocoder.cn/Fight/Spring-Boot-RabbitMQ-deferred-message-implementation-full-version/?self) 文章。

这两种方案，生产环境下，还是**推荐直接使用 RabbitMQ Delayed Message Plugin 插件的方式**。毕竟，这是 RabbitMQ 官方认可的插件，使用起来肯定是没错的。

# 9. 消息模式

> 示例代码对应仓库：[lab-04-rabbitmq-demo-message-model](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model)

在消息队列中，有两种经典的消息模式：「点对点」和「发布订阅」。具体的概念，艿艿就先不解释，胖友可以看看[《消息队列两种模式：点对点与发布订阅》](http://www.iocoder.cn/Fight/There-are-two-modes-of-message-queuing-point-to-point-and-publish-subscription/?self)文章。

如果胖友有使用过 RocketMQ 或者 Kafka 消息队列，可能比较习惯的叫法是：

> **集群消费（Clustering）**：对应「点对点」 集群消费模式下，相同 Consumer Group 的每个 Consumer 实例平均分摊消息。
>
> **广播消费（Broadcasting）**：对应「发布订阅」 广播消费模式下，相同 Consumer Group 的每个 Consumer 实例都接收全量的消息。

😈 考虑到艿艿自己的习惯，下文我们统一采用集群消费和广播消费叫法。

下面，我们分别在[「9.1 集群消费」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)和[「9.2 广播消费」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的示例代码。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-message-model](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model) 项目。

## 9.1 集群消费

在 RabbitMQ 中，如果多个 Consumer 订阅相同的 Queue ，那么每一条消息有且仅会被一个 Consumer 所消费。这个特性，就为我们实现集群消费提供了基础。

在本示例中，我们会把一个 Queue 作为一个 Consumer Group ，同时创建消费该 Queue 的 Consumer 。这样，在我们启动多个 JVM 进程时，就会有多个 Consumer 消费该 Queue ，从而实现集群消费的效果。

下面，让我们开始集群消费的示例。

### 9.1.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/pom.xml) 文件。

### 9.1.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/resources/application.yaml) 文件。

### 9.1.3 ClusteringMessage

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [ClusteringMessage](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/ClusteringMessage.java) 消息类，提供给当前示例使用。代码如下：



```
// ClusteringMessage.java

public class ClusteringMessage implements Serializable {

    public static final String QUEUE = "QUEUE_CLUSTERING";

    public static final String EXCHANGE = "EXCHANGE_CLUSTERING";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 在这里，我们并没有定义 RoutingKey 的枚举，答案我们在[「9.1.6 ClusteringConsumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)揭晓。

### 9.1.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加集群消费需要的 Exchange 的配置。代码如下：



```
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * 集群消费的示例的配置
     */
    public static class ClusteringConfiguration {

        // 创建 Topic Exchange
        @Bean
        public TopicExchange clusteringExchange() {
            return new TopicExchange(ClusteringMessage.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

    }

}
```



- 在这里，我们创建了 Exchange 类型是 **Topic** 。

为什么不选择 Exchange 类型是 **Direct** 呢？考虑到集群消费的模式，会存在多 Consumer Group 消费的情况，显然我们要支持一条消息投递到多个 Queue 中，所以 Direct Exchange 基本就被排除了。

为什么不选择 Exchange 类型是 **Fanout** 或者 **Headers** 呢？实际是可以的，不过询问了朋友(didi) [Spring Cloud Stream RabbitMQ](https://github.com/spring-cloud/spring-cloud-stream-binder-rabbit) 是怎么实现的。得知答案是[默认](https://raw.githubusercontent.com/spring-cloud/spring-cloud-stream-binder-rabbit/master/docs/src/main/asciidoc/images/rabbit-binder.png)是使用 Topic Exchange 的，所以艿艿示例这里也就使用 Topic Exchange 类型了。

### 9.1.5 ClusteringProducer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [ClusteringProducer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/ClusteringProducer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// ClusteringProducer.java

@Component
public class ClusteringProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 ClusteringMessage  消息
        ClusteringMessage message = new ClusteringMessage();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(ClusteringMessage.EXCHANGE, null, message);
    }

}
```



- 和[「3.2.3 Demo02Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)是基本一致的，除了调用 RabbitTemplate 发送消息时，我们传递的 `routingKey` 参数为 `null` 。为什么呢？答案我们也在[「9.1.6 ClusteringConsumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)揭晓。

### 9.1.6 ClusteringConsumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [ClusteringConsumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/ClusteringConsumer.java) 类，**集群**消费消息。代码如下：



```
// ClusteringConsumer.java

@Component
@RabbitListener(
        bindings = @QueueBinding(
                value = @Queue(
                        name = ClusteringMessage.QUEUE + "-" + "GROUP-01"
                ),
                exchange = @Exchange(
                        name = ClusteringMessage.EXCHANGE,
                        type = ExchangeTypes.TOPIC,
                        declare = "false"
                ),
                key = "#"
        )
)
public class ClusteringConsumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(ClusteringMessage message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 相比其它 Consumer 示例来说，这里添加的 `@RabbitListener` 注解复杂很多。

- 在 `bindings` 属性，我们添加了 [`@QueueBinding`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/annotation/QueueBinding.java) 注解，来定义了一个 Binding 。通过 `key` 属性，设置使用的 RoutingKey 为 `#` ，**匹配所有**。这就是为什么我们在[「9.1.3 ClusteringMessage」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)未定义 RoutingKey ，以及在[「9.1.5 ClusteringProducer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中使用 `routingKey = null` 的原因。

- 在

   

  ```
  exchange
  ```

   

  属性，我们添加了

   

  `@Exchange`

   

  注解，设置了对应

   

  ```
  EXCHANGE_CLUSTERING
  ```

   

  这个 Exchange 。

  - `type` 属性，设置是 TopicExchange 。
  - `declare` 属性，因为[「9.1.4 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中，已经配置创建这个 Exchange 了。

- 在 `value` 属性，我们添加了 [`@Queue`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/annotation/Queue.java) 注解，设置消费 `QUEUE_CLUSTERING-GROUP-01` 这个 Queue 的消息。

注意，通过添加 `@Exchange`、`@Queue`、`@QueueBinding` 注解，如果未声明 `declare="false"` 时，会**自动创建对应**的 Exchange、Queue、Binding 。

### 9.1.7 简单测试

创建 [ClusteringProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/ClusteringProducerTest.java) 测试类，用于测试集群消费。代码如下：



```
// ClusteringProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ClusteringProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private ClusteringProducer producer;

    @Test
    public void mock() throws InterruptedException {
        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testSyncSend() throws InterruptedException {
        // 发送 3 条消息
        for (int i = 0; i < 3; i++) {
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);
            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



首先，执行 `#mock()` 测试方法，先启动一个消费 `"QUEUE_CLUSTERING-GROUP-01"` 这个 Queue 的 Consumer 节点。

然后，执行 `#testSyncSend()` 测试方法，再启动一个消费 `"QUEUE_CLUSTERING-GROUP-01"` 这个 Queue 的 Consumer 节点。同时，该测试方法，调用 `ClusteringProducer#syncSend(id)` 方法，同步发送了 3 条消息。控制台输出如下：



```
// #### testSyncSend 方法对应的控制台 ####

# Producer 同步发送消息成功
2019-12-15 22:13:44.372  INFO 43363 --- [           main] c.i.s.l.r.p.ClusteringProducerTest       : [testSyncSend][发送编号：[1576073624] 发送成功]
2019-12-15 22:13:44.373  INFO 43363 --- [           main] c.i.s.l.r.p.ClusteringProducerTest       : [testSyncSend][发送编号：[1576073624] 发送成功]
2019-12-15 22:13:44.374  INFO 43363 --- [           main] c.i.s.l.r.p.ClusteringProducerTest       : [testSyncSend][发送编号：[1576073624] 发送成功]

# ClusteringConsumer 消费了 1 条消息
2019-12-15 22:13:44.393  INFO 43363 --- [ntContainer#1-1] c.i.s.l.r.consumer.ClusteringConsumer    : [onMessage][线程编号:19 消息内容：ClusteringtMessage{id=1576073624}]

// ### mock 方法对应的控制台 ####

# ClusteringConsumer 消费了 2 条消息
2019-12-15 22:13:44.396  INFO 43308 --- [ntContainer#1-1] c.i.s.l.r.consumer.ClusteringConsumer    : [onMessage][线程编号:19 消息内容：ClusteringtMessage{id=1576073624}]
2019-12-15 22:13:44.398  INFO 43308 --- [ntContainer#1-1] c.i.s.l.r.consumer.ClusteringConsumer    : [onMessage][线程编号:19 消息内容：ClusteringtMessage{id=1576073624}]
```



- 3 条消息，都仅被 **两个** Consumer 节点的**一个**进行消费。符合集群消费的预期~

因为考虑让集群消费的示例做的比较简单，所以并未提供一条消息投递到多个 Queue 中，从而实现多集群下的集群消费的效果。不过比较简单，胖友可以自行在创建一个类似[「9.1.6 ClusteringConsumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的消费者类，设置消费另外一个 Queue 即可。例如说：



```
@Component
@RabbitListener(
        bindings = @QueueBinding(
                value = @Queue(
                        name = ClusteringMessage.QUEUE + "-" + "GROUP-02" // 这里从 "GROUP-01" 改成了 "GROUP-02" 。
                ),
                exchange = @Exchange(
                        name = ClusteringMessage.EXCHANGE,
                        type = ExchangeTypes.TOPIC,
                        declare = "false"
                ),
                key = "#"
        )
)
```



## 9.2 广播消费

在[「9.1 集群消费」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中，我们通过“在 RabbitMQ 中，如果多个 Consumer 订阅相同的 Queue ，那么每一条消息有且仅会被一个 Consumer 所消费”特性，来实现了集群消费。但是，在实现广播消费时，这个特性恰恰成为了一种阻碍。

不过机智的我们，我们可以通过给每个 Consumer 创建一个其**独有** Queue ，从而保证都能接收到全量的消息。同时，RabbitMQ 支持队列的自动删除，所以我们可以在 Consumer 关闭的时候，通过该功能删除其**独有**的 Queue 。

下面，让我们开始集群消费的示例。考虑到简便，我们直接继续在 [lab-04-rabbitmq-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo) 项目改造。

### 9.2.1 BroadcastMessage

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [BroadcastMessage](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/BroadcastMessage.java) 消息类，提供给当前示例使用。代码如下：



```
// BroadcastMessage.java

public class BroadcastMessage implements Serializable {

    public static final String QUEUE = "QUEUE_BROADCASTING";

    public static final String EXCHANGE = "EXCHANGE_BROADCASTING";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 和[「9.1.3 ClusteringMessage」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Queue 和 Exchange 的名字不同。

### 9.2.2 RabbitConfig

修改 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加自定义的 [SimpleRabbitListenerContainerFactory](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/config/SimpleRabbitListenerContainerFactory.java) Bean ，添加广播消费需要的 Exchange 的配置。代码如下：



```
// RabbitConfig.java

/**
 * 广播消费的示例的配置
 */
public static class BroadcastingConfiguration {

    // 创建 Topic Exchange
    @Bean
    public TopicExchange broadcastingExchange() {
        return new TopicExchange(BroadcastMessage.EXCHANGE,
                true,  // durable: 是否持久化
                false);  // exclusive: 是否排它
    }

}
```



- 和[「9.1.4 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的 ClusteringConfiguration 配置类是一致，只是创建了不同的 Exchange 。

### 9.2.3 BroadcastProducer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [BroadcastProducer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/BroadcastProducer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// BroadcastProducer.java

@Component
public class BroadcastProducer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 BroadcastMessage 消息
        BroadcastMessage message = new BroadcastMessage();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(BroadcastMessage.EXCHANGE, null, message);
    }

}
```



- 和[「9.1.5 ClusteringProducer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)是一致，只是使用了不同的 Exchange 和消息。

### 9.2.4 BroadcastConsumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [BroadcastConsumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/ClusteringConsumer.java) 类，**广播**消费消息。代码如下：



```
// BroadcastConsumer.java

@Component
@RabbitListener(
        bindings = @QueueBinding(
                value = @Queue(
                        name = BroadcastMessage.QUEUE + "-" + "#{T(java.util.UUID).randomUUID()}",
                        autoDelete = "true"
                ),
                exchange = @Exchange(
                        name = BroadcastMessage.EXCHANGE,
                        type = ExchangeTypes.TOPIC,
                        declare = "false"
                )
        )
)
public class BroadcastConsumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(BroadcastMessage message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 总体和[「9.1.6 ClusteringConsumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)是一致，主要差异在两点。
- 第一点，在 `@Queue` 注解的 `name` 属性，我们通过 Spring EL 表达式，在 Queue 的名字上，使用 UUID 生成其后缀。这样，我们就能保证每个项目启动的 Consumer 的 Queue 不同，以达到广播消费的目的。
- 第二点，在 `@Queue` 注解的 `autoDelete` 属性，我们设置为 `"true"` ，这样在 Consumer 关闭时，该队列就可以被自动删除了。

### 9.2.5 简单测试

创建 [BroadcastProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-message-model/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/BroadcastProducerTest.java) 测试类，用于测试广播消费。代码如下：



```
// BroadcastProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class BroadcastProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private BroadcastProducer producer;

    @Test
    public void mock() throws InterruptedException {
        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



首先，执行 `#mock()` 测试方法，先启动一个消费 `"QUEUE_BROADCAST-${UUID1}"` 这个 Queue 的 Consumer 节点。

然后，执行 `#testSyncSend()` 测试方法，再启动一个消费 `"QUEUE_BROADCAST-${UUID2}"` 这个 Queue 的 Consumer 节点。同时，该测试方法，调用 `BroadcastProducer#syncSend(id)` 方法，同步发送了 3 条消息。控制台输出如下：



```
// #### testSyncSend 方法对应的控制台 ####

# Producer 同步发送消息成功
2019-12-15 00:11:41.459  INFO 64479 --- [           main] c.i.s.l.r.p.BroadcastProducerTest        : [testSyncSend][发送编号：[1576080701] 发送成功]
2019-12-15 00:11:41.460  INFO 64479 --- [           main] c.i.s.l.r.p.BroadcastProducerTest        : [testSyncSend][发送编号：[1576080701] 发送成功]
2019-12-15 00:11:41.461  INFO 64479 --- [           main] c.i.s.l.r.p.BroadcastProducerTest        : [testSyncSend][发送编号：[1576080701] 发送成功]

# BroadcastConsumer 消费了 3 条消息
2019-12-15 00:11:41.478  INFO 64479 --- [ntContainer#0-1] c.i.s.l.r.consumer.BroadcastConsumer     : [onMessage][线程编号:17 消息内容：BroadcastMessage{id=1576080701}]
2019-12-15 00:11:41.479  INFO 64479 --- [ntContainer#0-1] c.i.s.l.r.consumer.BroadcastConsumer     : [onMessage][线程编号:17 消息内容：BroadcastMessage{id=1576080701}]
2019-12-15 00:11:41.480  INFO 64479 --- [ntContainer#0-1] c.i.s.l.r.consumer.BroadcastConsumer     : [onMessage][线程编号:17 消息内容：BroadcastMessage{id=1576080701}]

// ### mock 方法对应的控制台 ####

# BroadcastConsumer 也消费了 3 条消
2019-12-15 00:11:41.460  INFO 63795 --- [ntContainer#0-1] c.i.s.l.r.consumer.BroadcastConsumer     : [onMessage][线程编号:17 消息内容：BroadcastMessage{id=1576080701}]
2019-12-15 00:11:41.462  INFO 63795 --- [ntContainer#0-1] c.i.s.l.r.consumer.BroadcastConsumer     : [onMessage][线程编号:17 消息内容：BroadcastMessage{id=1576080701}]
2019-12-15 00:11:41.462  INFO 63795 --- [ntContainer#0-1] c.i.s.l.r.consumer.BroadcastConsumer     : [onMessage][线程编号:17 消息内容：BroadcastMessage{id=1576080701}]
```



- **两个** Consumer 节点，都消费了这条发送的消息。符合广播消费的预期~

# 10. 并发消费

> 示例代码对应仓库：[lab-04-rabbitmq-demo-concurrency](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency)

在上述的示例中，我们配置的每一个 Spring-AMQP `@RabbitListener` ，都是**串行**消费的。显然，这在监听的 Queue 每秒消息量比较大的时候，会导致消费不及时，导致消息积压的问题。

虽然说，我们可以通过启动多个 JVM 进程，实现**多进程的并发消费**，从而加速消费的速度。但是问题是，否能够实现**多线程**的并发消费呢？答案是**有**。

在 `@RabbitListener` 注解中，有 `concurrency` 属性，它可以指定并发消费的线程数。例如说，如果设置 `concurrency=4` 时，Spring-AMQP 就会为**该** `@RabbitListener` 创建 4 个线程，进行并发消费。

考虑到让胖友能够更好的理解 `concurrency` 属性，我们来简单说说 Spring-AMQP 在这块的实现方式。我们来举个例子：

- 首先，我们来创建一个 Queue 为 `"DEMO_09"` 。
- 然后，我们创建一个 Demo9Consumer 类，并在其消费方法上，添加 `@RabbitListener(concurrency=2)` 注解。
- 再然后，我们启动项目。Spring-AMQP 会根据 `@RabbitListener(concurrency=2)` 注解，创建 **2** 个 RabbitMQ Consumer 。注意噢，是 **2** 个 RabbitMQ Consumer 呢！！！后续，每个 RabbitMQ Consumer 会被**单独**分配到一个线程中，进行拉取消息，消费消息。

酱紫讲解一下，胖友对 Spring-AMQP 实现**多线程**的并发消费的机制，是否理解了。

下面，我们开始本小节的示例。本示例就是上述举例的具体实现。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-concurrency](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency) 项目。

## 10.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/pom.xml) 文件。

## 10.2 应用配置文件

在开始看具体的应用配置文件之前，我们先来了了解下 Spring-AMQP 的两个 [ContainerType](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java#L566-L579) 容器类型，枚举如下：



```
// RabbitProperties.java

public enum ContainerType {

	/**
	 * Container where the RabbitMQ consumer dispatches messages to an invoker thread.
	 */
	SIMPLE,

	/**
	 * Container where the listener is invoked directly on the RabbitMQ consumer
	 * thread.
	 */
	DIRECT

}
```



① 第一种类型，`SIMPLE` 对应 [SimpleMessageListenerContainer](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/listener/SimpleMessageListenerContainer.java) 消息监听器容器。它一共有两**类**线程：

- Consumer 线程，负责从 RabbitMQ Broker 获取 Queue 中的消息，存储到**内存中**的 [BlockingQueue](https://github.com/openjdk-mirror/jdk7u-jdk/blob/master/src/share/classes/java/util/concurrent/BlockingQueue.java) 阻塞队列中。
- Listener 线程，负责从**内存中**的 BlockingQueue 获取消息，进行消费逻辑。

注意，每一个 Consumer 线程，对应一个 RabbitMQ Consumer ，对应一个 Listener 线程。也就是说，它们三者是**一一对应**的。

② 第二种类型，`DIRECT` 对应 [DirectMessageListenerContainer](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/listener/DirectMessageListenerContainer.java) 消息监听器容器。它只有**一类**线程，即做 `SIMPLE` 的 Consumer 线程的工作，也做 `SIMPLE` 的 Listener 线程工作。

注意，因为只有**一类**线程，所以它要么正在获取消息，要么正在消费消息，也就是**串行**的。

🔥 默认情况下，Spring-AMQP 选择使用第一种类型，即 `SIMPLE` 容器类型。

下面，让我们一起看看 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
    listener:
      type: simple # 选择的 ListenerContainer 的类型。默认为 simple 类型
      simple:
        concurrency: 2 # 每个 @ListenerContainer 的并发消费的线程数
        max-concurrency: 10 # 每个 @ListenerContainer 允许的并发消费的线程数
#      direct:
#        consumers-per-queue: 2 # 对于每一个 @RabbitListener ，一个 Queue ，对应创建几个 Consumer 。
```



- 相比

   

  「3.1.2 应用配置文件」

   

  来说，额外三个参数：

  - `spring.rabbitmq.listener.type`
  - `spring.rabbitmq.listener.simple.concurrency`
  - `spring.rabbitmq.listener.simple.max-concurrency`

要**注意**，是 `spring.rabbitmq.listener.simple.max-concurrency` 配置，是**限制**每个 `@RabbitListener` 的**允许**配置的 `concurrency` 最大大小。如果超过，则会抛出 IllegalArgumentException 异常。在[「10.6 Demo09Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中，我们会看到 `@RabbitListener` 注解，有一个 `concurrency` 属性，可以自定义每个 `@RabbitListener` 的并发消费的线程数。

## 10.3 Demo09Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo09Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo09Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 10.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，配置相关的 Exchange、Queue、Binding 。

和[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 10.5 Demo09Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo09Producer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo09Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。

和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、RoutingKey 名字不同。

## 10.6 Demo09Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo09Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo09Consumer.java) 类，消费消息。代码如下：



```
// Demo06Consumer.java

@Component
@RabbitListener(queues = Demo09Message.QUEUE,
    concurrency = "2")
public class Demo09Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo09Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
    }

}
```



- 和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只差在消费不同的队列。
- 另外，可以通过 `@RabbitListener` 注解，设置并发数。优先级最高，可覆盖配置文件中的 `spring.rabbitmq.listener.simple.concurrency` 配置项。

不过个人建议，还是每个 `@RabbitListener` 各自配置，毕竟每个 Queue 的消息数量，都是不同的。当然，也可以结合使用 😈 。

## 10.7 简单测试

创建 [Demo09ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-concurrency/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo09ProducerTest.java) 测试类，编写一个单元测试方法，发送 10 条消息，观察并发消费情况。代码如下：



```
// Demo09ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo09ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo09Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            int id = (int) (System.currentTimeMillis() / 1000);
            producer.syncSend(id);
//            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行单元测试方法，控制台输出如下：



```
# 线程编号为 17
2019-12-15 10:48:20.013  INFO 2937 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:17 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.015  INFO 2937 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:17 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.016  INFO 2937 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:17 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.017  INFO 2937 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:17 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.017  INFO 2937 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:17 消息内容：Demo09Message{id=1576118899}]

# 线程编号 18
2019-12-15 10:48:20.013  INFO 2937 --- [ntContainer#0-2] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:18 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.015  INFO 2937 --- [ntContainer#0-2] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:18 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.016  INFO 2937 --- [ntContainer#0-2] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:18 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.016  INFO 2937 --- [ntContainer#0-2] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:18 消息内容：Demo09Message{id=1576118899}]
2019-12-15 10:48:20.017  INFO 2937 --- [ntContainer#0-2] c.i.s.l.r.consumer.Demo09Consumer        : [onMessage][线程编号:18 消息内容：Demo09Message{id=1576118899}]
```



- 我们可以看到，两个线程在消费 `"QUEUE_DEMO_09"` 下的消息。

此时，如果我们使用 [RabbitMQ Management](https://www.rabbitmq.com/management.html) 来查看 `"QUEUE_DEMO_09"` 的消费者列表：![ 的消费者列表](E:\Development\Typora\images\01-16592018324545.png)

# 11. 顺序消息

> 示例代码对应仓库：[lab-04-rabbitmq-demo-orderly](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly)

我们先来一起了解下顺序消息的**顺序消息**的定义：

- 普通顺序消息 ：Producer 将相关联的消息发送到相同的消息队列。
- 完全严格顺序 ：在【普通顺序消息】的基础上，Consumer 严格顺序消费。

那么，让我们来思考下，如果我们希望在 RabbitMQ 上，实现顺序消息需要做两个事情。

① **事情一**，我们需要保证 RabbitMQ Producer 发送相关联的消息发送到相同的 Queue 中。例如说，我们要发送用户信息发生变更的 Message ，那么如果我们希望使用顺序消息的情况下，可以将**用户编号**相同的消息发送到相同的 Queue 中。

② **事情二**，我们在有**且仅启动一个** Consumer 消费该队列，保证 Consumer 严格顺序消费。

不过如果这样做，会存在两个问题，我们逐个来看看。

① **问题一**，正如我们在[「10. 并发消费」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中提到，如果我们将消息仅仅投递到一个 Queue 中，并且采用单个 Consumer **串行**消费，在监听的 Queue 每秒消息量比较大的时候，会导致消费不及时，导致消息积压的问题。

此时，我们有两种方案来解决：

- 方案一，在 Producer 端，将 Queue 拆成多个**子** Queue 。假设原先 Queue 是 `QUEUE_USER` ，那么我们就分拆成 `QUEUE_USER_00` 至 `QUEUE_USER_..${N-1}` 这样 N 个队列，然后基于消息的用户编号取余，路由到对应的**子** Queue 中。
- 方案二，在 Consumer 端，将 Queue 拉取到的消息，将相关联的消息发送到**相同的线程**中来消费。例如说，还是 Queue 是 `QUEUE_USER` 的例子，我们创建 N 个线程池大小为 1 的 [ExecutorService](https://github.com/JetBrains/jdk8u_jdk/blob/master/src/share/classes/java/util/concurrent/ExecutorService.java) 数组，然后基于消息的用户编号取余，提交到对应的 ExecutorService 中的单个线程来执行。

两个方案，并不冲突，可以结合使用。

② **问题二**，如果我们启动相同 Consumer 的**多个进程**，会导致相同 Queue 的消息被分配到多个 Consumer 进行消费，破坏 Consumer 严格顺序消费。

此时，我们有两种方案来解决：

- 方案一，引入 ZooKeeper 来协调，动态设置多个进程中的**相同的** Consumer 的开关，保证有且仅有一个 Consumer 开启对**同一个** Queue 的消费。
- 方案二，仅适用于【问题一】的【方案一】。还是引入 ZooKeeper 来协调，动态设置多个进程中的**相同的** Consumer 消费的 Queue 的分配，保证有且仅有一个 Consumer 开启对**同一个** Queue 的消费。

下面，我们开始本小节的示例。本示例就是上述举例的具体实现。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-orderly](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly) 项目。

- 对于问题一，我们采用方案一。因为在 Spring-AMQP 中，自己定义线程来消费消息，无法和现有的 [MessageListenerContainer](https://github.com/spring-projects/spring-framework/blob/master/spring-jms/src/main/java/org/springframework/jms/listener/MessageListenerContainer.java) 的实现所结合，除非自定义一个 MessageListenerContainer 实现类。
- 对于问题二，因为实现起来比较复杂，暂时先不提供。

## 11.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/pom.xml) 文件。

## 11.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/resources/application.yaml) 文件。

## 11.3 Demo10Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo10Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo10Message.java) 消息类，提供给当前示例使用。代码如下：



```
// Demo10Message.java

public class Demo10Message implements Serializable {

    private static final String QUEUE_BASE = "QUEUE_DEMO_10-";
    public static final String QUEUE_0 = QUEUE_BASE + "0";
    public static final String QUEUE_1 = QUEUE_BASE + "1";
    public static final String QUEUE_2 = QUEUE_BASE + "2";
    public static final String QUEUE_3 = QUEUE_BASE + "3";

    public static final int QUEUE_COUNT = 4;

    public static final String EXCHANGE = "EXCHANGE_DEMO_10";

    /**
     * 编号
     */
    private Integer id;

    // ... 省略 set/get/toString 方法

}
```



- 定义了 `QUEUE_DEMO_10-` 的四个**子** Queue 。
- 定义了统一的 Exchange 。
- 暂未定义 RoutingKey 的名字，我们会使用“队列编号”作为 RoutingKey ，然后路由消息到每个子 Queue 中。

## 11.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 Direct Exchange 示例相关的 Exchange、Queue、Binding 的配置。代码如下：



```
// RabbitConfig.java


@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo10Queue0() {
            return new Queue(Demo10Message.QUEUE_0);
        }
        @Bean
        public Queue demo10Queue1() {
            return new Queue(Demo10Message.QUEUE_1);
        }
        @Bean
        public Queue demo10Queue2() {
            return new Queue(Demo10Message.QUEUE_2);
        }
        @Bean
        public Queue demo10Queue3() {
            return new Queue(Demo10Message.QUEUE_3);
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo10Exchange() {
            return new DirectExchange(Demo10Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        @Bean
        public Binding demo10Binding0() {
            return BindingBuilder.bind(demo10Queue0()).to(demo10Exchange()).with("0");
        }
        @Bean
        public Binding demo10Binding1() {
            return BindingBuilder.bind(demo10Queue1()).to(demo10Exchange()).with("1");
        }
        @Bean
        public Binding demo10Binding2() {
            return BindingBuilder.bind(demo10Queue2()).to(demo10Exchange()).with("2");
        }
        @Bean
        public Binding demo10Binding3() {
            return BindingBuilder.bind(demo10Queue3()).to(demo10Exchange()).with("3");
        }

    }

}
```



- 首先，创建了四个**子** Queue 。
- 然后，创建了一个 Exchange 。
- 最后，创建了四个 Bingding ，对应每一个**子** Queue ，RoutingKey 为队列编号。

## 11.5 Demo10Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo01Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo01Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息到**子** Queue 中。代码如下：



```
// Demo10Producer.java

@Component
public class Demo10Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo10Message 消息
        Demo10Message message = new Demo10Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo10Message.EXCHANGE, this.getRoutingKey(id), message);
    }

    private String getRoutingKey(Integer id) {
        return String.valueOf(id % Demo10Message.QUEUE_COUNT);
    }

}
```



- 发送消息时，我们对 `Demo10Message.id % 队列编号` 进行取余，获得**队列编号**作为 RoutingKey ，从而路由消息到对应的**子** Queue 中。

## 11.6 Demo10Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo10Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo10Consumer.java) 类，**严格**消费**顺序**消息。代码如下：



```
// Demo10Consumer.java

@Component
@RabbitListener(queues = Demo10Message.QUEUE_0)
@RabbitListener(queues = Demo10Message.QUEUE_1)
@RabbitListener(queues = Demo10Message.QUEUE_2)
@RabbitListener(queues = Demo10Message.QUEUE_3)
public class Demo10Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler(isDefault = true)
    public void onMessage(Message<Demo10Message> message) {
        logger.info("[onMessage][线程编号:{} Queue:{} 消息编号：{}]", Thread.currentThread().getId(), getQueue(message),
                message.getPayload().getId());
    }

    private static String getQueue(Message<Demo10Message> message) {
        return message.getHeaders().get("amqp_consumerQueue", String.class);
    }

}
```



- 为了实现每个**子** Queue 能够被每个 Consumer **串行**消费，从而实现基于**子** Queue 的**并行**的**严格**消费**顺序**消息，我们只好在类上添了四个 `@RabbitListener` 注解，每个对应一个**子** Queue 。
- 如果胖友使用一个 `@RabbitListener` 注解，并添加四个**子** Queue ，然后设置 `concurrency = 4` 时，实际是并发四个线程，消费四个**子** Queue 的消息，无法保证**严格**消费**顺序**消息。

## 11.7 简单测试

创建 [Demo10ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-orderly/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo10ProducerTest.java) 测试类，编写一个单元测试方法，发送 8 条消息，观察顺序消费情况。代码如下：



```
// Demo10ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo10ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo10Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int i = 0; i < 2; i++) {
            for (int id = 0; id < 4; id++) {
                producer.syncSend(id);
//            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
            }
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



- 发送 2 轮消息，每一轮消息的编号都是 0 至 3 。

执行单元测试方法，控制台输出如下：



```
# 线程编号为 21
2019-12-15 20:04:59.262  INFO 99104 --- [ntContainer#2-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:21 Queue:QUEUE_DEMO_10-2 消息编号：2]
2019-12-15 20:04:59.265  INFO 99104 --- [ntContainer#2-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:21 Queue:QUEUE_DEMO_10-2 消息编号：2]

# 线程编号为 17
2019-12-15 20:04:59.262  INFO 99104 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:17 Queue:QUEUE_DEMO_10-0 消息编号：0]
2019-12-15 20:04:59.265  INFO 99104 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:17 Queue:QUEUE_DEMO_10-0 消息编号：0]

# 线程编号为 23
2019-12-15 20:04:59.262  INFO 99104 --- [ntContainer#3-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:23 Queue:QUEUE_DEMO_10-3 消息编号：3]
2019-12-15 20:04:59.265  INFO 99104 --- [ntContainer#3-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:23 Queue:QUEUE_DEMO_10-3 消息编号：3]

# 线程编号为 19
2019-12-15 20:04:59.262  INFO 99104 --- [ntContainer#1-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:19 Queue:QUEUE_DEMO_10-1 消息编号：1]
2019-12-15 20:04:59.265  INFO 99104 --- [ntContainer#1-1] c.i.s.l.r.consumer.Demo10Consumer        : [onMessage][线程编号:19 Queue:QUEUE_DEMO_10-1 消息编号：1]
```



- 相同编号的消息，被投递到相同的**子** Queue ，被相同的线程所消费。符合预期~

# 12. 事务消息

> 示例代码对应仓库：[lab-04-rabbitmq-demo-transaction](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction) 。

RabbitMQ 内置提供事务消息的支持。对事务消息的概念不了解的胖友，可以看看 [《RabbitMQ 之消息确认机制（事务 + Confirm）》](http://www.iocoder.cn/RabbitMQ/message-confirmation-mechanism-transaction-Confirm/?self) 文章的[「事务机制」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节。

不过 RabbitMQ 提供的并不是**完整的**的事务消息的支持，缺少了**回查机制**。目前，常用的分布式消息队列，只有 RocketMQ 提供了完整的事务消息的支持，具体的可以看看[《芋道 Spring Boot 消息队列 RocketMQ 入门》](http://www.iocoder.cn/Spring-Boot/RocketMQ/?self) 的[「9. 事务消息」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节，😈 暂时不拓展开来讲。

下面，我们开始本小节的示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-transaction](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction) 项目。

## 12.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/pom.xml) 文件。

## 12.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/resources/application.yaml) 文件。

## 12.3 Demo11Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo11Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo11Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 12.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，额外配置事务相关的。代码如下：



```
// RabbitConfig.java

@Configuration
@EnableTransactionManagement
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo11Queue() {
            return new Queue(Demo11Message.QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo11Exchange() {
            return new DirectExchange(Demo11Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo11Message.EXCHANGE
        // Routing key：Demo11Message.ROUTING_KEY
        // Queue：Demo11Message.QUEUE
        @Bean
        public Binding demo11Binding() {
            return BindingBuilder.bind(demo11Queue()).to(demo11Exchange()).with(Demo11Message.ROUTING_KEY);
        }

    }

    @Bean
    public RabbitTransactionManager rabbitTransactionManager(ConnectionFactory connectionFactory, RabbitTemplate rabbitTemplate) {
        // <Y> 设置 RabbitTemplate 支持事务
        rabbitTemplate.setChannelTransacted(true);

        // 创建 RabbitTransactionManager 对象
        return new RabbitTransactionManager(connectionFactory);
    }

}
```



- DirectExchangeDemoConfiguration 配置类，用于定义 Queue、Exchange、Binding 的配置。
- 在类上，添加 `@EnableTransactionManagement` 注解，开启[Spring Transaction](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/transaction.html) 的支持。
- 在 `#rabbitTransactionManager()` 方法，创建 [RabbitTransactionManager](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/transaction/RabbitTransactionManager.java) 事务管理器 Bean 。
- 在 `<Y>` 处，标记创建的 RabbitMQ Channel 是事务性的，从而可以使用 RabbitMQ 的事务消息。

因为 Spring-AMQP 通过 RabbitTransactionManager 来实现对 Spring Transaction 的集成，所以在实际开发中，我们只需要配合使用 `@Transactional` 注解，来声明事务即可。

## 12.5 Demo11Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo11Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo11Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。代码如下：



```
// Demo11Producer.java

@Component
public class Demo11Producer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Transactional
    public void syncSend(Integer id) throws InterruptedException {
        // 创建 Demo11Message 消息
        Demo11Message message = new Demo11Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo11Message.EXCHANGE, Demo11Message.ROUTING_KEY, message);
        logger.info("[syncSend][发送编号：[{}] 发送成功]", id);

        // <X> 等待
        Thread.sleep(10 * 1000L);
    }

}
```



- 在发送消息方法上，我们添加了 `@Transactional` 注解，声明事务。因为我们创建了 RabbitTransactionManager 事务管理器，所以这里会创建 RabbitMQ 事务。

- 在

   

  ```
  <X>
  ```

   

  处，我们故意等待

   

  ```
  Thread#sleep(long millis)
  ```

   

  10 秒，判断 RabbitMQ 事务是否生效。

  - 如果同步发送消息成功后，Consumer 立即消费到该消息，说明未生效。
  - 如果 Consumer 是 10 秒之后，才消费到该消息，说明已生效。

## 12.6 Demo11Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo11Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo11Consumer.java) 类，消费消息。

和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_11"` 。

## 12.7 简单测试

创建 [Demo11ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-transaction/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo11ProducerTest.java) 测试类，编写单元测试方法，测试 Producer 发送**事务**消息的效果。代码如下：



```
// Demo11ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo11ProducerTest {

    @Autowired
    private Demo11Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行单元测试方法，控制台输出如下：



```
// Producer 成功同步发送了 1 条消息。此时，事务并未提交
2019-12-12 22:03:05.525  INFO 17729 --- [           main] c.i.s.l.r.producer.Demo11Producer        : [syncSend][发送编号：[1576159385] 发送成功]

// 10 秒后，Producer 提交事务。
// 此时，Consumer 消费到该消息。
2019-12-12 22:03:15.548  INFO 17729 --- [ntContainer#4-1] c.i.s.l.r.consumer.Demo11Consumer        : [onMessage][线程编号:25 消息内容：Demo11Message{id=1576159385}]
```



- Consumer 在事务消息提交后，消费到该消息。符合预期~

# 13. 消费者的消息确认

> 示例代码对应仓库：[lab-04-rabbitmq-demo-ack](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack) 。

在 RabbitMQ 中，Consumer 有两种消息确认的方式：

- 方式一，自动确认。
- 方式二，手动确认。

对于**自动确认**的方式，RabbitMQ Broker 只要将消息写入到 TCP Socket 中成功，就认为该消息投递成功，而无需 Consumer **手动确认**。

对于**手动确认**的方式，RabbitMQ Broker 将消息发送给 Consumer 之后，由 Consumer **手动确认**之后，才任务消息投递成功。

实际场景下，因为自动确认存在可能**丢失消息**的情况，所以在对**可靠性**有要求的场景下，我们基本采用手动确认。当然，如果允许消息有一定的丢失，对**性能**有更高的产经下，我们可以考虑采用自动确认。

😈 更多关于消费者的消息确认的内容，胖友可以阅读如下的文章：

- [《Consumer Acknowledgements and Publisher Confirms》](https://www.rabbitmq.com/confirms.html) 的消费者部分的内容，对应中文翻译为 [《消费者应答（ACK）和发布者确认（Confirm）》](https://blog.bossma.cn/rabbitmq/consumer-ack-and-publisher-confirm/) 。
- [《RabbitMQ 之消息确认机制（事务 + Confirm）》](http://www.iocoder.cn/RabbitMQ/message-confirmation-mechanism-transaction-Confirm/?self) 文章的[「消息确认（Consumer端）」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节。

在 Spring-AMQP 中，在 [AcknowledgeMode](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/AcknowledgeMode.java) 中，定义了三种消息确认的方式：



```
// AcknowledgeMode.java

/**
 * No acks - {@code autoAck=true} in {@code Channel.basicConsume()}.
 */
NONE, // 对应 Consumer 的自动确认

/**
 * Manual acks - user must ack/nack via a channel aware listener.
 */
MANUAL, // 对应 Consumer 的手动确认，由开发者在消费逻辑中，手动进行确认。

/**
 * Auto - the container will issue the ack/nack based on whether
 * the listener returns normally, or throws an exception.
 * <p><em>Do not confuse with RabbitMQ {@code autoAck} which is
 * represented by {@link #NONE} here</em>.
 */
AUTO; // 对应 Consumer 的手动确认，在消费消息完成（包括正常返回、和抛出异常）后，由 Spring-AMQP 框架来“自动”进行确认。
```



- 实际上，就是将**手动确认**进一步细分，提供了由 Spring-AMQP 提供 Consumer 级别的自动确认。

**在上述的示例中，我们都采用了 Spring-AMQP 默认的 `AUTO` 模式**。下面，我们来搭建一个 `MANUAL` 模式，手动确认的示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-ack](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack) 项目。

## 13.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/pom.xml) 文件。

## 13.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
    listener:
      simple:
        acknowledge-mode: manual # 配置 Consumer 手动提交
```



- 相比[「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，我们通过**新增** `spring.rabbitmq.listener.simple.acknowledge-mode=true` 配置项，来配置 Consumer 手动提交。

## 13.3 Demo12Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo12Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo12Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 13.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，配置相关的 Exchange、Queue、Binding 。

和[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 13.5 Demo12Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo12Producer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/Demo12Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。

和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、RoutingKey 名字不同。

## 13.6 Demo12Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo09Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo12Consumer.java) 类，消费消息。代码如下：



```
// Demo12Consumer.java

@Component
@RabbitListener(queues = Demo12Message.QUEUE)
public class Demo12Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo12Message message, Channel channel,
                          @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag) throws IOException {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // 提交消费进度
        if (message.getId() % 2 == 1) {
            // ack 确认消息
            // 第二个参数 multiple ，用于批量确认消息，为了减少网络流量，手动确认可以被批处。
            // 1. 当 multiple 为 true 时，则可以一次性确认 deliveryTag 小于等于传入值的所有消息
            // 2. 当 multiple 为 false 时，则只确认当前 deliveryTag 对应的消息
            channel.basicAck(deliveryTag, false);
        }
    }

}
```



- 在消费方法上，我们增加类型为 [Channel](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java) 的方法参数，和 `deliveryTag` 。通过调用其 `Channel#basicAck(deliveryTag, multiple)` 方法，可以进行消息的确认。这里，艿艿添加了比较详细的注释说明，胖友可以自己瞅瞅噢。
- 在 `@RabbitListener` 注解的 `ackMode` 属性，我们可以设置自定义的 AcknowledgeMode 模式。
- 在消费逻辑中，我们故意只提交消费的消息的 `Demo12Message.id` 为**奇数**的消息。😈 这样，我们只需要发送一条 `id=1` ，一条 `id=2` 的消息，如果第二条的消费进度没有被提交，就可以说明手动提交消费进度成功。

## 13.7 简单测试

创建 [Demo12ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-ack/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo12ProducerTest.java) 测试类，编写单元测试方法，测试手动提交消费进度。代码如下：



```
// Demo12ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo12ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo12Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        for (int id = 1; id <= 2; id++) {
            producer.syncSend(id);
            logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);
        }

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行 `#testSyncSend()` 单元测试，输出日志如下：



```
// Producer 同步发送 2 条消息成功
2019-12-13 00:19:33.403  INFO 45869 --- [           main] c.i.s.l.r.producer.Demo12ProducerTest    : [testSyncSend][发送编号：[1] 发送成功]
2019-12-13 00:19:33.406  INFO 45869 --- [           main] c.i.s.l.r.producer.Demo12ProducerTest    : [testSyncSend][发送编号：[2] 发送成功]

// Demo08Consumer 消费 2 条消息成功
2019-12-13 00:19:33.420  INFO 45869 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo12Consumer        : [onMessage][线程编号:17 消息内容：Demo12Message{id=1}]
2019-12-13 00:19:33.421  INFO 45869 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo12Consumer        : [onMessage][线程编号:17 消息内容：Demo12Message{id=2}]
```



此时，如果我们使用 [RabbitMQ Management](https://static.iocoder.cn/7c5541295505e7a3be4ac7ab2882feeb) 来查看 `"DEMO_12"` 的该消费者：![ 的消费者列](E:\Development\Typora\images\02-16592018324547.png)

- 有 1 条消息的未确认，符合预期~

# 14. 生产者的发送确认

在 RabbitMQ 中，**默认**情况下，Producer 发送消息的方法，只保证将消息写入到 TCP Socket 中成功，并不保证消息发送到 RabbitMQ Broker 成功，并且持久化消息到磁盘成功。

也就是说，我们上述的示例，Producer 在发送消息都不是绝对可靠，是存在丢失消息的可能性。

不过不用担心，在 RabbitMQ 中，Producer 采用 Confirm 模式，实现发送消息的确认机制，以保证消息发送的可靠性。实现原理如下：

- 首先，Producer 通过调用 [`Channel#confirmSelect()`](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java#L1278-L1283) 方法，将 Channel 设置为 Confirm 模式。
- 然后，在该 Channel 发送的消息时，需要先通过 [`Channel#getNextPublishSeqNo()`](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java#L1285-L1290) 方法，给发送的消息分配一个唯一的 ID 编号(`seqNo` 从 1 开始递增)，再发送该消息给 RabbitMQ Broker 。
- 之后，RabbitMQ Broker 在接收到该消息，并被路由到相应的队列之后，会发送一个包含消息的唯一编号(`deliveryTag`)的确认给 Producer 。

通过 `seqNo` 编号，将 Producer 发送消息的“请求”，和 RabbitMQ Broker 确认消息的“响应”串联在一起。

通过这样的方式，Producer 就可以知道消息是否成功发送到 RabbitMQ Broker 之中，保证消息发送的可靠性。不过要注意，整个执行的过程实际是**异步**，需要我们调用 [`Channel#waitForConfirms()`](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java#L1293-L1329) 方法，**同步**阻塞等待 RabbitMQ Broker 确认消息的“响应”。

也因此，Producer 采用 Confirm 模式时，有三种编程方式：

- 【同步】普通 Confirm 模式：Producer 每发送一条消息后，调用 `Channel#waitForConfirms()` 方法，等待服务器端 Confirm 。

- 【同步】批量 Confirm 模式：Producer 每发送一批消息后，调用`Channel#waitForConfirms()` 方法，等待服务器端 Confirm 。

  > 本质上，和「普通 Confirm 模式」是一样的。

- 【异步】异步 Confirm 模式：Producer 提供一个回调方法，RabbitMQ Broker 在 Confirm 了一条或者多条消息后，Producer 会回调这个方法。

😈 更多关于 Producer 的 Confirm 模式的内容，胖友可以阅读如下的文章：

- [《Consumer Acknowledgements and Publisher Confirms》](https://www.rabbitmq.com/confirms.html) 的生产者部分的内容，对应中文翻译为 [《消费者应答（ACK）和发布者确认（Confirm）》](https://blog.bossma.cn/rabbitmq/consumer-ack-and-publisher-confirm/) 。
- [《RabbitMQ 之消息确认机制（事务 + Confirm）》](http://www.iocoder.cn/RabbitMQ/message-confirmation-mechanism-transaction-Confirm/?self) 文章的[「Confirm 模式」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)小节。

在 Spring-AMQP 中，在 [ConfirmType](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/connection/CachingConnectionFactory.java#L145-L167) 中，定义了三种消息确认的方式：



```
// CachingConnectionFactory#ConfirmType.java

public enum ConfirmType {

	/**
	 * Use {@code RabbitTemplate#waitForConfirms()} (or {@code waitForConfirmsOrDie()}
	 * within scoped operations.
	 */
	SIMPLE, // 使用同步的 Confirm 模式

	/**
	 * Use with {@code CorrelationData} to correlate confirmations with sent
	 * messsages.
	 */
	CORRELATED, // 使用异步的 Confirm 模式

	/**
	 * Publisher confirms are disabled (default).
	 */
	NONE // 不使用 Confirm 模式

}
```



**在上述的示例中，我们都采用了 Spring-AMQP 默认的 `NONE` 模式**。下面，我们来搭建两个示例：

- 在[「14.1 同步 Confirm 模式」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 中，我们会使用 `SIMPLE` 类型，实现同步的 Confirm 模式。
- 在[「14.2 异步 Confirm 模式」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 中，我们会使用 `CORRELATED` 类型，使用异步的 Confirm 模式。

## 14.1 同步 Confirm 模式

> 示例代码对应仓库：[lab-04-rabbitmq-demo-confirm](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm) 。

在本小节中，我们会使用 `ConfirmType.SIMPLE` 类型，实现同步的 Confirm 模式。

要注意，这里的**同步**，指的是我们通过调用 [`Channel#waitForConfirms()`](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java#L1293-L1329) 方法，**同步**阻塞等待 RabbitMQ Broker 确认消息的“响应”。

考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-confirm](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm) 项目。

### 14.1.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/pom.xml) 文件。

### 14.1.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
    publisher-confirm-type: simple # 设置 Confirm 类型为 SIMPLE 。
```



- 相比[「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，我们通过**新增** `spring.rabbitmq.publisher-confirm-type=simple` 配置项，设置 Confirm 类型为 `ConfirmType.SIMPLE` 。

在该类型下，Spring-AMQP 在创建完 RabbitMQ Channel 之后，会**自动**调用 [`Channel#confirmSelect()`](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java#L1278-L1283) 方法，将 Channel 设置为 Confirm 模式。

### 14.1.3 Demo13Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo13Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo13Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

### 14.1.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，配置相关的 Exchange、Queue、Binding 。

和[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

### 14.1.4 Demo13Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo13Producer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo13Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。



```
// Demo13Producer.java

@Component
public class Demo13Producer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo13Message 消息
        Demo13Message message = new Demo13Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.invoke(new RabbitOperations.OperationsCallback<Object>() {

            @Override
            public Object doInRabbit(RabbitOperations operations) {
                // 同步发送消息
                operations.convertAndSend(Demo13Message.EXCHANGE, Demo13Message.ROUTING_KEY, message);
                logger.info("[doInRabbit][发送消息完成]");
                // 等待确认
                operations.waitForConfirms(0); // timeout 参数，如果传递 0 ，表示无限等待
                logger.info("[doInRabbit][等待 Confirm 完成]");
                return null;
            }

        }, new ConfirmCallback() {

            @Override
            public void handle(long deliveryTag, boolean multiple) throws IOException {
                logger.info("[handle][Confirm 成功]");
            }

        }, new ConfirmCallback() {

            @Override
            public void handle(long deliveryTag, boolean multiple) throws IOException {
                logger.info("[handle][Confirm 失败]");
            }

        });

    }

}
```



- 在 RabbitTemplate 提供的 API 方法中，如果 Producer 要使用同步的 Confirm 模式，需要调用 `#invoke(action, acks, nacks)` 方法。代码如下：

  ```
  // RabbitOperations.java
  // RabbitTemplate 实现了 RabbitOperations 接口
  
  /**
   * Invoke operations on the same channel.
   * If callbacks are needed, both callbacks must be supplied.
   * @param action the callback.
   * @param acks a confirm callback for acks.
   * @param nacks a confirm callback for nacks.
   * @param <T> the return type.
   * @return the result of the action method.
   * @since 2.1
   */
  @Nullable
  <T> T invoke(OperationsCallback<T> action, @Nullable com.rabbitmq.client.ConfirmCallback acks,
  		@Nullable com.rabbitmq.client.ConfirmCallback nacks);
  ```

  

  - 因为 Confirm 模式需要基于**相同** Channel ，所以我们需要使用该方法。
  - 在方法参数 `action` 中，我们可以自定义操作。
  - 在方法参数 `acks` 中，定义接收到 RabbitMQ Broker 的成功“响应”时的成回调。
  - 在方法参数 `nacks` 中，定义接收到 RabbitMQ Broker 的失败“响应”时的成回调。

  > - 当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息。
  > - 如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，生产者应用程序同样可以在回调方法中处理该 nack 消息。

- 具体的发送方法的代码，胖友根据艿艿的注释，进行理解下。

### 14.1.5 Demo13Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo13Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo13Consumer.java) 类，消费消息。

和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_13"` 。

### 14.1.6 简单测试

创建 [Demo13ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo13ProducerTest.java) 测试类，编写单元测试方法，调用 Demo13Producer 发送消息的方法。代码如下：



```
// Demo13ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo13ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo13Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行 `#testSyncSend()` 单元测试，输出日志如下：



```
// 主线程，Producer 发送 1 条消息完成。
2019-12-13 12:49:13.680  INFO 13247 --- [           main] c.i.s.l.r.producer.Demo13Producer        : [doInRabbit][发送消息完成]

// AMQConnection 线程，Producer 确认收到 RabbitMQ Broker 对该消息的成功“响应” 。
2019-12-13 12:49:13.689  INFO 13247 --- [ 127.0.0.1:5672] c.i.s.l.r.producer.Demo13Producer        : [handle][Confirm 成功]

// 主线程，Producer 等待该消息的 Confirm 完成。
2019-12-13 12:49:13.689  INFO 13247 --- [           main] c.i.s.l.r.producer.Demo13Producer        : [doInRabbit][等待 Confirm 完成]

// 单元测试，打印下日志，可以忽略
2019-12-13 12:49:13.694  INFO 13247 --- [           main] c.i.s.l.r.producer.Demo13ProducerTest    :
[testSyncSend][发送编号：[1576212553] 发送成功]

// 消费者的线程，Consumer 消费到该消息
2019-12-13 12:49:13.699  INFO 13247 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo13Consumer        : [onMessage][线程编号:17 消息内容：Demo13Message{id=1576212553}]
```



- 符合预期~整个过程，好好理解艿艿在日志上，添加的过程注释噢。

## 14.2 异步 Confirm 模式

> 示例代码对应仓库：[lab-04-rabbitmq-demo-confirm-async](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async) 。

在本小节中，我们会使用 `ConfirmType.SIMPLE` 类型，实现异步的 Confirm 模式。

考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-confirm-async](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async) 项目。

### 14.2.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/pom.xml) 文件。

### 14.2.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/resources/application.yaml) 配置文件。配置如下：



```
spring:
  # RabbitMQ 配置项，对应 RabbitProperties 配置类
  rabbitmq:
    host: 127.0.0.1 # RabbitMQ 服务的地址
    port: 5672 # RabbitMQ 服务的端口
    username: guest # RabbitMQ 服务的账号
    password: guest # RabbitMQ 服务的密码
    publisher-confirm-type: correlated # 设置 Confirm 类型为 CORRELATED 。
```



- 相比[「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，我们通过**新增** `spring.rabbitmq.publisher-confirm-type=correlated` 配置项，设置 Confirm 类型为 `ConfirmType.CORRELATED` 。

在该类型下，Spring-AMQP 在创建完 RabbitMQ Channel 之后，也会**自动**调用 [`Channel#confirmSelect()`](https://github.com/rabbitmq/rabbitmq-java-client/blob/master/src/main/java/com/rabbitmq/client/Channel.java#L1278-L1283) 方法，将 Channel 设置为 Confirm 模式。

### 14.2.3 Demo13Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/) 包下，创建 [Demo13Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo13Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

### 14.2.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，配置相关的 Exchange、Queue、Binding 。

和[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

### 14.2.5 RabbitProducerConfirmCallback

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.core`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core/) 包下，创建 [RabbitProducerConfirmCallback](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core/RabbitProducerConfirmCallback.java) 类，实现 [RabbitTemplate.ConfirmCallback](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/RabbitTemplate.java#L2712-L2727) 接口，提供 Producer 收到 RabbitMQ 确认消息的“响应”的回调。代码如下：



```
// RabbitProducerConfirmCallback.java

@Component
public class RabbitProducerConfirmCallback implements RabbitTemplate.ConfirmCallback {

    private Logger logger = LoggerFactory.getLogger(getClass());

    public RabbitProducerConfirmCallback(RabbitTemplate rabbitTemplate) {
        rabbitTemplate.setConfirmCallback(this);
    }

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack) {
            logger.info("[confirm][Confirm 成功 correlationData: {}]", correlationData);
        } else {
            logger.error("[confirm][Confirm 失败 correlationData: {} cause: {}]", correlationData, cause);
        }
    }

}
```



- 在构造方法中，把自己设置到 RabbitTemplate 中，作为 Confirm 的回调。
- 在 `#confirm(...)` 方法中，根据是否 `ack` 成功，打印不同的日志。

### 14.2.6 Demo13Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo13Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo13Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。



```
// Demo13Producer.java

@Component
public class Demo13Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void syncSend(Integer id) {
        // 创建 Demo13Message 消息
        Demo13Message message = new Demo13Message();
        message.setId(id);
        // 同步发送消息
        rabbitTemplate.convertAndSend(Demo13Message.EXCHANGE, Demo13Message.ROUTING_KEY, message);
    }

}
```



- 不需要像[「14.1.4 Demo13Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一样，而是直接像我们其它示例一样，直接使用 RabbitTemplate 的 `#convertAndSend(...)` 等等方法即可。

### 14.2.7 Demo13Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/) 包下，创建 [Demo13Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo13Consumer.java) 类，消费消息。

和[「3.1.7 Demo01Consumer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)基本一致，差别在于消费的队列是 `"QUEUE_DEMO_13"` 。

### 14.2.8 简单测试

创建 [Demo13ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 测试类，编写单元测试方法，调用 Demo13Producer 发送消息的方法。代码如下：



```
// Demo13ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo13ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo13Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行 `#testSyncSend()` 单元测试，输出日志如下：



```
// 单元测试，打印下日志，可以忽略
2019-12-13 17:17:45.849  INFO 69003 --- [           main] c.i.s.l.r.producer.Demo13ProducerTest    :
[testSyncSend][发送编号：[1576228665] 发送成功]

// RabbitConnectionFactory 线程，Producer 确认收到 RabbitMQ Broker 对该消息的成功“响应” 。
// 因为我们在 Demo13Producer 发送消息的时候，并未传入 CorrelationData 参数，所以为 null 。
2019-12-13 17:17:45.859  INFO 69003 --- [nectionFactory1] .i.s.l.r.c.RabbitProducerConfirmCallback : [confirm][Confirm 成功 correlationData: null]

// 消费者的线程，Consumer 消费到该消息
2019-12-13 17:17:45.873  INFO 69003 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo13Consumer        : [onMessage][线程编号:17 消息内容：Demo13Message{id=1576228665}]
```



- 符合预期~整个过程，好好理解艿艿在日志上，添加的过程注释噢。

## 14.3 ReturnCallback

> 示例代码对应仓库：[lab-04-rabbitmq-demo-confirm-async](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async) 。

当 Producer 成功发送消息到 RabbitMQ Broker 时，但是在通过 Exchange 进行**匹配不到** Queue 时，Broker 会将该消息回退给 Producer 。

下面，我们来创建一个使用示例，继续在 [lab-04-rabbitmq-demo-confirm-async](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async) 项目改造。

### 14.3.1 RabbitProducerReturnCallback

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.core`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core/) 包下，创建 [RabbitProducerReturnCallback](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core/RabbitProducerReturnCallback.java) 类，实现 [RabbitTemplate.ReturnCallback](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/core/RabbitTemplate.java#L2712-L2727) 接口，提供 Producer 收到 RabbitMQ Broker 回退消息的的回调。代码如下：



```
// RabbitProducerReturnCallback.java

@Component
public class RabbitProducerReturnCallback implements RabbitTemplate.ReturnCallback {

    private Logger logger = LoggerFactory.getLogger(getClass());

    public RabbitProducerReturnCallback(RabbitTemplate rabbitTemplate) {
        rabbitTemplate.setReturnCallback(this);
    }

    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        logger.error("[returnedMessage][message: [{}] replyCode: [{}] replyText: [{}] exchange: [{}] routingKey: [{}]]",
                message, replyCode, replyText, exchange, routingKey);
    }

}
```



- 在构造方法中，把自己设置到 RabbitTemplate 中，作为消息 Return 的回调。
- 在 `#returnedMessage(...)` 方法中，打印错误日志。当然，具体怎么处理，胖友可以根据自己的需要哈。

### 14.3.2 Demo13Producer

修改 [Demo13Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo13Producer.java) 类，增加一个发送无法匹配到 Queue 的消息的方法。代码如下：



```
// Demo13Producer.java

public void syncSendReturn(Integer id) {
    // 创建 Demo13Message 消息
    Demo13Message message = new Demo13Message();
    message.setId(id);
    // 同步发送消息
    rabbitTemplate.convertAndSend(Demo13Message.EXCHANGE, "error", message);
}
```



- 发送消息的 RoutingKey ，我们故意设置为 `error` ，达到消息无法匹配到 Queue 的效果。

### 14.3.3 简单测试

修改 [Demo13ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-confirm-async/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo13ProducerTest.java) 测试类，增加调用 Demo13Producer 新增的方法。代码如下：



```
// Demo13ProducerTest.java

@Test
public void testSyncSendReturn() throws InterruptedException {
    int id = (int) (System.currentTimeMillis() / 1000);
    producer.syncSendReturn(id);
    logger.info("[testSyncSendReturn][发送编号：[{}] 发送成功]", id);

    // 阻塞等待，保证消费
    new CountDownLatch(1).await();
}
```



执行 `#testSyncSendReturn()` 单元测试，输出日志如下：



```
// 单元测试，打印下日志，可以忽略
2019-12-13 17:40:57.130  INFO 74326 --- [           main] c.i.s.l.r.producer.Demo13ProducerTest    : [testSyncSendReturn][发送编号：[1576230057] 发送成功]

// RabbitConnectionFactory 线程，Producer 确认收到 RabbitMQ Broker 对该消息的退回 。
2019-12-13 17:41:02.817 ERROR 74326 --- [nectionFactory1] c.i.s.l.r.c.RabbitProducerReturnCallback : [returnedMessage][message: [(Body:'[B@4689be61(byte[187])' MessageProperties [headers={}, contentType=application/x-java-serialized-object, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, deliveryTag=0])] replyCode: [312] replyText: [NO_ROUTE] exchange: [EXCHANGE_DEMO_13] routingKey: []]

// RabbitConnectionFactory 线程，Producer 确认收到 RabbitMQ Broker 对该消息的成功“响应” 。
// 注意，即使存在 RabbitMQ Broker 回退消息的情况，依然会收到对该消息的成功“响应”
2019-12-13 17:41:02.819  INFO 74326 --- [nectionFactory1] .i.s.l.r.c.RabbitProducerConfirmCallback : [confirm][Confirm 成功 correlationData: null]
```



- 符合预期~整个过程，好好理解艿艿在日志上，添加的过程注释噢。

# 15. RPC 远程调用

> 示例代码对应仓库：[lab-04-rabbitmq-demo-rpc](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc) 。

在 RabbitMQ 中，我们可以通过 [Direct Reply-to](https://www.rabbitmq.com/direct-reply-to.html) 特性，实现 RPC 远程调用的功能。具体的实现原理，胖友自己看[《RabbitMQ 之 RPC 实现》](http://www.iocoder.cn/RabbitMQ/RPC-implementation/?self)文章，这里艿艿就不赘述了。

下面，我们来搭建一个 RPC 的示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-rpc](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc) 项目。

## 15.1 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/pom.xml) 文件。

## 15.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo14ProducerTest.java) 文件。

## 15.3 Demo14Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo14Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo14Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 15.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，添加 Direct Exchange 示例相关的 Exchange、Queue、Binding 的配置。代码如下：



```
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo01Queue() {
            return new Queue(Demo14Message.QUEUE, // Queue 名字
                    false, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo01Exchange() {
            return new DirectExchange(Demo14Message.EXCHANGE,
                    false,  // durable: 是否持久化
                    false);  // autoDelete: 是否自动删除
        }

        // 创建 Binding
        // Exchange：Demo01Message.EXCHANGE
        // Routing key：Demo01Message.ROUTING_KEY
        // Queue：Demo01Message.QUEUE
        @Bean
        public Binding demo01Binding() {
            return BindingBuilder.bind(demo01Queue()).to(demo01Exchange()).with(Demo14Message.ROUTING_KEY);
        }

    }

}
```



- 不同于[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的配置，我们设置队列里的消息无需持久化，毕竟 RPC 是个瞬态操作。

## 15.5 Demo14Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo14Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo14Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现 RPC 操作。代码如下：



```
// Demo14Producer.java

@Component
public class Demo14Producer {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public String syncSend(Integer id) {
        // 创建 Demo01Message 消息
        Demo14Message message = new Demo14Message();
        message.setId(id);
        // <1> 创建 CorrelationData 对象
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        // <2> 同步发送消息，并接收结果
        return (String) rabbitTemplate.convertSendAndReceive(Demo14Message.EXCHANGE, Demo14Message.ROUTING_KEY, message,
                correlationData);
    }

}
```



- `<1>` 处，创建 CorrelationData 对象，使用 UUID 作为唯一标识。
- `<2>` 处，调用 `RabbitTemplate#convertSendAndReceive(exchange, routingKey, message, correlationData)` 方法，Producer 发送消息，并等待结果。该结果，是 Consumer 消费消息，返回的结果。

## 15.6 Demo14Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/) 包下，创建 [Demo14Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo14Consumer.java) 类，消费消息。代码如下：



```
// Demo14Consumer.java

@Component
@RabbitListener(queues = Demo14Message.QUEUE)
public class Demo14Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public String onMessage(Demo14Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // 返回结果
        return "nicai";
    }

}
```



- 消费完成后，额外返回了 `"nicai"` 字符串。

## 15.7 简单测试

创建 [Demo14ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-rpc/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo14ProducerTest.java) 测试类，编写单元测试方法，调用 Demo14Producer 的 RPC 的方法。代码如下：



```
// Demo14ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo14ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo14Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        String result = producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功 消费结果：[{}]]", id, result);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行 `#testSyncSend()` 单元测试，输出日志如下：



```
# Demo14Consumer 成功消费发送的消息
2019-12-13 19:13:36.627  INFO 93696 --- [ntContainer#4-1] c.i.s.l.r.consumer.Demo14Consumer        : [onMessage][线程编号:25 消息内容：Demo14Message{id=1576235616}]

# Producer 打印发送消息的消费结果
2019-12-13 19:13:36.630  INFO 93696 --- [           main] c.i.s.l.r.producer.Demo14ProducerTest    : [testSyncSend][发送编号：[1576235616] 发送成功 消费结果：[nicai]]
```



- 符合预期~整个过程，好好理解艿艿在日志上，添加的过程注释噢。

😈 通过 RabbitMQ 来实现 RPC 的功能，看起来是比较酷炫的。不过艿艿暂时没有想到实际的使用场景，有了解的胖友，麻烦告知下艿艿噢，谢谢。

# 16. MessageConverter

> 示例代码对应仓库：[lab-04-rabbitmq-demo-json](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json) 。

在 Spring-AMQP 中，通过 [MessageConverter](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/support/converter/MessageConverter.java) 来作为消息转换器：

- 在 Producer 中，将 Java POJO 转换成 AMQP [Message](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/Message.java) 。
- 在 Consumer 中，将 AMQP [Message](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/Message.java) 转换成 Java POJO 。

默认情况下，RabbitTemplate 采用 [SimpleMessageConverter](https://github.com/spring-projects/spring-framework/blob/master/spring-messaging/src/main/java/org/springframework/messaging/converter/SimpleMessageConverter.java) 。而 SimpleMessageConverter 内部，采用 Java **自带**序列化方式，实现对 Java POJO 对象的序列化和反序列化，所以官方目前不是很推荐。主要缺点如下：

- 无法跨语言
- 序列化后的字节数组太大
- 序列化性能太低

因此一般情况下，我们建议采用 [Jackson2JsonMessageConverter](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/support/converter/Jackson2JsonMessageConverter.java) ，使用 **JSON** 实现对 Java POJO 对象的序列化和反序列化。

下面，我们来搭建一个 Jackson2JsonMessageConverter 的使用示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-json](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json) 项目。

## 16.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/pom.xml) 文件中，引入相关依赖。



```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-04-rabbitmq-demo-json</artifactId>

    <dependencies>
        <!-- 实现对 RabbitMQ 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <!-- Jackson 依赖  -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.9.10.1</version>
        </dependency>

        <!-- 方便等会写单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```



- 相比[「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)来说，额外引入 `jackson-databind` 依赖。

## 16.2 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/resources/https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/resources/application.yaml) 文件。

## 16.3 Demo15Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message) 包下，创建 [Demo15Message](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo15Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 16.4 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，额外添加创建 Jackson2JsonMessageConverter Bean 。代码如下：



```
// RabbitConfig.java

@Configuration
public class RabbitConfig {

    /**
     * Direct Exchange 示例的配置类
     */
    public static class DirectExchangeDemoConfiguration {

        // 创建 Queue
        @Bean
        public Queue demo15Queue() {
            return new Queue(Demo15Message.QUEUE, // Queue 名字
                    true, // durable: 是否持久化
                    false, // exclusive: 是否排它
                    false); // autoDelete: 是否自动删除
        }

        // 创建 Direct Exchange
        @Bean
        public DirectExchange demo15Exchange() {
            return new DirectExchange(Demo15Message.EXCHANGE,
                    true,  // durable: 是否持久化
                    false);  // exclusive: 是否排它
        }

        // 创建 Binding
        // Exchange：Demo15Message.EXCHANGE
        // Routing key：Demo15Message.ROUTING_KEY
        // Queue：Demo15Message.QUEUE
        @Bean
        public Binding demo15Binding() {
            return BindingBuilder.bind(demo15Queue()).to(demo15Exchange()).with(Demo15Message.ROUTING_KEY);
        }

    }

    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

}
```



- 在 `#messageConverter()` 方法，创建 Jackson2JsonMessageConverter Bean 对象。后续，[RabbitAutoConfiguration.RabbitTemplateConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitAutoConfiguration.java) 在创建 RabbitTemplate Bean 时，会自动注入它。

## 16.5 Demo15Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer) 包下，创建 [Demo15Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo15Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。

和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、RoutingKey 名字不同。

对于胖友来说，可能最关心的是，消息 Message 是怎么序列化的。

- 在序列化时，我们使用了 Jackson2JsonMessageConverter 序列化 Message 消息对象，它会在 RabbitMQ 消息 [MessageProperties](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/MessageProperties.java) 的 `__TypeId__` 上，值为 Message 消息对应的**类全名**。
- 在反序列化时，我们使用了 Jackson2JsonMessageConverter 序列化出 Message 消息对象，它会根据 RabbitMQ 消息 [MessageProperties](https://github.com/spring-projects/spring-amqp/blob/master/spring-amqp/src/main/java/org/springframework/amqp/core/MessageProperties.java) 的 `__TypeId__` 的值，反序列化消息内容成该 Message 对象。

## 16.6 Demo15Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer) 包下，创建 [Demo15Consumer](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo15Consumer.java) 类，消费消息。代码如下：



```
// Demo15Consumer.java

@Component
@RabbitListener(queues = Demo15Message.QUEUE)
public class Demo15Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler(isDefault = true)
    public void onMessage(Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(),
                new String(message.getBody()));
    }

}
```



- 因为我们希望通过查看具体消息内容，判断是不是真的使用 JSON 格式，所以采用 AMQP Message 接收消息。

## 16.7 简单测试

创建 [Demo15ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-json/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo15ProducerTest.java) 测试类，编写单元测试方法，调用 Demo15Producer 发送消息的方法。代码如下：



```
// Demo15ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo15ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo15Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行 `#testSyncSend()` 单元测试，输出日志如下：



```
// Producer 同步发送 1 条消息成功
2019-12-13 20:38:37.311  INFO 4285 --- [           main] c.i.s.l.r.producer.Demo15ProducerTest    : [testSyncSend][发送编号：[1576240717] 发送成功]

// Demo15Consumer 消费 1 条消息成功
// 从消息内容中，我们可以看到 JSON 格式，说明配置生效，嘿嘿。
2019-12-13 20:38:37.335  INFO 4285 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo15Consumer        : [onMessage][线程编号:17 消息内容：{"id":1576240717}]
```



- 符合预期~

# 17. 消费异常处理器

> 示例代码对应仓库：[lab-04-rabbitmq-demo-error-handler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler) 。

在[「7. 消费重试」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)中，我们一起看了下，在 Consumer 消费异常时，Spring-AMQP 提供的**消费重试**机制。除此之外，在 Spring-AMQP 中可以自定义消费异常时的处理器。目前有两个接口，可以实现对 Consumer 消费异常的处理：

- [`org.springframework.amqp.rabbit.listener.api.RabbitListenerErrorHandler`](https://github.com/spring-projects/spring-amqp/blob/master/spring-rabbit/src/main/java/org/springframework/amqp/rabbit/listener/api/RabbitListenerErrorHandler.java) 接口
- [`org.springframework.util.ErrorHandler`](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/util/ErrorHandler.java) 接口

下面，我们来搭建一个 RabbitListenerErrorHandler 和 ErrorHandler 的使用示例。考虑到不污染上述的示例，我们新建一个 [lab-04-rabbitmq-demo-error-handler](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler) 项目。

## 17.1 RabbitListenerErrorHandlerImpl

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.core`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core) 包下，创建 [RabbitListenerErrorHandlerImpl](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core/RabbitListenerErrorHandlerImpl.java) 类，实现 RabbitListenerErrorHandler 接口。代码如下：



```
// RabbitListenerErrorHandler.java

@Component("rabbitListenerErrorHandler")
public class RabbitListenerErrorHandlerImpl implements RabbitListenerErrorHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public Object handleError(Message amqpMessage, org.springframework.messaging.Message<?> message,
                              ListenerExecutionFailedException exception) {
        // 打印异常日志
        logger.error("[handleError][amqpMessage:[{}] message:[{}]]", amqpMessage, message, exception);

        // 直接继续抛出异常
        throw exception;
    }

}
```



- 在类上，添加 `@Component` 注解，并设置其 Bean 名为 `"rabbitListenerErrorHandler"` 。稍后，我们会使用到该 Bean 名字。
- 在 `#handleError(...)` 方法中，我们先打印异常日志，并继续抛出 ListenerExecutionFailedException 异常。**要注意**，如果此时我们不继续抛出异常，而是 `return` 结果，意味着 Consumer 消息成功。如果我们结合[「7. 消费重试」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一起使用的时候，一定要继续抛出该异常，否则消费重试机制将失效。

## 17.2 RabbitLoggingErrorHandler

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.core`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core) 包下，创建 [RabbitLoggingErrorHandler](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/core/RabbitLoggingErrorHandler.java) 类，实现 ErrorHandler 接口。代码如下：



```
// RabbitLoggingErrorHandler.java

@Component
public class RabbitLoggingErrorHandler implements ErrorHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    public RabbitLoggingErrorHandler(SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory) {
        rabbitListenerContainerFactory.setErrorHandler(this);
    }

    @Override
    public void handleError(Throwable t) {
        logger.error("[handleError][发生异常]]", t);
    }

}
```



- 在构造方法中，把自己设置到 SimpleRabbitListenerContainerFactory 中，作为其 ErrorHandler 异常处理器。
- 在 `#handleError(...)` 方法中，打印错误日志。当然，具体怎么处理，胖友可以根据自己的需要哈。

在执行**顺序**上，RabbitListenerErrorHandler **先**于 ErrorHandler 执行。不过这个需要建立在一个前提上，RabbitListenerErrorHandler 需要继续抛出异常。

另外，RabbitListenerErrorHandler 需要每个 `@RabbitListener` 注解上，需要每个手动设置下 `errorHandler` 属性。而 ErrorHandler 是相对全局的，所有 SimpleRabbitListenerContainerFactory 创建的 SimpleMessageListenerContainer 都会生效。

具体选择 ErrorHandler 还是 RabbitLoggingErrorHandler ，艿艿暂时没有答案。不过个人感觉，如果不需要对 Consumer 消费的结果（包括成功和异常）做进一步处理，还是考虑 ErrorHandler 即可。在 ErrorHandler 中，我们可以通过判断 Throwable 异常是不是 ListenerExecutionFailedException 异常，从而拿到 Message 相关的信息。

## 17.3 引入依赖

和 [「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/pom.xml) 文件。

## 17.4 应用配置文件

和 [「3.1.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#) 一致，见 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/resources/application.yaml) 文件。

## 17.5 Demo16Message

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.message`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/) 包下，创建 [Demo16Message](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/message/Demo16Message.java) 消息类，提供给当前示例使用。

和[「3.1.4 Demo01Message」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 17.6 RabbitConfig

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.config`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/) 包下，创建 [RabbitConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/config/RabbitConfig.java) 配置类，配置相关的 Exchange、Queue、Binding 。

和[「3.1.5 RabbitConfig」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、Queue、RoutingKey 名字不同。

## 17.7 Demo16Producer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.producer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/) 包下，创建 [Demo16Producer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo16Producer.java) 类，它会使用 Spring-AMQP 封装提供的 RabbitTemplate ，实现发送消息。

和[「3.1.6 Demo01Producer」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)一致，只是 Exchange、RoutingKey 名字不同。

## 17.8 Demo16Consumer

在 [`cn.iocoder.springboot.lab04.rabbitmqdemo.consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/) 包下，创建 [Demo16Consumer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/main/java/cn/iocoder/springboot/lab04/rabbitmqdemo/consumer/Demo16Consumer.java) 类，消费消息。代码如下：



```
// Demo16Consumer.java

@Component
@RabbitListener(queues = Demo16Message.QUEUE,
    errorHandler = "rabbitListenerErrorHandler")
public class Demo16Consumer {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @RabbitHandler
    public void onMessage(Demo16Message message) {
        logger.info("[onMessage][线程编号:{} 消息内容：{}]", Thread.currentThread().getId(), message);
        // 模拟消费异常
        throw new RuntimeException("你猜");
    }

}
```



- 在 `@RabbitListener` 注解上，我们通过设置 `errorHandler` 属性为[「17.1 RabbitListenerErrorHandlerImpl」](https://www.iocoder.cn/Spring-Boot/RabbitMQ/?github#)的名字。
- 在 `#onMessage(...)` 方法中，我们通过抛出 RuntimeException 异常，模拟消费异常。

## 17.9 简单测试

创建 [Demo16ProducerTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-04-rabbitmq/lab-04-rabbitmq-demo-error-handler/src/test/java/cn/iocoder/springboot/lab04/rabbitmqdemo/producer/Demo16ProducerTest.java) 测试类，编写单元测试方法，调用 Demo16Producer 发送消息的方法。代码如下：



```
// Demo16ProducerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class Demo16ProducerTest {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private Demo16Producer producer;

    @Test
    public void testSyncSend() throws InterruptedException {
        int id = (int) (System.currentTimeMillis() / 1000);
        producer.syncSend(id);
        logger.info("[testSyncSend][发送编号：[{}] 发送成功]", id);

        // 阻塞等待，保证消费
        new CountDownLatch(1).await();
    }

}
```



执行 `#testSyncSend()` 单元测试，输出日志如下：



```
// Producer 同步发送 1 条消息成功
2019-12-13 22:35:54.459  INFO 22515 --- [           main] c.i.s.l.r.producer.Demo16ProducerTest    : [testSyncSend][发送编号：[1576247754] 发送成功]

// Demo08Consumer 消费 1 条消息成功
2019-12-13 22:35:57.254  INFO 22515 --- [ntContainer#0-1] c.i.s.l.r.consumer.Demo16Consumer        : [onMessage][线程编号:18 消息内容：Demo16Message{id=1576247754}]

// RabbitListenerErrorHandler 先处理异常
2019-12-13 22:35:57.263 ERROR 22515 --- [ntContainer#0-1] i.s.l.r.c.RabbitListenerErrorHandlerImpl : [handleError][amqpMessage:[(Body:'[B@401abc48(byte[187])' MessageProperties [headers={}, contentType=application/x-java-serialized-object, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=EXCHANGE_DEMO_16, receivedRoutingKey=ROUTING_KEY_16, deliveryTag=1, consumerTag=amq.ctag-tUY94gzefw73JvHtgqwMnQ, consumerQueue=QUEUE_DEMO_16])] message:[GenericMessage [payload=Demo16Message{id=1576247754}, headers={amqp_receivedDeliveryMode=PERSISTENT, amqp_receivedRoutingKey=ROUTING_KEY_16, amqp_receivedExchange=EXCHANGE_DEMO_16, amqp_deliveryTag=1, amqp_consumerQueue=QUEUE_DEMO_16, amqp_channel=Cached Rabbit Channel: AMQChannel(amqp://guest@127.0.0.1:5672/,1), conn: Proxy@10bd9df0 Shared Rabbit Connection: SimpleConnection@68217d41 [delegate=amqp://guest@127.0.0.1:5672/, localPort= 56809], amqp_redelivered=false, id=34327625-9ef4-0433-3514-a6633bfad100, amqp_consumerTag=amq.ctag-tUY94gzefw73JvHtgqwMnQ, amqp_lastInBatch=false, contentType=application/x-java-serialized-object, timestamp=1576247757255}]]]

org.springframework.amqp.rabbit.support.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab04.rabbitmqdemo.consumer.Demo16Consumer.onMessage(cn.iocoder.springboot.lab04.rabbitmqdemo.message.Demo16Message)' threw exception
// ... 省略异常堆栈

// ErrorHandler 再处理器异常
2019-12-13 22:36:00.175 ERROR 22515 --- [ntContainer#0-1] c.i.s.l.r.c.RabbitLoggingErrorHandler    : [handleError][发生异常]]

org.springframework.amqp.rabbit.support.ListenerExecutionFailedException: Listener method 'public void cn.iocoder.springboot.lab04.rabbitmqdemo.consumer.Demo16Consumer.onMessage(cn.iocoder.springboot.lab04.rabbitmqdemo.message.Demo16Message)' threw exception
// ... 省略异常堆栈
```



- 符合预期~

# 666. 彩蛋

Spring-AMQP 涉及到的内容非常多，艿艿在本文只覆盖了相对常用的部分，所以胖友可以在有需要的时候，看看 [《Spring AMQP 官方文档》](https://docs.spring.io/spring-amqp/docs/current/reference/html/) 。😈 不过即使只是常用的部分，= = 貌似这也是艿艿有史以来，写过最最最最最长的文章了。

因为艿艿个人在生产环境下，主要是使用 RocketMQ 作为消息队列。如果有写的不正确的地方，辛苦胖友帮忙指正。这里额外在推荐一些 RabbitMQ 不错的内容：

- [《RabbitMQ 实战指南》](https://item.jd.com/12277834.html)

  > 感谢厮大（本书作者）在艿艿写本文时，各种智障的问题的指导。
  >
  > 所以，本文有任何内容的错误，都是厮大教的不对。

- [《RabbitMQ 最佳实践》](https://www.cnblogs.com/davenkin/p/rabbitmq-best-practices.html)

最后弱弱的说一下，还是 RocketMQ 更加好用，哈哈哈哈~
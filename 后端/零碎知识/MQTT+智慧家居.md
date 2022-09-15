## 一、什么是 MQTT协议？

`MQTT` 全称(Message Queue Telemetry Transport)：一种基于发布/订阅（`publish`/`subscribe`）模式的`轻量级`通讯协议，通过订阅相应的主题来获取消息，是物联网（`Internet of Thing`）中的一个标准传输协议。

该协议将消息的发布者（`publisher`）与订阅者（`subscriber`）进行分离，因此可以在不可靠的网络环境中，为远程连接的设备提供可靠的消息服务，使用方式与传统的MQ有点类似。![图片](E:\Development\Typora\images\640.png)

`TCP`协议位于传输层，`MQTT` 协议位于应用层，`MQTT` 协议构建于`TCP/IP`协议上，也就是说只要支持`TCP/IP`协议栈的地方，都可以使用`MQTT`协议。

## 二、为什么要用 MQTT协议？

`MQTT`协议为什么在物联网（IOT）中如此受偏爱？而不是其它协议，比如我们更为熟悉的 `HTTP`协议呢？

- 首先`HTTP`协议它是一种同步协议，客户端请求后需要等待服务器的响应。而在物联网（IOT）环境中，设备会很受制于环境的影响，比如带宽低、网络延迟高、网络通信不稳定等，显然异步消息协议更为适合`IOT`应用程序。
- `HTTP`是单向的，如果要获取消息客户端必须发起连接，而在物联网（IOT）应用程序中，设备或传感器往往都是客户端，这意味着它们无法被动地接收来自网络的命令。
- 通常需要将一条命令或者消息，发送到网络上的所有设备上。`HTTP`要实现这样的功能不但很困难，而且成本极高。

## 三、MQTT协议介绍

前边说过`MQTT`是一种轻量级的协议，它只专注于发消息， 所以此协议的结构也非常简单。

### MQTT数据包

在`MQTT`协议中，一个`MQTT`数据包由：`固定头`（Fixed header）、 `可变头`（Variable header）、 `消息体`（payload）三部分构成。

- 固定头（Fixed header），所有数据包中都有固定头，包含数据包类型及数据包的分组标识。
- 可变头（Variable header），部分数据包类型中有可变头。
- 内容消息体（Payload），存在于部分数据包类，是客户端收到的具体消息内容。

![图片](E:\Development\Typora\images\640-16631646667831.png)在这里插入图片描述

#### 1、固定头

固定头部，使用两个字节，共16位：![图片](E:\Development\Typora\images\640-16631646667832.png)(4-7)位表示消息类型，使用4位二进制表示，可代表如下的16种消息类型，不过 0 和 15位置属于保留待用，所以共14种消息事件类型。![图片](E:\Development\Typora\images\640-16631646667833.png)

**DUP Flag（重试标识）**

DUP Flag：保证消息可靠传输，消息是否已送达的标识。默认为0，只占用一个字节，表示第一次发送，当值为1时，表示当前消息先前已经被传送过。

**QoS Level（消息质量等级）**

QoS Level：消息的质量等级，后边会详细介绍

**RETAIN（持久化）**

- 值为`1`：表示发送的消息需要一直持久保存，而且不受服务器重启影响，不但要发送给当前的订阅者，且以后新加入的客户端订阅了此`Topic`，订阅者也会马上得到推送。**注意**：新加入的订阅者，只会取出最新的一个`RETAIN flag = 1`的消息推送。
- 值为`0`：仅为当前订阅者推送此消息。

**Remaining Length（剩余长度）**

在当前消息中剩余的`byte`(字节)数，包含可变头部和消息体payload。

#### 2、可变头

固定头部仅定义了消息类型和一些标志位，一些消息的元数据需要放入可变头部中。可变头部内容字节长度 + 消息体payload = 剩余长度。

可变头部居于固定头部和payload中间，包含了协议名称，版本号，连接标志，用户授权，心跳时间等内容。

可变头存在于这些类型的消息：PUBLISH (QoS > 0)、PUBACK、PUBREC、PUBREL、PUBCOMP、SUBSCRIBE、SUBACK、UNSUBSCRIBE、UNSUBACK。

#### 3、消息体payload

消息体payload只存在于`CONNECT`、`PUBLISH`、`SUBSCRIBE`、`SUBACK`、`UNSUBSCRIBE`这几种类型的消息：

- `CONNECT`：包含客户端的`ClientId`、订阅的`Topic`、`Message`以及`用户名`和`密码`。
- `PUBLISH`：向对应主题发送消息。
- `SUBSCRIBE`：要订阅的主题以及`QoS`。
- `SUBACK`：服务器对于`SUBSCRIBE`所申请的主题及`QoS`进行确认和回复。
- `UNSUBSCRIBE`：取消要订阅的主题。

### 消息质量（QoS ）

`消息质量`（Quality of Service），即消息的发送质量，发布者（`publisher`）和订阅者（`subscriber`）都可以指定`qos`等级，有`QoS 0`、`QoS 1`、`QoS 2`三个等级。

下边分别说明一下这三个等级的区别。

#### 1、Qos 0

Qos 0：`At most once`（至多一次）只发送一次消息，不保证消息是否成功送达，没有确认机制，消息可能会丢失或重复。

![图片](E:\Development\Typora\images\640-16631646667834.jpeg)图片源于网络，如有侵权联系删除

#### 2、Qos 1

Qos 1：`At least once`（至少一次），相对于`QoS 0`而言`Qos 1`增加了`ack`确认机制，发送者（`publisher`）推送消息到MQTT代理（`broker`）时，两者自身都会先持久化消息，只有当`publisher` 或者 `Broker`分别收到 `PUBACK`确认时，才会删除自身持久化的消息，否则就会重发。

但有个问题，尽管我们可以通过确认来保证一定收到客户端 或 服务器的`message`，可我们却不能保证仅收到一次`message`，也就是当客户端`publisher`没收到`Broker`的`puback`或者 `Broker`没有收到`subscriber`的`puback`，那么就会一直重发。

**publisher -> broker 大致流程：**

1. publisher store msg -> publish ->broker （传递message）
2. broker -> puback -> publisher delete msg （确认传递成功）

![图片](E:\Development\Typora\images\640-16631646667835.jpeg)图片源于网络，如有侵权联系删除

#### 3、Qos 2

Qos 2：`Exactly once`（只有一次），相对于`QoS 1`，`QoS 2`升级实现了仅接受一次`message`，`publisher` 和 `broker` 同样对消息进行持久化，其中 `publisher` 缓存了`message`和 对应的`msgID`，而 `broker` 缓存了 `msgID`，可以保证消息不重复，由于又增加了一个`confirm` 机制，整个流程变得复杂很多。

**publisher -> broker 大致流程：**

1. publisher store msg -> publish ->broker -> broker store
2. msgID（传递message） broker -> puberc （确认传递成功）
3. publisher -> pubrel ->broker delete msgID （告诉broker删除msgID）
4. broker -> pubcomp -> publisher delete msg （告诉publisher删除msg）

![图片](E:\Development\Typora\images\640-16631646667836.jpeg)图片源于网络，如有侵权联系删除

### LWT（最后遗嘱）

`LWT` 全称为 `Last Will and Testament`，其实遗嘱是一个由客户端预先定义好的主题和对应消息，附加在`CONNECT`的数据包中，包括`遗愿主题`、`遗愿 QoS`、`遗愿消息`等。

当MQTT代理 `Broker` 检测到有客户端`client`非正常断开连接时，再由服务器主动发布此消息，然后相关的订阅者会收到消息。

**举个栗子**：聊天室中所有人都订阅一个叫`talk`的主题 ，但小富由于网络抖动突然断开了链接，这时聊天室中所有订阅主题 `talk`的客户端都会收到一个 “`小富离开聊天室`” 的遗愿消息。

遗嘱的相关参数：

- `Will Flag`：是否使用 LWT，1 开启
- `Will Topic`：遗愿主题名，不可使用通配符
- `Will Qos`：发布遗愿消息时使用的 QoS
- `Will Retain`：遗愿消息的 Retain 标识
- `Will Message`：遗愿消息内容

**那客户端`Client` 有哪些场景是非正常断开连接呢？**

- `Broker` 检测到底层的 I/O 异常；
- 客户端 未能在心跳 `Keep Alive` 的间隔内和 `Broker` 进行消息交互；
- 客户端 在关闭底层 `TCP` 连接前没有发送 `DISCONNECT` 数据包；
- 客户端 发送错误格式的数据包到 `Broker`，导致关闭和客户端的连接等。

**注意**：当客户端通过发布 `DISCONNECT` 数据包断开连接时，属于正常断开连接，并不会触发 `LWT` 的机制，与此同时`Broker` 还会丢弃掉当前客户端在连接时指定的相关 `LWT` 参数。

## 四、MQTT协议应用场景

`MQTT`协议广泛应用于物联网、移动互联网、智能硬件、车联网、电力能源等领域。使用的场景也是非常非常多，下边列举一些：

- 物联网M2M通信，物联网大数据采集
- Android消息推送，WEB消息推送
- 移动即时消息，例如Facebook Messenger
- 智能硬件、智能家具、智能电器
- 车联网通信，电动车站桩采集
- 智慧城市、远程医疗、远程教育
- 电力、石油与能源等行业市场

## 五、代码实现

注意rabbitmq环境搭建的时候需要开启mqtt 

![图片](E:\Development\Typora\images\640-16631646667847.jpeg)在这里插入图片描述

### 1、启用 rabbitmq的mqtt协议

我们先开启 `rabbitmq` 的 `mqtt`协议，因为默认安装下是关闭的，命令如下：

```bash
rabbitmq-plugins enable rabbitmq_mqtt
```

### 2、mqtt 客户端依赖包 

上一步中安装`rabbitmq`环境并开启 `mqtt`协议后，实际上`mqtt` 消息代理服务就搭建好了，接下来要做的就是实现客户端消息的推送和订阅。

这里使用`spring-integration-mqtt`、`org.eclipse.paho.client.mqttv3`两个工具包实现。

```xml
<!--mqtt依赖包-->
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-mqtt</artifactId>
</dependency>
<dependency>
    <groupId>org.eclipse.paho</groupId>
       <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.0</version>
</dependency>
```

### 3、消息发送者

消息的发送比较简单，主要是应用到`@ServiceActivator`注解，需要注意`messageHandler.setAsync`属性，如果设置成`false`，关闭异步模式发送消息时可能会阻塞。

```java
@Configuration
public class IotMqttProducerConfig {

    @Autowired
    private MqttConfig mqttConfig;

    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setServerURIs(mqttConfig.getServers());
        return factory;
    }

    @Bean
    public MessageChannel mqttOutboundChannel() {
        return new DirectChannel();
    }

    @Bean
    @ServiceActivator(inputChannel = "iotMqttInputChannel")
    public MessageHandler mqttOutbound() {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(mqttConfig.getServerClientId(), mqttClientFactory());
        messageHandler.setAsync(false);
        messageHandler.setDefaultTopic(mqttConfig.getDefaultTopic());
        return messageHandler;
    }
}
```

`MQTT` 对外提供发送消息的`API`时，需要使用`@MessagingGateway` 注解，去提供一个消息网关代理，参数`defaultRequestChannel` 指定发送消息绑定的`channel`。

可以实现三种`API`接口，`payload` 为发送的消息，`topic` 发送消息的主题，`qos` 消息质量。

```java
@MessagingGateway(defaultRequestChannel = "iotMqttInputChannel")
public interface IotMqttGateway {

    // 向默认的 topic 发送消息
    void sendMessage2Mqtt(String payload);
    // 向指定的 topic 发送消息
    void sendMessage2Mqtt(String payload,@Header(MqttHeaders.TOPIC) String topic);
    // 向指定的 topic 发送消息，并指定服务质量参数
    void sendMessage2Mqtt(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) int qos, String payload);
}
```



### 4、消息订阅

消息订阅和我们平时用的MQ消息监听实现思路基本相似，`@ServiceActivator`注解表明当前方法用于处理`MQTT`消息，`inputChannel` 参数指定了用于接收消息的`channel`。

```java
/**
 * @Author: xiaofu
 * @Description: 消息订阅配置
 * @date 2020/6/8 18:24
 */
@Configuration
public class IotMqttSubscriberConfig {

    @Autowired
    private MqttConfig mqttConfig;

    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        factory.setServerURIs(mqttConfig.getServers());
        return factory;
    }

    @Bean
    public MessageChannel iotMqttInputChannel() {
        return new DirectChannel();
    }

    @Bean
    public MessageProducer inbound() {
        MqttPahoMessageDrivenChannelAdapter adapter = new MqttPahoMessageDrivenChannelAdapter(mqttConfig.getClientId(), mqttClientFactory(), mqttConfig.getDefaultTopic());
        adapter.setCompletionTimeout(5000);
        adapter.setConverter(new DefaultPahoMessageConverter());
        adapter.setQos(1);
        adapter.setOutputChannel(iotMqttInputChannel());
        return adapter;
    }

    /**
     * @author xiaofu
     * @description 消息订阅
     * @date 2020/6/8 18:20
     */
    @Bean
    @ServiceActivator(inputChannel = "iotMqttInputChannel")
    public MessageHandler handlerTest() {

        return message -> {
            try {
                String string = message.getPayload().toString();
                System.out.println("接收到消息：" + string);
            } catch (MessagingException ex) {
                //logger.info(ex.getMessage());
            }
        };
    }
}
```

### application

```yml
iot:
    mqtt:
        clientId: client-1 # Client ID
        defaultTopic: push_message_topic # 默认topic
        serverClientId: server-1 # server-1
        servers: tcp://106.53.124.182:1883 # MQTT服务器地址
```

### controller

```java
@CrossOrigin("*")
@Controller
@RequestMapping("mqtt")
public class MqttController {

    @Autowired
    private IotMqttGateway mqttGateway;

    @RequestMapping("/index")
    public String index() {
        return "mqtt";
    }

    @RequestMapping("/sendMessage")
    @ResponseBody
    public String sendMqtt(@RequestParam(value = "topic") String topic, @RequestParam(value = "message") String message) {
        mqttGateway.sendMessage2Mqtt(message, topic);
        return "SUCCESS";
    }


}
```

### 前端

```js
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" lang="zh-CN">
<head>
    <title>未读消息</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.js" type="text/javascript"></script>
    <script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js" type="text/javascript"></script>
    <script src="/vue.min.js" type="text/javascript"></script>
    <script src="/js/index.js"></script>
    <link rel="stylesheet" href="/style.css" media="screen" type="text/css"/>
    <link rel="stylesheet" type="text/css" href="push.css"/>

    <script type="text/javascript">
        // mqtt协议rabbitmq服务
        //var brokerIp = location.hostname;
        var brokerIp = "106.53.124.182";
        // mqtt协议端口号
        var port = 15675;
        // 接受推送消息的主题
        var topic = "push_message_topic";

        // mqtt连接
        client = new Paho.MQTT.Client(brokerIp, port, "/ws", "clientId_" + parseInt(Math.random() * 100, 10));

        var options = {
            timeout: 3, //超时时间
            keepAliveInterval: 30,//心跳时间
            onSuccess: function () {
                console.log(("连接成功~"));
                client.subscribe(topic, {qos: 1});
            },
            onFailure: function (message) {
                console.log(("连接失败~" + message.errorMessage));
            }
        };
        // 考虑到https的情况
        if (location.protocol == "https:") {
            options.useSSL = true;
        }
        client.connect(options);
        console.log(("已经连接到" + brokerIp + ":" + port));

        // 连接断开事件
        client.onConnectionLost = function (responseObject) {
            console.log("失去连接 - " + responseObject.errorMessage);
        };

        // 接收消息事件
        client.onMessageArrived = function (message) {
            console.log("接受主题： " + message.destinationName + "的消息： " + message.payloadString);
            $("#arrivedDiv").append("<br/>"+message.payloadString);
            var count = $("#count").text();
            count = Number(count) + 1;
            $("#count").text(count);
        };

        // 推送给指定主题
        function sendMessage() {
            var a = $("#message").val();
            if (client.isConnected()) {
                var message = new Paho.MQTT.Message(a);
                message.destinationName = topic;
                client.send(message);
            }
        }


    </script>
</head>


<body>
<div>
    <ul>
        <li class="active"><i class="fa fa-home fa-lg"></i> 未读消息 <span id="count" class="unread">0</span></li>

    </ul>
    <div style="margin-left: 800px;">
        <input style="height: 25px; width: 180px;" maxlength="60" value="" id="message"/>
        <button class="button" id="mySendBtn" onclick="sendMessage()"> 点击发送</button>
        <div style="font-size: 20px;color: darkcyan"> 接收到的mqtt消息</div>
        <hr/>
        <div id="arrivedDiv" style="height:200px; width:300px; overflow:scroll; background:#EEEEEE;">
            <br/>
        </div>
    </div>
</div>
</body>

</html>
```



## 六、测试消息

额~ 由于本渣渣对硬件一窍不通，为了模拟硬件的发送消息，只能借助一下工具，其实硬件端实现`MQTT`协议，跟我们前边的基本没什么区别，只不过换种语言嵌入到硬件中而已。

这里选的测试工具为`mqttbox`，下载地址：http://workswithweb.com/mqttbox.html

### 1、测试消息发送

我们用先用`mqttbox`模拟向主题`mqtt_test_topic`发送消息，看后台是否能成功接收到。

![图片](E:\Development\Typora\images\640-16631646667848.png)看到后台成功拿到了向主题`mqtt_test_topic`发送的消息。![图片](E:\Development\Typora\images\640-16631646667849.png)

### 2、测试消息订阅

订阅我的有问题………..

用`mqttbox`模拟订阅主题`mqtt_test_topic`，在后台向主题`mqtt_test_topic`发送一条消息，这里我简单的写了个`controller`调用API发送消息。

http://127.0.0.1:8080/fun/testMqtt?topic=mqtt_test_topic&message=我是后台向主题 mqtt_test_topic 发送的消息![图片](E:\Development\Typora\images\640-166316466678410.png)我们看`mqttbox`的订阅消息，已经成功的接收到了后台的消息，到此我们的`MQTT`通信环境就算搭建成功了。如果把`mqttbox`工具换成具体硬件设备，整个流程就是我们常说的智能家居了，其实真的没那么难。![图片](E:\Development\Typora\images\640-166316466678411.png)

## 七、应用注意事项

在我们实际的生产环境中遇到过的问题，这里分享一下让大家少踩坑。

### clientId 要唯一

在客户端`connect`连接的时，会有一个`clientId` 参数，需要每个客户端都保持唯一的。但我们在开发测试阶段`clientId`直接在代码中写死了，而且服务都是单实例部署，并没有暴露出什么问题。

```java
MqttPahoMessageDrivenChannelAdapter(mqttConfig.getClientId(), mqttClientFactory(), mqttConfig.getDefaultTopic());
```

然而在生产环境内侧的时候，由于服务是多实例集群部署，结果出现了下边的奇怪问题。同一时间内只能有一个客户端能拿到消息，其他客户端不但不能消费消息，而且还在不断的掉线重连：`Lost connection: 已断开连接; retrying...`。

![图片](E:\Development\Typora\images\640-166316466678412.png)这就是由于`clientId`相同导致客户端间相互竞争消费，最后将`clientId`获取方式换成从发号器中拿，问题就好了，所以这个地方是需要特别注意的。

平时程序在开发环境没问题，可偏偏到了生产环境就一大堆问题，很多都是因为服务部署方式不同导致的。所以多学习分布式还是很有必要的。

## 八、其他中间件

`MQTT`它只是一种协议，支持`MQTT`协议的消息中间件产品非常多，下边的也只是其中的一部分

- Mosquitto
- Eclipse Paho
- RabbitMQ
- Apache ActiveMQ
- HiveMQ
- JoramMQ
- ThingMQ
- VerneMQ
- Apache Apollo
- emqttd Xively
- IBM Websphere .....

## 总结

我也是第一次做和硬件相关的项目，之前听到智能家居都会觉得好高大上，但实际上手开发后发现，技术嘛万变不离其宗，也只是换种用法而已。

双手奉上项目 demo 的`github`地址 ：https://github.com/chengxy-nds/springboot-rabbitmq-mqtt.git,感兴趣的小伙伴可以下载跑一跑，实现起来非常的简单。
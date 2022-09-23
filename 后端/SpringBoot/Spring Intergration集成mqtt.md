## 关于`Spring Intergration`

**介绍关于Spring Integration 的一些概念性的东西，如果只想知道怎么集成可跳过该部分**

> Spring Integration provides an extension of the Spring programming model to support the well known [Enterprise Integration Patterns](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.enterpriseintegrationpatterns.com%2F). It enables lightweight messaging within Spring-based applications and supports integration with external systems through declarative adapters. Those adapters provide a higher level of abstraction over Spring’s support for remoting, messaging, and scheduling.

大致意思是

- Spring Integration 为许多著名的企业级集成方案提供了扩展接口
- 它能够在基于Sping的应用中进行轻量级消息传递
- 支持声明式适配器与发布系统进行集成（见Mqtt send消息的实现）
- 这些适配器在 Spring对远程调用，消息传递和系统调度提供了更高层次的抽象

### Spring Intergration核心组件

#### Message（消息）

Message是对消息的包装，在Spring 系统中传递的任何消息都会被包装为Message。可以理解为是Spring Integration消息传递的基本单位

#### Message Channel（消息管道）

消息管道：Message 在Message Channel中进行传递，生产者向管道中投递消息，消费者从管道中取出消息。Spring Integration 支持两种消息传递模型，point-to-point（点对点模型），Publish-subscribe（发布订阅模型）有多种管道类型。

本次使用点对点模型，消息管道类型`DirectChannel`

#### Message Endpoint（消息切入点）

消息切入点：消息在管道中流动那必定会有某些流入或流出的点亦或是在某个位置（即特定函数）需要对消息进行处理，过滤，格式转换等。这些点即为Message Endpoint（实际为某些处理函数），例如消息发送，消息接收都是Message Endpoint。

##### Message Transformer（消息转化器）

消息转化器：是将消息进行特定转换例如将一个 Object 序列化为 Json 字符串

##### Message Filter

消息过滤器，过滤掉特定消息。例如在管道中发送的含username 和 age 属性的 User 对象，如果当前消息（一个User实例的包装）的age < 18则将其过滤掉，那么处在过滤器之后的消费者将无法接收到 age < 18的User对象。当然过滤条件不仅是消息负载的属性，也可以是消息本身的属性。

##### Message Router（消息路由）

消息路由：向管道投递消息时可由消息路由根据路由规则选择投递给那个管道

##### Splitter（分割器）

分割器：它从一个输入管道接收一条消息并将其分割为多条消息，再将每条消息发送到不同的输出管道上

##### Aggregator（聚合器）

聚合器：与分割器功能刚好相反

##### Service Activator

Service Activator: 它是一个用于将系统服务实例接入到消息系统的泛型切入点，该切入点必须配置输入管道。其返回值可是消息类型也可以是一个消息处理器，当返回值为消息类型时需要指定输出管道，即在该切入点对消息加工处理后再发送到指定的输出管道，如果返回值为消息处理器。那么消息交由消息处理器进行处理。下文中会为Mqtt消息出站配置Service Activator并且 返回值为消息处理器

##### Channel Adapter（管道适配器）

管道适配器：因为外部协议有无数种，消息适配器则用于连接不同协议的外部系统。从外部系统读入数据并对数据进行处理最终与Spring Integration 内部的消息系统适配。例如将要进行Mqtt集成，那么就需要一个Mqtt的管道适配器，事实上也确实有一个，下文中将会看到。

## 开始集成

> 依赖管理工具使用Gardle

### 引入spring-integration-mqtt依赖



```groovy
implementation "org.springframework.integration:spring-integration-mqtt:5.4.6"
```

### 创建Mqtt配置类



```java
@Configuration
public class MqttConfig {
    /**
     *  以下属性将在配置文件中读取
     **/
    //mqtt Broker 地址
    private String[] uris;
    //连接用户名
    private String username;
    //连接密码
    private String password;
    //入站Client ID
    private String inClientId;
    //出站Client ID
    private String outClientId;
    //默认订阅主题
    private String defaultTopic;

    public void setUris(String[] uris) {
        this.uris = uris;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setInClientId(String inClientId) {
        this.inClientId = inClientId;
    }

    public void setOutClientId(String outClientId) {
        this.outClientId = outClientId;
    }

    public void setDefaultTopic(String defaultTopic) {
        this.defaultTopic = defaultTopic;
    }
}
```

这里需要注意为什么创建两个client ID,Spring Integration 在集成的时候入站与出站消息处理并不使用同一个连接，所以如果clien ID相同，将会出现Mqtt反复重连现象，实为 mqtt 出入站连接交替踢对方下线。

### 修改配置文件 `application.yml`



```yaml
mqtt:
  uris: tcp://ip:port
  username: user
  password: pwd
  in-client-id: ${random.value} # 随机值，使出入站 client ID 不同
  out-client-id: ${random.value}
  default-topic: defaultTopic
```

在MqttConfig上使用注解 `@ConfigurationProperties(prefix = "mqtt")`将配置文件中属性注入到MqttConfig中,但别忘记在启动类上使用`@EnableConfigurationProperties`启用属性注入。

```java
@Configuration
@ConfigurationProperties(prefix = "mqtt")
public class MqttConfig {
    .....
}
```

### 创建三个管道

这三个管道分别用于处理入站消息，出站消息，错误消息



```java
@Bean
public MessageChannel mqttOutboundChannel(){ //出站
    return new DirectChannel();
}
@Bean
public MessageChannel mqttInboundChannel(){ //入站
    return new DirectChannel();
}
@Bean
public MessageChannel errorChannel(){ //错误
    return new DirectChannel();
}
```

### 添加 Mqtt 客户端工厂



```java
@Bean
public MqttPahoClientFactory mqttPahoClientFactory(){
    DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
    MqttConnectOptions options = new MqttConnectOptions();
    options.setServerURIs(uris);
    options.setUserName(username);
    options.setPassword(password.toCharArray());
    factory.setConnectionOptions(options);
    return factory;
}
```

### 添加 Mqtt 管道适配器



```java
@Bean
public MqttPahoMessageDrivenChannelAdapter adapter(MqttPahoClientFactory factory){
    return new MqttPahoMessageDrivenChannelAdapter(inClientId,factory,defaultTopic);
}
```

### 添加消息生产者



```java
@Bean
public MessageProducer mqttInbound(MqttPahoMessageDrivenChannelAdapter adapter){
    adapter.setCompletionTimeout(5000);
    adapter.setConverter(new DefaultPahoMessageConverter());
    //入站投递给入站管道
    adapter.setOutputChannel(mqttInboundChannel());
    adapter.setErrorChannel(errorChannel());
    adapter.setQos(0);
    return adapter;
}
```

### 添加出站处理器

出站处理器是一个`Service Activator`

```java
@Bean
@ServiceActivator(inputChannel = "mqttOutboundChannel")
public MessageHandler mqttOutbound(MqttPahoClientFactory factory){
    MqttPahoMessageHandler handler =
            new MqttPahoMessageHandler(outClientId,factory);
    handler.setAsync(true);
    handler.setConverter(new DefaultPahoMessageConverter());
    handler.setDefaultTopic(defaultTopic);
    return handler;
}
```

### 添加消息接收器

通过前文的配置，当Mqtt 订阅主题产生消息时会通过 `MessageProducer`（本例中是一个管道适配器）将消息投递到入站管道中，所以当需要接收并处理Mqtt消息时只需要从入站管道中取出消息即可。取出消息即可使用前文的 `Endpoint` 本例使用`Service Activator`

创建一个接收类，自定义任意类型。重点是使用其内部的方法，将其注册为`Endpoint`



```java
@Component
public class Receiver {
    @Bean
    //使用ServiceActivator 指定接收消息的管道为 mqttInboundChannel，投递到mqttInboundChannel管道中的消息会被该方法接收并执行
    @ServiceActivator(inputChannel = "mqttInboundChannel")
    public MessageHandler handleMessage() {
        return message -> {
            System.out.println(message.getPayload());
        };
    }
}
```

至此，整个服务即可接收Mqtt broker的消息了。默认接收的主题为配置文件中指定的 "defaultTopic"。

### 添加消息发送器

那么消息发送器无疑是向出站管道投递消息即可。如何实现，Spring Integration 提供了 @MessagingGateway注解，该注解提供一个`defaultRequestChannel`属性用于**指定出站管道**。如下

```java
@Component
@MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
public interface MqttSender {
    void sendToMqtt(String text);

    void sendWithTopic(@Header(MqttHeaders.TOPIC) String topic, String text);

    void sendWithTopicAndQos(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) Integer Qos, String text);
}
```

此接口内函数参数将作为消息的负载被包装成为消息并投递到出站管道中。同时可以看到，方法参数可以注解的方式自定义消息发送的 topic qos retain 等属性。

至此集成完成，用户可调用MqttSender发布消息或是冲Receiver中接收并处理消息

## 自定义编解码器

前文中添加消息生产者和添加出站处理器代码中都可以看到`setConverter(new DefaultPahoMessageConverter());`方法，该方法用与对消息负载进行编解码。多数情况下我们的消息都是可编码的。我在消息传递过程中使用的是json编码。下面我将以json编码为例，展示如何自定义编解码器



```java
//协议对象
public class User {
    private String username;
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

### 实现自己的编解码器

代码如下，详见注释



```java
@Component
public class AlMingConverter implements MqttMessageConverter {
    private final  static Logger log = LoggerFactory.getLogger(AlMingConverter.class);
    private int defaultQos = 0;
    private boolean defaultRetain = false;
    ObjectMapper om = new ObjectMapper();
    //入站消息解码
    @Override
    public Message<User> toMessage(String topic, MqttMessage mqttMessage) {
        
        User protocol = null;
        try {
            protocol = om.readValue(mqttMessage.getPayload(), User.class);
        } catch (IOException e) {
            if (e instanceof JsonProcessingException) {
                System.out.println();
                log.error("Converter only support json string");
            }
        }
        assert protocol != null;
        MessageBuilder<User> messageBuilder = MessageBuilder
                .withPayload(protocol);
        //使用withPayload初始化的消息缺少头信息，将原消息头信息填充进去
        messageBuilder.setHeader(MqttHeaders.ID, mqttMessage.getId())
                .setHeader(MqttHeaders.RECEIVED_QOS, mqttMessage.getQos())
                .setHeader(MqttHeaders.DUPLICATE, mqttMessage.isDuplicate())
                .setHeader(MqttHeaders.RECEIVED_RETAINED, mqttMessage.isRetained());
        if (topic != null) {
            messageBuilder.setHeader(MqttHeaders.TOPIC, topic);
        }
        return messageBuilder.build();
    }
    //出站消息编码
    @Override
    public Object fromMessage(Message<?> message, Class<?> targetClass) {
        MqttMessage mqttMessage = new MqttMessage();
        String msg = null;
        try {
            msg = om.writeValueAsString(message.getPayload());
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        assert msg != null;
        mqttMessage.setPayload(msg.getBytes(StandardCharsets.UTF_8));
        //这里的 mqtt_qos ,和 mqtt_retained 由 MqttHeaders 此类得出不可以随便取，如需其他属性自行查找
        Integer qos = (Integer) message.getHeaders().get("mqtt_qos");
        mqttMessage.setQos(qos == null ? defaultQos : qos);
        Boolean retained = (Boolean) message.getHeaders().get("mqtt_retained");
        mqttMessage.setRetained(retained == null ? defaultRetain : retained);
        return mqttMessage;
    }
    //此方法直接拿默认编码器的来用的，照抄即可
    @Override
    public Message<?> toMessage(Object payload, MessageHeaders headers) {
        Assert.isInstanceOf(MqttMessage.class, payload,
                () -> "This converter can only convert an 'MqttMessage'; received: " + payload.getClass().getName());
        return this.toMessage(null, (MqttMessage) payload);
    }

    public void setDefaultQos(int defaultQos) {
        this.defaultQos = defaultQos;
    }

    public void setDefaultRetain(boolean defaultRetain) {
        this.defaultRetain = defaultRetain;
    }
}
```

### 修改MqttConfig

将自定义编解码器注入到MqttConfig中。



```java
private final AlMingConverter alMingConverter;
@Autowired
public MqttConfig(AlMingConverter alMingConverter) {
    this.alMingConverter = alMingConverter;
}
```

将原来所有`DefaultPahoMessageConverter`实例更换为`alMingConverter`



```java
...
adapter.setConverter(alMingConverter);
...
handler.setConverter(alMingConverter);
```

### 修改MqttSender



```java
@Component
@MessagingGateway(defaultRequestChannel = "mqttOutboundChannel")
public interface MqttSender {
    void sendToMqtt(User user);

    void sendWithTopic(@Header(MqttHeaders.TOPIC) String topic, User user);

    void sendWithTopicAndQos(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) Integer Qos, User user);
}
```

### Receiver修改

此时Reveiver的消息负载虽然编译时为 Object ,但运行时为User可安全的进行 Object 到 User 的强转然后进行后续操作



```java
@Component
public class Receiver {
    @Bean
    @ServiceActivator(inputChannel = "mqttInboundChannel")
    public MessageHandler handleMessage() {
        return message -> {
            User user = (User) message.getPayload();
            System.out.println(user.getUsername()+":"+user.getPassword());
        };
    }
}
```

## 动态修改订阅主题

前文中发布订阅的主题都是默认主题，即在配置文件中指定的主题。生产环境肯定不止使用一个主题。那么就需要运行时动态修改主题。Spring Integration也确实为此提供了支持。上文中向Bean容器中注册的管道适配器（MqttPahoMessageDrivenChannelAdapter）提供了`addTopic` `removeTopic`等方法可用于运行时修改主题。所以可以创建一个MqttService专门用于添加和删除主题。

**但注意只有Spring Integration 4.1以上版本可以这么使用**



```java
@Service
public class MqttServiceImpl implements MqttService { //MqttService是自己定义的，仅包含如下方法均已重写
    MqttPahoMessageDrivenChannelAdapter adapter;
    @Autowired
    public MqttServiceImpl(MqttPahoMessageDrivenChannelAdapter adapter) {
        this.adapter = adapter;
    }

    @Override
    public void addTopic(String topic) {
        String[] topics = adapter.getTopic();
        if(!Arrays.asList(topics).contains(topic)){
            adapter.addTopic(topic,0);
        }
    }

    @Override
    public void removeTopic(String topic) {
        adapter.removeTopic(topic);
    }
}
```

增加或删除主题时，注入该服务调用addTopic，removeTopic即可

## End&附录

### 最终MqttConfig完整代码



```java
@Configuration
@ConfigurationProperties(prefix = "mqtt")
public class MqttConfig {
    /**
     *  以下属性将在配置文件中读取
     **/
    //mqtt Broker 地址
    private String[] uris;
    //连接用户名
    private String username;
    //连接密码
    private String password;
    //入站Client ID
    private String inClientId;
    //出站Client ID
    private String outClientId;
    //默认订阅主题
    private String defaultTopic;

    private final AlMingConverter alMingConverter;
    @Autowired
    public MqttConfig(AlMingConverter alMingConverter) {
        this.alMingConverter = alMingConverter;
    }

    @Bean
    public MqttPahoClientFactory mqttPahoClientFactory(){
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        MqttConnectOptions options = new MqttConnectOptions();
        options.setServerURIs(uris);
        options.setUserName(username);
        options.setPassword(password.toCharArray());
        factory.setConnectionOptions(options);
        return factory;
    }
    @Bean
    public MqttPahoMessageDrivenChannelAdapter adapter(MqttPahoClientFactory factory){
        return new MqttPahoMessageDrivenChannelAdapter(inClientId,factory,defaultTopic);
    }
    public void setUris(String[] uris) {
        this.uris = uris;
    }
    @Bean
    public MessageProducer mqttInbound(MqttPahoMessageDrivenChannelAdapter adapter){
        adapter.setCompletionTimeout(5000);
        adapter.setConverter(alMingConverter);
        //入站投递的通道
        adapter.setOutputChannel(mqttInboundChannel());
        adapter.setErrorChannel(errorChannel());
        adapter.setQos(0);
        return adapter;
    }
    @Bean
    @ServiceActivator(inputChannel = "mqttOutboundChannel")
    public MessageHandler mqttOutbound(MqttPahoClientFactory factory){
        MqttPahoMessageHandler handler =
                new MqttPahoMessageHandler(outClientId,factory);
        handler.setAsync(true);
        handler.setConverter(alMingConverter);
        handler.setDefaultTopic(defaultTopic);
        return handler;
    }
    @Bean
    public MessageChannel mqttOutboundChannel(){
        return new DirectChannel();
    }
    @Bean
    public MessageChannel mqttInboundChannel(){
        return new DirectChannel();
    }
    @Bean
    public MessageChannel errorChannel(){
        return new DirectChannel();
    }
    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public void setInClientId(String inClientId) {
        this.inClientId = inClientId;
    }

    public void setOutClientId(String outClientId) {
        this.outClientId = outClientId;
    }

    public void setDefaultTopic(String defaultTopic) {
        this.defaultTopic = defaultTopic;
    }
}
```
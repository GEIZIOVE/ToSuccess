# Spring Cloud Alibaba 配置中心 Nacos 入门

# 1. 概述

本文我们来学习 [Spring Cloud Alibaba](https://spring.io/projects/spring-cloud-alibaba) 提供的 [Spring Cloud Alibaba Nacos Config](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-nacos-config) 组件，基于 Spring Cloud 的编程模型，接入 Nacos 作为配置中心，实现服务的统一配置管理。

> FROM [《Spring Cloud Alibaba 官方文档 —— Nacos Config》](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config#spring-cloud-alibaba-nacos-config)
>
> Nacos 提供用于存储配置和其他元数据的 key/value 存储，为分布式系统中的外部化配置提供服务器端和客户端支持。使用 Spring Cloud Alibaba Nacos Config，您可以在 Nacos Server 集中管理你 Spring Cloud 应用的外部属性配置。
>
> Spring Cloud Alibaba Nacos Config 是 Config Server 和 Client 的替代方案，客户端和服务器上的概念与 Spring Environment 和 PropertySource 有着一致的抽象，在特殊的 bootstrap 阶段，配置被加载到 Spring 环境中。当应用程序通过部署管道从开发到测试再到生产时，您可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容。

在开始本文之前，胖友需要对 Nacos 进行简单的学习。可以阅读[《Nacos 极简入门》](http://www.iocoder.cn/Nacos/install/?self)文章，将第一二小节看完，在本机搭建一个 Nacos 服务。

# 2. 快速入门

> 示例代码对应仓库：[`labx-05-sca-nacos-config-demo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo)。

本小节，我们会在 Nacos 服务中定义配置，并使用 [`@ConfigurationProperties`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/context/properties/ConfigurationProperties.java) 和 [`@Value`](https://github.com/spring-projects/spring-framework/blob/master/spring-beans/src/main/java/org/springframework/beans/factory/annotation/Value.java) 注解，读取该配置。

> 友情提示：如果胖友看过[《芋道 Spring Boot 配置文件入门》](http://www.iocoder.cn/Spring-Boot/config-file/?self)的[「2. 自定义配置」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)小节，就会发现本小节是对标这块的内容。
>
> 如果没看过，也没关系，艿艿只是提一下而已，嘿嘿。继续往下看即可。

下面，我们来创建一个 [`labx-05-sca-nacos-config-demo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo) 示例项目，进行快速入门。最终项目代码如下图所示：![ 项目](E:\Development\Typora\images\03-16635041008887.png)

## 2.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo/pom.xml) 文件中，主要引入 Spring Cloud Nacos Config 相关依赖。代码如下：



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>labx-05</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>labx-05-sca-nacos-config-demo</artifactId>

    <properties>
        <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR1</spring.cloud.version>
        <spring.cloud.alibaba.version>2.2.0.RELEASE</spring.cloud.alibaba.version>
    </properties>

    <!--
        引入 Spring Boot、Spring Cloud、Spring Cloud Alibaba 三者 BOM 文件，进行依赖版本的管理，防止不兼容。
        在 https://dwz.cn/mcLIfNKt 文章中，Spring Cloud Alibaba 开发团队推荐了三者的依赖关系
     -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- 引入 SpringMVC 相关依赖，并实现对其的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 引入 Spring Cloud Alibaba Nacos Config 相关依赖，将 Nacos 作为配置中心，并实现对其的自动配置 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>

</project>
```



通过引入 [`spring-cloud-starter-alibaba-nacos-config`](https://mvnrepository.com/artifact/com.alibaba.cloud/spring-cloud-starter-alibaba-nacos-config) 依赖，引入并实现 Nacos Config 的自动配置。

## 2.2 配置文件

创建 [`bootstrap.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo/src/main/resources/bootstrap.yaml) 配置文件，添加 Nacos Config 相关配置。配置如下：

注意是bootstrap哦

```yml
spring:
  application:
    name: demo-application

  cloud:
    nacos:
      # Nacos Config 配置项，对应 NacosConfigProperties 配置属性类
      config:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: # 使用的 Nacos 的命名空间，默认为 null
        group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
        name: # 使用的 Nacos 配置集的 dataId，默认为 spring.application.name
        file-extension: yaml # 使用的 Nacos 配置集的 dataId 的文件拓展名，同时也是 Nacos 配置集的配置格式，默认为 properties
```



Nacos Config 配置项，以 `spring.cloud.nacos.config` 开头，对应 [NacosConfigProperties](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/java/com/alibaba/cloud/nacos/NacosConfigProperties.java) 配置属性类。

① `server-addr` 配置项，设置 Nacos 服务器地址。

② `namespace` 配置项，使用的 Nacos 的命名空间，默认为 `null`，表示使用 `public` 这个默认命名空间。

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **命名空间**
> 用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

③ `group` 配置项，使用的 Nacos 配置分组，默认为 `DEFAULT_GROUP`。

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **配置分组**
> Nacos 中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串（如 Buy 或 Trade ）对配置集进行分组，从而区分 Data ID 相同的配置集。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 `DEFAULT_GROUP` 。配置分组的常见场景：不同的应用或组件使用了相同的配置类型，如 `database_url` 配置和 `MQ_topic` 配置。

④ `name` 配置项，使用的 Nacos 配置集的 dataId，默认为 `spring.application.name`。

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **配置集**
> 一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件通常就是一个配置集，包含了系统各个方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。
>
> **配置集 ID**
> Nacos 中的某个配置集的 ID。配置集 ID 是组织划分配置的维度之一。Data ID 通常用于组织划分系统的配置集。一个系统或者应用可以包含多个配置集，每个配置集都可以被一个有意义的名称标识。Data ID 通常采用类 Java 包（如 `com.taobao.tc.refund.log.level`）的命名规则保证全局唯一性。此命名规则非强制。

因为这里我们未进行配置，所以使用 Nacos 配置集的 dataId 为 `demo-application`。这也是为什么我们将 `spring.application.name` 配置项添加到 `bootstrap.yaml` 配置文件中的原因。

⑤ `file-extension` 配置项，使用的 Nacos 配置集的 dataId 的**文件拓展名**，同时也是 Nacos 配置集的**配置格式**，默认为 `properties`。这里我们设置为 `yaml`，因为我们稍后使用的配置集的配置格式为 `YAML`。

可能胖友对“文件拓展名”的作用有点不太理解，我们来看一段 `NacosPropertySourceLocator` 的代码，会特别好理解。代码如下：



```java
// NacosPropertySourceLocator.java

private void loadApplicationConfiguration(
		CompositePropertySource compositePropertySource, String dataIdPrefix,
		NacosConfigProperties properties, Environment environment) {
	String fileExtension = properties.getFileExtension();
	String nacosGroup = properties.getGroup();
	// load directly once by default
	loadNacosDataIfPresent(compositePropertySource, dataIdPrefix, nacosGroup,
			fileExtension, true);
	// load with suffix, which have a higher priority than the default
	loadNacosDataIfPresent(compositePropertySource,
			dataIdPrefix + DOT + fileExtension, nacosGroup, fileExtension, true);
	// Loaded with profile, which have a higher priority than the suffix
	for (String profile : environment.getActiveProfiles()) {
		String dataId = dataIdPrefix + SEP1 + profile + DOT + fileExtension;
		loadNacosDataIfPresent(compositePropertySource, dataId, nacosGroup,
				fileExtension, true);
	}
}
```



代码的意思，按照 Nacos 配置集的 dataId 为 `{dataIdPrefix}`、`{dataIdPrefix}.{fileExtension}`、`${dataIdPrefix}-{profile}.{fileExtension}` 的三种情况下，分别从 Nacos 中加载对应的配置集。同时要注意，**优先级是反过来**的，即优先级为 `{dataIdPrefix}-{profile}.{fileExtension} > {dataIdPrefix}.{fileExtension} > {dataIdPrefix}`。

按照我们在 `bootstrap.yaml` 配置文件中的配置，会加载的 Nacos 配置集的 dataId 为 `demo-application` 和 `demo-application.yaml`，并且优先级是 `demo-application.yaml > demo-application`。

------

> 友情提示：可能会有胖友好奇，为什么要将 Nacos Config 的配置项添加到 `bootstrap.yaml` 配置文件，而不像我们之前的文章一样，添加到 `application.yaml` 配置文件中呢？
>
> 下面，我们来讲解下原因，可能有一点点难度，可以阅读一遍，选择性理解噢。

在 Spring Cloud 应用中，会先创建一个 **Bootstrap** Context（**引导**上下文），比 Spring Boot 创建 **Application** Context（**应用**上下文）**更早初始化**。

Bootstrap Context 新增了一个 `bootstrap.yaml` **配置文件**，保证和 Application Context 的 `application.yaml` **配置文件**的**隔离**。

有了配置文件的隔离之后，Bootstrap Context 初始化的 **Bean** 从哪里来？Spring Cloud 新定义了专属于 Bootstrap Context 的自动化配置类的拓展点 **BootstrapConfiguration**，和 Spring Boot 为 Application Context 的自动化配置类的拓展点 **EnableAutoConfiguration**的**隔离**，保证两个 Context 创建各自的 Bean。以 Spring Cloud Alibaba Nacos 的 [`spring.factories`](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/resources/META-INF/spring.factories) 举例子，如下图所示：![ 示例](E:\Development\Typora\images\01-16635041008889.png)

虽然说，Bootstrap Context 和 Application Context 做了这么多隔离，但是它们有一点是共享的，那就是 **Environment**。在 Spring 中，我们通过 Environment 获取属性配置，例如说 `spring.application.name` 对应的值是多少。

了解完这些之后，我们把它们串联在一起去思考一下，Bootstrap Context 的**目的**究竟是什么呢？通过 Bootstrap Context 的优先初始化，**将配置加载到 Environment 中**，提供给后面的 Application Context 使用。

举个贼重要的例子，稍后我们会在 `bootstrap.yaml` 添加 Spring Cloud Alibaba Nacos Config 相关的配置，这样 Bootstrap Context 在初始化时，通过 NacosConfigBootstrapConfiguration 创建 Nacos 相关的 Bean，然后实现从 Nacos 配置中心加载配置到 Environment 中。

如果我们把 Spring Cloud Alibaba Nacos Config 相关的配置添加在 `application.yaml` 中，那么可能无法保证 Nacos 相关的 Bean 被**最先**初始化，完成从 Nacos 获取配置，从而影响创建的 Bean。

> 友情提示：实际上将 Spring Cloud Alibaba Nacos Config 相关的配置添加在 `application.yaml` 中，也有办法解决，需要基于 Spring Boot 的 ApplicationContextInitializer 和 EnvironmentPostProcessor 拓展点，实现自定义的处理。
>
> 感兴趣的胖友，可以去看看 Apollo 或者 Nacos Config Spring Boot Starter 的源码。

😝 上面涉及的内容点有点多，不理解的胖友可以在看看如下两个资料：

- [《芋道 Spring Boot 自动配置原理》](http://www.iocoder.cn/Spring-Boot/autoconfigure/?self)
- [《Spring Cloud 中文文档 —— Spring Cloud Context》](https://www.docs4dev.com/docs/zh/spring-cloud/Greenwich.RELEASE/reference/multi__spring_cloud_context_application_context_services.html#_the_bootstrap_application_context)

## 2.3 创建 Nacos 配置集

① 打开 Nacos UI 界面的「配置列表」菜单，进入「配置管理」功能。如下图所示：![配置管理](E:\Development\Typora\images\01-16635041008791.png)

② 点击列表右上角的➕号，进入「新建配置」界面，创建一个 Nacos 配置集。输入如下内容，并点击「发布」按钮，完成创建。如下图所示：![新建配置](E:\Development\Typora\images\02-166350410088812.png)

其中，YAML 配置文件如下：



```yml
order:
  pay-timeout-seconds: 120 # 订单支付超时时长，单位：秒。
  create-frequency-seconds: 60 # 订单创建频率，单位：秒
```



## 2.4 OrderProperties

创建 [OrderProperties](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/config/OrderProperties.java) 配置类，读取 `order` 配置项。代码如下：



```java
@Component
@ConfigurationProperties(prefix = "order")
// @NacosConfigurationProperties(prefix = "order", dataId = "${nacos.config.data-id}", type = ConfigType.YAML)
public class OrderProperties {

    /**
     * 订单支付超时时长，单位：秒。
     */
    private Integer payTimeoutSeconds;

    /**
     * 订单创建频率，单位：秒
     */
    private Integer createFrequencySeconds;

    // ... 省略 setter/getter 方法

}
```



- 在类上，添加 `@Component` 注解，保证该配置类可以作为一个 Bean 被扫描到。
- 在类上，添加 `@ConfigurationProperties` 注解，并设置 `prefix = "order"` 属性，这样它就可以读取**前缀**为 `order` 配置项，设置到配置类对应的属性上。

😈 这里，我们注释了一段 [`@NacosConfigurationProperties`](https://github.com/nacos-group/nacos-spring-boot-project/blob/master/nacos-config-spring-boot-autoconfigure/src/main/java/com/alibaba/boot/nacos/config/properties/NacosConfigProperties.java) 注解的代码，该注解在功能上是对标(即可以互相替换)`@ConfigurationProperties` 注解，用于将 Nacos 配置注入 POJO 配置类中。为什么我们这里注释掉了呢？因为我们在[「2.2 配置文件」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)中，添加 Nacos Config 相关配置到 `bootstrap.yaml` 配置文件中，因此 Spring Cloud 应用在启动时，**预加载**了来自 Nacos 配置，所以可以直接使用 `@ConfigurationProperties` 注解即可。这样的好处，是可以更加通用，而无需和 Nacos 有耦合与依赖。

## 2.5 DemoController

创建 [DemoController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/controller/DemoController.java) 类，提供测试 `@ConfigurationProperties` 和 `@Value` 注入配置的两个 HTTP 接口。代码如下：



```java
@RestController
@RequestMapping("/demo")
public class DemoController {

    @Autowired
    private OrderProperties orderProperties;

    /**
     * 测试 @ConfigurationProperties 注解的配置属性类
     */
    @GetMapping("/test01")
    public OrderProperties test01() {
        return orderProperties;
    }

    @Value(value = "${order.pay-timeout-seconds}") // @NacosValue(value = "${order.pay-timeout-seconds}")
    private Integer payTimeoutSeconds;
    @Value(value = "${order.create-frequency-seconds}") // @NacosValue(value = "${order.create-frequency-seconds}")
    private Integer createFrequencySeconds;

    /**
     * 测试 @Value 注解的属性
     */
    @GetMapping("/test02")
    public Map<String, Object> test02() {
        return new JSONObject().fluentPut("payTimeoutSeconds", payTimeoutSeconds)
                .fluentPut("createFrequencySeconds", createFrequencySeconds);
    }

}
```



## 2.6 DemoApplication

创建 [DemoApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/DemoApplication.java) 类，作为应用启动类。代码如下：



```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class);
    }

}
```



## 2.7 简单测试

① 使用 DemoApplication 启动示例应用。在 IDEA 控制台中，可以看到 Nacos 相关的日志如下：



```bash
// 加载 Nacos 配置集 demo-application.yaml 
2020-02-16 20:08:56.955  INFO 13713 --- [           main] c.a.n.client.config.impl.ClientWorker    : [fixed-127.0.0.1_8848] [subscribe] demo-application.yaml+DEFAULT_GROUP
2020-02-16 20:08:56.956  INFO 13713 --- [           main] c.a.nacos.client.config.impl.CacheData   : [fixed-127.0.0.1_8848] [add-listener] ok, tenant=, dataId=demo-application.yaml, group=DEFAULT_GROUP, cnt=1

// 加载 Nacos 配置集 demo-application
2020-02-16 20:08:56.957  INFO 13713 --- [           main] c.a.n.client.config.impl.ClientWorker    : [fixed-127.0.0.1_8848] [subscribe] demo-application+DEFAULT_GROUP
2020-02-16 20:08:56.957  INFO 13713 --- [           main] c.a.nacos.client.config.impl.CacheData   : [fixed-127.0.0.1_8848] [add-listener] ok, tenant=, dataId=demo-application, group=DEFAULT_GROUP, cnt=1
```



② 使用浏览器，访问 http://127.0.0.1:8080/demo/test01 接口，测试 `@ConfigurationProperties` 注解的配置属性类，返回结果如下，符合预期：

```json
{
    "payTimeoutSeconds": 60,
    "createFrequencySeconds": 120
}
```

② 使用浏览器，访问 http://127.0.0.1:8080/demo/test02 接口，测试 `@Value` 注解的属性，返回结果如下，符合预期：

```json
{
    "payTimeoutSeconds": 60,
    "createFrequencySeconds": 120
}
```

# 3. 多环境配置

> 示例代码对应仓库：[labx-05-sca-nacos-config-demo-profiles](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/)。

在[《芋道 Spring Boot 配置文件入门》](http://www.iocoder.cn/Spring-Boot/config-file/?self)的[「6. 多环境配置」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)中，我们介绍如何基于 `spring.profiles.active` 配置项，在 Spring Boot 实现多环境的配置功能。在本小节中，我们会在该基础之上，实现结合 Nacos 的多环境配置。

实际上，Nacos 开发者已经告诉我们如何实现了，通过 Nacos **namespace** 命名空间。文档说明如下：

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **命名空间**
> 用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

如此，我们可以通过给每个环境创建一个 Nacos namespace。然后，在每个 `bootstrap-${profile}.yaml` 配置文件中，配置对应的 Nacos namespace。

下面，我们来创建一个 [`labx-05-sca-nacos-config-demo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo) 示例项目，搭建一个结合 Nacos 的多环境的示例。最终项目代码如下图所示：![ 项目](E:\Development\Typora\images\15-166350410088814.png)

> 首先，让我们进行 Nacos 命名空间和配置集的创建。

## 3.1 创建 Nacos 命名空间

① 打开 Nacos UI 界面的「命名空间」菜单，进入「命名空间」功能。如下图所示：![命名空间](E:\Development\Typora\images\11-166350410088816.png)

② 点击列表右上角的「新建命名空间」按钮，弹出「新建命名空间」窗口，创建一个 `dev` 命名空间。输入如下内容，并点击「确定」按钮，完成创建。如下图所示：![新建命名空间](E:\Development\Typora\images\12-166350410088818.png)

③ 重复该操作，继续创建一个 `prod` 命名空间。最终 `dev` 和 `prod` 信息如下图：![命名空间列表](E:\Development\Typora\images\13-166350410088820.png)

## 3.2 创建 Nacos 配置集

① 打开 Nacos UI 界面的「配置列表」菜单，进入「配置管理」功能。如下图所示：![配置管理](E:\Development\Typora\images\14-166350410088822.png)

我们会发现，红圈部分多了 `dev` 和 `prod` 两个命名空间。

② 点击 `dev` 命名空间，然后点击列表右上角的➕号，进入「新建配置」界面，创建一个 Nacos 配置集。输入如下内容，并点击「发布」按钮，完成创建。如下图所示：![新建配置](E:\Development\Typora\images\11-16635041008802.png)

③ 点击 `prod` 命名空间，然后点击列表右上角的➕号，进入「新建配置」界面，创建一个 Nacos 配置集。输入如下内容，并点击「发布」按钮，完成创建。如下图所示：![新建配置](E:\Development\Typora\images\12-16635041008803.png)

如此，我们在 Nacos 中有 `dev`、`prod` 命名空间。而这两命名空间下，都有一个 `example` 配置集。而这两配置集都有 `server.port` 配置项，用于启动不同端口的服务器。😈 为什么选择 `server.port` 配置呢？因为 Spring Cloud 项目启动后，从日志中就可以看到生效的服务器端口，嘿嘿~

> 然后，让我们进行 Spring Cloud 示例的搭建。

## 3.3 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/pom.xml) 文件中，主要引入 Spring Cloud Nacos Config 相关依赖。代码如下：



```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>labx-05</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>labx-05-sca-nacos-config-demo-profiles</artifactId>

    <properties>
        <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR1</spring.cloud.version>
        <spring.cloud.alibaba.version>2.2.0.RELEASE</spring.cloud.alibaba.version>
    </properties>

    <!--
        引入 Spring Boot、Spring Cloud、Spring Cloud Alibaba 三者 BOM 文件，进行依赖版本的管理，防止不兼容。
        在 https://dwz.cn/mcLIfNKt 文章中，Spring Cloud Alibaba 开发团队推荐了三者的依赖关系
     -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- 引入 SpringMVC 相关依赖，并实现对其的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 引入 Spring Cloud Alibaba Nacos Config 相关依赖，将 Nacos 作为配置中心，并实现对其的自动配置 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>

</project>
```



## 3.4 配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/src/main/resources/) 目录下，创建 2 个配置文件，对应不同的环境。如下：

- [`bootstrap-dev.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/src/main/resources/bootstrap-dev.yaml)，开发环境。

  ```yml
  spring:
    cloud:
      nacos:
        # Nacos Config 配置项，对应 NacosConfigProperties 配置属性类
        config:
          server-addr: 127.0.0.1:8848 # Nacos 服务器地址
          namespace: 14226a0d-799f-424d-8905-162f6a8bf409 # 使用的 Nacos 的命名空间，默认为 null
          group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
          name: # 使用的 Nacos 配置集的 dataId，默认为 spring.application.name
          file-extension: yaml # 使用的 Nacos 配置集的 dataId 的文件拓展名，同时也是 Nacos 配置集的配置格式，默认为 properties
  ```

  

  - 和[「2.2 配置文件」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)不同的点，主要是修改了 `spring.cloud.nacos.config.namespace` 配置项，设置为 `dev` 命名空间的编号。

- [`bootstrap-prod.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/src/main/resources/bootstrap-prod.yaml)，生产环境。

  ```yml
  spring:
    cloud:
      nacos:
        # Nacos Config 配置项，对应 NacosConfigProperties 配置属性类
        config:
          server-addr: 127.0.0.1:8848 # Nacos 服务器地址
          namespace: f1686f3b-a984-4cdf-8298-7caee3455d14 # 使用的 Nacos 的命名空间，默认为 null
          group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
          name: # 使用的 Nacos 配置集的 dataId，默认为 spring.application.name
          file-extension: yaml # 使用的 Nacos 配置集的 dataId 的文件拓展名，同时也是 Nacos 配置集的配置格式，默认为 properties
  ```

  

  - 和[「2.2 配置文件」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)不同的点，主要是修改了 `spring.cloud.nacos.config.namespace` 配置项，设置为 `prod` 命名空间的编号。

另外，我们会创建 [`bootstrap.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/src/main/resources/bootstrap.yaml) 配置文件，放不同环境的**相同配置**。例如说，`spring.application.name` 配置项，肯定是相同的啦。配置如下：



```yml
spring:
  application:
    name: demo-application
```



## 3.5 ProfilesApplication

创建 [ProfilesApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-profiles/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/DemoApplication.java) 类，作为应用启动类。代码如下：



```java
@SpringBootApplication
public class ProfilesApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProfilesApplication.class);
    }

}
```



## 3.6 简单测试

下面，我们使用命令行参数进行 `--spring.profiles.active` 配置项，实现不同环境，读取不同配置文件。

> 经过测试：
>
> - 使用**命令行参数**进行 `--spring.profiles.active` 配置，对 `bootstrap.yaml` 配置文件**无效**。
> - 使用 **VM 参数**进行 `-Dspring.profiles.active` 配置爱，对 `bootstrap.yaml` 配置文件**有效**。
>
> 具体的原因还不知道，先暂时这么解决哈~

① **开发环境**示例：直接在 IDEA 中，增加 `-Dspring.profiles.active=dev` 到 VM options 中。如下图所示：![IDEA 配置 - dev](E:\Development\Typora\images\13-16635041008804.png)

启动 Spring Cloud 应用，输出日志如下：



```bash
# 省略其它日志...

# Tomcat 启动日志
2020-02-16 22:55:58.293  INFO 16734 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8081 (http) with context path ''
```



- Tomcat 启动在 8081 端口，符合读取 `dev` 命名空间的 `demo-application` 配置集。

② **生产环境**示例：直接在 IDEA 中，增加 `-Dspring.profiles.active==prod` 到 VM options 中。如下图所示：![IDEA 配置 - prod](E:\Development\Typora\images\14-16635041008805.png)

启动 Spring Cloud 应用，输出日志如下：



```
# 省略其它日志...
2020-02-16 22:59:17.501  INFO 16803 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8084 (http)
```



- Tomcat 启动在 8084 端口，符合读取 `prod` 命名空间的 `demo-application` 配置集。

另外，关于 Spring Boot 应用的多环境部署，胖友也可以看看[《芋道 Spring Boot 持续交付 Jenkins 入门》](http://www.iocoder.cn/Spring-Boot/Jenkins/?self)文章。

## 3.7 其它方案

除了通过 Nacos **namespace** 命名空间的方案之外，我们还有两种可以实现多环境配置：

① 通过配置集的 **dataId** 来实现。例如说：开发环境使用 Nacos 配置集的 dataId 为 `{applicationName}-dev`，生产环境使用 Nacos 配置集的 `dataId` 为 `{applicationName}-prod`。

因为 Spring Cloud Alibaba Nacos Config 提供的 Profiles 功能，内置就提供在配置的 Nacos 配置集的 `dataId` 拼接 `-{profile}`，所以还是比较方便的。

② 通过配置集的 **group** 分组来实现。例如说：开发环境使用配置集的 group 为 `DEV_GROUP`，生产环境使用配置集的 group 为 `PPROD_GROUP`。

**📚 怎么选？**

相比来说，最最最推荐采用基于 **namespace** 的方案，因为官方推荐，同时未来 Nacos 的权限控制是按照 **namespace** 级别进行控制，可以更好的结合，毕竟不是人人都有生产环境的配置管理的权限。

次之推荐采用基于 **dataId** 的方案，因为 Spring Cloud Alibaba Nacos Config 提供了良好的支持，Spring Cloud Config 也是采用类似方案。

最最不推荐采用 **group** 的方案，毕竟 **group** 的定位，是将相同的应用或组件的配置集进行分组。例如说，**group** 可以有 `DATABASE_GROUP`、`MQ_GROUP`、`RPC_GROUP` 等等。

# 4. 自动刷新配置

> 示例代码对应仓库：[`labx-05-sca-nacos-config-auto-refresh`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-auto-refresh)。

在上面的示例中，我们已经实现从 Nacos 读取配置。那么，在应用已经启动的情况下，如果我们将读取的 Nacos 的配置进行修改时，应用是否会自动刷新本地的配置呢？

如果是我们上面的示例，答案是**部分会**。使用 `@ConfigurationProperties` 注解的**会**，使用 `@Value` 注解的**不会**。

下面，我们从[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)小节的 [`labx-05-sca-nacos-config-demo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo) 项目，复制出 [`labx-05-sca-nacos-config-auto-refresh`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-auto-refresh) 项目，用于演示 Nacos 的**自动刷新配置**的功能。最终项目代码如下图所示：![ 项目](E:\Development\Typora\images\22-166350410088927.png)

## 4.1 简单测试

① 使用 DemoApplication 启动示例应用。

② **获得**目前 Nacos 配置集 `demo-provider` 的配置内容为：



```yml
order:
  pay-timeout-seconds: 60 # 订单支付超时时长，单位：秒。
  create-frequency-seconds: 120 # 订单创建频率，单位：秒
```

使用 `curl` 命令，请求 DemoController 提供的两个测试接口，过程如下：

```bash
# 测试 `@ConfigurationProperties` 注解的配置属性类
$ curl http://127.0.0.1:8080/demo/test01
{"payTimeoutSeconds":60,"createFrequencySeconds":120}

# 测试 `@Value` 注解的属性
$ curl http://127.0.0.1:8080/demo/test02
{"payTimeoutSeconds":60,"createFrequencySeconds":120}
```



③ **修改**配置集 `demo-provider` 的配置内容为：



```yml
order:
  pay-timeout-seconds: 60 # 订单支付超时时长，单位：秒。
  create-frequency-seconds: 480 # 订单创建频率，单位：秒
```



此时，我们可以看到 IDEA 控制台打印出了好多 Nacos 相关的日志，如下图所示：![Nacos 日志](E:\Development\Typora\images\21-166350410088929.png)

使用 `curl` 命令，请求 DemoController 提供的两个测试接口，过程如下：



```bash
# 测试 `@ConfigurationProperties` 注解的配置属性类
$ curl http://127.0.0.1:8080/demo/test01
{"payTimeoutSeconds":60,"createFrequencySeconds":480}

# 测试 `@Value` 注解的属性
$ curl http://127.0.0.1:8080/demo/test02
{"payTimeoutSeconds":60,"createFrequencySeconds":120}
```



- 使用 `@ConfigurationProperties` 注解的**成功**刷新，使用 `@Value` 注解的**失败**刷新。

## 4.2 @RefreshScope

在 Spring Cloud 中，提供了 [`@RefreshScope`](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/scope/refresh/RefreshScope.java) 注解，可以声明在 Bean 上，实现该 Bean 的配置刷新。

> 友情提示：对 `@RefreshScope` 注解的实现原理感兴趣的胖友，可以阅读[《@RefreshScope 那些事》](http://www.iocoder.cn/Fight/RefreshScope-those-things/?self)文章。

下面，我们将 `@RefreshScope` 注解添加在 DemoController 类上，实现 `@Value` 注解的属性的刷新。代码如下：



```java
@RestController
@RequestMapping("/demo")
@RefreshScope // 加我加我加我！
public class DemoController {

    // ... 省略其它代码

}
```



## 4.3 重新测试

① 使用 DemoApplication 启动示例应用。

② **获得**目前 Nacos 配置集 `demo-provider` 的配置内容为：



```yml
order:
  pay-timeout-seconds: 60 # 订单支付超时时长，单位：秒。
  create-frequency-seconds: 480 # 订单创建频率，单位：秒
```



使用 `curl` 命令，请求 DemoController 提供的两个测试接口，过程如下：



```bash
# 测试 `@ConfigurationProperties` 注解的配置属性类
$ curl http://127.0.0.1:8080/demo/test01
{"payTimeoutSeconds":60,"createFrequencySeconds":480}

# 测试 `@Value` 注解的属性
$ curl http://127.0.0.1:8080/demo/test02
{"payTimeoutSeconds":60,"createFrequencySeconds":480}
```



③ **修改**配置集 `demo-provider` 的配置内容为：



```yml
order:
  pay-timeout-seconds: 60 # 订单支付超时时长，单位：秒。
  create-frequency-seconds: 960 # 订单创建频率，单位：秒
```



使用 `curl` 命令，请求 DemoController 提供的两个测试接口，过程如下：



```bash
# 测试 `@ConfigurationProperties` 注解的配置属性类
$ curl http://127.0.0.1:8080/demo/test01
{"payTimeoutSeconds":60,"createFrequencySeconds":960}

# 测试 `@Value` 注解的属性
$ curl http://127.0.0.1:8080/demo/test02
{"payTimeoutSeconds":60,"createFrequencySeconds":960}
```



- 使用 `@ConfigurationProperties` 注解的**成功**刷新，使用 `@Value` 注解的**成功**刷新。完美~

## 4.4 EnvironmentChangeEvent

通过 `@ConfigurationProperties` 或者 `@Value` + `@RefreshScope` 注解，已经能够满足我们绝大多数场景下的自动刷新配置的功能。但是，在一些场景下，我们仍然需要**实现对配置的监听，执行自定义的逻辑**。

例如说，当数据库连接的配置发生变更时，我们需要通过监听该配置的变更，重新初始化应用中的数据库连接，从而访问到新的数据库地址。

又例如说，当日志级别发生变更时，我们需要通过监听该配置的变更，设置应用中的 Logger 的日志级别，从而后续的日志打印可以根据新的日志级别。

在 Spring Cloud 中，在 Environment 的属性配置发生变化时，会发布 [EnvironmentChangeEvent](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/environment/EnvironmentChangeEvent.java) 事件。这样，我们只需要实现 EnvironmentChangeEvent 事件的监听器，就可以进行自定义的逻辑处理。

例如说，Spring Cloud 内置了 [LoggingRebinder](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/logging/LoggingRebinder.java) 监听器，实现了对 EnvironmentChangeEvent 事件的监听，在发现 `logging.level` 配置项发生变化时，重新设置日志组件的日志级别。如下图所示：![LoggingRebinder 源码](E:\Development\Typora\images\51-166350410088935.png)

下面，我们来实现一个自定义的 [DemoEnvironmentChangeListener](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-auto-refresh/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/listener/DemoEnvironmentChangeListener/DemoEnvironmentChangeListener.java) 监听器，打印下变化配置项的最新值。代码如下：



```java
@Component
public class DemoEnvironmentChangeListener implements ApplicationListener<EnvironmentChangeEvent> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    private ConfigurableEnvironment environment;

    @Override
    public void onApplicationEvent(EnvironmentChangeEvent event) {
        for (String key : event.getKeys()) {
            logger.info("[onApplicationEvent][key({}) 最新 value 为 {}]", key, environment.getProperty(key));
        }
    }

}
```



> 补充：最近看了下文档，可以通过 `@RefreshScope` 注解，实现数据源的动态变化，具体可以看看[《SpringCloud 运行时刷新数据源相关配置》](http://www.iocoder.cn/Fight/The-SpringCloud-runtime-refreshes-the-datasource-related-configuration/?self)文章。也就是说，不一定需要通过实现对 EnvironmentChangeEvent 事件的监听处理。

## 4.5 再次测试

① 使用 DemoApplication 启动示例应用。

② **修改**配置集 `demo-provider` 的配置内容，增加 `test` 配置项为 `true`。此时在 IDEA 控制台可以看到 DemoEnvironmentChangeListener 打印的日志如下：



```
2020-02-16 09:58:58.503  INFO 21072 --- [-127.0.0.1_8848] .i.s.l.n.l.DemoEnvironmentChangeListener : [onApplicationEvent][key(test) 最新 value 为 true]
```



# 5. 配置加密

> 示例代码对应仓库：[`labx-05-sca-nacos-config-demo-jasypt`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt)。

考虑到安全性，我们可能最好将配置文件中的敏感信息进行加密。例如说，MySQL 的用户名密码、第三方平台的 Token 令牌等等。不过，Nacos 暂时未内置配置加密的功能。官方文档说明如下：

> FROM https://nacos.io/zh-cn/docs/faq.html
>
> **Nacos如何对配置进行加密**
> Nacos 计划在 1.X 版本提供加密的能力，目前还不支持加密，只能靠 sdk 做好了加密再存到 nacos 中。

因此，我们暂时只能在客户端进行配置的加解密。这里，我们继续采用在[《芋道 Spring Boot 配置文件入门》](http://www.iocoder.cn/Spring-Boot/config-file/?self)的[「8. 配置加密」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)小节中使用的 [Jasypt](https://github.com/jasypt/jasypt)。

下面，我们来使用 Nacos + Jasypt 搭建一个配置加密的示例。最终项目代码如下图所示：![ 项目](E:\Development\Typora\images\34.png)

## 5.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>labx-05</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>labx-05-sca-nacos-config-demo-jasypt</artifactId>

    <properties>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
        <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR1</spring.cloud.version>
        <spring.cloud.alibaba.version>2.2.0.RELEASE</spring.cloud.alibaba.version>
    </properties>

    <!--
        引入 Spring Boot、Spring Cloud、Spring Cloud Alibaba 三者 BOM 文件，进行依赖版本的管理，防止不兼容。
        在 https://dwz.cn/mcLIfNKt 文章中，Spring Cloud Alibaba 开发团队推荐了三者的依赖关系
     -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- 引入 SpringMVC 相关依赖，并实现对其的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 引入 Spring Cloud Alibaba Nacos Config 相关依赖，将 Nacos 作为配置中心，并实现对其的自动配置 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>

        <!-- 实现对 Jasypt 实现自动化配置 -->
        <dependency>
            <groupId>com.github.ulisesbocchio</groupId>
            <artifactId>jasypt-spring-boot-starter</artifactId>
            <version>3.0.2</version>
            <!--            <scope>test</scope>-->
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



引入 [`jasypt-spring-boot-starter`](https://mvnrepository.com/artifact/com.github.ulisesbocchio/jasypt-spring-boot-starter) 依赖，实现对 Jasypt 的自动化配置。

## 5.2 创建 Nacos 配置集

在 Nacos 中，创建一个用于测试配置加密的配置集 `demo-application-jasypt`，具体内容如下图：![创建 Nacos 配置集](E:\Development\Typora\images\31-166350410088932.png)

这里为了测试简便，我们直接添加加密秘钥 `jasypt.encryptor.password` 配置项在该 Nacos 配置集中。如果为了安全性更高，实际建议把加密秘钥和配置隔离。不然，如果配置泄露，岂不是可以拿着加密秘钥，直接进行解密。

## 5.3 配置文件

在 [`bootstrap.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt/src/main/resources/bootstrap.yaml) 中，添加 Nacos Config 配置，如下：



```yml
spring:
  application:
    name: demo-application-jasypt

  cloud:
    nacos:
      # Nacos Config 配置项，对应 NacosConfigProperties 配置属性类
      config:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: # 使用的 Nacos 的命名空间，默认为 null
        group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
        name: # 使用的 Nacos 配置集的 dataId，默认为 spring.application.name
        file-extension: yaml # 使用的 Nacos 配置集的 dataId 的文件拓展名，同时也是 Nacos 配置集的配置格式，默认为 properties
```



- 和[「2.2 配置文件」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)一样，就是换了一个配置集为 `demo-application-jasypt`。

## 5.4 Application

创建 [DemoApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/DemoApplication.java) 类，作为应用启动类。代码如下：



```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class);
    }

}
```



## 5.5 JasyptTest

创建 [JasyptTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt/src/test/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/JasyptTest.java) 测试类，使用 Jasypt 将配置项的值进行加密。代码如下：



```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class JasyptTest {

    @Autowired
    private StringEncryptor encryptor;

    @Test
    public void encode() {
        // 第一个加密
        String password = "woshimima";
        System.out.println(encryptor.encrypt(password));

        // 第二个加密
        password = "bushimima";
        System.out.println(encryptor.encrypt(password));
    }

    @Value("${xxx-password:}")
    private String xxxPassword;

    @Test
    public void print() {
        System.out.println(xxxPassword);
    }

}
```



下面，我们进行下简单测试。

① 首先，执行 `#encode()` 方法，**手动**使用 Jasypt 将 `"woshimima"` 和 `"bushimima"` 进行加密，获得加密结果。加密结果如下：



```
// "woshimima" 的加密结果
yCJ8JgVNG8Ns+vikAvZBKfJu/7LVejQWzwMxyoVFoR8=
// "bushimima" 的加密结果
NGgajNjjhKcgm7ncXvdVNsShSsueysdcCOTbOmtHXRc=
```



② 然后，将 `"woshimima"` 加密结果 `"yCJ8JgVNG8Ns+vikAvZBKfJu/7LVejQWzwMxyoVFoR8="`，赋值到 Nacos 配置集 `demo-application-jasypt` 的 `xxx-password` 配置项。如下图所示：![修改 Nacos 配置集](E:\Development\Typora\images\32-166350410088937.png)

之后，执行 `#print()` 方法，**自动**使用 Jasypt 将 `xxx-password` 配置项解密。解密结果如下：



```
woshimima
```



- 成功正确解密，符合预期。

## 5.6 DemoController

创建 [DemoController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/controller/DemoController.java) 类，提供查询 `xxx-password` 配置项的 HTTP 接口，并方便我们稍后测试 Nacos 自动配置刷新功能和 Jasypt 的兼容性问题。代码如下：



```
@RestController
@RequestMapping("/demo")
@RefreshScope
public class DemoController {

    @Value("${xxx-password:}")
    private String xxxPassword;

    @GetMapping("/test")
    public String test() {
        return xxxPassword;
    }

}
```



下面，我们进行下简单测试。

① 使用 DemoApplication 启动示例应用。

之后，访问 http://127.0.0.1:8080/demo/test 接口，返回结果为 `woshimima`，**符合预期**。

② 然后，将 `"bushimima"` 加密结果 `"NGgajNjjhKcgm7ncXvdVNsShSsueysdcCOTbOmtHXRc="`，赋值到 Nacos 配置集 `demo-application-jasypt` 的 `xxx-password` 配置项。如下图所示：![修改 Nacos 配置集](E:\Development\Typora\images\33.png)

之后，访问 http://127.0.0.1:8080/demo/test 接口，返回结果为 `ENC(NGgajNjjhKcgm7ncXvdVNsShSsueysdcCOTbOmtHXRc=)`，**不符合预期**。理论来说，返回结果应该是 `bushimima`，说明 Jasypt **并未**对 Nacos 自动配置刷新获取到的最新配置进行**解密**。

## 5.7 JasyptEnvironmentChangeListener

针对 Jasypt **并未**对 Nacos 自动配置刷新获取到的最新配置进行**解密**，我们可以通过自定义 [JasyptEnvironmentChangeListener](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-jasypt/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/listener/JasyptEnvironmentChangeListener.java) 监听器，发现变更的配置项的值是使用 Jasypt 进行加密的，则进行解密，并设置到 Environment 中。代码如下：



```
@Component
public class JasyptEnvironmentChangeListener implements ApplicationListener<EnvironmentChangeEvent> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    // Environment 管理器，可以实现配置项的获取和修改
    @Autowired
    private EnvironmentManager environmentManager;

    // Jasypt 加密器，可以对配置项进行加密和加密
    @Autowired
    private StringEncryptor encryptor;

    @Override
    public void onApplicationEvent(EnvironmentChangeEvent event) {
        for (String key : event.getKeys()) {
            // <1> 获得 value
            Object valueObj = environmentManager.getProperty(key);
            if (!(valueObj instanceof String)) {
                continue;
            }
            String value = (String) valueObj;
            // <2> 判断 value 是否为加密。如果是，则进行解密
            if (value.startsWith("ENC(") && value.endsWith(")")) {
                value = encryptor.decrypt(StringUtils.substringBetween(value, "ENC(", ")"));
                logger.info("[onApplicationEvent][key({}) 解密后为 {}]", key, value);
                // <3> 设置到 Environment 中
                environmentManager.setProperty(key, value);
            }
        }
    }

}
```



- 代码比较简单，胖友跟着艿艿添加的注释，可以快速的就理解了噢。

下面，我们进行下简单测试。

① 使用 DemoApplication 启动示例应用。

之后，访问 http://127.0.0.1:8080/demo/test 接口，返回结果为 `bushimima`，**符合预期**。

② 然后，将 `"woshimima"` 加密结果 `"yCJ8JgVNG8Ns+vikAvZBKfJu/7LVejQWzwMxyoVFoR8="`，赋值到 Nacos 配置集 `demo-application-jasypt` 的 `xxx-password` 配置项。如下图所示：![修改 Nacos 配置集](E:\Development\Typora\images\32-166350410088937.png)

之后，访问 http://127.0.0.1:8080/demo/test 接口，返回结果为 `woshimima`，**符合预期**。

😈 不过这个方案暂时只是**临时解决方案**，因为在 JasyptEnvironmentChangeListener 处理完成之前，实际获取到经过 Jasypt 加密的配置项的值，都是未经过解密的，这个可能导致不可预期的问题，需要胖友进行一定的测试噢。

# 6. 监控端点

> 示例代码对应仓库：[`labx-05-sca-nacos-config-demo-actuator`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-actuator/)。

Spring Cloud Alibaba Nacos Config 的 [`endpoint`](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/java/com/alibaba/cloud/nacos/endpoint/) 模块，基于 Spring Boot Actuator，提供了自定义监控端点 [`nacos-config`](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/java/com/alibaba/cloud/nacos/endpoint/NacosConfigEndpoint.java)，获取 Nacos 的**配置项**、配置集的更新时间、自动配置刷新历史。

同时，Nacos Config 拓展了 Spring Boot Actuator 内置的 `health` 端点，通过自定义的 [NacosConfigHealthIndicator](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/java/com/alibaba/cloud/nacos/endpoint/NacosConfigHealthIndicator.java)，获取和 Nacos 配置中心的连接状态。

> 友情提示：对 Spring Boot Actuator 不了解的胖友，可以后续阅读[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)文章。

下面，我们从[「2. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self#)小节的 [`labx-05-sca-nacos-config-demo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo) 项目，复制出 [`labx-05-sca-nacos-config-demo-actuator`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-actuator/) 项目，快速搭建 Nacos Config 监控端点的使用示例。

## 6.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-actuator/pom.xml) 文件中，额外引入 Spring Boot Actuator 相关依赖。代码如下：



```
<!-- 实现对 Actuator 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



## 6.2 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-actuator/src/main/resources/application.yaml) 配置文件，增加 Spring Boot Actuator 配置项。配置如下：



```
management:
  endpoints:
    web:
      exposure:
        include: '*' # 需要开放的端点。默认值只打开 health 和 info 两个端点。通过设置 * ，可以开放所有端点。
  endpoint:
    # Health 端点配置项，对应 HealthProperties 配置类
    health:
      enabled: true # 是否开启。默认为 true 开启。
      show-details: ALWAYS # 何时显示完整的健康信息。默认为 NEVER 都不展示。可选 WHEN_AUTHORIZED 当经过授权的用户；可选 ALWAYS 总是展示。
```



每个配置项的作用，胖友看下艿艿添加的注释。如果还不理解的话，后续看下[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)文章。

## 6.3 简单测试

① 使用 DemoApplication 启动示例应用。

② 访问应用的 `nacos-config` 监控端点 http://127.0.0.1:8080/actuator/nacos-config，返回结果如下图：![ 监控端点](E:\Development\Typora\images\41-166350410088940.png)

③ 访问应用的 `health` 监控端点 http://127.0.0.1:8080/actuator/health，返回结果如下图：![ 监控端点](E:\Development\Typora\images\42-166350410088944.png)

# 7. 配置加载顺序

> 示例代码对应仓库：[`labx-05-sca-nacos-config-demo-multi`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-multi/)。

Nacos Config 提供了三种配置 Nacos 配置集的方式：

- A：通过 `spring.cloud.nacos.config.shared-configs` 配置项，支持**多个**共享 Nacos 配置集。
- B：通过 `spring.cloud.nacos.config.extension-configs` 配置项，支持**多个**拓展 Nacos 配置集。
- C：通过 `spring.cloud.nacos.config.name` 配置项，支持**一个** Nacos 配置集。

当三种方式共同使用时，它们的优先级关系是：`A < B < C`。另外，A 和 B 的命名带有“共享”或是“拓展”，没有任何含义，只是优先级不同。

下面，我们来创建一个 [`labx-05-sca-nacos-config-demo-multi`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo) 示例项目，搭建**同时**使用三种配置 Nacos 配置集的方式的示例。最终项目代码如下图所示：![labx-05-sca-nacos-config-demo-multi` 项目](E:\Development\Typora\images\52.png)

## 7.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-multi/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>labx-05</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>labx-05-sca-nacos-config-demo-multi</artifactId>

    <properties>
        <spring.boot.version>2.2.4.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR1</spring.cloud.version>
        <spring.cloud.alibaba.version>2.2.0.RELEASE</spring.cloud.alibaba.version>
    </properties>

    <!--
        引入 Spring Boot、Spring Cloud、Spring Cloud Alibaba 三者 BOM 文件，进行依赖版本的管理，防止不兼容。
        在 https://dwz.cn/mcLIfNKt 文章中，Spring Cloud Alibaba 开发团队推荐了三者的依赖关系
     -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- 引入 SpringMVC 相关依赖，并实现对其的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 引入 Spring Cloud Alibaba Nacos Config 相关依赖，将 Nacos 作为配置中心，并实现对其的自动配置 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>

</project>
```



## 7.2 配置文件

创建 [`bootstrap.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-multi/src/main/resources/bootstrap.yaml) 配置文件，添加 Nacos Config 相关配置。配置如下：



```yml
spring:
  application:
    name: demo-application

  cloud:
    nacos:
      # Nacos Config 配置项，对应 NacosConfigProperties 配置属性类
      config:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: # 使用的 Nacos 的命名空间，默认为 null
        group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
        name: ${spring.application.name} # 使用的 Nacos 配置集的 dataId，默认为 spring.application.name
        file-extension: yaml # 使用的 Nacos 配置集的 dataId 的文件拓展名，同时也是 Nacos 配置集的配置格式，默认为 properties
        # 拓展配置集数组，对应 Config 数组
        extension-configs:
          - data-id: extension-dataId-01.yaml
            group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
            refresh: true # 是否自动刷新配置，默认为 false
          - data-id: extension-dataId-02.yaml
            group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
            refresh: true # 是否自动刷新配置，默认为 false
        # 共享配置集数组，对应 Config 数组
        shared-configs:
          - data-id: shared-dataId-01.yaml
            group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
            refresh: true # 是否自动刷新配置，默认为 false
          - data-id: shared-dataId-02.yaml
            group: DEFAULT_GROUP # 使用的 Nacos 配置分组，默认为 DEFAULT_GROUP
            refresh: true # 是否自动刷新配置，默认为 false

  profiles:
    active: dev # 设置开启的 Profiles
```



① 在 `spring.cloud.nacos.config` 配置项下，**同时**使用三种配置 Nacos 配置集的方式。

② 在 `spring.profiles.active` 配置项下，设置为 `dev`，用于测试结合 Profiles 的情况。

## 7.3 DemoApplication

创建 [DemoApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-05-spring-cloud-alibaba-nacos-config/labx-05-sca-nacos-config-demo-multi/src/main/java/cn/iocoder/springcloudalibaba/labx5/nacosdemo/DemoApplication.java) 类，作为应用启动类。代码如下：



```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        ConfigurableApplicationContext context = SpringApplication.run(DemoApplication.class, args);

        // 查看 Environment
        Environment environment = context.getEnvironment();
        System.out.println(environment);
    }

}
```



在代码中，我们去获取了 Spring [Environment](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/env/Environment.java) 对象，因为我们要从其中获取到 [PropertySource](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/env/PropertySource.java) 配置来源。**DEBUG** 运行 Application，并记得在 `System.out.println(environment);` 代码块打一个断点，可以看到如下图的调试信息：![调试信息](E:\Development\Typora\images\51-16635041008816.png)

- 每一个 Nacos 配置集，对应一个 PropertySource 对象，并且优先级符合 `C > B > A`。另外，注意每一组内部的优先级。
- 所有 Nacos 配置集的 PropertySource 对象，排在 `application.yaml` 配置文件的 PropertySource 对象前面，基本是最高优先级。

如果胖友好奇三种配置 Nacos 配置集的优先级是这样，可以看看 NacosPropertySourceLocator 的代码，会特别好理解。代码如下：



```java
// NacosPropertySourceLocator.java

@Override
public PropertySource<?> locate(Environment env) {
    // ... 省略其他代码

	CompositePropertySource composite = new CompositePropertySource(
			NACOS_PROPERTY_SOURCE_NAME);

	loadSharedConfiguration(composite); // A
	loadExtConfiguration(composite); // B
	loadApplicationConfiguration(composite, dataIdPrefix, nacosConfigProperties, env); // C

	return composite;
}
```



# 666. 彩蛋

至此，我们已经完成 Spring Cloud Alibaba Nacos Config 的学习。如下是 Nacos 相关的官方文档：

- [《Nacos 官方文档》](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [《Spring Cloud Alibaba 官方**文档** —— Nacos Config》](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config)
- [《Spring Cloud Alibaba 官方**示例** —— Nacos Config》](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-config-example/readme-zh.md)

另外，Spring Cloud Alibaba Nacos Config 还有一个 [`spring-cloud-alibaba-nacos-config-server`](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config-server/) 子项目，将 Nacos 适配成 [Spring Cloud Config Server](https://github.com/spring-cloud/spring-cloud-config/tree/master/spring-cloud-config-server)，或者说将 Nacos 作为 [Spring Cloud Config Server](https://github.com/spring-cloud/spring-cloud-config/tree/master/spring-cloud-config-server) 的存储器。这样，目前正在使用 [Spring Cloud Config Client](https://github.com/spring-cloud/spring-cloud-config/blob/master/spring-cloud-config-client/) 的项目，再不做任何代码变动的情况下，就可以无缝切换到 Nacos 配置中心。

如果，想要在 Spring Boot 项目中使用 Nacos 作为配置中心的胖友，可以阅读[《芋道 Spring Boot 配置中心 Nacos 入门》](http://www.iocoder.cn/Spring-Boot/config-nacos/?self)文章。
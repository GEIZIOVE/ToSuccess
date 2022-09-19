# Spring Cloud Alibaba 注册中心 Nacos 入门

# 1. 概述

本文我们来学习 [Spring Cloud Alibaba](https://spring.io/projects/spring-cloud-alibaba) 提供的 [Spring Cloud Alibaba Nacos Discovery](https://github.com/alibaba/spring-cloud-alibaba/tree/master/spring-cloud-alibaba-nacos-discovery) 组件，基于 Spring Cloud 的编程模型，接入 Nacos 作为注册中心，实现服务的注册与发现。

> [服务注册/发现: Nacos Discovery](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-docs/src/main/asciidoc-zh/nacos-discovery.adoc#服务注册发现-nacos-discovery)
>
> 服务发现是微服务架构体系中最关键的组件之一。如果尝试着用手动的方式来给每一个客户端来配置所有服务提供者的服务列表是一件非常困难的事，而且也不利于服务的动态扩缩容。
>
> Nacos Discovery 可以帮助您将服务自动注册到 Nacos 服务端并且能够动态感知和刷新某个服务实例的服务列表。
>
> 除此之外，Nacos Discovery 也将服务实例自身的一些元数据信息-例如 host，port, 健康检查URL，主页等内容注册到 Nacos。

在开始本文之前，胖友需要对 Nacos 进行简单的学习。可以阅读[《Nacos 极简入门》](http://www.iocoder.cn/Nacos/install/?self)文章，将第一二小节看完，在本机搭建一个 Nacos 服务。

# 2. 注册中心原理

在开始搭建 Nacos Discovery 的示例之前，我们先来简单了解下注册中心的原理。

在使用注册中心时，一共有三种角色：服务提供者（Service Provider）、服务消费者（Service Consumer）、注册中心（Registry）。

> 在一些文章中，服务提供者被称为 Server，服务消费者被称为 Client。胖友们知道即可。

三个角色交互如下图所示：![注册中心原理](E:\Development\Typora\images\01-16634272204342.png)

① `Provider`：

- 启动时，向 Registry **注册**自己为一个服务（Service）的实例（Instance）。
- 同时，定期向 Registry 发送**心跳**，告诉自己还存活。
- 关闭时，向 Registry **取消注册**。

② `Consumer`：

- 启动时，向 Registry **订阅**使用到的服务，并缓存服务的实例列表在内存中。
- 后续，Consumer 向对应服务的 Provider 发起**调用**时，从内存中的该服务的实例列表选择一个，进行远程调用。
- 关闭时，向 Registry **取消订阅**。

③ `Registry`：

- Provider 超过一定时间未**心跳**时，从服务的实例列表移除。
- 服务的实例列表发生变化（新增或者移除）时，通知订阅该服务的 Consumer，从而让 Consumer 能够刷新本地缓存。

当然，不同的注册中心可能在实现原理上会略有差异。例如说，[Eureka](https://github.com/Netflix/eureka/) 注册中心，并不提供通知功能，而是 Eureka Client 自己定期轮询，实现本地缓存的更新。

另外，Provider 和 Consumer 是角色上的定义，一个服务**同时**即可以是 Provider 也可以作为 Consumer。例如说，**优惠劵服务可以给订单服务提供接口，同时又调用用户服务提供的接口。**

# 3. 快速入门

> 示例代码对应仓库：
>
> - 服务提供者：[`labx-01-sca-nacos-discovery-demo01-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider)
> - 服务消费者：[`labx-01-sca-nacos-discovery-demo01-consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer/)

本小节，我们来搭建一个 Nacos Discovery 组件的快速入门示例。步骤如下：

- 首先，搭建一个服务提供者 `demo-provider` ，注册服务到 Nacos 中。
- 然后，搭建一个服务消费者 `demo-consumer`，从 Nacos 获取到 `demo-provider` 服务的实例列表，选择其中一个示例，进行 HTTP 远程调用。

## 3.1 搭建服务提供者

创建 [`labx-01-sca-nacos-discovery-demo01-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider) 项目，作为服务提供者 `demo-provider`。最终项目代码如下图所示：![ 项目](E:\Development\Typora\images\01.jpg)

### 3.1.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider/pom.xml) 文件中，主要引入 Spring Cloud Nacos Discovery 相关依赖。代码如下：



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>labx-01</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>labx-01-sca-nacos-discovery-demo01-provider</artifactId>

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

        <!-- 引入 Spring Cloud Alibaba Nacos Discovery 相关依赖，将 Nacos 作为注册中心，并实现对其的自动配置 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>

</project>
```



> 友情提示：有点小长，不要慌~

在 `<dependencyManagement />` 中，我们引入了 Spring Boot、Spring Cloud、Spring Cloud Alibaba 三者 BOM 文件，进行依赖版本的管理，防止不兼容。在[《Spring Cloud 官方文档 —— 版本说明》](https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明)文档中，推荐了三者的依赖关系。如下表格：

| Spring Cloud Version   | Spring Cloud Alibaba Version | Spring Boot Version |
| :--------------------- | :--------------------------- | :------------------ |
| Spring Cloud Greenwich | 2.1.1.RELEASE                | 2.1.X.RELEASE       |
| Spring Cloud Finchley  | 2.0.1.RELEASE                | 2.0.X.RELEASE       |
| Spring Cloud Edgware   | 1.5.1.RELEASE                | 1.5.X.RELEASE       |

- 这里，我们选择了 Spring Cloud Alibaba 版本为 `2.2.0.RELEASE`。
- 当前版版本下，我们使用的 Nacos 版本为 `1.1.4`。

引入 [`spring-cloud-starter-alibaba-nacos-discovery`](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-alibaba-nacos-discovery) 依赖，将 Nacos 作为注册中心，并实现对它的自动配置。

### 3.1.2 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider/src/main/resources/application.yaml) 配置文件，添加 Nacos Discovery 配置项。配置如下：



```yml
spring:
  application:
    name: demo-provider # Spring 应用名
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        service: ${spring.application.name} # 注册到 Nacos 的服务名。默认值为 ${spring.application.name}。

server:
  port: 18080 # 服务器端口。默认为 8080
```



重点看 `spring.cloud.nacos.dis	covery` 配置项，它是 Nacos Discovery 配置项的前缀，对应 [NacosDiscoveryProperties](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-discovery/src/main/java/com/alibaba/cloud/nacos/NacosDiscoveryProperties.java) 配置项。

### 3.1.3 DemoProviderApplication

创建 [DemoProviderApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider/src/main/java/cn/iocoder/springcloudalibaba/labx01/nacosdemo/provider/DemoProviderApplication.java) 类，创建应用启动类，并提供 HTTP 接口。代码如下：



```java
@SpringBootApplication
@EnableDiscoveryClient
public class DemoProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoProviderApplication.class, args);
    }

    @RestController
    static class TestController {

        @GetMapping("/echo")
        public String echo(String name) {
            return "provider:" + name;
        }

    }

}
```



① `@SpringBootApplication` 注解，被添加在类上，声明这是一个 Spring Boot 应用。Spring Cloud 是构建在 Spring Boot 之上的，所以需要添加。

② [`@EnableDiscoveryClient`](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/EnableDiscoveryClient.java) 注解，开启 Spring Cloud 的注册发现功能。不过从 Spring Cloud Edgware 版本开始，实际上已经不需要添加 `@EnableDiscoveryClient` 注解，只需要引入 Spring Cloud 注册发现组件，就会自动开启注册发现的功能。例如说，我们这里已经引入了 `spring-cloud-starter-alibaba-nacos-discovery` 依赖，就不用再添加 `@EnableDiscoveryClient` 注解了。

> 拓展小知识：在 Spring Cloud Common 项目中，定义了 [DiscoveryClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/discovery/DiscoveryClient.java) 接口，作为通用的发现客户端，提供读取服务和读取服务列表的 API 方法。而想要集成到 Spring Cloud 体系的注册中心的组件，需要提供对应的 DiscoveryClient 实现类。
>
> 例如说，Spring Cloud Alibaba Nacos Discovery 提供了 [NacosDiscoveryClient](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-discovery/src/main/java/com/alibaba/cloud/nacos/discovery/NacosDiscoveryClient.java) 实现，Spring Cloud Netflix Eureka 提供了 [EurekaDiscoveryClient](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaDiscoveryClient.java) 实现。
>
> 如此，所有需要使用到的地方，只需要获取到 DiscoveryClient 客户端，而无需关注具体实现，保证其通用性。

③ TestController 类，提供了 `/echo` 接口，返回 `provider:${name}` 结果。

### 3.1.4 简单测试

① 通过 DemoProviderApplication 启动服务提供者，IDEA 控制台输出日志如：



```bash
// ... 省略其它日志
2020-02-08 15:25:57.406  INFO 27805 --- [           main] c.a.c.n.registry.NacosServiceRegistry    : nacos registry, DEFAULT_GROUP demo-provider 10.171.1.115:18080 register finished
```



- 服务 `demo-provider` 注册到 Nacos 上的日志。

② 打开 Nacos 控制台，可以在服务列表看到服务 `demo-provider`。如下图：![服务列表](E:\Development\Typora\images\02-16634272204355.png)

## 3.2 搭建服务消费者

创建 [`labx-01-sca-nacos-discovery-demo01-consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer/) 项目，作为服务提供者 `demo-consumer`。最终项目代码如下图所示：![ 项目](E:\Development\Typora\images\03-16634272204357.png)

整个项目的代码，和服务提供者是基本一致的，毕竟是示例代码 😜

### 3.2.1 引入依赖

和[「3.1.1 引入依赖」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)一样，只是修改 Maven `<artifactId />` 为 `labx-01-sca-nacos-discovery-demo01-consumer`，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer/pom.xml) 文件。

### 3.2.2 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer/src/main/resources/application.yaml) 配置文件，添加相应配置项。配置如下：



```yml
spring:
  application:
    name: demo-consumer # Spring 应用名
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址

server:
  port: 28080 # 服务器端口。默认为 8080
```



和[「3.1.2 配置文件」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)基本一致，主要是将配置项目 `spring.application.name` 修改为 `demo-consumer`。

### 3.2.3 DemoConsumerApplication

创建 [DemoConsumerApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer/src/main/java/cn/iocoder/springcloudalibaba/labx01/nacosdemo/consumer/DemoConsumerApplication.java) 类，创建应用启动类，并提供一个调用服务提供者的 HTTP 接口。代码如下：



```java
@SpringBootApplication
// @EnableDiscoveryClient
public class DemoConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoConsumerApplication.class, args);
    }

    @Configuration
    public class RestTemplateConfiguration {

        @Bean
        public RestTemplate restTemplate() {
            return new RestTemplate();
        }

    }

    @RestController
    static class TestController {

        @Autowired
        private DiscoveryClient discoveryClient;
        @Autowired
        private RestTemplate restTemplate;
        @Autowired
        private LoadBalancerClient loadBalancerClient;

        @GetMapping("/hello")
        public String hello(String name) {
            // <1> 获得服务 `demo-provider` 的一个实例
            ServiceInstance instance;
            if (true) {
                // 获取服务 `demo-provider` 对应的实例列表
                List<ServiceInstance> instances = discoveryClient.getInstances("demo-provider");
                // 选择第一个
                instance = instances.size() > 0 ? instances.get(0) : null;
            } else {
                instance = loadBalancerClient.choose("demo-provider");
            }
            // <2> 发起调用
            if (instance == null) {
                throw new IllegalStateException("获取不到实例");
            }
            String targetUrl = instance.getUri() + "/echo?name=" + name;
            String response = restTemplate.getForObject(targetUrl, String.class);
            // 返回结果
            return "consumer:" + response;
        }

    }

}
```



① `@EnableDiscoveryClient` 注解，因为已经无需添加，所以我们进行了注释，原因在上面已经解释过。

② RestTemplateConfiguration 配置类，创建 [RestTemplate](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/client/RestTemplate.java) Bean。RestTemplate 是 Spring 提供的 HTTP 调用模板工具类，可以方便我们稍后调用服务提供者的 HTTP API。

③ TestController 提供了 `/hello` 接口，用于调用服务提供者的 `/demo` 接口。代码略微有几行，我们来稍微解释下哈。

**`discoveryClient` 属性，DiscoveryClient 对象，服务发现客户端，上文我们已经介绍过。这里我们注入的不是 Nacos Discovery 提供的 NacosDiscoveryClient，保证通用性。未来如果我们不使用 Nacos 作为注册中心，而是使用 Eureka 或则 Zookeeper 时，则无需改动这里的代码。**

`loadBalancerClient` 属性，[LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java) 对象，负载均衡客户端。稍后我们会使用它，从 Nacos 获取的服务 `demo-provider` 的实例列表中，选择一个进行 HTTP 调用。

> 拓展小知识：在 Spring Cloud Common 项目中，定义了[LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java) 接口，作为通用的负载均衡客户端，提供从指定服务中选择一个实例、对指定服务发起请求等 API 方法。而想要集成到 Spring Cloud 体系的负载均衡的组件，需要提供对应的 LoadBalancerClient 实现类。
>
> 例如说，Spring Cloud Netflix Ribbon 提供了 [RibbonLoadBalancerClient](https://github.com/spring-cloud/spring-cloud-netflix/blob/2.2.x/spring-cloud-netflix-ribbon/src/main/java/org/springframework/cloud/netflix/ribbon/RibbonLoadBalancerClient.java) 实现。
>
> 如此，所有需要使用到的地方，只需要获取到 DiscoveryClient 客户端，而无需关注具体实现，保证其通用性。😈 不过貌似 Spring Cloud 体系中，暂时只有 Ribbon 一个负载均衡组件。
>
> 当然，LoadBalancerClient 的服务的实例列表，是来自 DiscoveryClient 提供的。

`/hello` 接口，示例接口，对服务提供者发起一次 HTTP 调用。

- `<1>` 处，获得服务 `demo-provider` 的一个实例。这里我们提供了两种方式的代码，分别基于 DiscoveryClient 和 LoadBalancerClient。
- `<2>` 处，通过获取到的服务实例 ServiceInstance 对象，拼接请求的目标 URL，之后使用 RestTemplate 发起 HTTP 调用。

### 3.2.4 简单测试

① 通过 DemoConsumerApplication 启动服务消费者，IDEA 控制台输出日志如：



```bash
// ... 省略其它日志
2020-02-08 18:05:35.810  INFO 35047 --- [           main] c.a.c.n.registry.NacosServiceRegistry    : nacos registry, DEFAULT_GROUP demo-consumer 10.171.1.115:28080 register finished
```



- 服务 `demo-consumer` 注册到 Nacos 上的日志。

注意，服务消费者和服务提供是一种角色的概念，本质都是一种服务，都是可以注册自己到注册中心的。

② 打开 Nacos 控制台，可以在服务列表看到服务 `demo-consumer`。如下图：![服务列表](E:\Development\Typora\images\04-16634272204359.png)

③ 访问服务**消费者**的 http://127.0.0.1:28080/hello?name=yudaoyuanma 接口，返回结果为 `"consumer:provider:yudaoyuanma"`。说明，调用远程的**服务提供者**成功。

④ 打开 Nacos 控制台，可以在订阅者列表看到订阅关系。如下图：![订阅者列表](E:\Development\Typora\images\05-166342722043511.png)

⑤ 关闭服务**提供者**后，再次访问 http://127.0.0.1:28080/hello?name=yudaoyuanma 接口，返回结果为报错提示 `"获取不到实例"`，说明我们本地缓存的服务 `demo-provider` 的实例列表已刷新，没有任何实例。

😈 这里我们并没有演示启动多个服务提供者的测试，胖友可以自己尝试下哟。

# 4. Nacos 概念详解

> 友情提示：本小节的内容，基于如下两篇文档梳理，推荐胖友后续也看看：
>
> - [《Nacos 官方文档 —— 概念》](https://nacos.io/zh-cn/docs/what-is-nacos.html)
> - [《Nacos 官方文档 —— 架构》](https://nacos.io/zh-cn/docs/architecture.html)

## 4.1 数据模型

Nacos 数据模型 Key 由三元组唯一确认。如下图所示：![Nacos 数据模型](E:\Development\Typora\images\b9ac1d085e4224ae5137e6bab0a4fdde.jpg)

- 作为注册中心时，`Namespace + Group + Service`
- 作为配置中心时，`Namespace + Group + DataId`

我们来看看 Namespace、Group、Service 的概念。

### 4.1.1 Namespace 命名空间

用于进行租户粒度的配置隔离。默认为 public（公共命名空间）。

不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

稍后在[「6. 多环境配置」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节中，我们会通过 Namespace 隔离不同环境的服务。

### 4.1.2 Group 服务分组

不同的服务可以归类到同一分组。默认为 `DEFAULT_GROUP`（默认分组）。

### 4.1.3 Service 服务

例如说，用户服务、订单服务、商品服务等等。

## 4.2 服务领域模型

Service 可以进一步细拆服务领域模型，如下图：![Nacos 服务领域模型](E:\Development\Typora\images\f6629a647c82262f80463ed813b49647.jpg)

我们来看看图中的每个“节点”的概念。

### 4.2.1 Instance 实例

提供一个或多个服务的具有可访问网络地址（IP:Port）的进程。

我们以[「3.1 搭建服务提供者」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节来举例子：

- 如果我们启动**一个** JVM 进程，就是服务 `demo-provider` 下的**一个**实例。
- 如果我们启动**多个** JVM 进程，就是服务 `demo-provider` 下的**多个**实例。

### 4.2.2 Cluster 集群

`同一个服务下的所有服务实例组成一个默认集群（Default）`。集群可以被进一步按需求划分，划分的单位可以是虚拟集群。

例如说，我们将服务部署在多个机房之中，每个机房可以创建为一个虚拟集群。每个服务在注册到 Nacos 时，设置所在机房的虚拟集群。这样，服务在调用其它服务时，可以通过虚拟集群，优先调用本机房的服务。如此，在提升服务的可用性的同时，保证了性能。

### 4.2.3 Metadata 元数据

Nacos 元数据（如配置和服务）描述信息，如**服务版本、权重、容灾策略、负载均衡策略、鉴权配置、各种自定义标签 (label)。**

从作用范围来看，分为服务级别的元信息、集群的元信息及实例的元信息。如下图：![元数据](E:\Development\Typora\images\11.gif)

> FROM [《Dubbo 官方文档 —— 多版本》](http://dubbo.apache.org/zh-cn/docs/user/demos/multi-versions.html)

以 Nacos 元数据的**服务版本**举例子。当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用。

可以按照以下的步骤进行版本迁移：

1. 在低压力时间段，先升级一半提供者为新版本
2. 再将所有消费者升级为新版本
3. 然后将剩下的一半提供者升级为新版本

> FROM [《Dubbo 官方文档 —— 令牌验证》](http://dubbo.apache.org/zh-cn/docs/user/demos/token-authorization.html)

再次 Nacos 元数据的**鉴权配置**举例子。通过令牌验证在注册中心控制权限，以决定要不要下发令牌给消费者，可以防止消费者绕过注册中心访问提供者。另外，通过注册中心可灵活改变授权方式，而不需修改或升级提供者。

![令牌验证](E:\Development\Typora\images\5e16512d6d12f9e1086d0f4eb71ec481.jpg)

### 4.2.4 Health Check 健康检查

以指定方式检查服务下挂载的实例的健康度，从而确认该实例是否能提供服务。根据检查结果，实例会被判断为健康或不健康。

对服务发起解析请求时，不健康的实例不会返回给客户端。

**健康保护阈值**

为了防止因过多实例不健康导致流量全部流向健康实例，继而造成流量压力把健康实例实例压垮并形成雪崩效应，应将健康保护阈值定义为一个 0 到 1 之间的浮点数。

**当域名健康实例占总服务实例的比例小于该值时，无论实例是否健康，都会将这个实例返回给客户端。这样做虽然损失了一部分流量，但是保证了集群的剩余健康实例能正常工作。**

## 4.3 小结

为了让胖友更好理解，我们把数据模型和服务领域模型整理如下图所示：![Nacos 概念小结](E:\Development\Typora\images\12-166342722043617.png)

# 5. 更多的配置项信息

在[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节中，我们为了快速入门，只使用了 Nacos Discovery Starter 两个配置项。实际上，Nacos Discovery Starter 提供的配置项挺多的，我们参考[文档](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-docs/src/main/asciidoc-zh/nacos-discovery.adoc#服务注册发现-nacos-discovery)将配置项一起梳理下。

**Nacos 服务器相关**

| 配置项     | Key                                        | 说明                                       |
| :--------- | :----------------------------------------- | :----------------------------------------- |
| 服务端地址 | `spring.cloud.nacos.discovery.server-addr` | Nacos Server 启动监听的ip地址和端口        |
| AccessKey  | `spring.cloud.nacos.discovery.access-key`  | 当要上阿里云时，阿里云上面的一个云账号名   |
| SecretKey  | `spring.cloud.nacos.discovery.secret-key`  | 当要上阿里云时，阿里云上面的一个云账号密码 |

**服务相关**

| 配置项              | Key                                          | 说明                                                         |
| :------------------ | :------------------------------------------- | :----------------------------------------------------------- |
| 命名空间            | `spring.cloud.nacos.discovery.namespace`     | 常用场景之一是不同环境的注册的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等 |
| 服务分组            | `spring.cloud.nacos.discovery.group`         | 不同的服务可以归类到同一分组。默认为 `DEFAULT_GROUP`         |
| 服务名              | `spring.cloud.nacos.discovery.service`       | 注册的服务名。默认为 `${spring.application.name}`            |
| 集群                | `spring.cloud.nacos.discovery.cluster-name`  | Nacos 集群名称。默认为 `DEFAULT`                             |
| 权重                | `spring.cloud.nacos.discovery.weight`        | 取值范围 1 到 100，数值越大，权重越大。默认为 1              |
| Metadata            | `spring.cloud.nacos.discovery.metadata`      | 使用Map格式配置，用户可以根据自己的需要自定义一些和服务相关的元数据信息 |
| 是否开启Nacos Watch | `spring.cloud.nacos.discovery.watch.enabled` | 可以设置成 `false` 来关闭 watch。默认为 `true`               |

**网络相关**

| 配置项       | Key                                              | 说明                                                         |
| :----------- | :----------------------------------------------- | :----------------------------------------------------------- |
| 网卡名       | `spring.cloud.nacos.discovery.network-interface` | 当 IP未配置时，注册的 IP 为此网卡所对应的 IP 地址，如果此项也未配置，则默认取第一块网卡的地址 |
| 注册的IP地址 | `spring.cloud.nacos.discovery.ip`                | 优先级最高                                                   |
| 注册的端口   | `spring.cloud.nacos.discovery.port`              | 默认情况下不用配置，会自动探测。默认为 `-1`                  |

**其它相关**

| 配置项          | Key                                     | 说明                                                         |
| :-------------- | :-------------------------------------- | :----------------------------------------------------------- |
| 是否集成 Ribbon | `ribbon.nacos.enabled`                  | 一般都设置成`true` 即可。默认为 `true`                       |
| 日志文件名      | `spring.cloud.nacos.discovery.log-name` |                                                              |
| 接入点          | `spring.cloud.nacos.discovery.endpoint` | 地域的某个服务的入口域名，通过此域名可以动态地拿到服务端地址 |

# 6. 多环境配置

> 示例代码对应仓库：
>
> - 服务提供者：[`labx-01-sca-nacos-discovery-demo02-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-provider)
> - 服务消费者：[`labx-01-sca-nacos-discovery-demo02-consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-consumer/)

同一个服务，我们会部署到开发、测试、预发布、生产等环境中，那么我们需要在项目中，添加不同环境的 Nacos 配置。一般情况下，开发和测试使用同一个 Nacos，预发布和生产使用另一个 Nacos。那么针对相同的 Nacos，我们怎么实现不同环境的隔离呢？

实际上，Nacos 开发者已经告诉我们如何实现了，通过 Nacos **Namespace** 命名空间。文档说明如下：

> FROM [《Nacos 文档 —— Nacos 概念》](https://nacos.io/zh-cn/docs/concepts.html)
>
> **命名空间**，用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 Data ID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

下面，我们来搭建一个多环境配置的示例。步骤如下：

- 首先，我们会在 Nacos 中创建开发环境使用的 Namespace 为 `dev`，测试环境使用的 Namespace 为 `uat`。

- 然后，搭建一个服务提供者 `demo-provider`，使用**开发**环境配置，注册服务到 Nacos 的 `dev` Namespace 下。

- 之后，搭建一个服务消费者`demo-consumer`，调用服务提供者`demo-provider`提供的 HTTP 接口。

  - 先使用**开发**环境配置，因为服务 `demo-provider` **是**在 Nacos `dev` Namespace 下注册，所以调用它**成功**。
- 后使用**测试**环境配置，因为服务 `demo-provider` **不**在 Nacos `uat` Namespace 下注册，所以调用它**失败**，

> 友情提示：在 Spring Boot（Spring Cloud）项目中，可以使用 Profiles 机制，基于 `spring.profiles.active` 配置项，实现不同环境读取不同的配置文件。
>
> 不了解的胖友，可以看看[《芋道 Spring Boot 配置文件入门》](http://www.iocoder.cn/Spring-Boot/config-file/?self)的[「6. 多环境配置」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节。

## 6.1 创建 Nacos 命名空间

① 打开 Nacos UI 界面的「命名空间」菜单，进入「命名空间」功能。如下图所示：![命名空间](E:\Development\Typora\images\11-166342722043619.png)

② 点击列表右上角的「新建命名空间」按钮，弹出「新建命名空间」窗口，创建一个 `dev` 命名空间。输入如下内容，并点击「确定」按钮，完成创建。如下图所示：![新建命名空间](E:\Development\Typora\images\12-16634272203591.png)

③ 重复该操作，继续创建一个 `uat` 命名空间。最终 `dev` 和 `uat` 信息如下图：![命名空间列表](E:\Development\Typora\images\21-166342722043622.png)

## 6.2 搭建服务提供者

从[「3.1 搭建服务提供者」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节的 [`labx-01-sca-nacos-discovery-demo01-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider) 项目，复制出 [`labx-01-sca-nacos-discovery-demo02-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-provider) 项目。然后在其上进行修改，方便搭建~

### 6.2.1 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-provider/src/main/resources/application.yaml) 配置文件，将 Nacos Discovery 配置项删除，稍后添加在不同环境的配置文件中。配置如下：

```yml
spring:
  application:
    name: demo-provider # Spring 应用名

server:
  port: 18080 # 服务器端口。默认为 8080
```

创建**开发环境**使用的 [`application-dev.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-provider/src/main/resources/application-dev.yaml) 配置文件，增加 Namespace 为 `dev` 的 Nacos Discovery 配置项。配置如下：

```yml
spring:
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: 14226a0d-799f-424d-8905-162f6a8bf409 # Nacos 命名空间 dev 的编号
```

创建**测试环境**使用的 [`application-uat.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-provider/src/main/resources/application-uat.yaml) 配置文件，增加 Namespace 为 `uat` 的 Nacos Discovery 配置项。配置如下：

```yml
spring:
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: bc8c8c2d-bd85-42bb-ada3-1a8f940ceb20 # Nacos 命名空间 uat 的编号
```

### 6.2.2 简单测试

下面，我们使用命令行参数进行 `--spring.profiles.active` 配置项，实现不同环境，读取不同配置文件。

① 先配置 `--spring.profiles.active` 为 `dev`，设置 DemoProviderApplication 读取 `application-dev.yaml` 配置文件。如下图所示：![IDEA 配置 - dev](E:\Development\Typora\images\22-166342722043624.png)

之后通过 DemoProviderApplication 启动服务提供者。

② 打开 Nacos 控制台，可以在服务列表看到服务 `demo-provider` 注册在命名空间 `dev` 下。如下图：![服务列表](E:\Development\Typora\images\23-166342722043626.png)

## 6.3 搭建服务消费者

从[「3.2 搭建服务消费者」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节的 [`labx-01-sca-nacos-discovery-demo01-consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer) 项目，复制出 [`labx-01-sca-nacos-discovery-demo02-consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-consumer) 项目。然后在其上进行修改，方便搭建~

### 6.3.1 配置文件

> 友情提示：和[「6.2.1 配置文件」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节的内容是基本一致的，重复唠叨一遍。

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-consumer/src/main/resources/application.yaml) 配置文件，将 Nacos Discovery 配置项删除，稍后添加在不同环境的配置文件中。配置如下：

```yml
spring:
  application:
    name: demo-consumer # Spring 应用名

server:
  port: 28080 # 服务器端口。默认为 8080
```

创建**开发环境**使用的 [`application-dev.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-consumer/src/main/resources/application-dev.yaml) 配置文件，增加 Namespace 为 `dev` 的 Nacos Discovery 配置项。配置如下：

```yml
spring:
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: 14226a0d-799f-424d-8905-162f6a8bf409 # Nacos 命名空间 dev 的编号
```



创建**测试环境**使用的 [`application-uat.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo02-consumer/src/main/resources/application-uat.yaml) 配置文件，增加 Namespace 为 `uat` 的 Nacos Discovery 配置项。配置如下：



```yml
spring:
  cloud:
    nacos:
      # Nacos 作为注册中心的配置项，对应 NacosDiscoveryProperties 配置类
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
        namespace: bc8c8c2d-bd85-42bb-ada3-1a8f940ceb20 # Nacos 命名空间 uat 的编号
```



### 6.2.3 简单测试

下面，我们使用命令行参数进行 `--spring.profiles.active` 配置项，实现不同环境，读取不同配置文件。

① 先配置 `--spring.profiles.active` 为 `dev`，设置 DemoConsumerApplication 读取 `application-dev.yaml` 配置文件。如下图所示：![IDEA 配置 - dev](E:\Development\Typora\images\24-166342722043628.png)

之后通过 DemoConsumerApplication 启动服务消费者。

访问服务**消费者**的 http://127.0.0.1:28080/hello?name=yudaoyuanma 接口，返回结果为 `"consumer:provider:yudaoyuanma"`。说明，调用远程的**服务提供者**【成功】。

② 再配置 `--spring.profiles.active` 为 `uat`，设置 DemoConsumerApplication 读取 `application-uat.yaml` 配置文件。如下图所示：![IDEA 配置 - uat](E:\Development\Typora\images\25.png)

之后通过 DemoConsumerApplication 启动服务消费者。

访问服务**消费者**的 http://127.0.0.1:28080/hello?name=yudaoyuanma 接口，返回结果为 报错提示 `"获取不到实例"`。说明，调用远程的**服务提供者**【失败】。

原因是，虽然说服务 `demo-provider` 已经启动，因为其注册在 Nacos 的 Namespace 为 `dev`，这就导致第 ① 步启动的服务 `demo-consumer` 可以调用到该服务，而第② 步启动的服务 `demo-consumer` 无法调用到该服务。

**即，我们可以通过 Nacos 的 Namespace 实现不同环境下的服务隔离**。未来，在开源版本 Nacos 权限完善之后，每个 Namespace 提供不同的 AccessKey、SecretKey，保证只有知道账号密码的服务，才能连到对应的 Namespace，进一步提升安全性。

# 7. 监控端点

> 示例代码对应仓库：
>
> - 服务提供者：[`labx-01-sca-nacos-discovery-demo01-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider)
> - 服务消费者：[`labx-01-sca-nacos-discovery-demo03-consumer`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo03-consumer/)

Nacos Discovery 基于 Spring Boot Actuator，提供了自定义监控端点 [`nacos-discovery`](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-discovery/src/main/java/com/alibaba/cloud/nacos/endpoint/NacosDiscoveryEndpoint.java)，获取 Nacos Discovery 配置项，和订阅的服务信息。

同时，Nacos Discovery 拓展了 Spring Boot Actuator 内置的 `health` 端点，通过自定义的 [NacosDiscoveryHealthIndicator](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-discovery/src/main/java/com/alibaba/cloud/nacos/discovery/actuate/health/NacosDiscoveryHealthIndicator.java)，获取和 Nacos 服务器的连接状态。

> 友情提示：对 Spring Boot Actuator 不了解的胖友，可以后续阅读[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)文章。

下面，我们来搭建一个 Nacos Discovery 监控端点的示例。步骤如下：

- 首先，搭建一个服务提供者 `demo-provider` ，注册服务到 Nacos 中。
- 然后，搭建一个服务消费者 `demo-consumer`，调用服务提供者 `demo-provider` 提供的 HTTP 接口。同时，配置开启服务消费者的 Nacos Discovery 监控端点。
- 最后，访问服务消费者的 Nacos Discovery 监控端点，查看下返回的监控数据。

## 7.1 搭建服务提供者

直接复用[「3.1 搭建服务提供者」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节的 [`labx-01-sca-nacos-discovery-demo01-provider`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-provider) 项目即可。

因为 `labx-01-sca-nacos-discovery-demo01-provider` 项目没有从 Nacos 订阅任何服务，无法完整看到 `nacos-discovery` 端点的完整效果，所以我们暂时不配置该项目的 Nacos Discovery 监控端点。

不过实际项目中，配置下开启 Nacos Discovery 监控端点 还是可以的，至少可以看到 Nacos Discovery 配置项。

## 7.2 搭建服务消费者

从[「3.2 搭建服务消费者」](https://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self#)小节的 [`labx-01-sca-nacos-discovery-demo01-consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo01-consumer) 项目，复制出 [`labx-01-sca-nacos-discovery-demo03-consumer`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo03-consumer) 项目。然后在其上进行修改，方便搭建~

### 7.2.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo03-consumer/pom.xml) 文件中，额外引入 Spring Boot Actuator 相关依赖。代码如下：

```yml
<!-- 实现对 Actuator 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 7.2.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-01-spring-cloud-alibaba-nacos-discovery/labx-01-sca-nacos-discovery-demo03-consumer/src/main/resources/application.yaml) 配置文件，增加 Spring Boot Actuator 配置项。配置如下：

```yml
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

## 7.3 简单测试

① 通过 DemoProviderApplication 启动服务提供者，通过 DemoConsumerApplication 启动服务消费者。

之后，访问服务**消费者**的 http://127.0.0.1:28080/hello?name=yudaoyuanma 接口，返回结果为 `"consumer:provider:yudaoyuanma"`。a说明，调用远程的**服务提供者**成功。

② 访问服务消费者的 `nacos-discovery` 监控端点 http://127.0.0.1:28080/actuator/nacos-discovery，返回结果如下图：![ 监控端点](E:\Development\Typora\images\31-166342722043631.png)

理论来说，`"subscribe"` 字段应该返回订阅的服务 `demo-provider` 的信息，结果这里返回的是空。后来翻看了下源码，是需要主动向 Nacos [EventDispatcher](https://github.com/alibaba/nacos/blob/master/client/src/main/java/com/alibaba/nacos/client/naming/core/EventDispatcher.java) 注册 [EventListener](https://github.com/alibaba/nacos/blob/master/api/src/main/java/com/alibaba/nacos/api/naming/listener/EventListener.java) 才可以。咳咳咳，感觉这个设定有点神奇~

③ 访问服务消费者的 `health` 监控端点 http://127.0.0.1:28080/actuator/health，返回结果如下图：![ 监控端点](E:\Development\Typora\images\32.png)

# 666. 彩蛋

至此，我们已经完成 Spring Cloud Alibaba Nacos Discovery 的学习。如下是 Nacos 相关的官方文档：

- [《Nacos 官方文档》](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [《Spring Cloud Alibaba 官方文档 —— Nacos Discovery》](https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-discovery)
- [《Spring Cloud Alibaba 官方示例 —— Nacos Discovery》](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-examples/nacos-example/nacos-discovery-example/readme-zh.md)

另外，想要在 Spring Boot 项目中使用 Nacos 作为注册中心的胖友，可以阅读[《芋道 Spring Boot 注册中心 Nacos 入门》](http://www.iocoder.cn/Spring-Boot/registry-nacos/?self)文章。
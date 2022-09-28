# Spring Cloud 网关 Spring Cloud Gateway 入门

> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 [labx-08-spring-cloud-gateway](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway) 目录。
>
> 原创不易，给点个 [Star](https://github.com/YunaiV/SpringBoot-Labs/stargazers) 嘿，一起冲鸭！

# 1. 概述

[Spring Cloud Gateway](https://github.com/spring-cloud/spring-cloud-gateway) 是由 [WebFlux](http://www.iocoder.cn/Spring-Boot/WebFlux/?self) + [Netty](https://github.com/YunaiV/netty) + [Reactor](https://github.com/reactor/reactor) 实现的**响应式**的 API 网关。

Spring Cloud Gateway 旨在为微服务架构提供一种简单且有效的 API 路由的管理方式，并基于 Filter 的方式提供网关的基本功能，例如说安全认证、监控、限流等等。

Spring Cloud Gateway 定位于取代 Netflix [Zuul](https://github.com/Netflix/zuul)，成为 Spring Cloud 生态系统的新一代网关。目前看下来非常成功，老的项目的网关逐步从 Zuul 迁移到 Spring Cloud Gateway，新项目的网关直接采用 Spring Cloud Gateway。相比 Zuul 来说，Spring Cloud Gateway 提供**更优秀的性能，更强大的有功能**。

> 友情提示：感兴趣的胖友可以看看[《性能测试 —— Spring Cloud Gateway、Zuul 基准测试》](http://www.iocoder.cn/Performance-Testing/SpringCloudGateway-Zuul-benchmark/?self)

Spring Cloud Gateway 的特征如下：

> FROM [spring-cloud-gateway#features](https://github.com/spring-cloud/spring-cloud-gateway#features) 翻译
>
> - 基于 Java 8 编码
> - 基于 Spring Framework 5 + Project Reactor + Spring Boot 2.0 构建
> - 支持动态路由，能够匹配任何请求属性上的路由
> - 支持内置到 Spring Handler 映射中的路由匹配
> - 支持基于 HTTP 请求的路由匹配（Path、Method、Header、Host 等等）
> - 集成了 Hystrix 断路器
> - 过滤器作用于匹配的路由
> - 过滤器可以修改 HTTP 请求和 HTTP 响应（增加/修改 Header、增加/修改请求参数、改写请求 Path 等等）
> - 支持 Spring Cloud DiscoveryClient 配置路由，与服务发现与注册配合使用
> - 支持限流

特性有点多，一边入门一边了解。实际上 Spring Cloud Gateway 还提供了很多其它特性，例如说还能够作为 WebSocket 的网关。

# 2. 为什么使用网关？

胖友可以后续阅读如下文章：

- [《一文带你 API 网关从入门到放弃》](http://www.iocoder.cn/Fight/This-article-takes-you-through-the-API-gateway-from-entry-to-abandonment/?self)
- [《微服务中的网关到底是个什么鬼？》](http://www.iocoder.cn/Fight/What-the-hell-is-a-gateway-in-microservices/?self)
- [《为什么微服务一定要有网关？》](http://www.iocoder.cn/Fight/Why-must-microservices-have-gateways/?self)

🙂 先继续 Spring Cloud Gateway 的入门。

# 3. 快速入门

> 示例代码对应仓库：[`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01)

本小节我们来对 Spring Cloud Gateway 进行快速入门。创建一个 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/01-16641133763022.png)

## 3.1 引入依赖

创建 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01/pom.xml) 文件中，主要引入 Spring Cloud Gateway 相关依赖。代码如下：



```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>labx-08</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>labx-08-sc-gateway-demo01</artifactId>

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
        <!-- 引入 Spring Cloud Gateway 相关依赖，使用它作为网关，并实现对其的自动配置 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
    </dependencies>

</project>
```



在 `spring-cloud-starter-gateway` 依赖中，会引入 WebFlux、Reactor、Netty 等等依赖，如下图所示：![ 依赖](E:/Development/Typora/images/02-16641133763024.png)

## 3.2 配置文件

创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01/src/main/resources/application.yaml) 配置文件，添加 Spring Cloud Gateway 相关配置。配置如下：



```
server:
  port: 8888

spring:
  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/blog
          filters:
            - StripPrefix=1
        - id: oschina # 路由的编号
          uri: https://www.oschina.net # 路由的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/oschina
          filters: # 过滤器，对请求进行拦截，实现自定义的功能，对应 FilterDefinition 数组
            - StripPrefix=1
```



① `server.port` 配置项，设置网关的服务器端口。

② `spring.cloud.gateway` 配置项，Spring Cloud Gateway 配置项，对应 [GatewayProperties](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/config/GatewayProperties.java) 类。

这里我们主要使用了 `routes` 路由配置项，对应 [RouteDefinition](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/Route.java) 数组。路由（[Route](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/Route.java)）是 Gateway 中最基本的组件之一，由一个 ID、URI、一组谓语（Predicate）、过滤器（Filter）组成。一个请求如果满足某个路由的所有谓语，则匹配上该路由，最终过程如下图：![路由过程](E:/Development/Typora/images/03-16641133763026.png)

- ID：编号，路由的唯一标识。

- URI：路由指向的目标 URI，即请求最终被转发的目的地。

  > 例如说，这里配置的 `http://www.iocoder.cn` 或 `https://www.oschina.net`，就是被转发的地址。

- [Predicate](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/AsyncPredicate.java)：谓语，作为路由的匹配条件。Gateway 内置了多种 Predicate 的[实现](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/)，提供了多种请求的匹配条件，比如说基于请求的 Path、Method 等等。

  > 例如说，这里配置的 [`Path`](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/PathRoutePredicateFactory.java) 匹配请求的 Path 地址。

- [Filter](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/GatewayFilter.java)：过滤器，对请求进行拦截，实现自定义的功能。Gateway 内置了多种 Filter 的[实现](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/)，提供了多种请求的处理逻辑，比如说限流、熔断等等。

  > 例如说，这里配置的 [`StripPrefix`](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/StripPrefixGatewayFilterFactory.java) 会将请求的 Path 去除掉前缀。可能有点不好理解，我们以第一个 Route 举例子，假设我们请求 http://127.0.0.1:8888/blog 时：
  >
  > - 如果**有**配置 `StripPrefix` 过滤器，则转发到的最终 URI 为 [http://www.iocoder.cn](http://www.iocoder.cn/)，正确返回首页
  > - 如果**未**配置 `StripPrefix` 过滤器，转发到的最终 URI 为 http://www.iocoder.cn/blog，错误返回 404

## 3.3 GatewayApplication

创建 [GatewayApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/GatewayApplication.java) 类，网关的启动类。代码如下：



```
@SpringBootApplication
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```



## 3.4 简单测试

① 使用 GatewayApplication 启动网关。可以看到控制台打印 Spring Cloud Gateway 相关日志如下：



```
// 加载 Predicate 工厂
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [After]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Before]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Between]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Cookie]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Header]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Host]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Method]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Path]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Query]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [ReadBodyPredicateFactory]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [RemoteAddr]
2020-02-25 01:07:58.205  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [Weight]
2020-02-25 01:07:58.206  INFO 58159 --- [           main] o.s.c.g.r.RouteDefinitionRouteLocator    : Loaded RoutePredicateFactory [CloudFoundryRouteService]

// 使用 Netty 作为服务器，占用 8888 端口
2020-02-25 01:07:58.620  INFO 58159 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port(s): 8888
```



② 使用浏览器，访问 `http://127.0.0.1:8888/blog`，成功转发到目标 URI `http://www.iocoder.cn`，如下图所示：![http://www.iocoder.cn](E:/Development/Typora/images/03-16641133762921.png)

③ 使用浏览器，访问 `http://127.0.0.1:8888/oschina`，成功转发到目标 URI `http://www.oschina.net`，如下图所示：![http://www.oschina.net](E:/Development/Typora/images/04-166411337630311.png)

## 3.5 整体架构

**注意**，一定要好好理解 Route、Predicate、Filter 这三个基础组件，Gateway 绝大多数代码都是围绕它们来运转的，如下图所示：![代码](E:/Development/Typora/images/05-16641133763029.png)

我们再来看看它们在 Gateway 的整体工作流程中的作用，如下图所示：![流程图](E:/Development/Typora/images/06-166411337630313.png)

- ① Gateway 接收客户端请求。

- ② 请求与 Predicate 进行匹配，获得到对应的 Route。匹配成功后，才能继续往下执行。

- ③ 请求经过 Filter 过滤器链，执行前置（prev）处理逻辑。

  > 例如说，修改请求头信息等。

- ④ 请求被 Proxy Filter 转发至目标 URI，并最终获得响应。

  > 一般来说，目标 URI 是被代理的微服务，如果是在 Spring Cloud 架构中。

- ⑤ 响应经过 Filter 过滤器链，执行后置（post）处理逻辑。

- ⑥ Gateway 返回响应给客户端。

其实吧，整体流程和 SpringMVC 的 DispatcherServlet 差不多，只是说 SpringMVC 最终转发到 JVM 进程内的指定方法，而 Gateway 最终转发到远程的目标 URI。

> 友情提示：撸过 [SpringMVC 源码](http://www.iocoder.cn/Spring-MVC/good-collection/?self) 的胖友，欢迎来撸艿艿写的 [《芋道 Spring Cloud Gateway 源码解析系列》](http://www.iocoder.cn/categories/Spring-Cloud-Gateway/?self)。

无意中在网上翻到一张 Gateway 更清晰的整体工作流程的图，胖友可以在看看：

> FROM https://www.javainuse.com/spring/cloud-filter
>
> ![img](E:/Development/Typora/images/e3e5773f5efbcb1445355ccf9e171021.jpg)

# 4. 基于注册中心实现动态路由

> 示例代码对应仓库：
>
> - 网关：[`labx-08-sc-gateway-demo02-registry`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo02-registry)
> - 用户服务：[`labx-08-sc-user-service`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-user-service/)

在[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节，我们通过**配置文件**的方式，**手动**配置了 Gateway 路由信息。本小节，我们使用 Gateway 提供的与 Spring Cloud 注册中心的集成，从注册中心获取服务列表，并以服务名作为目标 URI 来**自动**创建**动态路由**。

我们直接从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo02-registry`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo02-registry) 项目来复制，**搭建 Gateway 基于注册中心实现动态路由的示例**。最终项目结构如下图：![ 项目](E:/Development/Typora/images/13-166411337630316.png)

> 分割线：先进行网关项目的改造。

## 4.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo02-registry/pom.xml) 文件，引入注册中心 Nacos 相关的依赖如下：



```
<!-- 引入 Spring Cloud Alibaba Nacos Discovery 相关依赖，将 Nacos 作为注册中心，并实现对其的自动配置 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```



考虑到 Nacos 作为 Spring Cloud 架构中的注册中心，已经越来越流行了，所以本小节我们使用它。感兴趣的胖友，可以后面看看艿艿写的[《芋道 Spring Cloud Alibaba 注册中心 Nacos 入门》](http://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Discovery/?self)文章。

## 4.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo02-registry/src/main/resources/application.yaml) 配置文件，增加注册中心相关的配置项。完整配置如下：



```
server:
  port: 8888

spring:
  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/blog
          filters:
            - StripPrefix=1
        - id: oschina # 路由的编号
          uri: https://www.oschina.net # 路由的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/oschina
          filters: # 过滤器，对请求进行拦截，实现自定义的功能，对应 FilterDefinition 数组
            - StripPrefix=1
      # 与 Spring Cloud 注册中心的集成，对应 DiscoveryLocatorProperties 类
      discovery:
        locator:
          enabled: true # 是否开启，默认为 false 关闭
          url-expression: "'lb://' + serviceId" # 路由的目标地址的表达式，默认为 "'lb://' + serviceId"

    # Nacos 作为注册中心的配置项
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
```



① `spring.cloud.nacos.discovery` 配置项，使用 Nacos 作为 Spring Cloud 注册中心的配置项。这里就不详细解释，毕竟 Nacos 不是主角。

② `spring.cloud.gateway.discovery` 配置项，Gateway 与 Spring Cloud 注册中心的集成，对应 [DiscoveryLocatorProperties](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/discovery/DiscoveryLocatorProperties.java) 类。

- `enable`：是否开启，默认为 `false` 关闭。这里我们设置为 `true`，开启与 Spring Cloud 注册中心的集成的功能。
- `url-expression`：路由的目标地址的 [Spring EL](https://docs.spring.io/spring/docs/4.3.10.RELEASE/spring-framework-reference/html/expressions.html) 表达式，默认为 `"'lb://' + serviceId"`。这里，我们设置的就是默认值。

可能 `url-expression` 配置项有点费解，我们来重点解释下。

- `lb://` 前缀，表示将请求**负载均衡**转发到对应的服务的实例。
- `"'lb://' + serviceId"` Spring EL 表达式，将从注册中心获得到的服务列表，每一个服务的名字对应 `serviceId`，最终使用 Spring EL 表达式进行格式化。

我们来举个例子，假设我们从注册中心假造到了 `user-service` 和 `order-service` 两个服务，最终效果和如下配置等价：



```
spring:
  cloud:
    gateway:
        routes:
            - id: ReactiveCompositeDiscoveryClient_user-service
              uri: lb://user-service
              predicates:
                - Path=/user-service/**
              filters:
                - RewritePath=/user-service/(?<remaining>.*), /${remaining} # 将 /user-service 前缀剔除
            - id: ReactiveCompositeDiscoveryClient_order-service
              uri: lb://order-service
              predicates:
                - Path=/order-service/**
              filters:
                - RewritePath=/order-service/(?<remaining>.*), /${remaining} # 将 /order-service 前缀剔除
```



友情提示：感兴趣的胖友，可以后续看看 [DiscoveryClientRouteDefinitionLocator](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/discovery/DiscoveryClientRouteDefinitionLocator.java) 类的源码，对应[《Spring-Cloud-Gateway 源码解析 —— 路由（1.4）之 DiscoveryClientRouteDefinitionLocator 注册中心》](http://www.iocoder.cn/Spring-Cloud-Gateway/route-definition-locator-discover-client/?self)文章。

> 分割线：再搭建用户服务来被 API 网关代理

## 4.3 搭建用户服务

创建 [`labx-08-sc-user-service`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-user-service/) 项目，作为 `user-service` 用户服务。代码比较简单，艿艿就不瞎哔哔了。最终项目如下图所示：![img](E:/Development/Typora/images/11-166411337630318.png)

## 4.4 简单测试

① 执行 UserServiceApplication 两次，启动两个 `user-service` 服务。启动完成后，在 Nacos 注册中心可以看到该服务的两个实例，如下图所示：![Nacos  服务](E:/Development/Typora/images/12-166411337630320.png)

② 执行 GatewayApplication 启动网关。

③ 访问 http://127.0.0.1:8888/user-service/user/get?id=1 地址，返回 JSON 结果如下：



```
{
    "id": 1,
    "name": "没有昵称：1",
    "gender": 2
}
```



请求经过网关后，转发到 `user-service` 服务成功。

# 5. 基于配置中心 Apollo 实现动态路由

> 示例代码对应仓库：
>
> - 网关：[`labx-08-sc-gateway-demo03-config-apollo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-apollo)
> - 用户服务：[`labx-08-sc-user-service`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-user-service/)

在[「4. 基于注册中心实现动态路由」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节中，我们使用 Gateway 提供的基于注册中心来**自动**创建**动态路由**的功能。但是很多时候，这个功能并不能满足我们的需求，例如说：

- 注册中心的服务这么多，我们并不想通过网关暴露所有的服务出去
- 每个服务的路由信息可能不同，会存在配置不同过滤器的情况

因此，我们可以引入配置中心 Apollo 来实现动态路由的功能，将 `spring.cloud.gateway` 配置项统一存储在 Apollo 中。同时，通过通过 Apollo 的实时监听器，在 `spring.cloud.gateway` 发生变化时，刷新内存中的路由信息。

当然，Gateway 中我们还是会使用注册中心，目的是为了获取服务的实例列表，只是不再使用 Gateway 基于注册中心来的动态路由功能而已。

我们直接从[「4. 基于注册中心实现动态路由」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo02-registry`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo02-registry) 项目，复制出本小节的 [`labx-08-sc-gateway-demo03-config-apollo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-apollo) 项目，**搭建 Gateway 基于配置中心 Apollo 实现动态路由的示例**。最终项目结构如下图：![ 项目](E:/Development/Typora/images/24-166411337630328.png)

> 分割线：先进行网关项目的改造。

## 5.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-apollo/pom.xml) 文件，引入配置中心 Apollo 相关的依赖如下：



```
<!--  引入 Apollo 客户端，内置对 Apollo 的自动化配置 -->
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.5.1</version>
</dependency>
```



## 5.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-apollo/src/main/resources/application.yaml) 配置文件，增加 Apollo 相关的配置项。完整配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，全部配置在 Apollo 中
#    gateway:

    # Nacos 作为注册中心的配置项
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址

# Apollo 相关配置项
app:
  id: ${spring.application.name} # 使用的 Apollo 的项目（应用）编号
apollo:
  meta: http://127.0.0.1:8080 # Apollo Meta Server 地址
  bootstrap:
    enabled: true # 是否开启 Apollo 配置预加载功能。默认为 false。
    eagerLoad:
      enable: true # 是否开启 Apollo 支持日志级别的加载时机。默认为 false。
    namespaces: application # 使用的 Apollo 的命名空间，默认为 application。
```



① `spring.cloud.gateway` 配置项，我们都删除了，统一在 Apollo 中进行配置。

为了演示 Gateway 启动时，从 Apollo 加载 `spring.cloud.gateway` 配置项，作为初始的路由信息，我们在 Apollo 配置如下：![Apollo 配置项](E:/Development/Typora/images/21-166411337630322.png)

- 默认将请求转发到艿艿的博客 [http://www.iocoder.cn](http://www.iocoder.cn/)，嘿嘿~

配置对应文本内容如下：



```
spring.cloud.gateway.routes[0].id = github_route
spring.cloud.gateway.routes[0].uri = http://www.iocoder.cn/
spring.cloud.gateway.routes[0].predicates[0] = Path=/**
```



② `app.id` 和 `apollo` 配置项，为 Apollo 相关配置项。这里就不详细解释，毕竟 Apollo 不是主角。感兴趣的胖友，可以阅读[《芋道 Spring Boot 配置中心 Apollo 入门》](http://www.iocoder.cn/Spring-Boot/config-apollo/?self)文章。

## 5.3 GatewayPropertiesRefresher

创建 [GatewayPropertiesRefresher](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-apollo/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/GatewayPropertiesRefresher.java) 类，监听 Apollo 中的`spring.cloud.gateway` 发生变化时，刷新内存中的路由信息。代码如下：

> 友情提示：如下的代码，我们省略了部分，避免 100 多行吓到胖友哦。



```
@Component
public class GatewayPropertiesRefresher implements ApplicationContextAware, ApplicationEventPublisherAware {

    private static final Logger logger = LoggerFactory.getLogger(GatewayPropertiesRefresher.class);

    private ApplicationContext applicationContext;
    private ApplicationEventPublisher publisher;
    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher = applicationEventPublisher;
    }

    @ApolloConfigChangeListener(interestedKeyPrefixes = "spring.cloud.gateway.") // <1>
    public void onChange(ConfigChangeEvent changeEvent) {
        refreshGatewayProperties(changeEvent);
    }

    private void refreshGatewayProperties(ConfigChangeEvent changeEvent) {
        logger.info("Refreshing GatewayProperties!");
        // <2>
        preDestroyGatewayProperties(changeEvent);
        // <3>
        this.applicationContext.publishEvent(new EnvironmentChangeEvent(changeEvent.changedKeys()));
        // <4>
        refreshGatewayRouteDefinition();
        logger.info("GatewayProperties refreshed!");
    }

    // ... 省略被调用的方法，一会说。

}
```



① `<1>` 处，通过 Apollo 提供的 [`@ApolloConfigChangeListener`](https://github.com/ctripcorp/apollo/blob/master/apollo-client/src/main/java/com/ctrip/framework/apollo/spring/annotation/ApolloConfigChangeListener.java) 注解，声明监听 `spring.cloud.gateway.` 配置项的刷新。

② `<2>` 处，调用 `#preDestroyGatewayProperties(ConfigChangeEvent changeEvent)` 方法，处理 `spring.cloud.gateway.routes` 或 `spring.cloud.gateway.default-filters` 配置项可能被**删除光**的特殊骚操作。代码如下：



```
private static final String ID_PATTERN = "spring\\.cloud\\.gateway\\.routes\\[\\d+\\]\\.id";
private static final String DEFAULT_FILTER_PATTERN = "spring\\.cloud\\.gateway\\.default-filters\\[\\d+\\]\\.name";

@Autowired
private GatewayProperties gatewayProperties;

private synchronized void preDestroyGatewayProperties(ConfigChangeEvent changeEvent) {
    logger.info("Pre Destroy GatewayProperties!");
    // 判断 `spring.cloud.gateway.routes` 配置项，是否被全部删除。如果是，则置空 GatewayProperties 的 `routes` 属性
    final boolean needClearRoutes = this.checkNeedClear(changeEvent, ID_PATTERN, this.gatewayProperties.getRoutes().size());
    if (needClearRoutes) {
        this.gatewayProperties.setRoutes(new ArrayList<>());
    }
    // 判断 `spring.cloud.gateway.default-filters` 配置项，是否被全部删除。如果是，则置空 GatewayProperties 的 `defaultFilters` 属性
    final boolean needClearDefaultFilters = this.checkNeedClear(changeEvent, DEFAULT_FILTER_PATTERN, this.gatewayProperties.getDefaultFilters().size());
    if (needClearDefaultFilters) {
        this.gatewayProperties.setRoutes(new ArrayList<>());
    }
    logger.info("Pre Destroy GatewayProperties finished!");
}

// 判断是否清除的标准，是通过指定配置项被删除的数量，是否和内存中的该配置项的数量一样。如果一样，说明被清空了
private boolean checkNeedClear(ConfigChangeEvent changeEvent, String pattern, int existSize) {
    return changeEvent.changedKeys().stream().filter(key -> key.matches(pattern))
            .filter(key -> {
                ConfigChange change = changeEvent.getChange(key);
                return PropertyChangeType.DELETED.equals(change.getChangeType());
            }).count() == existSize;
}
```



因为 GatewayProperties 没有 `@PreDestroy` 注解的 destroy 方法，所以 Spring Cloud Context 的 [`ConfigurationPropertiesRebinder#rebind(String name)`](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/properties/ConfigurationPropertiesRebinder.java#L87-L116) 中 [destroyBean](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/properties/ConfigurationPropertiesRebinder.java#L99-L100) 无法销毁**当前的** GatewayProperties Bean 对象。

- 如果把 `spring.cloud.gateway.` 前缀的配置项全部删除（例如需要动态删除最后一个路由的场景），[initializeBean](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/properties/ConfigurationPropertiesRebinder.java#L101-L102) 时也无法创建**新的** GatewayProperties Bean 对象，则 `return` **当前的** GatewayProperties Bean 对象。
- 若仍保留有 `spring.cloud.gateway.routes[n]` 或 `spring.cloud.gateway.default-filters[n]` 等配置项，[initializeBean](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/properties/ConfigurationPropertiesRebinder.java#L101-L102) 会时会注入**新的** GatewayProperties Bean 对象。

`#preDestroyGatewayProperties(ConfigChangeEvent changeEvent)` 提供类似 `@PreDestroy` 的操作，根据 `spring.cloud.gateway.` 配置项的实际情况把 `GatewayProperties.routes` 和 `GatewayProperties.defaultFilters` 两个集合情况。

> 友情提示：这块逻辑艿艿也理解了蛮久，现在还是理解的还不是很透彻，胖友可以选择性理解。

③ `<3>` 处，发布 Spring Cloud Context [EnvironmentChangeEvent](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-context/src/main/java/org/springframework/cloud/context/environment/EnvironmentChangeEvent.java) 事件，通知 `spring.cloud.gateway` 配置项发生变化，从而实现使用到注入**新的**GatewayProperties Bean。例如说，我们这里 `@Autowired` 注入的 GatewayProperties。

④ `<4>` 处，调用 `#refreshGatewayRouteDefinition()` 方法，发布 Gateway [RefreshRoutesEvent](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/event/RefreshRoutesEvent.java) 事件，从而刷新内存中的路由信息。代码如下：



```
private void refreshGatewayRouteDefinition() {
    logger.info("Refreshing Gateway RouteDefinition!");
    this.publisher.publishEvent(new RefreshRoutesEvent(this));
    logger.info("Gateway RouteDefinition refreshed!");
}
```



目前具体监听该事件的有两个：

- [CachingRouteDefinitionLocator](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/CachingRouteDefinitionLocator.java) 刷新路由**定义**
- [CachingRouteLocator](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/CachingRouteLocator.java) 刷新路由**信息**

> 分割线：再搭建用户服务来被 API 网关代理

## 5.4 搭建用户服务

创建 [`labx-08-sc-user-service`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-user-service/) 项目，作为 `user-service` 用户服务。代码比较简单，艿艿就不瞎哔哔了。最终项目如下图所示：![img](E:/Development/Typora/images/11-166411337630318.png)

## 5.5 简单测试

① 执行 UserServiceApplication 两次，启动两个 `user-service` 服务。

② 执行 GatewayApplication 启动网关。

使用浏览器，访问 http://127.0.0.1:8888/ 地址，返回艿艿的博客首页，如下图所示：![博客首页](E:/Development/Typora/images/22-166411337630324.png)

③ 修改在 Apollo 的 `spring.cloud.gateway` 配置项，转发请求到用户服务。如下图所示：![Apollo 配置项](E:/Development/Typora/images/23-166411337630326.png)

配置对应文本内容如下：



```
spring.cloud.gateway.routes[0].id = ReactiveCompositeDiscoveryClient_user-service
spring.cloud.gateway.routes[0].uri = lb://user-service
spring.cloud.gateway.routes[0].predicates[0] = Path=/user/**
spring.cloud.gateway.routes[0].filters[0] = StripPrefix=1
```



此时 IDEA 控制台看到 GatewayPropertiesRefresher 监听到 `spring.cloud.gateway` 配置项刷新，并打印日志如下：



```
2020-02-26 01:11:48.974  INFO 77076 --- [Apollo-Config-1] c.i.s.l.g.GatewayPropertiesRefresher     : Refreshing GatewayProperties!
2020-02-26 01:11:48.974  INFO 77076 --- [Apollo-Config-1] c.i.s.l.g.GatewayPropertiesRefresher     : Pre Destroy GatewayProperties!
2020-02-26 01:11:48.975  INFO 77076 --- [Apollo-Config-1] c.i.s.l.g.GatewayPropertiesRefresher     : Pre Destroy GatewayProperties finished!
2020-02-26 01:11:49.000  INFO 77076 --- [Apollo-Config-1] c.i.s.l.g.GatewayPropertiesRefresher     : Refreshing Gateway RouteDefinition!
2020-02-26 01:11:49.001  INFO 77076 --- [Apollo-Config-1] c.i.s.l.g.GatewayPropertiesRefresher     : Gateway RouteDefinition refreshed!
2020-02-26 01:11:49.001  INFO 77076 --- [Apollo-Config-1] c.i.s.l.g.GatewayPropertiesRefresher     : GatewayProperties refreshed!
```



④ 访问 http://127.0.0.1:8888/user/user/get?id=1 地址，返回 JSON 结果如下：



```
{
    "id": 1,
    "name": "没有昵称：1",
    "gender": 2
}
```



请求经过网关后，转发到 `user-service` 服务成功。

# 6. 基于配置中心 Nacos 实现动态路由

> 示例代码对应仓库：
>
> - 网关：[`labx-08-sc-gateway-demo03-config-nacos`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-nacos)
> - 用户服务：[`labx-08-sc-user-service`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-user-service/)

在[「4. 基于注册中心实现动态路由」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节中，我们使用 Gateway 提供的基于注册中心来**自动**创建**动态路由**的功能。但是很多时候，这个功能并不能满足我们的需求，例如说：

- 注册中心的服务这么多，我们并不想通过网关暴露所有的服务出去
- 每个服务的路由信息可能不同，会存在配置不同过滤器的情况

因此，我们可以引入配置中心 Nacos 来实现动态路由的功能，将 `spring.cloud.gateway` 配置项统一存储在 Nacos 中。同时，通过通过 Nacos 的实时监听器，在 `spring.cloud.gateway` 发生变化时，刷新内存中的路由信息。

当然，Gateway 中我们还是会使用注册中心，目的是为了获取服务的实例列表，只是不再使用 Gateway 基于注册中心来的动态路由功能而已。

我们直接从[「4. 基于注册中心实现动态路由」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo02-registry`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo02-registry) 项目，复制出本小节的 [`labx-08-sc-gateway-demo03-config-nacos`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-nacos) 项目，**搭建 Gateway 基于配置中心 Nacos 实现动态路由的示例**。最终项目结构如下图：![ 项目](E:/Development/Typora/images/35.png)

> 分割线：先进行网关项目的改造。

## 6.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-nacos/pom.xml) 文件，引入配置中心 Nacos 相关的依赖如下：



```
<!-- 引入 Spring Cloud Alibaba Nacos Config 相关依赖，将 Nacos 作为配置中心，并实现对其的自动配置 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```



## 6.2 配置文件

① 创建 [`bootstrap.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-nacos/src/main/resources/bootstrap.yaml) 配置文件，添加配置中心 Nacos 相关的配置。配置如下：



```
spring:
  application:
    name: gateway-application

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



`spring.cloud.nacos.config` 配置项，为配置中心 Nacos 相关配置项。这里就不详细解释，毕竟 Nacos 不是主角。感兴趣的胖友，可以阅读[《芋道 Spring Cloud Alibaba 配置中心 Nacos 入门》](http://www.iocoder.cn/Spring-Cloud-Alibaba/Nacos-Config/?self)文章。

② 修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo03-config-nacos/src/main/resources/application.yaml) 配置文件，删除 Gateway 相关的配置。完整配置如下：



```
server:
  port: 8888

spring:

  cloud:
    # Spring Cloud Gateway 配置项，全部配置在 Nacos 中
#    gateway:

    # Nacos 作为注册中心的配置项
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 # Nacos 服务器地址
```



`spring.cloud.gateway` 配置项，我们都删除了，统一在配置中心 Nacos 中进行配置。

为了演示 Gateway 启动时，从 Nacos 加载 `spring.cloud.gateway` 配置项，作为初始的路由信息，我们在 Nacos 配置如下：![Nacos 配置项](E:/Development/Typora/images/31-166411341286633.png)

- 默认将请求转发到艿艿的博客 [http://www.iocoder.cn](http://www.iocoder.cn/)，嘿嘿~

配置对应文本内容如下：



```
spring:
    cloud:
        gateway:
            routes:
            -   id: github_route
                predicates:
                - Path=/**
                uri: http://www.iocoder.cn/
```



## 6.3 Nacos 配置监听器

在 Nacos 配置发生变化时，Spring Cloud Alibaba Nacos Config [内置的监听器](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-nacos-config/src/main/java/com/alibaba/cloud/nacos/refresh/NacosContextRefresher.java#L123-L150) 会监听到配置刷新，最终触发 Gateway 的路由信息刷新。完整流程如下图所示：![配置刷新的流程](E:/Development/Typora/images/32-166411341286637.png)

- [CachingRouteDefinitionLocator](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/CachingRouteDefinitionLocator.java) 刷新路由**定义**
- [CachingRouteLocator](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/CachingRouteLocator.java) 刷新路由**信息**

> 友情提示：极端情况下，假设胖友把 `spring.cloud.gateway` 配置项**完全**删除，无法实现 Gateway 的路由信息的**全部**删除。不过一般情况下，我们貌似也不会这么干，嘿嘿~
>
> 感兴趣的胖友，可以看看[「5.3 GatewayPropertiesRefresher」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的内容，略微进行改造，进行支持。

> 分割线：再搭建用户服务来被 API 网关代理

## 6.4 搭建用户服务

创建 [`labx-08-sc-user-service`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-user-service/) 项目，作为 `user-service` 用户服务。代码比较简单，艿艿就不瞎哔哔了。最终项目如下图所示：![img](E:/Development/Typora/images/11-166411341286635.png)

## 6.5 简单测试

① 执行 UserServiceApplication 两次，启动两个 `user-service` 服务。

② 执行 GatewayApplication 启动网关。

使用浏览器，访问 http://127.0.0.1:8888/ 地址，返回艿艿的博客首页，如下图所示：![博客首页](E:/Development/Typora/images/22-166411341286639.png)

③ 修改在 Nacos 的 `spring.cloud.gateway` 配置项，转发请求到用户服务。如下图所示：[Nacos 配置项](https://static.iocoder.cn/images/Spring-Cloud/2020-08-01/33.png)

配置对应文本内容如下：



```
spring:
    cloud:
        gateway:
            routes:
            -   filters:
                - StripPrefix=1
                id: ReactiveCompositeDiscoveryClient_user-service
                predicates:
                - Path=/user/**
                uri: lb://user-service
```



此时 IDEA 控制台看到 GatewayPropertiesRefresher 监听到 `spring.cloud.gateway` 配置项刷新，并打印日志如下图：![IDEA 控制台](E:/Development/Typora/images/34-166411341286642.png)

④ 访问 http://127.0.0.1:8888/user/user/get?id=1 地址，返回 JSON 结果如下：



```
{
    "id": 1,
    "name": "没有昵称：1",
    "gender": 2
}
```



请求经过网关后，转发到 `user-service` 服务成功。

# 7. Route Predicate

Gateway **内置**了多种 Route Predicate 实现，将请求匹配到对应的 Route 上。并且，多个 Route Predicate 是可以**组合**实现，满足我们绝大多数的路由匹配规则。

而 Predicate 的创建，实际是通过其对应的 [Predicate Factory](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/RoutePredicateFactory.java) **工厂**来完成。艿艿将它们使用脑图进行梳理，最终如下图所示：![Predicate Factory 脑图](E:/Development/Typora/images/02-166411341281631.png)

一般情况下，我们主要使用 `Path` 和 `Host` 两个 Predicate，甚至只使用 `Path`。所以具体每个 Predicate 的使用，胖友可以在有需要的时候，再看看如下两篇文章：

- [《Spring Cloud Gateway 源码解析 —— 处理器 (3.1) 之 RoutePredicateFactory 路由谓语工厂》](http://www.iocoder.cn/Spring-Cloud-Gateway/handler-route-predicate-factory/?self)
- [《Spring Cloud Gateway 官方文档 —— Route Predicate Factories》](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories)

如果内置的 Predicate 无法满足胖友的需求，我们参考[《Spring Cloud Gateway —— Predicate 断言使用与自定义》](http://www.iocoder.cn/Fight/Spring-Cloud-Gateway-Predicate-Predicate-usage-and-customization/?self)文章，实现自定义的 Predicate 实现。不过貌似真的想不太到，需要自定义的场景。

# 8. 灰度发布

> 示例代码对应仓库：[`labx-08-sc-gateway-demo04`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo04)

Gateway 内置的 [**Weight** Route Predicate](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/predicate/WeightRoutePredicateFactory.java) 实现非常有趣，提供了基于**权重**的匹配条件，为网关实现[**灰度发布**](http://www.iocoder.cn/Fight/Micro-service-deployment-blue-green-deployment-rolling-deployment-grayscale-release-canary-release/?self)提供了基础。

> 灰度发布（又名金丝雀发布）是指在黑与白之间，能够平滑过渡的一种发布方式。
>
> 在其上可以进行 A/B testing，即让一部分用户继续用产品特性 A，一部分用户开始用产品特性 B，如果用户对 B 没有什么反对意见，那么逐步扩大范围，把所有用户都迁移到 B 上面来。灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。

例如说，目前线上正在运行用户服务的版本为 `1.0.0`。现在我们新开发的 `1.1.0` 版本已经完成所有测试流程，准备发布上线。考虑到平滑发布，先只将 5% 的流量转发到**新**版本，而 95% 的流量继续转发到**老**版本。如下图所示：![灰度发布](E:/Development/Typora/images/41-166411341286644.png)

新版本在线上运行稳定后，逐步将所有流量都转发到其上。还是老样子，

下面，我们来**搭建 Gateway 权重路由的示例**。从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo04`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo04) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/42-166411341286647.png)

## 8.1 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo04/src/main/resources/application.yaml) 配置文件，增加权重匹配的路由。完整配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: user-service-prod
          uri: http://www.iocoder.cn
          predicates:
            - Path=/**
            - Weight=user-service, 90
        - id: user-service-canary
          uri: https://www.oschina.net
          predicates:
            - Path=/**
            - Weight=user-service, 10
```



一共创建 `user-service` 的两个路由配置项，分别是 `user-service-prod` 和 `user-service-canary`：

- 使用 `Path` 匹配条件为 `/**`，设置**相同**的路径条件。
- 使用 `Weight` 匹配条件，设置**不同**的权重条件。其中，第一个参数为权重**分组**，需要配置成**相同**，一般和服务名相同即可；第二个参数为权重**比例**。

这里我们配置的 `uri` 暂时不是胖友可能希望的 `lb://user-service`，这是为什么呢？Gateway 的权重路由仅仅提供了灰度发布的基础，实际还是需要做一定的改造，例如说：

- 第一，Spring Cloud 微服务在注册到注册中心时，需要在元数据中带上**版本号**。例如说，`version = 1.0.0`、`version = 1.1.0`。

- 第二，Gateway 在 `lb://serviceId` 负载均衡选择请求的服务实例时，需要增加基于**版本号**的选择规则。

  > 友情提示：目前常用的负载均衡组件 Ribbon 暂未提供基于版本的负载均衡规则，需要胖友自己去略微拓展下，并不复杂噢。

因此，考虑到是示例演示，艿艿就暂时只配置了两个可爱的普通地址，嘿嘿~

## 8.2 简单测试

① 执行 GatewayApplication 启动网关。

② 使用浏览器，访问 [http://127.0.0.1:8888](http://127.0.0.1:8888/) 地址，90% 的情况下返回 [http://www.iocoder.cn](http://www.iocoder.cn/)，10% 的情况下返回 [https://www.oschina.net](https://www.oschina.net/)，符合预期~

ps：如果胖友对灰度发布比较感兴趣，可以看看 https://github.com/Nepxion/Discovery 项目。

# 9. Route Filter

> 示例代码对应仓库：[`labx-08-sc-gateway-demo05-custom-gateway-filter`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo05-custom-gateway-filter)

Gateway **内置**了多种 Route Filter 实现，对请求进行拦截，实现自定义的功能，例如说限流、熔断等等功能。并且，多个 Route Filter 是可以**组合**实现，满足我们绝大多数的路由的处理逻辑。

而 Filter 的创建，实际是通过其对应的 [Filter Factory](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/GatewayFilterFactory.java) **工厂**来完成。艿艿将它们使用脑图进行梳理，最终如下图所示：

![Filter Factory 脑图](E:/Development/Typora/images/01-166411341281632.png)

具体每个 Filter 的使用，胖友可以在有需要的时候，再看看如下两篇文章：

- [《Spring-Cloud-Gateway 源码解析 —— 过滤器 (4.2) 之 GatewayFilterFactory 过滤器工厂》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-factory/?self)
- [《Spring Cloud Gateway 官方文档 —— GatewayFilter Factories》](https://cloud.spring.io/spring-cloud-gateway/reference/html/#gatewayfilter-factories)

一般情况下，我们在 Gateway 上会去做的拓展，**主要集中在 Filter 上**，例如说接入认证服务。因此，我们来**搭建 Gateway 自定义 Filter 实现的示例**，提供“伪劣”的认证功能。还是老样子，从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo05-custom-gateway-filter`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo05-custom-gateway-filter) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/53.png)

## 9.1 AuthGatewayFilterFactory

创建 [AuthGatewayFilterFactory](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo05-custom-gateway-filter/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/filter/AuthGatewayFilterFactory.java) 类，创建认证 Filter 的工厂。代码如下：



```
@Component
public class AuthGatewayFilterFactory extends AbstractGatewayFilterFactory<AuthGatewayFilterFactory.Config> {

    public AuthGatewayFilterFactory() {
        super(AuthGatewayFilterFactory.Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        // <1> token 和 userId 的映射
        Map<String, Integer> tokenMap = new HashMap<>();
        tokenMap.put("yunai", 1);

        // 创建 GatewayFilter 对象
        return new GatewayFilter() {

            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                // <2> 获得 token
                ServerHttpRequest request = exchange.getRequest();
                HttpHeaders headers = request.getHeaders();
                String token = headers.getFirst(config.getTokenHeaderName());

                // <3> 如果没有 token，则不进行认证。因为可能是无需认证的 API 接口
                if (!StringUtils.hasText(token)) {
                    return chain.filter(exchange);
                }

                // <4> 进行认证
                ServerHttpResponse response = exchange.getResponse();
                Integer userId = tokenMap.get(token);

                // <5> 通过 token 获取不到 userId，说明认证不通过
                if (userId == null) {
                    // 响应 401 状态码
                    response.setStatusCode(HttpStatus.UNAUTHORIZED);
                    // 响应提示
                    DataBuffer buffer = exchange.getResponse().bufferFactory().wrap("认证不通过".getBytes());
                    return response.writeWith(Flux.just(buffer));
                }

                // <6> 认证通过，将 userId 添加到 Header 中
                request = request.mutate().header(config.getUserIdHeaderName(), String.valueOf(userId))
                        .build();
                return chain.filter(exchange.mutate().request(request).build());
            }

        };
    }

    public static class Config {

        private static final String DEFAULT_TOKEN_HEADER_NAME = "token";
        private static final String DEFAULT_HEADER_NAME = "user-id";

        private String tokenHeaderName = DEFAULT_TOKEN_HEADER_NAME;
        private String userIdHeaderName = DEFAULT_HEADER_NAME;

        public String getTokenHeaderName() {
            return tokenHeaderName;
        }

        public String getUserIdHeaderName() {
            return userIdHeaderName;
        }

        public Config setTokenHeaderName(String tokenHeaderName) {
            this.tokenHeaderName = tokenHeaderName;
            return this;
        }

        public Config setUserIdHeaderName(String userIdHeaderName) {
            this.userIdHeaderName = userIdHeaderName;
            return this;
        }

    }

}
```



① 在类上添加 `@Component` 注解，保证 Gateway 在加载所有 GatewayFilterFactory Bean 的时候，能够加载到我们定义的 AuthGatewayFilterFactory。

② 继承了 [AbstractGatewayFilterFactory](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/AbstractGatewayFilterFactory.java) 抽象类，并将泛型参数 `<C>` 设置为我们定义的 **AuthGatewayFilterFactory.Config** 配置类。这样，Gateway 在解析配置时，会转换成 Config 对象。

**注意**，在 AuthGatewayFilterFactory 构造方法中，也需要传递 Config 类给父构造方法，保证能够正确创建 Config 对象。

在 Config 类中，我们定义了两个属性：

- `tokenHeaderName` 属性：认证 Token 的 Header 名字，默认值为 `token`。
- `userIdHeaderName` 属性：认证后的 UserId 的 Header 名字，默认为 `user-id`。

③ 在 `#apply(Config config)` 方法中，我们通过内部类定义了需要创建的 GatewayFilter。我们来解释下整个 Filter 的逻辑：

- `<1>` 处，定义了一个存储 `token` 和 `userId` 映射的 Map，毕竟咱仅仅是一个提供“伪劣”的认证功能的 Filter。
- `<2>` 处，从请求 Header 中获取 `token`，作为认证标识。
- `<3>` 处，如果没有 `token`，则不进行认证。因为可能是无需认证的 API 接口。
- `<4>` 处，“伪劣”的认证逻辑，哈哈哈~实际场景下，一般调用远程的认证服务。
- `<5>` 处，通过 `token` 获取不到 `userId`，说明认证**不通过**，直接返回 401 状态码 + 提示文案，并**不继续** Gateway 的过滤链，最终不会转发请求给目标 URI。
- `<6>` 处，通过 `token` 获取到 `userId`，说明认证**通过**，将 `userId` 添加到请求 Header，从而实现将 `userId` 传递给目标 URI。同时，**继续** Gateway 的过滤链，执行后续的过滤器。

> 友情提示：在 [Spring Cloud Security](https://github.com/spring-cloud/spring-cloud-security) 项目中，提供了 Gateway 的支持，胖友可以后续去愁一愁噢。

## 9.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo05-custom-gateway-filter/src/main/resources/application.yaml) 配置文件，添加自定义 Filter 的配置。完整配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/blog
          filters:
            - StripPrefix=1
        - id: oschina # 路由的编号
          uri: https://www.oschina.net # 路由的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/oschina
          filters: # 过滤器，对请求进行拦截，实现自定义的功能，对应 FilterDefinition 数组
            - StripPrefix=1
      # 默认过滤器，对应 FilterDefinition 数组
      default-filters:
        - name: Auth
          args:
            token-header-name: access-token
```



`spring.cloud.gateway.default-filters` 配置项，Gateway **默认**过滤器，对所有路由都生效，对应 [FilterDefinition](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/FilterDefinition.java) 数组。这里配置了一个我们自定义的 Filter 的配置：

- `name`：过滤器名。这里我们设置为 `Auth`，因为 Gateway 默认使用 **XXX**GatewayFilterFactory 的前缀 **XXX** 为名字，因此 AuthGatewayFilterFactory 就是 Auth。
- `args`：过滤器的配置参数，对应 Map 类。这里我们设置 `token-header-name` 配置项为 `access-token`，表示从请求 Header `access-token` 获得认证 Token。

如果胖友只想给指定路由配置过滤器，可以在 `spring.cloud.gateway.routes[x].filters` 配置项中，进行自定义设置。

## 9.3 简单测试

① 执行 GatewayApplication 启动网关。

② 使用 Postman 模拟请求 Header `access-token` 为 `yutou`，演示认证**不通过**的情况，结果如下图：![认证不通过](E:/Development/Typora/images/51-166411341286650.png)

③ 使用 Postman 模拟请求 Header `access-token` 为 `yunai`，演示认证**通过**的情况，结果如下图：![认证通过](E:/Development/Typora/images/52-166411341286752.png)

# 10. 请求限流

> 示例代码对应仓库：[`labx-08-sc-gateway-demo06-rate-limiter`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo06-rate-limiter)

Gateway 内置 [RequestRateLimiterGatewayFilterFactory](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/RequestRateLimiterGatewayFilterFactory.java) 提供**请求限流**的功能。该 Filter 是基于 **Token Bucket Algorithm（令牌桶算法）\**实现的限流，同时搭配上 Redis 实现\**分布式限流**。

> FROM [《接口限流实践》](http://www.iocoder.cn/Fight/Interface-current-limiting-practices/?self)
>
> 对于很多应用场景来说，除了要求能够限制数据的平均传输速率外，还要求允许某种程度的突发传输。这时候漏桶算法可能就不合适了，令牌桶算法更为适合。如下图所示，令牌桶算法的原理是系统会以一个恒定的速度往桶里放入令牌，而如果请求需要被处理，则需要先从桶里获取一个令牌，当桶里没有令牌可取时，则拒绝服务。
>
> ![令牌桶算法示意图](E:/Development/Typora/images/ccc8a747a7ebf698ec5ec9f2e85a4203.jpg)

下面，我们来**搭建 Gateway 请求限流的使用示例**。还是老样子，从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo06-rate-limiter`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo06-rate-limiter) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/62-166411341286758.png)

## 10.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo06-rate-limiter/pom.xml) 文件，引入 Redis 相关的依赖如下：



```
<!-- 实现对 Spring Data Redis 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



## 10.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo06-rate-limiter/src/main/resources/application.yaml) 配置文件，增加限流 Filter 的配置项。完整配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/blog
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 1 # 令牌桶的每秒放的数量
                redis-rate-limiter.burstCapacity: 2 # 令牌桶的最大令牌数
                key-resolver: "#{@ipKeyResolver}" # 获取限流 KEY 的 Bean 的名字
        - id: oschina # 路由的编号
          uri: https://www.oschina.net # 路由的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/oschina
          filters: # 过滤器，对请求进行拦截，实现自定义的功能，对应 FilterDefinition 数组
            - StripPrefix=1

  # Redis 配置项
  redis:
    host: 127.0.0.1
    port: 6379
```



① 针对编号为 `yudaoyuanma` 的路由，我们在 `filter` 配置项，添加了限流过滤器 `RequestRateLimiter`，其配置参数如下：

- `redis-rate-limiter.replenishRate`：令牌桶的每秒放的数量。

- `redis-rate-limiter.burstCapacity`：令牌桶的最大令牌数。

  > `burstCapacity` 参数，我们可以近似理解为是每秒最大的请求数。因此每请求一次，都会从桶里获取掉一块令牌。
  > `replenishRate` 参数，我们可以近似理解为是每秒平均的请求数。假设在令牌桶为空的情况下，一秒最多放这么多令牌，所以最大请求书当然也是这么多。
  >
  > 实际上，在令牌桶满的情况下，每秒最大的请求数是 `burstCapacity + replenishRate`，好好理解下。

- `key-resolver`：获取限流 KEY 的 Bean 的名字。

② `spring.redis` 配置项，设置使用的 Redis 的配置。感兴趣的胖友，可以阅读[《芋道 Spring Boot Redis 入门》](http://www.iocoder.cn/Spring-Boot/Redis/?self)文章。

有一点要注意，使用的 Redis 客户端需要提供 Reative 的操作，目前能够支持的是 Lettuce 和 Redisson，也就是说我们暂时不能使用 Jedis。

## 10.3 GatewayConfig

创建 [GatewayConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo06-rate-limiter/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/config/GatewayConfig.java) 配置类，创建**获取限流 KEY 的 Bean**。代码如下：



```
@Configuration
public class GatewayConfig {

    @Bean
    public KeyResolver ipKeyResolver() {
        return new KeyResolver() {

            @Override
            public Mono<String> resolve(ServerWebExchange exchange) {
                // 获取请求的 IP
                return Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
            }

        };
    }

}
```



创建的 `ipKeyResolver` Bean 是通过解析请求的来源 IP 作为限流 KEY，这样我们就能实现基于 IP 的请求限流。

如果说，我们想要实现基于用户的请求限流，那么我们可以创建从请求中解析用户身份的 KeyResolver Bean。也就是说，通过自定义的 KeyResolver 来实现**不同粒度**的请求限流，美滋滋~

## 10.4 简单测试

① 执行 GatewayApplication 启动网关。

② 使用浏览器，连续快速访问 http://127.0.0.1:8888/blog 地址，将会出现被限流，返回 429 错误页，如下图所示：![429 错误页](E:/Development/Typora/images/61-166411341286755.png)

> 友情提示：对 Gateway 请求限流的源码感兴趣的胖友，可以阅读[《Spring Cloud Gateway 源码解析 —— 过滤器 (4.10) 之 RequestRateLimiterGatewayFilterFactory 请求限流》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-request-rate-limiter)文章。

# 11. 基于 Hystrix 实现服务容错

> 示例代码对应仓库：[`labx-08-sc-gateway-demo07-hystrix`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-hystrix/) 。

Gateway 内置 [HystrixGatewayFilterFactory](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/factory/HystrixGatewayFilterFactory.java) 整合 [Hystrix](https://github.com/Netflix/Hystrix) 组件，实现服务的容错处理。

> Hystrix 库，是 Netflix 开源的一个针对分布式系统的延迟和容错库。
>
> Hystrix 供分布式系统使用，提供延迟和容错功能，隔离远程系统、访问和第三方程序库的访问点，防止级联失败，保证复杂的分布系统在面临不可避免的失败时，仍能有其弹性。

下面，我们来**搭建 Gateway 基于 Hystrix 实现服务容错的使用示例**。还是老样子，从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo07-hystrix`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-hystrix/) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/71.png)

## 11.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-hystrix/pom.xml) 文件，引入 Hystrix 相关的依赖如下：



```
<!-- 引入 Hystrix 相关依赖，使用它实现服务容错，并实现对其的自动配置 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```



## 11.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-hystrix/src/main/resources/application.yaml) 配置文件，增加 Hystrix Filter 的配置项。完整配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: hystrix_test
          uri: http://127.0.0.1:18181
          predicates:
            - Path=/**
          filters:
            - name: Hystrix
              args:
                name: fallbackcmd # 对应的 Hystrix Command 名字
                fallbackUri: forward:/fallback # 处理 Hystrix fallback 的情况，重定向到指定地址
```



创建了一个编号为 `hystrix_test` 的路由，我们在 `filter` 配置项，添加了 `Hystrix` 过滤器，其配置参数如下：

- `name`：对应的 Hystrix Command 名字，后续可以通过 `hystrix.command.{name}` 配置项，来设置 `name` 对应的 Hystrix Command 的配置，例如说超时时间、隔离策略等等。
- `fallbackUri`：处理 Hystrix fallback 的情况，重定向到指定地址。注意，要么为空，要么必须以 `forward:` 开头。

另外，设置路由的 `uri` 为 `http://127.0.0.1:18181`，一个不存在的目标 URI，用于模拟转发请求失败，从而触发 Hystrix fallback 的情况。

## 11.3 FallbackController

创建 [FallbackController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-hystrix/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/controller/FallbackController.java) 类，提供 `/fallback` 接口，用于 Hystrix fallback 时的重定向。代码如下：



```
@RestController
public class FallbackController {

    private Logger logger = LoggerFactory.getLogger(FallbackController.class);

    @GetMapping("/fallback")
    public String fallback(ServerWebExchange exchange) {
//        URI requestUrl = exchange.getAttribute(ServerWebExchangeUtils.GATEWAY_REQUEST_URL_ATTR);
        Throwable executionException = exchange.getAttribute(ServerWebExchangeUtils.HYSTRIX_EXECUTION_EXCEPTION_ATTR);
        logger.error("[fallback][发生异常]", executionException);

        return "服务降级..." + executionException.getMessage();
    }

}
```



通过 [`ServerWebExchangeUtils.HYSTRIX_EXECUTION_EXCEPTION_ATTR`](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/support/ServerWebExchangeUtils.java#L125-L129) 属性，可以获得具体的 fallback 异常，进行具体的逻辑处理。

## 11.4 简单测试

① 执行 GatewayApplication 启动网关。

② 使用浏览器，快速访问 http://127.0.0.1:8888/ 地址，返回结果为 `服务降级...Connection refused: /127.0.0.1:18181`。因为，无法访问目标地址 `http://127.0.0.1:18181`，所以触发 Hystrix fallback 处理，转发请求到 `/fallback` 地址。

> 友情提示：对 Hystrix 的源码感兴趣的胖友，可以阅读[《芋道 Hystrix 源码解析系列》](http://www.iocoder.cn/categories/Hystrix/?self)。
>
> 另外，也推荐下 [《Spring Cloud Gateway 集成 Hystrix实战》](http://www.iocoder.cn/Fight/Spring-Cloud-Gateway-integrates-Hystrix-combat/?self)文章，提供了 HystrixGatewayFilterFactory 的源码解析。

# 12. 基于 Sentinel 实现服务容错

> 示例代码对应仓库：[`labx-08-sc-gateway-demo07-sentinel`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/) 。

本小节我们来进行 Gateway 和 Sentinel 的整合，使用 Sentinel 进行 Gateway 的流量保护。

> [Sentinel](https://github.com/alibaba/Sentinel) 是阿里中间件团队开源的，面向分布式服务架构的轻量级流量控制产品，主要以流量为切入点，从**流量控制**、**熔断降级**、**系统负载保护**等多个维度来帮助用户保护服务的稳定性。

Sentinel 提供了 [`sentinel-spring-cloud-gateway-adapter`](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-spring-cloud-gateway-adapter/) 子项目，已经对 Gateway 进行适配，所以我们只要引入它，基本就完成了 Gateway 和 Sentinel 的整合，贼方便。

> 友情提示：本小节会引用[《Sentinel 官方文档 —— 网关限流》](https://github.com/alibaba/Sentinel/wiki/网关限流)的内容。

![ 图](E:/Development/Typora/images/e7dedcb935a15730c9b5cc665edb3f9a.jpg)

下面，我们来**搭建 Gateway 基于 Sentinel 实现服务容错的使用示例**。还是老样子，从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo07-sentinel`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/85.png)

## 12.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/pom.xml) 文件，额外引入 Sentinel 相关的依赖如下：



```
<!-- 引入 Spring Cloud Alibaba Sentinel 相关依赖，使用 Sentinel 提供服务保障，并实现对其的自动配置 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>
```



## 12.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/src/main/resources/application.yaml) 配置文件，增加 Sentinel 的配置项。完整配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/**

    sentinel:
      eager: true # 是否饥饿加载。默认为 false 关闭
      transport:
        dashboard: localhost:7070 # 是否饥饿加载。
      # Sentinel 对 Spring Cloud Gateway 的专属配置项，对应 SentinelGatewayProperties 类
      scg:
        order: -2147483648 # 过滤器顺序，默认为 -2147483648 最高优先级
        fallback:
          mode: # fallback 模式，目前有三种：response、redirect、空
          # 专属 response 模式
          response-status: 429 # 响应状态码，默认为 429
          response-body: 你被 block 了... # 响应内容，默认为空
          content-type: application/json # 内容类型，默认为 application/json
          # 专属 redirect 模式
          redirect: http://www.baidu.com
```



① 为了测试方便，我们修改 `spring.cloud.gateway.routes` 配置项，所有请求都转发到艿艿的博客 [http://www.iocoder.cn](http://www.iocoder.cn/)。

② `spring.cloud.sentinel` 配置项，是 Spring Cloud Sentinel 的配置项，后续胖友可以看看[《芋道 Spring Cloud Alibaba 服务容错 Sentinel 入门》](http://www.iocoder.cn/Spring-Cloud-Alibaba/Sentinel/?self)文章。

> 友情提示：艿艿本机搭建的 Sentinel 控制台启动在 7070 端口。

③ `spring.cloud.sentinel.scg` 配置项，是 Sentinel 对 Spring Cloud Gateway 的专属配置项，对应 [SentinelGatewayProperties](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-sentinel-gateway/src/main/java/com/alibaba/cloud/sentinel/gateway/scg/SentinelGatewayProperties.java) 类。

- `order`：过滤器顺序，默认为 -2147483648 最高优先级。
- `fallback`：Sentinel fallback 的处理模式。一共有 `response`、`redirect`、**空**三种选择。

当 `fallback.mode` 选择**空**时，使用 Sentinel 定义的 [BlockRequestHandler](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-spring-cloud-gateway-adapter/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/sc/callback/BlockRequestHandler.java) 处理 fallback 的情况。默认情况下，使用 BlockRequestHandler 默认实现类 [DefaultBlockRequestHandler](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-spring-cloud-gateway-adapter/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/sc/callback/DefaultBlockRequestHandler.java)。

当然，我们也可以自己实现 BlockRequestHandler 接口，实现对 fallback 的自定义处理逻辑。例如说本示例，我们就创建了 [CustomBlockRequestHandler](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/CustomBlockRequestHandler.java) 类，进行下演示，代码如下：



```
@Component
public class CustomBlockRequestHandler implements BlockRequestHandler {

    private static final String DEFAULT_BLOCK_MSG_PREFIX = "HAHAHA ~ Blocked by Sentinel: ";

    @Override
    public Mono<ServerResponse> handleRequest(ServerWebExchange exchange, Throwable ex) {
        return ServerResponse.status(HttpStatus.TOO_MANY_REQUESTS) // 状态码
                .contentType(MediaType.TEXT_PLAIN) // 内容类型为 text/plain 纯文本
                .bodyValue(DEFAULT_BLOCK_MSG_PREFIX + ex.getClass().getSimpleName()); // 错误提示
    }

}
```



- 实际项目中，胖友可以根据自己的需要来拓展噢。

## 12.3 GatewayApplication

修改 [GatewayApplication](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/src/main/java/cn/iocoder/springcloud/labx08/gatewaydemo/GatewayApplication.java) 类的代码，声明这是一个 Spring Cloud Gateway 应用。代码如下：



```
@SpringBootApplication
public class GatewayApplication {

    public static void main(String[] args) {
        System.setProperty(SentinelConfig.APP_TYPE, ConfigConstants.APP_TYPE_SCG_GATEWAY); // 【重点】设置应用类型为 Spring Cloud Gateway
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```



## 12.4 简单测试

① 执行 GatewayApplication 启动网关。

访问 Sentinel 控制台，可以看到网关已经成功注册上。如下图所示：![Sentinel 控制台](E:/Development/Typora/images/81.png)

② 点击「流控规则」菜单，我们来给路由 `yudaoyuanma` 创建一个[**网关流控规则**](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-api-gateway-adapter-common/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/common/rule/GatewayFlowRule.java)，如下图所示：![创建流控规则](E:/Development/Typora/images/82.png)

- 创建了针对路由 `yudaoyuanma` 的**统一**流控规则，允许**每个 URL** 的 QPS 上限为 3。

使用浏览器，快速访问 [http://www.iocoder.cn](http://www.iocoder.cn/) 地址 4 次，会发现被 Sentinel 限流，返回 `HAHAHA ~ Blocked by Sentinel: ParamFlowException` 结果。该文字提示，就是我们自定义的 CustomBlockRequestHandler 提供的。

③ 下面，我们来给 `/categories/**` 路径，配置**单独**流控规则。

点击「API 管理」菜单，我们先创建一个包含 `/categories/**` 的 [**API 分组**](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-api-gateway-adapter-common/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/common/api/ApiDefinition.java)，如下图所示：![创建 API 分组](E:/Development/Typora/images/83.png)

点击「流控规则」菜单，我们再给 API 分组创建一个[**网关流控规则**](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-api-gateway-adapter-common/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/common/rule/GatewayFlowRule.java)，如下图所示：![创建流控规则](E:/Development/Typora/images/84.png)

- 创建了针对该 API 分组的**单独**流控规则，允许**每个 URL** 的 QPS 上限为 1。

使用浏览器，快速访问 http://127.0.0.1:8888/categories/Spring-Cloud/ 地址 2 次，会发现被 Sentinel 限流，返回 `HAHAHA ~ Blocked by Sentinel: ParamFlowException` 结果。

**注意**，虽然我们给 `/categories/**` 配置了**单独**的流控规则，但是通过路由配置的**统一**的流控规则也是生效的，也会作用到 `/categories/**` 上，即**叠加**的效果。

## 12.4 网关限流规则

[`sentinel-spring-cloud-gateway-adapter`](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-spring-cloud-gateway-adapter/) 项目增加了**网关限流规则**（[GatewayFlowRule](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-api-gateway-adapter-common/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/common/rule/GatewayFlowRule.java)），针对 API Gateway 的场景定制的限流规则，可以针对不同 route 或自定义的 API 分组进行限流，支持针对请求中的参数、Header、来源 IP 等进行定制化的限流。

GatewayFlowRule 的字段解释如下：

- `resource`：资源名称，可以是网关中的 route 名称或者用户自定义的 API 分组名称。

- `resourceMode`：规则是针对 API Gateway 的 route 还是用户在 Sentinel 中定义的 API 分组，默认是 route。

- `grade`：限流指标维度，同[限流规则](https://github.com/alibaba/Sentinel/wiki/流量控制)的 `grade` 字段。

- `count`：限流阈值

- `intervalSec`：统计时间窗口，单位是秒，默认是 1 秒。

- `controlBehavior`：流量整形的控制效果，同[限流规则](https://github.com/alibaba/Sentinel/wiki/流量控制)的 `controlBehavior` 字段，目前支持快速失败和匀速排队两种模式，默认是快速失败。

- `burst`：应对突发请求时额外允许的请求数目。

- `maxQueueingTimeoutMs`：匀速排队模式下的最长排队时间，单位是毫秒，仅在匀速排队模式下生效。

- ```
  paramItem
  ```

  ：参数限流配置。若不提供，则代表不针对参数进行限流，该网关规则将会被转换成普通流控规则；否则会转换成热点规则。其中的字段：

  - `parseStrategy`：从请求中提取参数的策略，目前支持提取来源 IP、Host、任意 Header 和任意 URL 参数四种策略。
  - `fieldName`：若提取策略选择 Header 模式或 URL 参数模式，则需要指定对应的 header 名称或 URL 参数名称。
  - `pattern`：参数值的匹配模式，只有匹配该模式的请求属性值会纳入统计和流控；若为空则统计该请求属性的所有值。
  - `matchStrategy`：参数值的匹配策略，目前支持精确匹配、子串匹配和正则匹配三种策略。

具体的示例，可以看看 [`sentinel-gw-flow.json`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/src/main/resources/sentinel-gw-flow.json) 配置文件，内容如下：



```
[
  {
    "resource": "yudaoyuanma",
    "count": 3
  },
  {
    "resource": "yudaoyuanma_customized_api",
    "count": 1
  }
]
```



## 12.5 API 定义分组

[`sentinel-spring-cloud-gateway-adapter`](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-spring-cloud-gateway-adapter/) 项目增加了**API 定义分组**（[ApiDefinition](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-api-gateway-adapter-common/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/common/api/ApiDefinition.java)），用户自定义的 API 定义分组，可以看做是一些 URL 匹配的组合。比如我们可以定义一个 API 叫 `my_api`，请求 path 模式为 `/foo/**` 和 `/baz/**` 的都归到 `my_api` 这个 API 分组下面。限流的时候可以针对这个自定义的 API 分组维度进行限流。

ApiDefinition 的字段解释如下：

- `apiName`：分组名。
- `predicateItems`：匹配规则([ApiPathPredicateItem](https://github.com/alibaba/Sentinel/blob/master/sentinel-adapter/sentinel-api-gateway-adapter-common/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/common/api/ApiPathPredicateItem.java))数组。

具体的示例，可以看看 [`sentinel-gw-api-group.json`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo07-sentinel/src/main/resources/sentinel-gw-api-group.json) 配置文件，内容如下：



```
[
  {
    "apiName": "yudaoyuanma_customized_api",
    "predicateItems": [
      {
        "pattern": "/categories/**",
        "matchStrategy": 1
      },
      {
        "items": [
          {
            "pattern": "/Dubbo/good-collection/",
            "matchStrategy": 0
          },
          {
            "pattern": "/SkyWalking/**",
            "matchStrategy": 1
          }
        ]
      }
    ]
  }
]
```



## 12.6 网关流控实现原理

当通过 `GatewayRuleManager` 加载网关流控规则（`GatewayFlowRule`）时，无论是否针对请求属性进行限流，Sentinel 底层都会将网关流控规则转化为热点参数规则（`ParamFlowRule`），存储在 `GatewayRuleManager` 中，与正常的热点参数规则相隔离。转换时 Sentinel 会根据请求属性配置，为网关流控规则设置参数索引（`idx`），并同步到生成的热点参数规则中。

外部请求进入 API Gateway 时会经过 Sentinel 实现的 filter，其中会依次进行 **路由/API 分组匹配**、**请求属性解析**和**参数组装**。Sentinel 会根据配置的网关流控规则来解析请求属性，并依照参数索引顺序组装参数数组，最终传入 `SphU.entry(res, args)` 中。Sentinel API Gateway Adapter Common 模块向 Slot Chain 中添加了一个 `GatewayFlowSlot`，专门用来做网关规则的检查。`GatewayFlowSlot` 会从 `GatewayRuleManager` 中提取生成的热点参数规则，根据传入的参数依次进行规则检查。若某条规则不针对请求属性，则会在参数最后一个位置置入预设的常量，达到普通流控的效果。

![img](E:/Development/Typora/images/d31d0c6f3689c2d4de609b6303c92724.jpg)

另外，Sentinel 是通过实现 GlobalFilter 接口，实现 [SentinelGatewayFilter](https://github.com/alibaba/Sentinel/blob/1.6.2/sentinel-adapter/sentinel-spring-cloud-gateway-adapter/src/main/java/com/alibaba/csp/sentinel/adapter/gateway/sc/SentinelGatewayFilter.java) 类来提供流量控制的功能。感兴趣的胖友，可以搂一搂它的源码~

# 13. Global Filter

在 Gateway 中，有**两类**过滤器：

- Route Filter 路由过滤器，对应 [GatewayFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/GatewayFilter.java) 接口
- Global Filter 全局过滤器，对应 [GlobalFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/GlobalFilter.java) 接口

两者基本是**等价**的，差别在于 Route Filter **不是全局，而是可以配置到指定路由上**。接口对比如下图：![GatewayFilter 对比 GlobalFilter](E:/Development/Typora/images/91.png)

绝大多数情况下，在 Route Filter 能满足我们的拓展需求的情况下，**优先**使用它。并且如果想要作用到所有路由上，可以通过 `spring.cloud.gateway.default-filters` 配置项来设置。另外，Global Filter 可能在未来的版本有一定的变化，官方文档描述如下：

> This interface and its usage are subject to change in future milestone releases.

下面，我们逐个看看 Gateway 提供的 Global Filter 实现，内容上会基于[《Spring Cloud Gateway 官方文档 —— Global Filters》](https://cloud.spring.io/spring-cloud-gateway/reference/html/#global-filters) 翻译 + 优化。

## 13.1 组合 RouteFilter 和 GlobalFilter 的排序

① 当请求与路由匹配时，[FilteringWebHandler](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/FilteringWebHandler.java) 将所有的 GlobalFilter 和该路由的 GatewayFilter 都到 [DefaultGatewayFilterChain](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/handler/FilteringWebHandler.java#L91-L126) 过滤器**链**中。这个合并的过滤器链的**排序**是基 [`@Ordered`](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/annotation/Order.java) 注解，或者 [Ordered](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/Ordered.java) 接口来获取排序值的。代码如下：



```
// FilteringWebHandler.java
@Override
public Mono<Void> handle(ServerWebExchange exchange) {
	// <1> 获得路由的 GatewayFilter 数组
	Route route = exchange.getRequiredAttribute(GATEWAY_ROUTE_ATTR);
	List<GatewayFilter> gatewayFilters = route.getFilters();
	
	// <2> 获得所有的 GlobalFilter 数组。注意，在 Gateway 的内部实现，GlobalFilter 会转换成 GatewayFilter。
	List<GatewayFilter> combined = new ArrayList<>(this.globalFilters);
	combined.addAll(gatewayFilters); // 合并
	// <3> 排序
	AnnotationAwareOrderComparator.sort(combined);
	// <4> 创建过滤器链
	return new DefaultGatewayFilterChain(combined).filter(exchange);
}
```



② Gateway 过滤器的执行分成了前置 **pre** 和后置 **post** 两个阶段，其中低排序值的过滤器在 pre 阶段先执行，高排序值的过滤器在 post 阶段先执行。示例代码如下：



```
// 配置三个 GlobalFilter
@Configuration
public class GatewayConfig {

    private Logger logger = LoggerFactory.getLogger(GatewayConfig.class);

    @Bean
    @Order(1)
    public GlobalFilter firstGlobalFilter() {
        return (exchange, chain) -> {
            logger.info("[first][pre]");
            return chain.filter(exchange)
                    .then(Mono.<Void>fromRunnable(() -> logger.info("[first][post]")));
        };
    }

    @Bean
    @Order(2)
    public GlobalFilter secondGatewayFilter() {
        return (exchange, chain) -> {
            logger.info("[second][pre]");
            return chain.filter(exchange)
                    .then(Mono.<Void>fromRunnable(() -> logger.info("[second][post]")));
        };
    }

    @Bean
    @Order(3)
    public GlobalFilter thirdGlobalFilter() {
        return (exchange, chain) -> {
            logger.info("[third][pre]");
            return chain.filter(exchange)
                    .then(Mono.<Void>fromRunnable(() -> logger.info("[third][post]")));
        };
    }

}

// 执行结果
## pre 正序
[first][pre]
[second][pre]
[third][pre]
## post 倒序
[third][post]
[second][post]
[first][post]
```



③ 在配置文件中，我们可以通过 `spring.cloud.routes[n].filters` 和 `spring.cloud.gateway.default-filters` 配置项，进行路由的 GatewayFilter 的设置，它们的排序值**分别**从 1 开始。示例如下：



```
# 配置文件
spring:
  cloud:
    gateway:
      routes:
        - id: oschina
          uri: https://www.oschina.net
          predicates:
            - Path=/oschina
          filters: # 指定过滤器
            - StripPrefix=100 # 排序值为 1
            - StripPrefix=200 # 排序值为 2
      default-filters: # 默认过滤器
        - StripPrefix=1 # 排序值为 1
        - StripPrefix=2 # 排序值为 2
        - StripPrefix=3 # 排序值为 3

# 排序后结果
## 相同排序值，`default-filters` 排在 `routes[n].filters` 前面
- StripPrefix=1 # 排序值为 1
- StripPrefix=100 # 排序值为 1
- StripPrefix=2 # 排序值为 2
- StripPrefix=200 # 排序值为 2
- StripPrefix=3 # 排序值为 3
```



另外，GlobalFilter 一般排序值**比较大**，所以主要排在 GatewayFilter 后面。

## 13.2 Forward Routing Filter

对应 [ForwardRoutingFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/ForwardRoutingFilter.java) 实现类，排序值为 `Integer.MAX_VALUE`，在路由的目标 URL 为 **`forward://` 开头** 的时候，则会使用 WebFlux 的 [DispatcherHandler](https://github.com/spring-projects/spring-framework/blob/master/spring-webflux/src/main/java/org/springframework/web/reactive/DispatcherHandler.java) 处理该请求，即转发该请求到本地 WebFlux 定义的 API 接口。

> 感兴趣的胖友，可以阅读[《Spring Cloud Gateway 源码解析 —— 过滤器 (4.5) 之 ForwardRoutingFilter》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-forward-routing/?self)文章。

## 13.3 The LoadBalancerClient Filter

对应 [LoadBalancerClientFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/LoadBalancerClientFilter.java) 实现类，排序值为 `10100`，在路由的目标 URL 为 **`lb://` 开始** 的时候，会是使用 Spring Cloud [LoadBalancerClient](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-commons/src/main/java/org/springframework/cloud/client/loadbalancer/LoadBalancerClient.java) 客户端，从服务 `serviceId` 的实例列表中选择一个，然后交给后面负责转发请求的 Filter 来调用该服务实例（例如说 [NettyRoutingFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/NettyRoutingFilter.java)）。通过这样的方式，实现 Gateway 对服务的**负载均衡**功能。

> 感兴趣的胖友，可以阅读[《Spring Cloud Gateway 源码解析 —— 过滤器 (4.5) 之 ForwardRoutingFilter》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-forward-routing/?self)文章。

## 13.4 The ReactiveLoadBalancerClientFilter

对应 [ReactiveLoadBalancerClientFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/ReactiveLoadBalancerClientFilter.java) 实现类，排序值为 `10150`，和 LoadBalancerClientFilter 在功能上是**等价**的，差别在于使用 Spring Cloud 响应式的 [ReactorLoadBalancer](https://github.com/spring-cloud/spring-cloud-commons/blob/master/spring-cloud-loadbalancer/src/main/java/org/springframework/cloud/loadbalancer/core/ReactorLoadBalancer.java) 客户端，进一步提升 Gateway 的性能。

## 13.5 The Netty Routing Filter

对应 [NettyRoutingFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/NettyRoutingFilter.java) 实现类，排序值为 `Integer.MAX_VALUE`，在路由的目标 URL 为 **`http://` 和 `https://` 开头** 的时候，使用基于 Netty 封装的 [HttpClient](https://github.com/reactor/reactor-netty/blob/master/src/main/java/reactor/netty/http/client/HttpClient.java) 请求目标 URL。

> 感兴趣的胖友，可以阅读[《Spring Cloud Gateway 源码解析 —— 过滤器 (4.7) 之 NettyRoutingFilter》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-netty-routing/?self)文章。

## 13.6 NettyWriteResponseFilter

对应 [NettyWriteResponseFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/NettyWriteResponseFilter.java) 实现类，排序值为 `-1`，搭配 NettyRoutingFilter 一起使用，将请求目标 URL 的响应结果写回给客户端。

> 感兴趣的胖友，可以阅读[《Spring Cloud Gateway 源码解析 —— 过滤器 (4.7) 之 NettyRoutingFilter》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-netty-routing/?self)文章。

## 13.7 The RouteToRequestUrl Filter

对应 [RouteToRequestUrl](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/RouteToRequestUrlFilter.java) 实现类，排序值为 `10000`，从 Route 路由中提取目标 URL，结合客户端当前请求网关的 URL，进行**拼接**后，添加到 [`GATEWAY_REQUEST_URL_ATTR`](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/support/ServerWebExchangeUtils.java#L86-L89) 中。这样，后续的 LoadBalancerClientFilter、NettyRoutingFilter、ForwardRoutingFilter 等过滤器，通过从 `GATEWAY_REQUEST_URL_ATTR` 来获取路由的目标 URL。

简单来说，RouteToRequestUrl 生成**初始的**目标 URL。

> 感兴趣的胖友，可以阅读[《Spring Cloud Gateway 源码解析 —— 过滤器 (4.3) 之 RouteToRequestUrlFilter》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-route-to-request/?self)文章。

## 13.8 Websocket Routing Filter

对应 [WebsocketRoutingFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/WebsocketRoutingFilter.java) 实现类，排序值为 `Integer.MAX_VALUE - 1`，在路由的目标 URL 为 **`ws://` 和 `wss://` 开头** 的时候，和被代理的 WebSocket 服务建立连接，并转发请求到其上。

> 感兴趣的胖友，可以阅读[《Spring-Cloud-Gateway 源码解析 —— 过滤器 (4.6) 之 WebSocketRoutingFilter》](http://www.iocoder.cn/Spring-Cloud-Gateway/filter-websocket-routing/?self)文章。

## 13.9 The Gateway Metrics Filter

对应 [GatewayMetricsFilter](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/GatewayMetricsFilter.java) 实现类，排序值为 `-2`，负责 Gateway 监控 Metrics 数据的收集。GatewayMetricsFilter 目前提供一个名为 `gateway.request` 的 [Timer](https://github.com/micrometer-metrics/micrometer/blob/master/micrometer-core/src/main/java/io/micrometer/core/instrument/Timer.java) 类型 Metrics，包含如下信息：

- `routeId`：路由的编号

- `routeUri`：路由的目标 URI。

- `httpMethod`：请求方法。

- `outcome`：结果，由 [HttpStatus.Series](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/http/HttpStatus.java#L444-L450) 分类。

- `status`：返回给客户端的请求的 HTTP 状态。

  > status: The HTTP status of the request returned to the client.

- `httpStatusCode`：返回给客户端的请求的 HTTP 状态。

  > httpStatusCode: The HTTP Status of the request returned to the client.

**开启** Gateway 的 Metrics 功能，需要引入 `spring-boot-starter-actuator` 依赖，并不主动设置 `spring.cloud.gateway.metrics.enabled` 配置项为 `false`。

> 对 Spring Boot Actuator 感兴趣的胖友，可以阅读[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)文章。

同时，Gateway Metrics 数据会暴露在 `/actuator/metrics/gateway.requests` 端点上。后续我们可以使用 Prometheus 从该端点采集 Gateway Metrics 数据，并使用 Grafana 创建 dashboard 来监控 Gateway。如下图所示：

> 通过 Spring Cloud Gateway 提供的 [gateway-grafana-dashboard.json](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/docs/src/main/asciidoc/gateway-grafana-dashboard.json)

![Gateway 监控](E:/Development/Typora/images/a79c67e29f7c6094946b67ed813351c0.jpg)

> 对 Prometheus + Grafana 感兴趣的胖友，可以阅读[《芋道 Spring Boot 监控平台 Prometheus + Grafana 入门》](http://www.iocoder.cn/Spring-Boot/Prometheus-and-Grafana/?self)文章。

## 13.10 标记请求已完成路由

当 Gateway 完成路由时，例如说符合 NettyRoutingFilter 转发请求到路由的目标 URL，则会调用 [`ServerWebExchangeUtils#setAlreadyRouted(ServerWebExchange exchange)`](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/support/ServerWebExchangeUtils.java#L171-L173) 方法，标记请求已经完成路由。

后续，其它也负责转发请求的过滤器，会调用 [`ServerWebExchangeUtils#isAlreadyRouted()`](https://github.com/spring-cloud/spring-cloud-gateway/blob/2.2.x/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/support/ServerWebExchangeUtils.java#L179-L181) 方法，判断请求是否已经完成路由，如果是，则不再转发请求到路由的目标 URL。如下图所示：![ServerWebExchangeUtils](E:/Development/Typora/images/92.png)

通过这样的方式，**避免重复转发请求**。

## 13.11 自定义 GlobalFilter

在 Gateway 的 [`filter`](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/filter/) 包下，还提供了其它 GlobalFilter 的实现类，胖友可以自己再挖掘挖掘。

另外，如果胖友想要自定义 GlobalFilter 实现类，可以参考[《Spring Cloud Gateway 自定义 Token 校验过滤器》](http://www.iocoder.cn/Fight/Spring-Cloud-Gateway-custom-Token-validation-filters/?self)文章。

# 14. 监控端点

> 示例代码对应仓库：[`labx-08-sc-gateway-demo09-actuator`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo09-actuator/) 。

Spring Cloud Gateway 的 [`actuate`](https://github.com/spring-cloud/spring-cloud-gateway/tree/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/actuate) 模块，基于 Spring Boot Actuator，提供了自定义监控端点 `gateway`，提供了 Gateway 的各种监控管理的功能。整理如下表格：

| 路径                               | 用途                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| GET `/globalfilters`               | 获得所有 GlobalFilter                                        |
| GET `/routefilters`                | 获得所有 GatewayFilterFactory                                |
| GET `/routepredicates`             | 获得所有 RoutePredicateFactory                               |
| GET `/routes`                      | 获得所有路由                                                 |
| GET `/routes/{id}`                 | 获得指定路由                                                 |
| GET `/routes/{id}/combinedfilters` | 获得指定路由的过滤器                                         |
| POST `/routes/{id}`                | 新增或修改路由，参数为 [RouteDefinition](https://github.com/spring-cloud/spring-cloud-gateway/blob/master/spring-cloud-gateway-core/src/main/java/org/springframework/cloud/gateway/route/RouteDefinition.java) |
| DELETE `/routes/{id}`              | 删除指定路由                                                 |
| POST `/refresh`                    | 刷新路由缓存                                                 |

注意，所有的**基础路径**为 `/actuator/gateway/`，所以 `/globalfilters` 对应的**完整路径**是 `/actuator/gateway/globalfilters`。

下面，我们来**搭建 Gateway 的监控端点的使用示例**。还是老样子，从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo09-actuator`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo09-actuator/) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/104.png)

## 14.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo09-actuator/pom.xml) 文件中，额外引入 Spring Boot Actuator 相关依赖。代码如下：



```
<!-- 实现对 Actuator 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



## 14.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-07-spring-cloud-alibaba-dubbo/labx-07-sca-dubbo-demo02/labx-07-sca-dubbo-demo02-provider-springmvc/src/main/resources/application.yaml) 配置文件，**额外**增加 Spring Boot Actuator 配置项。完成配置如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

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



① 增加 `management` 配置项，设置的 Actuator 配置。每个配置项的作用，胖友看下艿艿添加的注释。如果还不理解的话，后续看下[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)文章。

② 删除 `spring.cloud.gateway` 配置项，我们稍后使用 Gateway 提供的监控端点 `gateway` 来新增路由。

## 14.3 获取 Filter 和 Predicate

执行 GatewayApplication 启动网关。

① 访问 http://127.0.0.1:8888/actuator/gateway/globalfilters 地址，获得所有 GlobalFilter。结果如下：



```
{
    "org.springframework.cloud.gateway.filter.NettyWriteResponseFilter@655f69da": -1,
    "org.springframework.cloud.gateway.filter.ForwardPathFilter@2e86807a": 0,
    "org.springframework.cloud.gateway.filter.NettyRoutingFilter@6bee793f": 2147483647,
    "org.springframework.cloud.gateway.filter.AdaptCachedBodyGlobalFilter@2b0b7e5a": -2147482648,
    "org.springframework.cloud.gateway.config.GatewayNoLoadBalancerClientAutoConfiguration$NoLoadBalancerClientFilter@208f0007": 10100,
    "org.springframework.cloud.gateway.filter.ForwardRoutingFilter@30893e08": 2147483647,
    "org.springframework.cloud.gateway.filter.WebsocketRoutingFilter@4548d254": 2147483646,
    "org.springframework.cloud.gateway.filter.RemoveCachedBodyFilter@43b5021c": -2147483648,
    "org.springframework.cloud.gateway.filter.GatewayMetricsFilter@590f0c50": 0,
    "org.springframework.cloud.gateway.filter.RouteToRequestUrlFilter@28369db0": 10000
}
```



可以方便的看到每个 GlobalFilter 的排序值。

② 访问 http://127.0.0.1:8888/actuator/gateway/routefilters 地址，获得所有 GatewayFilterFactory。结果如下：



```
{
    "[RetryGatewayFilterFactory@2ceee4b6 configClass = RetryGatewayFilterFactory.RetryConfig]": null,
    "[RemoveRequestHeaderGatewayFilterFactory@6968c1d6 configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[RemoveResponseHeaderGatewayFilterFactory@77f991c configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[RewriteResponseHeaderGatewayFilterFactory@3f6bf8aa configClass = RewriteResponseHeaderGatewayFilterFactory.Config]": null,
    "[ModifyRequestBodyGatewayFilterFactory@1319bc2a configClass = ModifyRequestBodyGatewayFilterFactory.Config]": null,
    "[PrefixPathGatewayFilterFactory@798dad3d configClass = PrefixPathGatewayFilterFactory.Config]": null,
    "[SetResponseHeaderGatewayFilterFactory@e784320 configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[RewriteLocationResponseHeaderGatewayFilterFactory@13c8ac77 configClass = RewriteLocationResponseHeaderGatewayFilterFactory.Config]": null,
    "[AddRequestParameterGatewayFilterFactory@72976b4 configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[SetPathGatewayFilterFactory@12a470dd configClass = SetPathGatewayFilterFactory.Config]": null,
    "[RequestHeaderToRequestUriGatewayFilterFactory@7a9eccc4 configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[SecureHeadersGatewayFilterFactory@6968bcec configClass = Object]": null,
    "[MapRequestHeaderGatewayFilterFactory@6a1ef65c configClass = MapRequestHeaderGatewayFilterFactory.Config]": null,
    "[SetStatusGatewayFilterFactory@4bdf configClass = SetStatusGatewayFilterFactory.Config]": null,
    "[SaveSessionGatewayFilterFactory@4cad79bc configClass = Object]": null,
    "[DedupeResponseHeaderGatewayFilterFactory@e280403 configClass = DedupeResponseHeaderGatewayFilterFactory.Config]": null,
    "[SetRequestHeaderGatewayFilterFactory@5568c66f configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[RequestSizeGatewayFilterFactory@78422efb configClass = RequestSizeGatewayFilterFactory.RequestSizeConfig]": null,
    "[RemoveRequestParameterGatewayFilterFactory@7d986d83 configClass = AbstractGatewayFilterFactory.NameConfig]": null,
    "[RewritePathGatewayFilterFactory@3a7e365 configClass = RewritePathGatewayFilterFactory.Config]": null,
    "[AddRequestHeaderGatewayFilterFactory@92d1782 configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[StripPrefixGatewayFilterFactory@2c63762b configClass = StripPrefixGatewayFilterFactory.Config]": null,
    "[PreserveHostHeaderGatewayFilterFactory@27abb6ca configClass = Object]": null,
    "[ModifyResponseBodyGatewayFilterFactory@42f85fa4 configClass = ModifyResponseBodyGatewayFilterFactory.Config]": null,
    "[AddResponseHeaderGatewayFilterFactory@726934e2 configClass = AbstractNameValueGatewayFilterFactory.NameValueConfig]": null,
    "[RedirectToGatewayFilterFactory@696db620 configClass = RedirectToGatewayFilterFactory.Config]": null,
    "[RequestHeaderSizeGatewayFilterFactory@5f05c8d6 configClass = RequestHeaderSizeGatewayFilterFactory.Config]": null
}
```



③ 访问 http://127.0.0.1:8888/actuator/gateway/routepredicates 地址，获得所有 RoutePredicateFactory。结果如下：



```
{
    "[QueryRoutePredicateFactory@2e13f304 configClass = QueryRoutePredicateFactory.Config]": null,
    "[PathRoutePredicateFactory@60a7e509 configClass = PathRoutePredicateFactory.Config]": null,
    "[BetweenRoutePredicateFactory@59b32539 configClass = BetweenRoutePredicateFactory.Config]": null,
    "[MethodRoutePredicateFactory@18a19e configClass = MethodRoutePredicateFactory.Config]": null,
    "[ReadBodyPredicateFactory@787508ca configClass = ReadBodyPredicateFactory.Config]": null,
    "[WeightRoutePredicateFactory@6274670b configClass = WeightConfig]": null,
    "[HeaderRoutePredicateFactory@233db8e9 configClass = HeaderRoutePredicateFactory.Config]": null,
    "[RemoteAddrRoutePredicateFactory@3d24420b configClass = RemoteAddrRoutePredicateFactory.Config]": null,
    "[CookieRoutePredicateFactory@5b47731f configClass = CookieRoutePredicateFactory.Config]": null,
    "[CloudFoundryRouteServiceRoutePredicateFactory@53ce2392 configClass = Object]": null,
    "[AfterRoutePredicateFactory@5d94a2dc configClass = AfterRoutePredicateFactory.Config]": null,
    "[BeforeRoutePredicateFactory@cedee22 configClass = BeforeRoutePredicateFactory.Config]": null,
    "[HostRoutePredicateFactory@40c2ce52 configClass = HostRoutePredicateFactory.Config]": null
}
```



## 14.4 路由管理

① 使用 Postman 请求 http://127.0.0.1:8888/actuator/gateway/routes/{id} 地址，新增一个路由。如下图所示：![新增一个路由](E:/Development/Typora/images/101.png)

② 访问 http://127.0.0.1:8888/actuator/gateway/routes/ 地址，获得所有路由。结果如下：



```
[]
```



竟然是空，啥子情况？！因为 `/routes` 获取的是**缓存**的路由，需要调用 `/refresh` 来刷新缓存。实际上，Gateway 生效的也是**缓存**的路由。

③ 使用 Postman 请求 http://127.0.0.1:8888/actuator/gateway/refresh 地址，刷新路由缓存。如下图所示：![刷新路由缓存](E:/Development/Typora/images/102.png)

④ 再访问 http://127.0.0.1:8888/actuator/gateway/routes/ 地址，获得所有路由。结果如下：



```
[
    {
        "predicate": "Paths: [/**], match trailing slash: true",
        "route_id": "yudaoyuanma",
        "filters": [
            "[[AddRequestHeader X-Request-Foo = 'Bar'], order = 1]"
        ],
        "uri": "http://www.iocoder.cn:80",
        "order": 0
    }
]
```



美滋滋，看到刚新增的路由了。

⑤ 使用浏览器，访问 [http://127.0.0.1:888](http://127.0.0.1:888/) 地址，成功路由到艿艿博客的首页。如下图所示：![博客首页](E:/Development/Typora/images/103.png)

## 14.5 Gateway Metrics

访问 http://127.0.0.1:8888/actuator/metrics/gateway.requests 地址，获得名字为 `gateway.requests` 的 Gateway Metrics 数据。结果如下：



```
{
  "name": "gateway.requests",
  "description": null,
  "baseUnit": "seconds",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 1
    },
    {
      "statistic": "TOTAL_TIME",
      "value": 0.114374697
    },
    {
      "statistic": "MAX",
      "value": 0.114374697
    }
  ],
  "availableTags": [
    {
      "tag": "routeUri",
      "values": [
        "http://www.iocoder.cn:80"
      ]
    },
    {
      "tag": "routeId",
      "values": [
        "yudaoyuanma"
      ]
    },
    {
      "tag": "httpMethod",
      "values": [
        "GET"
      ]
    },
    {
      "tag": "outcome",
      "values": [
        "REDIRECTION"
      ]
    },
    {
      "tag": "status",
      "values": [
        "NOT_MODIFIED"
      ]
    },
    {
      "tag": "httpStatusCode",
      "values": [
        "304"
      ]
    }
  ]
}
```



如果胖友想使用 Prometheus 采集 `gateway.requests` 的 Metrics 数据，可以参考[《芋道 Spring Boot 监控平台 Prometheus + Grafana 入门》](http://www.iocoder.cn/Spring-Boot/Prometheus-and-Grafana/?self)文章。

# 15. 故障排查

> 示例代码对应仓库：[`labx-08-sc-gateway-demo10-troubleshooting`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo10-troubleshooting/) 。

在[《Spring Cloud Gateway 官方文档 —— Troubleshooting》](https://cloud.spring.io/spring-cloud-gateway/reference/html/#troubleshooting)中，分享了有哪些手段帮助我们进行 Gateway 的故障排查。例如说，路由为什么不匹配，请求和响应的具体内容是什么等等。

下面，我们来**搭建 Gateway 故障排查的示例**。还是老样子，从[「3. 快速入门」](https://www.iocoder.cn/Spring-Cloud/Spring-Cloud-Gateway/?github#)小节的 [`labx-08-sc-gateway-demo01`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo01) 项目，复制出本小节的 [`labx-08-sc-gateway-demo10-troubleshooting`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo10-troubleshooting/) 项目，最终项目结构如下图：![ 项目](E:/Development/Typora/images/113.png)

## 15.1 日志级别

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo10-troubleshooting/src/main/resources/application.yaml) 配置文件，修改日志级别相关的配置如下：



```
logging:
  level:
    reactor.netty: DEBUG
    org.springframework.cloud.gateway: TRACE
```



这样，Spring Cloud Gateway 和 Reactor Netty 可以打印更多的日志。

**简单测试**

① 执行 GatewayApplication 启动网关。

② 访问 http://127.0.0.1:8888/blog 地址，可以看到控制台打印日志如下图：![日志](E:/Development/Typora/images/111.png)

## 15.2 Wiretap

Reactor Netty 提供 Wiretap（窃听）功能，让 Reactor Netty 打印包含请求和响应信息的日志，例如说请求和响应的 Header、Body 等等。

开启 Reactor Netty 的 Wiretap 功能一共有三个配置项：

- 设置 `reactor.netty` 配置项为 `DEBUG` 或 `TRACE`。
- 设置 `spring.cloud.gateway.httpserver.wiretap` 配置项为 `true`，开启 [HttpServer Wiretap](https://projectreactor.io/docs/netty/milestone/reference/index.html#_wire_logger_2) 功能。
- 设置 `spring.cloud.gateway.httpclient.wiretap` 配置项为 `true`，开启 [HttpClient Wiretap](https://projectreactor.io/docs/netty/milestone/reference/index.html#_wire_logger) 功能。

因此 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/labx-08-spring-cloud-gateway/labx-08-sc-gateway-demo10-troubleshooting/src/main/resources/application.yaml) 配置文件的完整内容如下：



```
server:
  port: 8888

spring:
  application:
    name: gateway-application

  cloud:
    # Spring Cloud Gateway 配置项，对应 GatewayProperties 类
    gateway:
      # 路由配置项，对应 RouteDefinition 数组
      routes:
        - id: yudaoyuanma # 路由的编号
          uri: http://www.iocoder.cn # 路由到的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/blog
          filters:
            - StripPrefix=1
        - id: oschina # 路由的编号
          uri: https://www.oschina.net # 路由的目标地址
          predicates: # 断言，作为路由的匹配条件，对应 RouteDefinition 数组
            - Path=/oschina
          filters: # 过滤器，对请求进行拦截，实现自定义的功能，对应 FilterDefinition 数组
            - StripPrefix=1

      # Reactor Netty 相关配置
      httpserver:
        wiretap: true
      httpclient:
        wiretap: true

logging:
  level:
    reactor.netty: DEBUG
    org.springframework.cloud.gateway: TRACE
```



**简单测试**

① 执行 GatewayApplication 启动网关。

② 访问 http://127.0.0.1:8888/blog 地址，可以看到控制台打印日志如下图：![日志](E:/Development/Typora/images/112.png)

# 666. 彩蛋

至此，我们已经完成 Spring Cloud Gateway 的学习。如下是 Gateway 相关的官方文档：

- [《Spring Cloud Gateway 官方文档》](https://cloud.spring.io/spring-cloud-gateway/reference/html/)
- [《Spring Cloud 中文文档 —— Spring Cloud Gateway》](https://www.docs4dev.com/docs/zh/spring-cloud/Greenwich.RELEASE/reference/multi__spring_cloud_gateway.html#xv-spring-cloud-gateway)

哈哈，如果胖友对 Gateway 的源码感兴趣，可以看看[《Spring Cloud Gateway 源码解析系列》](http://www.iocoder.cn/categories/Spring-Cloud-Gateway/?self)
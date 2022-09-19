# Spring Boot-自动配置原理

# 1. 概述

Spring Boot 自动配置，顾名思义，是希望能够自动配置，将我们从配置的苦海中解脱出来。那么既然要自动配置，它需要解三个问题：

- 满足什么样的**条件**？
- 创建**哪些** Bean？
- 创建的 Bean 的**属性**？

我们来举个示例，对照下这三个问题。在我们引入 [`spring-boot-starter-web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) 依赖，会创建一个 8080 端口的内嵌 Tomcat，同时可以通过 `application.yaml` 配置文件中的 `server.port` 配置项自定义端口。那么这三个问题的答案如下：

> 友情提示：为了更易懂，这里的答案暂时是表象的，不绝对精准。

- 满足什么样的**条件**？因为我们引入了 `spring-boot-starter-web` 依赖。
- 创建**哪些** Bean？创建了一个内嵌的 Tomcat Bean，并进行启动。
- 创建的 Bean 的**属性**？通过 `application.yaml` 配置文件的 `server.port` 配置项，定义 Tomcat Bean 的启动端口属性，并且默认值为 8080。

壮着胆子，我们来看看 Spring Boot 提供的 [EmbeddedWebServerFactoryCustomizerAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/embedded/EmbeddedWebServerFactoryCustomizerAutoConfiguration.java) 类，负责创建内嵌的 Tomcat、Jetty 等等 Web 服务器的配置类。代码如下：



```java
@Configuration // <1.1>
@ConditionalOnWebApplication // <2.1>
@EnableConfigurationProperties(ServerProperties.class) // <3.1>
public class  EmbeddedWebServerFactoryCustomizerAutoConfiguration {

	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration // <1.2>
	@ConditionalOnClass({ Tomcat.class, UpgradeProtocol.class })
	public static class TomcatWebServerFactoryCustomizerConfiguration {

		@Bean
		public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			// <3.2>
			return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration // <1.3>
	@ConditionalOnClass({ Server.class, Loader.class, WebAppContext.class })
	public static class JettyWebServerFactoryCustomizerConfiguration {

		@Bean
		public JettyWebServerFactoryCustomizer jettyWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			 // <3.3>
			return new JettyWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	// ... 省略 UndertowWebServerFactoryCustomizerConfiguration 代码

	/**
	 * Nested configuration if Netty is being used.
	 */
	// ... 省略 NettyWebServerFactoryCustomizerConfiguration 代码

}
```



在开始看代码之前，我们先来简单科普下 [Spring JavaConfig](https://docs.spring.io/spring-javaconfig/docs/1.0.0.M4/reference/html/) 的小知识。在 Spring3.0 开始，Spring 提供了 JavaConfig 的方式，允许我们使用 Java 代码的方式，进行 Spring Bean 的创建。示例代码如下：

```java
@Configuration
public class DemoConfiguration {

    @Bean
    public void object() {
        return new Obejct();
    }

}
```



- 通过在**类**上添加 [`@Configuration`](https://docs.spring.io/spring-javaconfig/docs/1.0.0.M4/reference/html/ch02.html#d0e270) 注解，声明这是一个 Spring 配置类。
- 通过在**方法**上添加 [`@Bean`](https://docs.spring.io/spring-javaconfig/docs/1.0.0.M4/reference/html/ch02s02.html) 注解，声明该方法创建一个 Spring Bean。



OK，现在我们在回过头看看 `EmbeddedWebServerFactoryCustomizerAutoConfiguration` 的代码，我们分成三块内容来讲，刚好解决我们上面说的三个问题：

- ① 配置类
- ② 条件注解
- ③ 配置属性

**① 配置类**

`<1.1>` 处，在类上添加了 `@Configuration` 注解，声明这是一个**配置类**。**因为它的目的是自动配置，所以类名以 AutoConfiguration 作为后缀。**

`<1.2>`、`<1.3>` 处，分别是用于初始化 Tomcat、Jetty 相关 Bean 的配置类。

- TomcatWebServerFactoryCustomizerConfiguration 配置类，负责创建 [TomcatWebServerFactoryCustomizer](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/embedded/TomcatWebServerFactoryCustomizer.java) Bean，从而初始化内嵌的 Tomcat 并进行启动。
- JettyWebServerFactoryCustomizer 配置类，负责创建 [JettyWebServerFactoryCustomizer](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/embedded/JettyWebServerFactoryCustomizer.java) Bean，从而初始化内嵌的 Jetty 并进行启动。

**如此，我们可以得到结论一，通过 `@Configuration` 注解的配置类，可以解决“创建哪些 Bean”的问题。**

实际上，Spring Boot 的 [spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure) 项目，提供了大量框架的自动配置类，稍后我们在[「2. 自动配置类」](https://www.iocoder.cn/Spring-Boot/autoconfigure/?self#)小节详细展开。

**② 条件注解**

`<2>` 处，在类上添加了 [`@ConditionalOnWebApplication`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/condition/ConditionalOnWebApplication.java) **条件注解**，表示当前配置类需要在当前项目是 Web 项目的条件下，才能生效。在 Spring Boot 项目中，会将项目类型分成 Web 项目（使用 SpringMVC 或者 WebFlux）和非 Web 项目。这样我们就很容易理解，为什么 EmbeddedWebServerFactoryCustomizerAutoConfiguration 配置类会要求在项目类型是 Web 项目，只有 Web 项目才有必要创建内嵌的 Web 服务器呀。

`<2.1>`、`<2.2>` 处，在类上添加了 [`@ConditionalOnClass`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/condition/ConditionalOnClass.java) **条件注解**，表示当前配置类需要在当前项目有指定类的条件下，才能生效。

- TomcatWebServerFactoryCustomizerConfiguration 配置类，需要有 [`tomcat-embed-core`](https://mvnrepository.com/search?q=tomcat-embed-core) 依赖提供的 Tomcat、UpgradeProtocol 依赖类，才能创建内嵌的 Tomcat 服务器。
- JettyWebServerFactoryCustomizer 配置类，需要有 [`jetty-server`](https://mvnrepository.com/artifact/org.eclipse.jetty/jetty-server) 依赖提供的 Server、Loader、WebAppContext 类，才能创建内嵌的 Jetty 服务器。

**如此，我们可以得到结论二，通过条件注解，可以解决“满足什么样的条件？”的问题。**

实际上，Spring Boot 的 [`condition`](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/condition) 包下，提供了大量的条件注解，稍后我们在[「2. 条件注解」](https://www.iocoder.cn/Spring-Boot/autoconfigure/?self#)小节详细展开。

**③ 配置属性**

`<3.1>` 处，使用 [`@EnableConfigurationProperties`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/context/properties/EnableConfigurationProperties.java) 注解，让 [ServerProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java) **配置属性类**生效。

​	在 Spring Boot 定义了 [`@ConfigurationProperties`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/context/properties/ConfigurationProperties.java) 注解，用于声明配置属性类，将指定前缀的配置项批量注入到该类中。例如 ServerProperties 代码如下：



```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties
		implements EmbeddedServletContainerCustomizer, EnvironmentAware, Ordered {

	/**
	 * Server HTTP port.
	 */
	private Integer port;

	/**
	 * Context path of the application.
	 */
	private String contextPath;
	
	// ... 省略其它属性
	
}
```



- 通过 `@ConfigurationProperties` 注解，声明将 `server` 前缀的配置项，设置到 ServerProperties 配置属性类中。

`<3.2>`、`<3.3>` 处，在创建 `TomcatWebServerFactoryCustomizer` 和 `JettyWebServerFactoryCustomizer` 对象时，都会将 `ServerProperties` 传入其中，作为后续创建的 Web 服务器的配置。也就是说，我们通过修改在配置文件的配置项，就可以自定义 Web 服务器的配置。

**如此，我们可以得到结论三，通过配置属性，可以解决“创建的 Bean 的属性？”的问题。**

------

🐶 至此，我们已经比较清晰的理解 Spring Boot 是怎么解决我们上面提出的三个问题，但是这样还是无法实现自动配置。例如说，我们引入的 `spring-boot-starter-web` 等依赖，Spring Boot 是怎么知道要扫码哪些配置类的。下面，继续我们的旅途，继续抽丝剥茧。

# 2. 自动配置类

在 Spring Boot 的 [spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure) 项目，提供了大量框架的自动配置，如下图所示：![img](E:\Development\Typora\images\01-16635581518401.png)

在我们通过 [`SpringApplication#run(Class primarySource, String... args)`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L1218-L1227) 方法，启动 Spring Boot 应用的时候，有个非常重要的组件 [SpringFactoriesLoader](https://github.com/spring-projects/spring-framework/blob/master/spring-core/src/main/java/org/springframework/core/io/support/SpringFactoriesLoader.java) 类，会读取 `META-INF` 目录下的 `spring.factories` 文件，获得**每个框架定义的需要自动配置的配置类**。

我们以 [spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-autoconfigure) 项目的 [Spring Boot `spring.factories`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) 文件来举个例子，如下图所示：![img](E:\Development\Typora\images\02-16635581518403.png)

如此，原先 `@Configuration` 注解的配置类，就**升级**成类自动配置类。这样，Spring Boot 在获取到需要自动配置的配置类后，就可以自动创建相应的 Bean，完成自动配置的功能。

> 旁白君：这里其实还有一个非常有意思的话题，作为拓展知识，胖友可以后续去看看。实际上，我们可以把 `spring.factories` 理解成 Spring Boot 自己的 SPI 机制。感兴趣的胖友，可以看看如下的文章：
>
> - [《Spring Boot 的 SPI 机制》](http://www.iocoder.cn/Fight/SPI-mechanism-in-Spring-Boot/?self)
> - [《Java 的 SPI 机制》](http://www.iocoder.cn/Fight/xuma/spi/?self)
> - [《Dubbo 的 SPI 机制》](http://dubbo.apache.org/zh-cn/docs/dev/SPI.html)
>
> 实际上，自动配置只是 Spring Boot 基于 `spring.factories` 的一个拓展点 EnableAutoConfiguration。我们从上图中，还可以看到如下的拓展点：
>
> - ApplicationContextInitializer
> - ApplicationListener
> - AutoConfigurationImportListener
> - AutoConfigurationImportFilter
> - FailureAnalyzer
> - TemplateAvailabilityProvider

因为 spring-boot-autoconfigure 项目提供的是它选择的主流框架的自动配置，所以其它框架需要自己实现。例如说，Dubbo 通过 [dubbo-spring-boot-project](https://github.com/apache/dubbo-spring-boot-project) 项目，提供 Dubbo 的自动配置。如下图所示：![Dubbo ](E:\Development\Typora\images\03-16635581518415.png)

# 3. 条件注解

条件注解并不是 Spring Boot 所独有，而是在 Spring3.1 版本时，为了满足不同环境注册不同的 Bean ，引入了 [`@Profile`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/context/annotation/Profile.java) 注解。示例代码如下：



```java
@Configuration
public class DataSourceConfiguration {

    @Bean
    @Profile("DEV")
    public DataSource devDataSource() {
        // ... 单机 MySQL
    }

    @Bean
    @Profile("PROD")
    public DataSource prodDataSource() {
        // ... 集群 MySQL
    }
    
}
```



- 在测试环境下，我们注册单机 MySQL 的 DataSource Bean。
- 在生产环境下，我们注册集群 MySQL 的 DataSource Bean。

在 Spring4 版本时，提供了 [`@Conditional`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/context/annotation/Conditional.java) 注解，用于声明在配置类或者创建 Bean 的方法上，表示需要满足指定条件才能生效。示例代码如下：



```java
@Configuration
public class TestConfiguration {

    @Bean
    @Conditional(XXXCondition.class)
    public Object xxxObject() {
        return new Object();
    }
    
}
```



- 其中，XXXCondition 需要我们自己实现 [Condition](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/context/annotation/Condition.java) 接口，提供具体的条件实现。

显然，Spring4 提交的 `@Conditional` 注解非常不方便，需要我们自己去拓展。因此，Spring Boot 进一步增强，提供了常用的条件注解：

- `@ConditionalOnBean`：当容器里有指定 Bean 的条件下
- `@ConditionalOnMissingBean`：当容器里没有指定 Bean 的情况下
- `@ConditionalOnSingleCandidate`：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
- `@ConditionalOnClass`：当类路径下有指定类的条件下
- `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
- `@ConditionalOnProperty`：指定的属性是否有指定的值
- `@ConditionalOnResource`：类路径是否有指定的值
- `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
- `@ConditionalOnJava`：基于 Java 版本作为判断条件
- `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
- `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
- `@ConditionalOnWebApplication`：当前项目是 Web项 目的条件下



# 4. 配置属性

Spring Boot 约定读取 `application.yaml`、`application.properties` 等配置文件，从而实现创建 Bean 的自定义属性配置，甚至可以搭配 `@ConditionalOnProperty` 注解来取消 Bean 的创建。

咳咳咳，貌似这个小节没有太多可以分享的内容，更多胖友可以阅读[《芋道 Spring Boot 配置文件入门》](http://www.iocoder.cn/Spring-Boot/config-file/?self)文章。

# 5. 内置 Starter

我们在使用 Spring Boot 时，并不会直接引入 [`spring-boot-autoconfigure`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-autoconfigure) 依赖，而是使用 Spring Boot 内置提供的 Starter 依赖。例如说，我们想要使用 SpringMVC 时，引入的是 [`spring-boot-starter-web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) 依赖。这是为什么呢？

> 因为 Spring Boot 提供的自动配置类，基本都有 `@ConditionalOnClass` 条件注解，判断我们项目中存在指定的类，才会创建对应的 Bean。而拥有指定类的前提，一般是需要我们引入对应框架的依赖。

因此，在我们引入 `spring-boot-starter-web` 依赖时，它会帮我们自动引入相关依赖，从而保证自动配置类能够生效，创建对应的 Bean。如下图所示：![img](E:\Development\Typora\images\11-16635581518417.png)

Spring Boot 内置了非常多的 Starter，方便我们引入不同框架，并实现自动配置。如下图所示：![Spring Boot Starter](E:\Development\Typora\images\12-16635581518419.png)

# 6. 自定义 Starter

在一些场景下，我们需要自己实现自定义 Starter 来达到自动配置的目的。例如说：

- 三方框架并没有提供 Starter，比如说 [Swagger](https://github.com/swagger-api)、[XXL-JOB](https://github.com/xuxueli/xxl-job) 等。
- Spring Boot 内置的 Starter 无法满足自己的需求，比如说 [`spring-boot-starter-jdbc`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-jdbc) 不提供多数据源的配置。
- 随着项目越来越大，想要提供适合自己团队的 Starter 来方便配置项目，比如说永辉彩食鲜 [csx-bsf-all](https://gitee.com/yhcsx/csx-bsf-all) 项目。

下面，我们一起来实现一个自定义 Starter，实现一个 Java 内置 [HttpServer](https://docs.oracle.com/javase/8/docs/jre/api/net/httpserver/spec/com/sun/net/httpserver/HttpServer.html) 服务器的自动化配置。最终项目如下图所示：![最终项目](E:\Development\Typora\images\21-166355815184111.png)

在开始示例之前，我们要了解下 Spring Boot Starter 的**命名规则**，显得我们更加专业（装逼）。命名规则如下：

| 场景                         | 命名规则                            | 示例                                                         |
| :--------------------------- | :---------------------------------- | :----------------------------------------------------------- |
| **Spring Boot 内置** Starter | `spring-boot-starter-{框架}`        | `spring-boot-starter-web`                                    |
| 框架 **自定义** Starter      | `{框架}-spring-boot-starter`        | [`mybatis-spring-boot-starter`](https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter) |
| 公司 **自定义** Starter      | `{公司}-spring-boot-starter-{框架}` | 暂无，艿艿自己的想法哈                                       |

## 6.1 yunai-server-spring-boot-starter 项目

创建 [yunai-server-spring-boot-starter](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-47/yunai-server-spring-boot-starter) 项目，实现一个 Java 内置 HttpServer 服务器的自动化配置。考虑到示例比较简单，我们就不像 Spring Boot 拆分成 `spring-boot-autoconfigure` 和 `spring-boot-starter-{框架}` 两个项目。

### 6.1.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/yunai-server-spring-boot-starter/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lab-47</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>yunai-server-spring-boot-starter</artifactId>

    <dependencies>
        <!-- 引入 Spring Boot Starter 基础库 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
    </dependencies>

</project>
```



### 6.1.2 YunaiServerProperties

在 [`cn.iocoder.springboot.lab47.yunaiserver.autoconfigure`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/yunai-server-spring-boot-starter/src/main/java/cn/iocoder/springboot/lab47/yunaiserver/autoconfigure/) 包下，创建 [YunaiServerProperties](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/yunai-server-spring-boot-starter/src/main/java/cn/iocoder/springboot/lab47/yunaiserver/autoconfigure/YunaiServerProperties.java) 配置属性类，读取 `yunai.server` 前缀的配置项。代码如下：



```java
@ConfigurationProperties(prefix = "yunai.server")
public class YunaiServerProperties {

    /**
     * 默认端口
     */
    private static final Integer DEFAULT_PORT = 8000;

    /**
     * 端口
     */
    private Integer port = DEFAULT_PORT;

    public static Integer getDefaultPort() {
        return DEFAULT_PORT;
    }

    public Integer getPort() {
        return port;
    }

    public YunaiServerProperties setPort(Integer port) {
        this.port = port;
        return this;
    }

}
```



### 6.1.3 YunaiServerAutoConfiguration

在 [`cn.iocoder.springboot.lab47.yunaiserver.autoconfigure`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/yunai-server-spring-boot-starter/src/main/java/cn/iocoder/springboot/lab47/yunaiserver/autoconfigure/) 包下，创建 [YunaiServerAutoConfiguration](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/yunai-server-spring-boot-starter/src/main/java/cn/iocoder/springboot/lab47/yunaiserver/autoconfigure/YunaiServerAutoConfiguration.java) 自动配置类，在项目中存在 `com.sun.net.httpserver.HttpServer` 类时，创建 HttpServer Bean，并启动该服务器。代码如下：



```java
@Configuration // 声明配置类
@EnableConfigurationProperties(YunaiServerProperties.class) // 使 YunaiServerProperties 配置属性类生效
public class YunaiServerAutoConfiguration {

    private Logger logger = LoggerFactory.getLogger(YunaiServerAutoConfiguration.class);

    @Bean // 声明创建 Bean
    @ConditionalOnClass(HttpServer.class) // 需要项目中存在 com.sun.net.httpserver.HttpServer 类。该类为 JDK 自带，所以一定成立。
    public HttpServer httpServer(YunaiServerProperties serverProperties) throws IOException {
        // 创建 HttpServer 对象，并启动
        HttpServer server = HttpServer.create(new InetSocketAddress(serverProperties.getPort()), 0);
        server.start();
        logger.info("[httpServer][启动服务器成功，端口为:{}]", serverProperties.getPort());

        // 返回
        return server;
    }

}
```

- 代码比较简单，胖友看看艿艿在代码上添加的注释哟。

### 6.1.4 spring.factories

在 `resources` 目录下创建，创建 `META-INF` 目录，然后在该目录下创建 [`spring.factories`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/yunai-server-spring-boot-starter/src/main/resources/META-INF/spring.factories) 文件，添加自动化配置类为 YunaiServerAutoConfiguration。内容如下：



```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
cn.iocoder.springboot.lab47.yunaiserver.autoconfigure.YunaiServerAutoConfiguration
```



至此，我们已经完成了一个自定义的 Starter。下面，我们在[「6.2 lab-47-demo 项目」](https://www.iocoder.cn/Spring-Boot/autoconfigure/?self#)中引入，然后进行测试。

## 6.2 lab-47-demo 项目

创建 [lab-47-demo](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/lab-47-demo/pom.xml) 项目，引入我们自定义 Starter。

### 6.2.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/lab-47-demo/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>lab-47</artifactId>
        <groupId>cn.iocoder.springboot.labs</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-47-demo</artifactId>

    <dependencies>
        <!-- 引入自定义 Starter -->
        <dependency>
            <groupId>cn.iocoder.springboot.labs</groupId>
            <artifactId>yunai-server-spring-boot-starter</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```



### 6.2.2 配置文件

在 `resource` 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/lab-47-demo/src/main/resources/application.yaml) 配置文件，设置 `yunai.server.port` 配置项来自定义 HttpServer 端口。配置如下：



```yml
yunai:
  server:
    port: 8888 # 自定义 HttpServer 端口
```



### 6.2.3 DemoApplication

创建 [`DemoApplication.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-47/lab-47-demo/src/main/java/cn/iocoder/springboot/lab47/demo/DemoApplication.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```



### 6.2.4 简单测试

执行 `DemoApplication#main(String[] args)` 方法，启动 Spring Boot 应用。打印日志如下：



```
2020-02-02 13:03:12.156  INFO 76469 --- [           main] c.i.s.lab47.demo.DemoApplication         : Starting DemoApplication on MacBook-Pro-8 with PID 76469 (/Users/yunai/Java/SpringBoot-Labs/lab-47/lab-47-demo/target/classes started by yunai in /Users/yunai/Java/SpringBoot-Labs)
2020-02-02 13:03:12.158  INFO 76469 --- [           main] c.i.s.lab47.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2020-02-02 13:03:12.873  INFO 76469 --- [           main] c.i.s.l.y.a.YunaiServerAutoConfiguration : [httpServer][启动服务器成功，端口为:8888]
2020-02-02 13:03:12.927  INFO 76469 --- [           main] c.i.s.lab47.demo.DemoApplication         : Started DemoApplication in 1.053 seconds (JVM running for 1.47)
```



- YunaiServerAutoConfiguration 成功自动配置 HttpServer Bean，并启动该服务器在 8888 端口。

此时，我们使用浏览器访问 http://127.0.0.1:8888/ 地址，返回结果为 404 Not Found。因为我们没有给 HttpServer 相应的 Handler。

# 666. 彩蛋

至此，我们已经完成了 Spring Boot 自动配置的原理学习。如果有不理解的地方，请给艿艿留言哟。

在理解 Spring Boot 自动配置的原理的过程中，我们会发现，无论是配置类，还是条件注解也好，实际 Spring 原本都已经进行提供。甚至说，SpringFactoriesLoader 竟然也是 Spring 提供的。所以，Spring Boot 是在 Spring 的基础之上，实现了一套 Boot 启动机制。

Spring 的核心之一是 IOC，负责管理 Bean 的生命周期。而 Spring Boot 则是对 Java 应用的生命周期的管理。

- 在 Spring 的年代，我们都是使用 Tomcat 外部容器来实现 Java 应用的运行，Spring 只是其中的一个组件。
- 在 Spring Boot 的年代，我们使用 Spring Boot 来管理 Java 应用的运行，内嵌的 Tomcat 反而成为其中的一个组件。

😈 另外，在推荐如下的文章，方便胖友进一步对 Spring Boot 有深入理解：

- [《给你一份 Spring Boot 知识清单》](http://www.iocoder.cn/Fight/Give-you-a-list-of-Spring-Boot-knowledge/?self)
- [《详解 Spring Boot 自动配置机制》](http://www.iocoder.ccn/Fight/Details-the-Spring-Boot-automatic-configuration-mechanism/?self)
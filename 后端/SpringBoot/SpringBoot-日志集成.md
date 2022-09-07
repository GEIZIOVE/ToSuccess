# SpringBoot-日志集成

摘要: 原创出处 http://www.iocoder.cn/Spring-Boot/Logging/ 「芋道源码」欢迎转载，保留摘要，谢谢！

- [1. 概述](http://www.iocoder.cn/Spring-Boot/Logging/)
- [2. 快速入门](http://www.iocoder.cn/Spring-Boot/Logging/)
- [3. 动态修改日志级别](http://www.iocoder.cn/Spring-Boot/Logging/)
- [4. 调试模式](http://www.iocoder.cn/Spring-Boot/Logging/)
- [5. 日志分组](http://www.iocoder.cn/Spring-Boot/Logging/)
- [6. 不同环境下的日志配置](http://www.iocoder.cn/Spring-Boot/Logging/)
- [7. Logback 扩展](http://www.iocoder.cn/Spring-Boot/Logging/)
- [8. 集成 Log4j2](http://www.iocoder.cn/Spring-Boot/Logging/)
- [9. 访问日志](http://www.iocoder.cn/Spring-Boot/Logging/)
- [666. 彩蛋](http://www.iocoder.cn/Spring-Boot/Logging/)

------

------

> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 [lab-37](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37) 目录。
>
> 原创不易，给点个 [Star](https://github.com/YunaiV/SpringBoot-Labs/stargazers) 嘿，一起冲鸭！

# 1. 概述

在开始讲解在 Spring Boot 如何使用日志框架之前，我们想来了解下 Java 日志框架的生态，以便我们更深入的入门。当然，胖友也可以选择跳过，直接从[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#)小节开始。

## 1.1 日志框架

在 Java 日志框架的生态中，存在多种的**日志实现框架**。例如说：

- [JUL](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html)

  > Java 自带 `java.util.logging` 组件的简称。

- [Apache Log4j1](https://logging.apache.org/log4j/1.2/)

- [Apache Log4j2](https://logging.apache.org/log4j/2.x/)

- [Logback](http://logback.qos.ch/)

那么，对于 Spring、Hibernate 等框架，会有打日志的需求，那么就需要选择相应的日志框架。但是，它们无论选择任一一个日志框架，可能使用 Spring、Hibernate 等框架的项目，希望选择另外一个日志框架。此时，项目中就需要添加多个日志框架的**配置文件**，十分麻烦不便。

所幸，在 Java 日志框架的生态中，存在多种**日志门面框架**，基于 [Facade Pattern](http://www.iocoder.cn/DesignPattern/xiaomingge/Facade-Pattern/) 设计模式的思想，提供通用的日志 API 给调用方，而自己去实现不同日志框架的适配。例如说：

- [SLF4J](http://www.slf4j.org/)

  > Simple Logging Facade for Java

- [JCL](https://commons.apache.org/proper/commons-logging/)

  > Apache Commons Logging

- [jboss-logging](https://github.com/jboss-logging)

这样就变成，Spring 采用 JCL 日志门面框架，Hibernate 采用 jboss-logging 日志门面框架，而将选择具体的日志实现框架的权利，交给使用者。

当然，也有框架，是自己实现简单的日志门面功能。例如说：

- Dubbo [logger](https://github.com/apache/dubbo/tree/master/dubbo-common/src/main/java/org/apache/dubbo/common/logger) 组件
- MyBatis [logging](https://github.com/mybatis/mybatis-3/tree/master/src/main/java/org/apache/ibatis/logging) 组件

所以，在 Java 日志框架的生态中，一共存在两种角色：**日志门面框架**和**日志实现框架**。下面，我们把它们整理成下图：![门面 + 实现](E:\Development\Typora\images\01.png)

不过要注意，日志门面框架**仅**提供通用的日志 API 给调用方，不包括每个日志实现框架的**配置**。因此我们在使用时，还是需要添加我们使用的具体的日志实现框架的**配置文件**。😈 也就是说，日志门面框架的**作用**是，帮助我们在项目中，**统一**日志实现框架的使用。

另外，即使在自己项目的代码中，如果有打日志的需求，也是推荐使用日志门面框架的 API ，而不是直接调用日志实现框架。在《阿里巴巴 Java 开发手册》中，也提出了这样的日志规约：![日志规约](E:\Development\Typora\images\02.png)

**目前，比较主流的选择是，选择 SLF4J 作为日志门面框架，Logback 作为日志实现框**。

## 1.2 SLF4J 原理

那么，SLF4J 是如何实现对多个日志框架的适配的呢？在[《SLF4J 文档 —— Bridging legacy APIs》](http://www.slf4j.org/legacy.html)中，已经详细讲解了。

我们先来看 SLF4J + Logback 的组合。如下图所示：

> ![SLF4J + Logback 的组合](E:\Development\Typora\images\03.png)

- 【红框】将使用

  其它

  日志

  门面

  框架的，替换成

   

  SLF4J

   

  日志门面框架。例如说，Spring 采用 JCL 日志门面框架，就是通过该方式来解决的。

  - [`jcl-over-slf4j`](https://mvnrepository.com/artifact/org.slf4j/jcl-over-slf4j) 库，将调用 JCL 打日志的地方，适配成调用 SLF4J API。实现原理是，`jcl-over-slf4j` 通过在包名／类名／方法名上和 JCL **保持完全一致**，内部重写来调用的是 SLF4J 的 API 。如下图所示：![img](E:\Development\Typora\images\04.png)

- 【绿框】将使用

  其它

  日志

  实现

  框架的，替换成

   

  SLF4J

   

  日志门面框架。

  - [`log4j-over-slf4j`](https://mvnrepository.com/artifact/org.slf4j/log4j-over-slf4j) 库，将调用 Log4j1 打日志的地方，适配成调用 SLF4J API。实现原理是，同 `jcl-over-slf4j` 库。如下图所示：![img](E:\Development\Typora\images\05.png)
  - [`jul-over-slf4j`](https://mvnrepository.com/artifact/org.slf4j/jul-to-slf4j) 库，将调用 JUL 打日志的地方，适配成调用 SLF4J API。实现原理是，实现自定义的 `java.util.logging.Handler` 处理器，来调用的是 SLF4J 的 API 。如下图所示：![img](E:\Development\Typora\images\06.png)

- 【黄框】

  SLF4J

   

  日志门面框架，调用具体的日志实现框架，打印日志。

  - [`logback-classic`](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic) 库，内置了对 SLF4J 框架的实现。如下图所示：![img](E:\Development\Typora\images\07.png)

我们再来看 SLF4J + Log4j1 的组合。如下图所示：

> ![SLF4J + Log4J1 的组合](E:\Development\Typora\images\08.png)

- 整体实现原理来说，和 SLF4J + Logback 的组合是一致的。
- 【绿框】[`slf4j-log4j12`](https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12) 库，因为 Log4j1 并未内置对 SLF4J 框架的实现，所以 SLF4J 只能自己实现。如下图所示：![img](E:\Development\Typora\images\09.png)

## 1.3 Spring Boot 对日志框架的封装

在 Spring Boot 内部，保持和 Spring 一致，采用 JCL 日志**门面**框架来记录日志。但是，其 [`logging`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/logging/package-info.java) 包下，提供了对 JUL、Log4j2、Logback 日志**实现**框架的封装，提供默认的日志配置。我们以 Logback 举例子，其在 [DefaultLogbackConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/logging/logback/DefaultLogbackConfiguration.java) 类中，定义了[文件日志格式](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/logging/logback/DefaultLogbackConfiguration.java#L58-L59)如下：



```
// DefaultLogbackConfiguration.java

private static final String FILE_LOG_PATTERN = "%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} "
		+ "${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}";
```



- 日志输出示例如下：

  ```
  2019-12-29 17:24:57.935  INFO 97606 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
  2019-12-29 17:24:57.938  INFO 97606 --- [           main] c.i.s.lab36.prometheusdemo.Application   : Started Application in 2.624 seconds (JVM running for 3.228)
  2019-12-29 17:24:58.412  INFO 97606 --- [on(4)-127.0.0.1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
  ```

  

- 文件日志格式解释：

  - `%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}` 日期和时间：毫秒精度。
  - `${LOG_LEVEL_PATTERN:-%5p}` 日志级别：`ERROR`、`WARN`、`INFO`、`DEBUG` 或 `TRACE`。
  - `${PID:- }` 进程 ID。
  - `---` 分隔符：用于区分实际日志内容的开始。
  - `[%t]` 线程名称：在方括号中（可能会截断控制台输出）。
  - `%-40.40logger{39}` 日志记录器名称：这通常是源类名称（通常为缩写）。
  - `%m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}` 日志内容。

同时，允许通过 `logging.` 配置项来，进行自定义设置。例如说：

- 自定义日志**格式**

  - `logging.pattern.file` ：文件的日志格式。
  - `logging.pattern.console` ：控制台的日志格式。
  - `logging.pattern.dateformat` ：日期和时间。
  - `logging.pattern.level` ：日志级别。
  - `logging.pattern.pid` ： 进程 ID。
  - `logging.exception-conversion-word` ：记录异常时使用的转换字

- 自定义日志**文件**

  - `logging.file.max-history` ：日志文件要保留的归档的最大天数。
  - `logging.file.max-size` ：日志文件的最大大小。
  - `logging.file` ：日志文件名。
  - `logging.path` ：日志文件路径。

- `logging.config` ：日志框架使用的配置文件地址。因为根据不同的日志实现框架，Spring Boot 按如下“约定规则”加载配置文件：

  - Logback ：对应 `logback-spring.xml`、`logback-spring.groovy`、`logback.xml`、`logback.groovy` 配置文件。
  - Log4j2 ：对应 `log4j2-spring.xml`、`log4j2.xml` 配置文件。
  - JUL ：对应 `logging.properties` 配置文件。

- `logging.level.*` ：日志等级，通过使用 `logging.level.<logger-name>=<level>` 来设置。例如说：

  ```
  logging.level.root=WARN
  logging.level.org.springframework.web=DEBUG
  logging.level.org.hibernate=ERROR
  ```

  

虽然 `logging.` 自定义配置项很多，但是一般情况下，我们只使用 `logging.level.*` 配置项，设置不同 Logger 的日志级别。

目前，Spring Boot 内置了两个日志相关的 Starter ：

- [`spring-boot-starter-logging`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-logging) ：使用 SLF4J + Logback 的组合。其 `pom.xml` 文件的依赖如下：

  ```
  <dependencies>
  <dependency>
  	<groupId>ch.qos.logback</groupId>
  	<artifactId>logback-classic</artifactId>
  </dependency>
  <dependency>
  	<groupId>org.apache.logging.log4j</groupId>
  	<artifactId>log4j-to-slf4j</artifactId>
  </dependency>
  <dependency>
  	<groupId>org.slf4j</groupId>
  	<artifactId>jul-to-slf4j</artifactId>
  </dependency>
  </dependencies>
  ```

  

  - **符合**我们在[「1.2 SLF4J 原理」](https://www.iocoder.cn/Spring-Boot/Logging/#)的 SLF4J + Logback 的组合的原理说明。

- [`spring-boot-starter-log4j2`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-log4j2) ：使用 SLF4J + Log4j2 的组合。其 `pom.xml` 文件的依赖如下：

  ```
  <dependencies>
  	<dependency>
  		<groupId>org.apache.logging.log4j</groupId>
  		<artifactId>log4j-slf4j-impl</artifactId>
  	</dependency>
  	<dependency>
  		<groupId>org.apache.logging.log4j</groupId>
  		<artifactId>log4j-core</artifactId>
  	</dependency>
  	<dependency>
  		<groupId>org.apache.logging.log4j</groupId>
  		<artifactId>log4j-jul</artifactId>
  	</dependency>
  	<dependency>
  		<groupId>org.slf4j</groupId>
  		<artifactId>jul-to-slf4j</artifactId>
  	</dependency>
  </dependencies>
  ```

  

  - **类似**我们在[「1.2 SLF4J 原理」](https://www.iocoder.cn/Spring-Boot/Logging/#)的 SLF4J + Log4j1 的组合的原理说明。

默认情况下，[`spring-boot-starter`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter) 推荐采用 `spring-boot-starter-logging` ，即 SLF4J + Logback 的组合。因为其 `pom.xml` 文件引入了 `spring-boot-starter-logging` 。

# 2. 快速入门

> 示例代码对应仓库：[lab-37-logging-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-demo) 。

本小节，我们来进行下 Spring Boot 集成日志功能的快速入门，使用 SLF4J + Logback 的组合。主要功能包括：

- 控制台打印日志。
- 控制台颜色输出。
- 日志文件打印日志。
- 自定义日志级别。

下面，我们就来搭建一个 [lab-37-logging-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-demo) 项目。

## 2.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-demo/pom.xml) 文件中，引入相关依赖。



```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-37-logging-demo</artifactId>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```



- 引入 `spring-boot-starter-web` 依赖的原因，是因为稍后我们会在[「2.4 DemoController」](https://www.iocoder.cn/Spring-Boot/Logging/#)中，定义 HTTP API 接口，方便安我们进行自定义日志级别的测试。
- 为什么并未引入 `spring-boot-starter-logging` 依赖呢？因为 `spring-boot-starter-web` 包含了 `spring-boot-starter` ，而 `spring-boot-starter` 又已经包含了 `spring-boot-starter-logging`。

## 2.2 配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-demo/src/main/resources/application.yaml) 中，添加日志相关配置，如下：



```yml
spring:
  application:
    name: demo-application # 应用名

logging:
  # 日志文件配置
  file:
#    path: /Users/yunai/logs/ # 日志文件路径。
    name: /Users/yunai/logs/${spring.application.name}.log # 日志文件名。
    max-history: 7 # 日志文件要保留的归档的最大天数。默认为 7 天。
    max-size: 10MB # 日志文件的最大大小。默认为 10MB 。

  # 日志级别
  level:
    cn:
      iocoder:
        springboot:
          lab37:
            loggingdemo:
              controller: DEBUG
```



**① 日志文件**

在 `logging.file.*` 配置项下，设置 Spring Boot **日志文件**。

默认情况下，Spring Boot 日志只会打印到控制台。所以需要通过 `logging.file.path` 或 `logging.file.name` 配置项来设置。不过要注意，这两者是**二选一**，并不是共同作用。

- `logging.file.name` ：日志文件全路径名。可以是相对路径，也可以是绝对路径。这里，我们设置为 `"/Users/yunai/logs/${spring.application.name}.log"` ，绝对路径。
- `logging.file.path` ：日志文件目录。会在该目录下，创建 `spring.log` 日志文件。例如说：`/Users/yunai/logs/` 。
- 上述两者都配置的情况下，以 `logging.file.name` 配置项为准。

**② 日志级别**

在 `logging.level.*` 配置项，设置自定义的**日志级别**。Spring Boot 定义的[日志级别](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/logging/LogLevel.java)为 `TRACE`、`DEBUG`、`INFO`、`WARN`、`ERROR`、`FATAL`、`OFF`。

默认情况下，控制台和日志文件的日志级别 `INFO`。即，`INFO`、`WARN`、`ERROR` 级别的日志，都会记录到控制台和日志文件中。

可以通过使用 `logging.level.<logger-name>=<level>` 配置项，自定义 Logger 对应的日志级别。这里，我们设置了名字为 `"cn.iocoder.springboot.lab37.loggingdemo.controller"` 的 Logger 的日志级别，为 `DEBUG` 。

**③ 颜色输出**

如果我们的控制台支持 [ANSI](https://www.iocoder.cn/Spring-Boot/Logging/ANSI转义序列) ，则可以使用**颜色输出**，来提高可读性。通过 `spring.output.ansi.enabled` 配置项，设置 ANSI 功能的状态，有三种选项：

- `ALWAYS` ：启用 `ANSI` 颜色的输出。
- `NEVER` ：禁用 `ANSI` 颜色的输出。
- `DETECT` ：自动检测控制台是否支持 ANSI 功能。如果支持则进行开启，否则则进行禁用。默认情况下，使用 `DETECT` 这种选项。

默认情况下，Spring Boot 已经配置好颜色输出的日志格式，我们并不需要做自定义，所以这里也就不多做介绍了。我们只需要知道，不同日志级别，对应不同的颜色的映射关系即可：

| 级别    | 颜色         |
| :------ | :----------- |
| `FATAL` | 红（Red）    |
| `ERROR` | 红（Red）    |
| `WARN`  | 黄（Yellow） |
| `INFO`  | 绿（Green）  |
| `DEBUG` | 绿（Green）  |
| `TRACE` | 绿（Green）  |

①、②、③ 配置项，我们也会在[「2.5 简单测试」](https://www.iocoder.cn/Spring-Boot/Logging/#)详细测试说明。

## 2.3 Application

创建 [Application](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-demo/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/Application.java) 类，用于启动示例项目。代码如下：



```
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



## 2.4 DemoController

在 [`cn.iocoder.springboot.lab37.loggingdemo.controller`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-demo/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/controller/) 包路径下，创建 [DemoController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-demo/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/controller/DemoController.java) 类，提供不同 API 接口，打印不同级别的日志。代码如下：



```
// DemoController.java

@RestController
@RequestMapping("/demo")
public class DemoController {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @GetMapping("/debug")
    public void debug() {
        logger.debug("debug");
    }

    @GetMapping("/info")
    public void info() {
        logger.info("info");
    }

    @GetMapping("/error")
    public void error() {
        logger.error("error");
    }

}
```



## 2.5 简单测试

① 测试“颜色输出”

执行 `Application#main(String[] args)` 方法，启动项目。控制台输出带有颜色的日志，如下图所示：![颜色输出](E:\Development\Typora\images\10.png)

② 测试“日志文件”

查看日志文件 `/Users/yunai/logs/demo-application.log` ，已经有打印日志。日志内容如下：



```
// ... 省略其它日志

2020-01-01 15:19:41.991  INFO 7553 --- [main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-01-01 15:19:42.130  INFO 7553 --- [main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-01-01 15:19:42.132  INFO 7553 --- [main] c.i.s.lab37.loggingdemo.Application      : Started Application in 1.549 seconds (JVM running for 2.053)
```



③ 测试“日志级别”

使用浏览器，访问 `/demo/debug`、`/demo/info`、`/demo/error` 三个接口。控制台输出日志如下：



```
2020-01-01 15:25:05.630 DEBUG 7553 --- [nio-8080-exec-2] c.i.s.l.l.controller.DemoController      : debug
2020-01-01 15:25:14.717  INFO 7553 --- [nio-8080-exec-4] c.i.s.l.l.controller.DemoController      : info
2020-01-01 15:25:19.590 ERROR 7553 --- [nio-8080-exec-5] c.i.s.l.l.controller.DemoController      : error
```



- 符合预期。因为我们配置 `"cn.iocoder.springboot.lab37.loggingdemo.controller"`的 Logger 的日志级别，为 `DEBUG` 。否则，按照 Spring Boot 默认日志级别为 `INFO` 的情况下，`DEBUG` 日志应该不会输出。

# 3. 动态修改日志级别

> 示例代码对应仓库：[lab-37-logging-actuator](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-actuator/) 。

在排查线上运行问题时，我们可能需要动态修改 Logger 的日志级别。一般情况下，我们需要手动修改配置文件的 Logger 的日志级别，然后重启应用，以使修改的 Logger 的日志级别生效。

显然，这样的操作过程略微繁琐，一点都不骚气。所以本小节，我们来通过[《芋道 Spring Boot 监控端点 Actuator 入门》](http://www.iocoder.cn/Spring-Boot/Actuator/?self)的[「14. loggers 端点」](https://www.iocoder.cn/Spring-Boot/Logging/#)，使用该端点提供的 `POST /actuator/loggers/{name}` 接口，修改指定 Logger 的日志级别。

下面，我们就来看看动态修改日志级别的示例。考虑到不污染[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#) 的示例，我们在 [lab-37-logging-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-demo) 项目的基础上，复制出一个 [lab-37-logging-actuator](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-actuator) 项目。😈 酱紫，我们也能少写点代码，哈哈哈~

## 3.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-actuator/pom.xml) 文件中，**额外**引入 `spring-boot-starter-actuator` 依赖。如下：



```
<!-- 实现对 Actuator 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



## 3.2 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-actuator/src/main/resources/application.yaml) 配置文件，**额外**增加 Spring Boot Actuator 相关配置。如下：



```
management:
  endpoints:
    # Actuator HTTP 配置项，对应 WebEndpointProperties 配置类
    web:
      exposure:
        include: '*' # 需要开放的端点。默认值只打开 health 和 info 两个端点。通过设置 * ，可以开放所有端点。
```



## 3.3 简单测试

本小节，我们会将名字为 `"cn.iocoder.springboot.lab37.loggingdemo.controller"` 的 Logger 的日志级别，修改成 `INFO` 级别，实现 `/demo/debug` 接口无法打印出 DEBUG 日志。

调用 `Application#main(Object[] args)` 方法，启动 Spring Boot 应用。

① 使用浏览器，访问 http://127.0.0.1:8080/actuator/loggers/cn.iocoder.springboot.lab37.loggingdemo.controller/ 地址，获得该 Logger 的日志级别。响应结果如下：



```
{
    "configuredLevel": "DEBUG",
    "effectiveLevel": "DEBUG"
}
```



- 日志级别是 `DEBUG` 。

② 使用浏览器，访问 http://127.0.0.1:8080/demo/debug/ 地址，打印出 DEBUG 日志如下：



```
2020-01-01 16:15:54.019 DEBUG 8507 --- [nio-8080-exec-3] c.i.s.l.l.controller.DemoController      : debug
```



③ 使用 Postman ，请求 `POST http://127.0.0.1:8080/actuator/loggers/cn.iocoder.springboot.lab37.loggingdemo.controller/` 地址，修改该 Logger 的日志级别。如下图：![修改日志级别](E:\Development\Typora\images\11.png)

④ 使用浏览器，访问 http://127.0.0.1:8080/actuator/loggers/cn.iocoder.springboot.lab37.loggingdemo.controller/ 地址，获得该 Logger 的日志级别。响应结果如下：



```
{
    "configuredLevel": "INFO",
    "effectiveLevel": "INFO"
}
```



- 日志级别**修改**成了 `INFO` 。

⑤ 使用浏览器，访问 http://127.0.0.1:8080/demo/debug/ 地址，并未打印出日志。符合预期~

另外，如果胖友有使用 Spring Boot Admin 监控工具，可以参考[《芋道 Spring Boot 监控工具 Admin 入门》](http://www.iocoder.cn/Spring-Boot/Admin/?self)的[「3.2 Loggers」](https://www.iocoder.cn/Spring-Boot/Logging/#)，也可以动态修改日志级别。

# 4. 调试模式

> 示例代码对应仓库：[lab-37-logging-debug](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-debug/) 。

默认情况下，Spring Boot **只**会记录 `ERROR`、`WARN` 和 `INFO` 级别的日志。可能有时我们启动 Spring Boot 失败时，需要通过启动 Spring Boot **调试模式**：

- 让**核心 Logger**（内嵌容器、Hibernate 和 Spring Boot 等等）打印 `DEBUG` 级别的日志，方便排查原因
- 应用中的**其它 Logger** 还是保持**原有**的日志级别。

目前，有两种方式开启 Spring Boot 应用的调试模式：

- 通过启动参数带上 `--debug` 标识，进行开启。例如说 `$ java -jar myapp.jar --debug`。
- 通过配置文件的 `debug = true` 配置项，进行开启。

下面，我们就来看看动态修改日志级别的示例。考虑到不污染[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#) 的示例，我们在 [lab-37-logging-demo](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-demo/) 项目的基础上，复制出一个 [lab-37-logging-debug](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-debug/) 项目。😈 酱紫，我们也能少写点代码，哈哈哈~

## 4.1 配置文件

修改 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-debug/src/main/resources/application.yaml) 配置文件，**额外**增加调试模式相关的配置。增加如下：



```
spring:
  application:
    name: demo-application # 应用名

logging:
  # 日志文件配置
  file:
#    path: /Users/yunai/logs/ # 日志文件路径。
    name: /Users/yunai/logs/${spring.application.name}.log # 日志文件名。
    max-history: 7 # 日志文件要保留的归档的最大天数。默认为 7 天。
    max-size: 10MB # 日志文件的最大大小。默认为 10MB 。

# 调试模式
debug: true
```



- 相比[「2.2 配置文件」](https://www.iocoder.cn/Spring-Boot/Logging/#)来说，主要有两点变化。
- 第一点，增加 `debug = true` 配置项，开启调试模式。
- 第二点，删除 `logging.level.*` 配置项，使用 `"cn.iocoder.springboot.lab37.loggingdemo.controller"` 的 Logger 的日志级别，恢复为默认的 `LOGGER`。

## 4.2 简单测试

① 调用 `Application#main(Object[] args)` 方法，启动 Spring Boot 应用。在控制台上，我们可以看到**核心 Logger** 打印了 `DEBUG` 级别的日志。如下图：![调试模式 - 启动日志](E:\Development\Typora\images\12.png)

② 使用浏览器，http://127.0.0.1:8080/demo/debug/ 地址，查看打印的日志。如下图：![调试模式 - 请求日志](E:\Development\Typora\images\13.png)

- 在控制台上，我们可以看到**核心 Logger** 打印了 `DEBUG` 级别的日志。
- `"cn.iocoder.springboot.lab37.loggingdemo.controller"` 的 Logger 的日志级别并未变成 `DEBUG` 级别，打印出 `DEBUG` 级别的日志，**符合预期**。

# 5. 日志分组

Spring Boot 提供了**日志分组**功能，可以相关 Logger 组合在一起，以便可以统一配置它们的日志级别。例如说：Spring Boot **内置**了两个分组：

| 日志分组名 | 对应 Logger                                                  |
| :--------- | :----------------------------------------------------------- |
| web        | `org.springframework.core.codec`、`org.springframework.http`、`org.springframework.web` |
| sql        | `org.springframework.jdbc.core`、`org.hibernate.SQL`         |

后续，我们仅需配置 `logging.level.sql = DEBUG`，就可以将 `sql` 日志分组对应的 Logger 们的日志级别，一次性修改成 `DEBUG` 级别。

同时，我们可以**自定义**日志分组。配置文件示例如下：



```
logging:
  group:
    tomcat: org.apache.catalina, org.apache.coyote, org.apache.tomcat
  level:
    tomcat: DEBUG
```



- 通过 `logging.group.<group-name>` 配置项，来定义一个日志分组，其值为 Logger 们。
- 通过 `logging.level.<group-name>` 配置项，来定义日志分组的日志级别。

# 6. 不同环境下的日志配置

> 示例代码对应仓库：[lab-37-logging-multi-env](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/) 。

同一个项目，在不同的环境下，日志配置可能存在差异性。例如说：

- 在开发环境下，**无需**日志文件，**只需要**打印日志到控制台。同时，使用 `DEBUG` 级别的日志级别，打印更多的日志，以便排查问题。
- 在生产环境下，**需要**日志文件，**可选**打印日志到控制台。同时，使用 `INFO` 级别的日志级别，毕竟生产环境下访问量比较大，日志量会很大。

因为我们可以在配置文件中，进行日志配置，所以我们可以基于 [Spring Profiles](https://www.baeldung.com/spring-profiles) 机制，每个环境对应一个 Profile 对应一个配置文件。

下面，我们开始本小节的示例。考虑到不污染上述的示例，我们新建一个 [lab-37-logging-multi-env](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/) 项目。

## 6.1 引入依赖

和[「2.1 引入依赖」」](https://www.iocoder.cn/Spring-Boot/Logging/#)一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/pom.xml) 文件。

## 6.2 配置文件

我们会创建两个配置文件，分别对应开发环境和测试环境。

**① 开发环境**

创建 [`application-dev.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/src/main/resources/application-dev.yaml) 配置文件，添加开发环境的日志相关配置，如下：



```yml
spring:
  application:
    name: demo-application # 应用名

logging:
  # 日志级别
  level:
    cn:
      iocoder:
        springboot:
          lab37:
            loggingdemo: DEBUG
```



- 未添加 `logging.file.path` 或 `logging.file.name` 配置项，所以不会创建日志文件。
- 添加 `logging.level.cn.iocoder.springboot.lab37.loggingdemo = DEBUG` 配置项，设置该 Logger 日志级别为 `DEBUG`。

**② 生产环境**

创建 [`application-prod.yaml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/src/main/resources/application-prod.yaml) 配置文件，添加生产环境的日志相关配置，如下：



```
spring:
  application:
    name: demo-application # 应用名

logging:
  # 日志文件配置
  file:
    name: /Users/yunai/logs/${spring.application.name}.log # 日志文件名。
```



- 添加 `logging.file.name` 配置项，所以会创建日志文件。
- 未添加 `logging.level.*` 配置项，所以整个日志级别为默认的 `LOGGER`。

## 6.3 Application

创建 [Application](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/Application.java) 类，用于启动示例项目。代码如下：



```
@SpringBootApplication
public class Application {

    private static Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);

        // <X> 打印日志
        logger.debug("just do it");
    }

}
```



- 在 `<X>` 处，打印 `DEBUG` 级别的日志，测试下不同环境下的日志级别。

## 6.4 简单测试

通过 IDEA 的 `Active profiles` 选项，设置启动的 Spring Boot 应用的 Profile 。如下图所示：![IDEA ](E:\Development\Typora\images\14.png)

**① 测试开发环境**

设置 Profile 为 `dev`，启动 Spring Boot 应用。控制台打印日志如下：



```
2019-12-30 21:43:19.736  INFO 12531 --- [           main] c.i.s.lab37.loggingdemo.Application      : Started Application in 1.361 seconds (JVM running for 1.764)
2019-12-30 21:43:19.737 DEBUG 12531 --- [           main] c.i.s.lab37.loggingdemo.Application      : just do its
```



- 我们在 `<X>` 处，打印的 `DEBUG` 日志，成功输出。符合预期~

**② 测试生产环境**

设置 Profile 为 `prod`，启动 Spring Boot 应用。控制台打印日志如下：



```
2019-12-30 21:44:35.292  INFO 12551 --- [           main] c.i.s.lab37.loggingdemo.Application      : Started Application in 1.393 seconds (JVM running for 1.837)
```



- 我们在 `<X>` 处，打印的 `DEBUG` 日志，并未输出。符合预期~
- 同时，查看 `/Users/yunai/logs/$demo-application.log` 日志文件，新增相应的日志。

# 7. Logback 扩展

> 示例代码对应仓库：[lab-37-logging-logback](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-logback/) 。

在[「1.3 Spring Boot 对日志框架的封装」](https://www.iocoder.cn/Spring-Boot/Logging/#)中，我们可以通过配置文件的 `logging.config` 配置项，设置日志**实现**框架的具体配置文件。在 Spring Boot 中，默认情况下，Logback 读取 `logback-spring.xml`、`logback-spring.groovy`、`logback.xml`、`logback.groovy` 配置文件。**一般情况下，我们是使用 `logback-spring.xml` 配置文件。**

Spring Boot 额外提供了 Logback 的两个 XML 标签的扩展。

**① 拓展一：`<springProfile />` 标签**

通过 `<springProfile />` 标签，设置**标签内**的 Logback 的配置，需要符合指定的 Spring Profile 才可以生效。通过该标签，我们也可以实现[「6. 不同环境下的日志配置」](https://www.iocoder.cn/Spring-Boot/Logging/#)的效果。示例如下：



```java
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```



**② 拓展二：`<springProperty />` 标签**

通过 `<springProperty />` 标签，可以从 Spring Boot 配置文件中读取配置项。示例如下：



```java
<!-- 从 Spring Boot 配置文件中，读取 spring.application.name 应用名 -->
<springProperty name="applicationName" scope="context" source="spring.application.name" />
<!-- 日志文件的路径 -->
<property name="LOG_FILE" value="/Users/yunai/logs/${applicationName}.log"/>
```



下面，我们开始本小节的示例，和[「6. 不同环境下的日志配置」](https://www.iocoder.cn/Spring-Boot/Logging/#)小节的效果**等价**。考虑到不污染上述的示例，我们新建一个 [lab-37-logging-logback](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-logback/) 项目。

## 7.1 引入依赖

和[「2.1 引入依赖」」](https://www.iocoder.cn/Spring-Boot/Logging/#)一致，见 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-logback/pom.xml) 文件。

## 7.2 应用配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-logback/src/main/resources/application.yaml) 中，仅添加应用名的配置即可，如下：



```
spring:
  application:
    name: demo-application # 应用名
```



## 7.3 Logback 配置文件

在 [`logback-spring.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-logback/src/main/resources/logback-spring.xml) 中，添加开发环境和生产环境的 Logback 配置，如下：



```
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

    <!-- <1> -->
    <!-- 引入 Spring Boot 默认的 logback XML 配置文件  -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <!-- <2.1> -->
    <!-- 控制台 Appender -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 日志的格式化 -->
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- 2.2 -->
    <!-- 从 Spring Boot 配置文件中，读取 spring.application.name 应用名 -->
    <springProperty name="applicationName" scope="context" source="spring.application.name" />
    <!-- 日志文件的路径 -->
    <property name="LOG_FILE" value="/Users/yunai/logs/${applicationName}.log"/>
    <!-- 日志文件 Appender -->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <!--滚动策略，基于时间 + 大小的分包策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>10MB</maxFileSize>
        </rollingPolicy>
        <!-- 日志的格式化 -->
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <!-- 3.1 -->
    <!-- 测试环境，独有的配置 -->
    <springProfile name="dev">
        <!-- 设置 Appender -->
        <root level="INFO">
            <appender-ref ref="console"/>
        </root>

        <!-- 设置 "cn.iocoder.springboot.lab37.loggingdemo" 的 Logger 的日志级别为 DEBUG -->
        <logger name="cn.iocoder.springboot.lab37.loggingdemo" level="DEBUG"/>
    </springProfile>

    <!-- 3.2 -->
    <!-- 生产环境，独有的配置 -->
    <springProfile name="prod">
        <!-- 设置 Appender -->
        <root level="INFO">
            <appender-ref ref="console"/>
            <appender-ref ref="file"/>
        </root>
    </springProfile>

</configuration>
```



- `<1>` 处，引入 Spring Boot **默认**的 logback XML 配置文件 [`defaults.xml`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)。在其中，定义了控制台的日志格式 `CONSOLE_LOG_PATTERN`、日志文件的日志格式 `FILE_LOG_PATTERN`、Tomcat 等常用 Logger 的日志级别。
- `<2.1>` 处，配置控制台的 Appender。
- `<2.2>` 处，配置日志文件的 Appender。这里，我们可以看到 Spring Boot `<springProperty />` 拓展标签的使用。
- `<3.1>` 处，通过 `<springProfile name="dev" />` 标签，设置**测试**环境的独有配置。
- `<3.2>` 处，通过 `<springProfile name="prod" />` 标签，设置**生产**环境的独有配置。

## 7.4 Application

创建 [Application](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-multi-env/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/Application.java) 类，用于启动示例项目。代码如下：



```
@SpringBootApplication
public class Application {

    private static Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);

        // <X> 打印日志
        logger.debug("just do it");
    }

}
```



- 在 `<X>` 处，打印 `DEBUG` 级别的日志，测试下不同环境下的日志级别。

## 7.5 简单测试

通过 IDEA 的 `Active profiles` 选项，设置启动的 Spring Boot 应用的 Profile 。如下图所示：![IDEA ](E:\Development\Typora\images\14.png)

**① 测试开发环境**

设置 Profile 为 `dev`，启动 Spring Boot 应用。控制台打印日志如下：



```
2019-12-30 23:53:44.469  INFO 14215 --- [           main] c.i.s.lab37.loggingdemo.Application      : Started Application in 1.583 seconds (JVM running for 2.069)
2019-12-30 23:53:44.469 DEBUG 14215 --- [           main] c.i.s.lab37.loggingdemo.Application      : just do it
```



- 我们在 `<X>` 处，打印的 `DEBUG` 日志，成功输出。符合预期~

**② 测试生产环境**

设置 Profile 为 `prod`，启动 Spring Boot 应用。控制台打印日志如下：



```
2019-12-30 23:54:19.202  INFO 14228 --- [           main] c.i.s.lab37.loggingdemo.Application      : Started Application in 1.38 seconds (JVM running for 1.781)
```



- 我们在 `<X>` 处，打印的 `DEBUG` 日志，并未输出。符合预期~
- 同时，查看 `/Users/yunai/logs/$demo-application.log` 日志文件，新增相应的日志。

# 8. 集成 Log4j2

> 示例代码对应仓库：[lab-37-logging-log4j2](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-log4j2/) 。

在[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#)小节中，我们已经使用 `spring-boot-starter-logging` 依赖，实现 SFL4J + Logback 打日志的功能。而在本小节，我们将使用 `spring-boot-starter-log4j2` 依赖，实现 SFL4J + **Log4j2** 打日志的功能。

考虑到在[「7. Logback 扩展」](https://www.iocoder.cn/Spring-Boot/Logging/#)小节中，采用了使用 Logback 配置文件的方式，所以本小节，我们也采用 Log4j2 配置文件的方式。在 Spring Boot 中，默认情况下，Log4j2 读取 `log4j2-spring.xml`、`log4j2.xml` 配置文件。**一般情况下，我们是使用 `log4j2-spring.xml` 配置文件。**

> 友情提示：Log4j2 也是支持[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#)小节中，采用 Spring Boot 配置文件的方式，😈 只是艿艿想稍微**“复杂”**下本小节的示例。

下面，我们开始本小节的示例，和[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#)小节的效果**等价**。考虑到不污染上述的示例，我们新建一个 [lab-37-logging-log4j2](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-log4j2/) 项目。

## 8.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-demo/pom.xml) 文件中，引入相关依赖。



```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-37-logging-log4j2</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <!-- 去掉对 spring-boot-starter-logging 的依赖-->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- 实现对 SLF4J + Log4j2 的依赖的引入 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```



- 具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢

## 8.2 应用配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-log4j2/src/main/resources/application.yaml) 中，仅添加应用名的配置即可，如下：



```
spring:
  application:
    name: demo-application # 应用名
```



## 8.3 Log4j2 配置文件

在 [`log4j2-spring.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-log4j2/src/main/resources/log4j2-spring.xml) 中，添加 Log4j2 配置，如下：



```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Properties>
        <Property name="PID">????</Property>
        <Property name="LOG_EXCEPTION_CONVERSION_WORD">%xwEx</Property>
        <Property name="LOG_LEVEL_PATTERN">%5p</Property>
        <Property name="LOG_DATEFORMAT_PATTERN">yyyy-MM-dd HH:mm:ss.SSS</Property>
        <!-- 1.1 -->
        <Property name="FILE_LOG_BASE_PATH">/Users/yunai/logs</Property>
        <Property name="APPLICATION_NAME">demo-application</Property>

        <!-- 1.2 -->
        <!-- 控制台的日志格式 -->
        <Property name="CONSOLE_LOG_PATTERN">%clr{%d{${LOG_DATEFORMAT_PATTERN}}}{faint} %clr{${LOG_LEVEL_PATTERN}} %clr{${sys:PID}}{magenta} %clr{---}{faint} %clr{[%15.15t]}{faint} %clr{%-40.40c{1.}}{cyan} %clr{:}{faint} %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
        <!-- 1.3 -->
        <!-- 日志文件的日志格式 -->
        <Property name="FILE_LOG_PATTERN">%d{${LOG_DATEFORMAT_PATTERN}} ${LOG_LEVEL_PATTERN} ${sys:PID} --- [%t] %-40.40c{1.} : %m%n${sys:LOG_EXCEPTION_CONVERSION_WORD}</Property>
    </Properties>

    <Appenders>
        <!-- 2.1 -->
        <!-- 控制台的 Appender -->
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout pattern="${sys:CONSOLE_LOG_PATTERN}" />
        </Console>

        <!-- 2.2 -->
        <!-- 日志文件的 Appender -->
        <RollingFile name="File" fileName="${FILE_LOG_BASE_PATH}/${sys:APPLICATION_NAME}"
                     filePattern="${FILE_LOG_BASE_PATH}/${sys:APPLICATION_NAME}-%d{yyyy-MM-dd-HH}-%i.log.gz">
            <!-- 日志的格式化 -->
            <PatternLayout>
                <Pattern>${sys:FILE_LOG_PATTERN}</Pattern>
            </PatternLayout>
            <!--滚动策略，基于时间 + 大小的分包策略 -->
            <Policies>
                <SizeBasedTriggeringPolicy size="10 MB" />
            </Policies>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- 3.1 -->
        <!-- 常用组件的 Logger 的日志级别 -->
        <Logger name="org.apache.catalina.startup.DigesterFactory" level="error" />
        <Logger name="org.apache.catalina.util.LifecycleBase" level="error" />
        <Logger name="org.apache.coyote.http11.Http11NioProtocol" level="warn" />
        <logger name="org.apache.sshd.common.util.SecurityUtils" level="warn"/>
        <Logger name="org.apache.tomcat.util.net.NioSelectorPool" level="warn" />
        <Logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="error" />
        <Logger name="org.hibernate.validator.internal.util.Version" level="warn" />
        <logger name="org.springframework.boot.actuate.endpoint.jmx" level="warn"/>

        <!-- 3.2 -->
        <!-- 自定义的 Logger 的日志级别 -->
        <logger name="cn.iocoder.springboot.lab37.loggingdemo" level="debug"/>

        <!-- 3.3 -->
        <!-- 设置 Appender ，同时 ROOT 的日志级别为 INFO -->
        <Root level="info">
            <AppenderRef ref="Console" />
            <AppenderRef ref="File" />
        </Root>
    </Loggers>
</Configuration>
```



- 参考 Spring Boot **默认**的 log4j2 XML 配置文件 [`log4j2-file.xml`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2-file.xml)，进行修改。
- `<1.1>` 处，定义日志文件的基础路径和文件名。因为 Spring Boot 并未提供 Log4j2 拓展，无法像[「7. Logback 拓展」](https://www.iocoder.cn/Spring-Boot/Logging/#)一样直接读取 Spring Boot 配置文件，所以这里我们只能直接定义。
- `<1.2>` 处，定义了控制台的日志格式。
- `<1.3>` 处，定义了日志文件的日志格式。
- `<2.1>` 处，配置控制台的 Appender 。
- `<2.2>` 处，配置日志文件的 Appender。
- `<3.1>` 处，设置常用组件的 Logger 的日志级别。
- `<3.2>` 处，自定义的 Logger 的日志级别。这里，我们设置名字为 `"cn.iocoder.springboot.lab37.loggingdemo"` 的 Logger 的日志级别为 `DEBUG`。
- `<3.3>` 处，设置 Root 的 Appender 为控制台和日志文件，日志级别为 `INFO`。

可能胖友对 Log4j2 日志实现框架不是很了解，后续可以看看 [《聊一聊 log4j2 配置文件 log4j2.xml》](http://www.iocoder.cn/Fight/Talk-about-the-log4j2-configuration-file-log4j2-xml/?self) 文章。

## 8.4 Application

创建 [Application](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-log4j2/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/Application.java) 类，用于启动示例项目。代码如下：



```
@SpringBootApplication
public class Application {

    private static Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);

        // <X> 打印日志
        logger.debug("just do it");
    }

}
```



- 在 `<X>` 处，打印 `DEBUG` 级别的日志，测试下不同环境下的日志级别。

## 8.5 简单测试

执行 `Application#main(String[] args)` 方法，启动 Spring Boot 应用。控制台打印日志如下：



```
2020-01-01 11:25:34.512  INFO 16166 --- [           main] c.i.s.l.l.Application                    : Started Application in 1.442 seconds (JVM running for 2.393)
2020-01-01 11:25:34.513 DEBUG 16166 --- [           main] c.i.s.l.l.Application                    : just do it
```



- 我们在 `<X>` 处，打印的 `DEBUG` 日志，成功输出。符合预期

另外，如果胖友想不同环境下的 Log4j2 日志配置的配置不同，并且使用不同的 Log4j2 配置文件，则可以基于[「6. 不同环境下的日志配置」](https://www.iocoder.cn/Spring-Boot/Logging/#)的方案之上，每个 `logging.config` 配置项设置对应的环境的 Log4j2 配置文件。例如说，开发环境对应 `log4j2-spring-dev.xml` 配置文件、生产环境对应 `log4j2-spring-prod.xml` 配置文件。

# 9. 访问日志

在项目中，我们需要记录 HTTP 访问日志。一般来说，有三种方式：

- 方式一，通过 [Filter](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/Filter.java) 过滤器
- 方式二，通过 Spring MVC [HandlerInterceptor](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/java/org/springframework/web/servlet/HandlerInterceptor.java) 处理器过滤器
- 方法三，通过 Spring [AOP](https://docs.spring.io/spring/docs/2.5.x/reference/aop.html) 切面

对于方式一，Spring Web 内置了 [CommonsRequestLoggingFilter](https://github.com/spring-projects/spring-framework/blob/master/spring-web/src/main/java/org/springframework/web/filter/CommonsRequestLoggingFilter.java) 过滤器，已经实现。如果日志内容不是胖友所需，胖友可以参考自定义实现。

对于方式二，艿艿的 [Onemall](https://github.com/YunaiV/onemall) 提供了 [AccessLogInterceptor](https://github.com/YunaiV/onemall/blob/master/common/mall-spring-boot/src/main/java/cn/iocoder/mall/spring/boot/web/interceptor/AccessLogInterceptor.java) 拦截器，已经实现。个人比较推荐这种方式。

对于方式三，本小节会基于 Controller 进行 Spring AOP 切面，记录访问日志。

下面，我们就来看看基于 AOP 实现访问日志的示例。考虑到不污染[「2. 快速入门」](https://www.iocoder.cn/Spring-Boot/Logging/#) 的示例，我们在 [lab-37-logging-demo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-37/lab-37-logging-demo) 项目的基础上，复制出一个 [lab-37-logging-aop](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-aop/) 项目。😈 酱紫，我们也能少写点代码，哈哈哈~

## 9.1 引入依赖

修改 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-aop/pom.xml) 文件中，**额外**引入 `spring-boot-starter-aop` 依赖。如下：



```
<!-- 实现对 Spring AOP 的自动化配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



## 9.2 DemoController

在 [`cn.iocoder.springboot.lab37.loggingdemo.controller`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-aop/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/controller/) 包路径下，创建 [DemoController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-aop/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/controller/DemoController.java) 类，提供不同 API 接口，打印不同级别的日志。代码如下：



```
// DemoController.java

@RestController
@RequestMapping("/demo")
public class DemoController {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @GetMapping("/debug")
    public void debug() {
        logger.debug("debug");
    }

    @GetMapping("/info")
    public void info() {
        logger.info("info");
    }

    @GetMapping("/error")
    public void error() {
        logger.error("error");
    }

}
```



## 9.3 HttpAccessAspect

创建 [HttpAccessAspect](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-37/lab-37-logging-aop/src/main/java/cn/iocoder/springboot/lab37/loggingdemo/aspect/HttpAccessAspect.java) 类，实现对 Controller 进行 Spring AOP 切面，记录访问日志。代码如下：



```
// HttpAccessAspect.java

@Component
@Aspect
public class HttpAccessAspect {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Around("@within(org.springframework.stereotype.Controller)" +
        "|| @within(org.springframework.web.bind.annotation.RestController)")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        // <1.1> 获取类名
        String className = point.getTarget().getClass().getName();
        // <1.2> 获取方法
        String methodName = point.getSignature().getName();
        // <1.3> 记录开始时间
        long beginTime = System.currentTimeMillis();
        // 记录返回结果
        Object result = null;
        Exception ex = null;
        try {
            // <2.1> 执行方法
            result = point.proceed();
            return result;
        } catch (Exception e) {
            // <2.2> 记录异常
            ex = e;
            throw e;
        } finally {
            // <3.1> 计算消耗时间
            long costTime = System.currentTimeMillis() - beginTime;
            // <3.2> 发生异常，则打印 ERROR 日志
            if (ex != null) {
                logger.error("[className: {}][methodName: {}][cost: {} ms][args: {}][发生异常]",
                        className, methodName, point.getArgs(), ex);
            // <3.3> 正常执行，则打印 INFO 日志
            } else {
                logger.info("[className: {}][methodName: {}][cost: {} ms][args: {}][return: {}]",
                        className, methodName, costTime, point.getArgs(), result);
            }
        }
    }

}
```



- 在**类**上，添加 `@Aspect` 注解，标记它是一个**切面类**。同时，添加 `@Component` 注解，保证被 Spring 扫描到，创建一个 Bean 对象。
- 在**方法**上，添加 `@Around(...)` 注解，切入 `@Controller` 和 `@RestController` 注解的 Controller 类的方法的调用。通过 `@Around` 注解，我们可以**手动**控制切入点的方法的调用，从而方便的计算执行时长。
- `<1.1>`、`<1.2>`、`<1.3>` 处，获取类名、方法、开始时间。
- `<2.1>` 处，执行切入点的方法的调用，并记录返回结果。如果发生异常，则在 `<2.2>` 处，进行记录。
- `<3.1>` 处，计算消耗时间。
- `<3.2>` 处，发生异常，则打印 `ERROR` 日志。
- `<3.3>` 处，正常执行，则打印 `INFO` 日志。
- 这里艿艿仅仅提供了较为简单的示例，更加接近实际项目所使用的，可以参考 [RequestLogAspect](https://github.com/chillzhuang/blade-tool/blob/master/blade-core-boot/src/main/java/org/springblade/core/boot/logger/RequestLogAspect.java) 类。

对 Spring AOP 不了解的胖友，可以简单看看 [《彻底征服 Spring AOP 之理论篇》](http://www.iocoder.cn/Fight/Thoroughly-conquer-the-theory-of-Spring-AOP/?self) 文章。

## 9.4 简单测试

执行 `Application#main(String[] args)` 方法，启动项目。

使用浏览器，访问 `/demo/debug` 接口，打印日志如下：



```
2020-01-01 14:43:33.284 DEBUG 18146 --- [nio-8080-exec-6] c.i.s.l.l.controller.DemoController      : debug
2020-01-01 14:43:33.285  INFO 18146 --- [nio-8080-exec-6] c.i.s.l.l.aspect.HttpAccessAspect        : [className: cn.iocoder.springboot.lab37.loggingdemo.controller.DemoController][methodName: debug][cost: 1 ms][args: []][return: null]
```



- HttpAccessAspect 成功切入 Controller 方法的调用，并记录访问日志。

如果胖友想要存储 HTTP 访问日志到数据库中，方便的后续查询使用，可以考虑采用如下几种方案：

- 使用 ELK 方案，通过 Logstash 采集日志文件，持久化到 Elasticsearch 数据库中，最终使用 Kibana 查询与分析。
- 使用 Logback 提供的 [DBAppender](https://github.com/qos-ch/logback/blob/master/logback-classic/src/main/java/ch/qos/logback/classic/db/DBAppender.java) ，将日志写入到数据库中。
- 不使用 Logger ，而是直接使用 DAO 操作，存储访问日志到数据库。例如说，[AccessLogInterceptor](https://github.com/YunaiV/onemall/blob/master/common/mall-spring-boot/src/main/java/cn/iocoder/mall/spring/boot/web/interceptor/AccessLogInterceptor.java) 拦截器。

# 666. 彩蛋

在生产环境下，我们会集群部署我们的应用。那么我们可能需要登录多台服务器，查看不用应用节点下的日志，这样会非常不方便。所以一般情况下，我们会采用 ELK 搭建日志平台，将日志进行采集，统一存储。感兴趣的胖友，可以阅读 [《芋道 Spring Boot 日志平台 ELK 入门》](http://www.iocoder.cn/Spring-Boot/ELK/?self)

这里额外在推荐一些 Logger 相关的不错的内容：

- [《SpringBoot 2.x 中文文档 —— 日志记录》](https://docshome.gitbooks.io/springboot/content/pages/spring-boot-features.html#boot-features-logging)
- [《Logback 中文手册/文档》](https://github.com/YLongo/logback-chinese-manual)
- [《Java 日志框架那些事儿》](http://www.iocoder.cn/Fight/something_about_java_log_framework/?self)
- [《惊讶！我定的日志规范被 CTO 在全公司推广了》](http://www.iocoder.cn/Fight/Fight/Surprise-The-logging-specification-I-ordered-was-promoted-by-the-CTO-throughout-the-company/?self)
SpringBoot-数据库连接池

# 1. 概述

在我们的项目中，数据库连接池基本是必不可少的组件。在目前数据库连接池的选型中，主要是

- [Druid](https://github.com/alibaba/druid) ，为**监控**而生的数据库连接池。
- [HikariCP](https://github.com/brettwooldridge/HikariCP) ，号称**性能**最好的数据库连接池。

至于怎么选择，两者都非常优秀，不用过多纠结。

- Spring Boot 2.X 版本，默认采用 HikariCP 。
- 阿里大规模采用 Druid 。

当然，如下有一些资料，胖友可以阅读参考：

- [《Druid 连接池介绍》](https://github.com/alibaba/druid/wiki/Druid连接池介绍)
- [《为什么 HikariCP 被号称为性能最好的 Java 数据库连接池，如何配置使用》](https://blog.csdn.net/clementad/article/details/46928621)
- [《alibaba/druid pool analysis》](https://github.com/brettwooldridge/HikariCP/issues/232) ，一个小小的“撕逼”。

下面，我们来进行 HikariCP 和 Druid 的入门，会配置单数据源和多数据源情况下的连接池。

# 2. HikariCP 单数据源

> 示例代码对应仓库：[lab-19-datasource-pool-hikaricp-single](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-19/lab-19-datasource-pool-hikaricp-single) 。

在本小节，我们会使用配置一个数据源的 HikariCP 连接池。

> 艿艿：推荐入门后，可以看看 HikariCP 的文档：https://github.com/brettwooldridge/HikariCP/wiki 。

## 2.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-single/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-19-datasource-pool-hikaricp-single</artifactId>

    <dependencies>
        <!-- 实现对数据库连接池的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency> <!-- 本示例，我们使用 MySQL -->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
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



- 无需主动引入 HikariCP 的依赖。因为在 Spring Boot 2.X 中，`spring-boot-starter-jdbc` 默认引入 [`com.zaxxer.HikariCP`](https://mvnrepository.com/artifact/com.zaxxer/HikariCP) 依赖。

## 2.2 应用配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-single/src/main/resources/application.yaml) 中，添加 HikariCP 配置，如下：



```
spring:
  # datasource 数据源配置内容，对应 DataSourceProperties 配置属性类
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: root # 数据库账号
    password: # 数据库密码yml
    # HikariCP 自定义配置，对应 HikariConfig 配置属性类
    hikari:
      minimum-idle: 10 # 池中维护的最小空闲连接数，默认为 10 个。
      maximum-pool-size: 10 # 池中最大连接数，包括闲置和使用中的连接，默认为 10 个。
```



- 在 `spring.datasource` 配置项下，我们可以添加数据源的**通用**配置。
- 在 `spring.datasource.hikari` 配置项下，我们可以添加 HikariCP 连接池的**自定义**配置。然后 [`DataSourceConfiguration.Hikari`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceConfiguration.java#L72-L92) 会自动化配置 HikariCP 连接池。

HikariCP 更多配置项，可以看看如下表格：s

> FROM [《HikariCP 连接池及其在 Spring Boot 中的配置》](https://blog.csdn.net/qq_32953079/article/details/81502237)

|          配置项           |                             描述                             | 构造器默认值                   | 默认配置validate之后的值 | validate重置                                                 |
| :-----------------------: | :----------------------------------------------------------: | :----------------------------- | :----------------------- | :----------------------------------------------------------- |
|        autoCommit         |                   自动提交从池中返回的连接                   | true                           | true                     | -                                                            |
|     connectionTimeout     |                 等待来自池的连接的最大毫秒数                 | SECONDS.toMillis(30) = 30000   | 30000                    | 如果小于250毫秒，则被重置回30秒                              |
|        idleTimeout        |                 连接允许在池中闲置的最长时间                 | MINUTES.toMillis(10) = 600000  | 600000                   | 如果idleTimeout+1秒>maxLifetime 且 maxLifetime>0，则会被重置为0（代表永远不会退出）；如果idleTimeout!=0且小于10秒，则会被重置为10秒 |
|        maxLifetime        |                     池中连接最长生命周期                     | MINUTES.toMillis(30) = 1800000 | 1800000                  | 如果不等于0且小于30秒则会被重置回30分钟                      |
|    connectionTestQuery    |   如果您的驱动程序支持JDBC4，我们强烈建议您不要设置此属性    | null                           | null                     | -                                                            |
|        minimumIdle        |                   池中维护的最小空闲连接数                   | -1                             | 10                       | minIdle<0或者minIdle>maxPoolSize,则被重置为maxPoolSize       |
|      maximumPoolSize      |            池中最大连接数，包括闲置和使用中的连接            | -1                             | 10                       | 如果maxPoolSize小于1，则会被重置。当minIdle<=0被重置为DEFAULT_POOL_SIZE则为10;如果minIdle>0则重置为minIdle的值 |
|      metricRegistry       | 该属性允许您指定一个 Codahale / Dropwizard `MetricRegistry` 的实例，供池使用以记录各种指标 | null                           | null                     | -                                                            |
|    healthCheckRegistry    | 该属性允许您指定池使用的Codahale / Dropwizard HealthCheckRegistry的实例来报告当前健康信息 | null                           | null                     | -                                                            |
|         poolName          | 连接池的用户定义名称，主要出现在日志记录和JMX管理控制台中以识别池和池配置 | null                           | HikariPool-1             | -                                                            |
| initializationFailTimeout |  如果池无法成功初始化连接，则此属性控制池是否将 `fail fast`  | 1                              | 1                        | -                                                            |
|  isolateInternalQueries   |     是否在其自己的事务中隔离内部池查询，例如连接活动测试     | false                          | false                    | -                                                            |
|    allowPoolSuspension    |               控制池是否可以通过JMX暂停和恢复                | false                          | false                    | -                                                            |
|         readOnly          |             从池中获取的连接是否默认处于只读模式             | false                          | false                    | -                                                            |
|      registerMbeans       |               是否注册JMX管理Bean（`MBeans`）                | false                          | false                    | -                                                            |
|          catalog          |       为支持 `catalog` 概念的数据库设置默认 `catalog`        | driver default                 | null                     | -                                                            |
|     connectionInitSql     | 该属性设置一个SQL语句，在将每个新连接创建后，将其添加到池中之前执行该语句。 | null                           | null                     | -                                                            |
|      driverClassName      | HikariCP将尝试通过仅基于jdbcUrl的DriverManager解析驱动程序，但对于一些较旧的驱动程序，还必须指定driverClassName | null                           | null                     | -                                                            |
|   transactionIsolation    |             控制从池返回的连接的默认事务隔离级别             | null                           | null                     | -                                                            |
|     validationTimeout     |                 连接将被测试活动的最大时间量                 | SECONDS.toMillis(5) = 5000     | 5000                     | 如果小于250毫秒，则会被重置回5秒                             |
|  leakDetectionThreshold   |    记录消息之前连接可能离开池的时间量，表示可能的连接泄漏    | 0                              | 0                        | 如果大于0且不是单元测试，则进一步判断：(leakDetectionThreshold < SECONDS.toMillis(2) or (leakDetectionThreshold > maxLifetime && maxLifetime > 0)，会被重置为0 . 即如果要生效则必须>0，而且不能小于2秒，而且当maxLifetime > 0时不能大于maxLifetime |
|        dataSource         | 这个属性允许你直接设置数据源的实例被池包装，而不是让HikariCP通过反射来构造它 | null                           | null                     | -                                                            |
|          schema           |           该属性为支持模式概念的数据库设置默认模式           | driver default                 | null                     | -                                                            |
|       threadFactory       | 此属性允许您设置将用于创建池使用的所有线程的java.util.concurrent.ThreadFactory的实例。 | null                           | null                     | -                                                            |
|     scheduledExecutor     | 此属性允许您设置将用于各种内部计划任务的java.util.concurrent.ScheduledExecutorService实例 | null                           | null                     | -                                                            |

## 2.3 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-single/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```java
// Application.java

@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        try (Connection conn = dataSource.getConnection()) {
            // 这里，可以做点什么
            logger.info("[run][获得连接：{}]", conn);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

}
```



通过实现 [CommandLineRunner](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-command-line-runner) 接口，应用启动完成后，回调 `#run(String... args)` 方法，输出下 Connection 信息。执行日志如下：



```bash
2019-11-12 11:15:32.303  INFO 41198 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2019-11-12 11:15:32.472  INFO 41198 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2019-11-12 11:15:32.473  INFO 41198 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得连接：HikariProxyConnection@1561745898 wrapping com.mysql.jdbc.JDBC4Connection@793138bd]
```

- 可以看到，HikariDataSource 成功初始化。

# 3. HikariCP 多数据源

> 示例代码对应仓库：[lab-19-datasource-pool-hikaricp-multiple](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-19/lab-19-datasource-pool-hikaricp-multiple) 。

在本小节，我们会使用配置**两个**数据源的 HikariCP 连接池。

## 3.1 引入依赖

和 [「2.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 是一致。

## 3.2 应用配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-multiple/src/main/resources/application.yaml) 中，添加 HikariCP 配置，如下：



```我现在存的也是md5
spring:
  # datasource 数据源配置内容
  datasource:
    # 订单数据源配置
    orders:
      url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password:
      # HikariCP 自定义配置，对应 HikariConfig 配置属性类
      hikari:
        minimum-idle: 20 # 池中维护的最小空闲连接数，默认为 10 个。
        maximum-pool-size: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 10 个。
    # 用户数据源配置
    users:
      url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password:
      # HikariCP 自定义配置，对应 HikariConfig 配置属性类
      hikari:
        minimum-idle: 15 # 池中维护的最小空闲连接数，默认为 10 个。
        maximum-pool-size: 15 # 池中最大连接数，包括闲置和使用中的连接，默认为 10 个。
```



- 我们在 `spring.datasource` 配置项下，定义了 `orders` 和 `users` 两个数据源的配置。而每个数据源的配置，和我们在 [「2.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 是一致的。

## 3.3 数据源配置类

### 3.3.1 错误的示例

在网上，我们会看到这样配置多个数据源的配置类。代码如下：



```java
@Bean(name = "ordersDataSource")
@ConfigurationProperties(prefix = "spring.datasource.orders")
public DataSource ordersDataSource() {
   return DataSourceBuilder.create().build();
}

@Bean(name = "usersDataSource")
@ConfigurationProperties(prefix = "spring.datasource.users")
public DataSource ordersDataSource() {
   return DataSourceBuilder.create().build();
}
```



- 实际上，这样做的话，在部分场景下，会存在问题，这是为什么呢？
- 我们先来理解这段程序的用途。以`#ordersDataSource()`方法为例子：
  - `DataSourceBuilder.create().build()` 代码段，会创建一个 DataSource 数据源。
  - 搭配上 `@Bean(name = "ordersDataSource")` 注解，会创建一个名字为 `"ordersDataSource"` 的 DataSource Bean 。这里，我们使用 HikariCP ，所以返回的是 HikariDataSource Bean 。
  - `@ConfigurationProperties(prefix = "spring.datasource.orders")` 注解，会将 `"spring.datasource.orders"` 配置项，逐个属性赋值给 DataSource Bean 。
- 看起来貌似没问题，但是如果每个数据源如果有 HikariCP 的 `"hikari"` 自定义配置项时，**它的自定义配置项无法设置到 HikariDataSource Bean 中**。因为，`"spring.datasource.orders.hikari"` 是 `"spring.datasource.orders"` 的**第二层**属性。而 [HikariDataSource](https://github.com/openbouquet/HikariCP/blob/master/src/main/java/com/zaxxer/hikari/HikariDataSource.java) 的[配置属性](https://github.com/openbouquet/HikariCP/blob/master/src/main/java/com/zaxxer/hikari/HikariConfig.java)在**第一层**，这就导致无法正确的设置。

虽然存在该问题，但是大多数项目，我们并不会自定义 HikariCP 的 `"hikari"` 配置项，所以这个问题就偷偷藏起来，**“不存在”**了。

### 3.3.2 正确的示例

当然，作为入门的示例，艿艿还是希望提供正确的做法。

在 [`cn.iocoder.springboot.lab19.datasourcepool.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-19/lab-19-datasource-pool-hikaricp-multiple/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/config) 包路径下，我们会创建 [DataSourceConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-multiple/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/config/DataSourceConfig.java) 配置类。代码如下：

```java
// DataSourceConfig.java

@Configuration
public class DataSourceConfig {

    /**
     * 创建 orders 数据源的配置对象
     */
    @Primary
    @Bean(name = "ordersDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.orders") // 读取 spring.datasource.orders 配置到 DataSourceProperties 对象
    public DataSourceProperties ordersDataSourceProperties() {
        return new DataSourceProperties();
    }

    /**
     * 创建 orders 数据源
     */
    @Bean(name = "ordersDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.orders.hikari") // 读取 spring.datasource.orders 配置到 HikariDataSource 对象
    public DataSource ordersDataSource() {
        // <1.1> 获得 DataSourceProperties 对象
        DataSourceProperties properties =  this.ordersDataSourceProperties();
        // <1.2> 创建 HikariDataSource 对象
        return createHikariDataSource(properties);
    }

    /**
     * 创建 users 数据源的配置对象
     */
    @Bean(name = "usersDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.users") // 读取 spring.datasource.users 配置到 DataSourceProperties 对象
    public DataSourceProperties usersDataSourceProperties() {
        return new DataSourceProperties();
    }

    /**
     * 创建 users 数据源
     */
    @Bean(name = "usersDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.users.hikari")
    public DataSource usersDataSource() {
        // 获得 DataSourceProperties 对象
        DataSourceProperties properties =  this.usersDataSourceProperties();
        // 创建 HikariDataSource 对象
        return createHikariDataSource(properties);
    }

    private static HikariDataSource createHikariDataSource(DataSourceProperties properties) {
        // 创建 HikariDataSource 对象
        HikariDataSource dataSource = properties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
        // 设置线程池名
        if (StringUtils.hasText(properties.getName())) {
            dataSource.setPoolName(properties.getName());
        }
        return dataSource;
    }

}
```



- 这块代码，我们是参考 Spring Boot [`DataSourceConfiguration.Hikari`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceConfiguration.java#L72-L92) 配置类来实现的。`#ordersDataSourceProperties()`方法，创建`"orders"`数据源的 DataSourceProperties 配置对象。

  - [`@Primary`](http://www.yihaomen.com/article/java/581.htm) 注解，保证项目中有一个**主**的 DataSourceProperties Bean 。
  - `new DataSourceProperties()` 代码段，会创建一个 DataSourceProperties 数据源的配置对象。
  - 搭配上 `@Bean(name = "ordersDataSourceProperties")` 注解，会创建一个名字为 `"ordersDataSourceProperties"` 的 DataSourceProperties Bean 。
  - `@ConfigurationProperties(prefix = "spring.datasource.orders")` 注解，会将 `"spring.datasource.orders"` 配置项，逐个属性赋值给 DataSourceProperties Bean 。`#ordersDataSource()`方法，创建`orders`数据源。

  - `<1.1>` 处，调用 `#ordersDataSourceProperties()` 方法，获得 `orders` 数据源的 DataSourceProperties 。
  - `<1.2>` 处，调用 `#createHikariDataSource(DataSourceProperties properties)` 方法，创建 HikariDataSource 对象。这样，`"spring.datasource.orders"` 配置项，逐个属性赋值给 HikariDataSource Bean 。
  - 搭配上 `@Bean(name = "ordersDataSource")` 注解，会创建一个名字为 `"ordersDataSource"` 的 HikariDataSource Bean 。
  - `@ConfigurationProperties(prefix = "spring.datasource.orders.hikari")` 注解，会将 HikariCP 的 `"spring.datasource.orders.hikari"` 自定义配置项，逐个属性赋值给 HikariDataSource Bean 。

- `users` 数据源的配置，同上，就不重复解释了。

## 3.4 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-multiple/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```java
// Application.java

@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Resource(name = "ordersDataSource")
    private DataSource ordersDataSource;

    @Resource(name = "usersDataSource")
    private DataSource usersDataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        // orders 数据源
        try (Connection conn = ordersDataSource.getConnection()) {
            // 这里，可以做点什么
            logger.info("[run][ordersDataSource 获得连接：{}]", conn);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }

        // users 数据源
        try (Connection conn = usersDataSource.getConnection()) {
            // 这里，可以做点什么
            logger.info("[run][usersDataSource 获得连接：{}]", conn);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

}
```



执行日志如下：



```bash
2019-11-12 15:30:35.060  INFO 45868 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2019-11-12 15:30:35.365  INFO 45868 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2019-11-12 15:30:35.367  INFO 45868 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][ordersDataSource 获得连接：HikariProxyConnection@1041547629 wrapping com.mysql.jdbc.JDBC4Connection@3c989952]
2019-11-12 15:30:35.371  INFO 45868 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Starting...
2019-11-12 15:30:35.376  INFO 45868 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-2 - Start completed.
2019-11-12 15:30:35.376  INFO 45868 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][usersDataSource 获得连接：HikariProxyConnection@795748540 wrapping com.mysql.jdbc.JDBC4Connection@7c098bb3]
```



- 可以看到，两个 HikariDataSource 成功初始化。

多数据源和 JPA、MyBatis、JdbcTemplate 的集成，可以看看 [《芋道 Spring Boot 多数据源（读写分离）入门》](http://www.iocoder.cn/Spring-Boot/dynamic-datasource/?self) 文章。

# 4. Druid 单数据源

> 示例代码对应仓库：[lab-19-datasource-pool-druid-single](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-19/lab-19-datasource-pool-druid-single) 。

在本小节，我们会使用配置一个数据源的 Druid 连接池。并简单看看 Druid 的监控功能。

> 艿艿：推荐入门后，可以看看 Druid 的文档：https://github.com/alibaba/druid/wiki/ 。

## 4.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-druid-single/pom.xml) 文件中，引入相关依赖。



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-19-datasource-pool-druid-single</artifactId>

    <dependencies>
        <!-- 保证 Spring JDBC 的依赖健全 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- 实现对 Druid 连接池的自动化配置 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.21</version>
        </dependency>
        <dependency> <!-- 本示例，我们使用 MySQL -->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
        </dependency>

        <!-- 实现对 Spring MVC 的自动化配置，因为我们需要看看 Druid 的监控功能 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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



具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。S

## 4.2 应用配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-druid-single/src/main/resources/application.yaml) 中，添加 Druid 配置，如下：



```yml
spring:
  # datasource 数据源配置内容，对应 DataSourceProperties 配置属性类
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: root # 数据库账号
    password: # 数据库密码
    type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
    # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
    druid:
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
      filter:
        stat: # 配置 StatFilter ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter
          log-slow-sql: true # 开启慢查询记录
          slow-sql-millis: 5000 # 慢 SQL 的标准，单位：毫秒
      stat-view-servlet: # 配置 StatViewServlet ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
        enabled: true # 是否开启 StatViewServlet
        login-username: yudaoyuanma # 账号
        login-password: javaniubi # 密码
```



- `spring.datasource` 配置项，设置 Spring 数据源的通用配置。其中，`spring.datasource.type` 配置项，**需要主动**设置使用 DruidDataSource 。因为默认情况下，`spring-boot-starter-jdbc` 的 [DataSourceBuilder](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/DataSourceBuilder.java#L49-L50) 会按照 [`DATA_SOURCE_TYPE_NAMES`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/DataSourceBuilder.java#L49-L50) 的顺序，尝试推断数据源的类型。`spring.datasource.druid`配置项，设置 Druid 连接池的自定义配置。然后DruidDataSourceAutoConfigure

   会自动化配置 Druid 连接池。

  - 在 [《Druid —— 配置属性》](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter#配置属性) 和 [《DruidDataSource 配置属性列表》](https://github.com/alibaba/druid/wiki/DruidDataSource配置属性列表) 下，提供了各种 Druid 的配置项，胖友可以自己看看。
  - `filter.stat` 配置项，我们配置了 Druid [StatFilter](https://github.com/alibaba/druid/blob/master/src/main/java/com/alibaba/druid/filter/stat/StatFilter.java) ，用于统计监控信息。对应文档 [《Druid —— 配置_StatFilter》](https://github.com/alibaba/druid/wiki/配置_StatFilter) 。要注意，StatFilter 并不是我们说的 [`javax.servlet.Filter`](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/Filter.java) ，而是 Druid 提供的 [Filter](https://github.com/alibaba/druid/wiki/配置_Filter配置) 拓展机制。
  - `filter.stat-view-servlet` 配置项，我们配置了 Druid [StatViewServlet](https://github.com/alibaba/druid/blob/master/src/main/java/com/alibaba/druid/support/http/StatViewServlet.java) ，用于提供监控信息的**展示的 html 页面**和 **JSON API** 。对应文档 [《Druid —— 配置_StatViewServlet 配置》](https://github.com/alibaba/druid/wiki/配置_StatViewServlet配置) 。StatViewServlet 就是我们说的 [`javax.servlet.Filter`](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/Servlet.java) 。

## 4.3 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-hikaricp-single/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```java
@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        logger.info("[run][获得数据源：{}]", dataSource.getClass());
    }

}
```



执行日志如下：



```bash
2019-11-12 19:34:12.079  INFO 48359 --- [           main] c.a.d.s.b.a.DruidDataSourceAutoConfigure : Init DruidDataSource
2019-11-12 19:34:12.156  INFO 48359 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited

2019-11-12 19:34:12.560  INFO 48359 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得数据源：class com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceWrapper]
```



- 可以看到，DruidDataSource 成功初始化。

## 4.4 监控功能

因为我们在 [「4.2 应用配置中」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 中，做了如下操作：

- 通过 `spring.datasource.filter.stat` 配置了 StatFilter ，统计监控信息。
- 通过 `spring.datasource.filter.stat-view-servlet` 配置了 StatViewServlet ，提供监控信息的展示的 html 页面和 JSON API 。

所以我们在启动项目后，访问 `http://127.0.0.1:8080/druid` 地址，可以看到监控 html 页面。如下图所示：![监控界面](E:\Development\Typora\images\01-16598824427651.png)

- 在界面的顶部，提供了数据源、SQL 监控、SQL 防火墙等等功能。

- 每个界面上，可以通过 [View JSON API](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 获得数据的来源。同时，我们可以在 [JSON API( `http://127.0.0.1:8080/druid/api.html` )](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 菜单对应的界面中，看到 StatViewServlet 内置的监控信息的 JSON API 列表。

- 因为监控信息是存储在 JVM 内存中，在 JVM 进程重启时，信息将会丢失。如果我们希望持久化到 MySQL、Elasticsearch、HBase 等存储器中，可以通过 StatViewServlet 提供的 JSON API 接口，采集监控信息。另外，有个 [druid-aggregated-monitor](https://github.com/bungder/druid-aggregated-monitor) 开源项目，提供了 集中监控分布式服务中的 Druid 连接池的方案和思路。

- 如果 StatViewServlet 提供的 JSON API 接口，无法满足我们的诉求，我们可以通过自定义 API 接口，使用 [DruidStatManagerFacade](https://github.com/alibaba/druid/blob/master/src/main/java/com/alibaba/druid/stat/DruidStatManagerFacade.java) 获得监控信息。使用示例 [DruidStatController](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-druid-single/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/controller/DruidStatController.java) 代码如下：

  

  ```java
  // DruidStatController.java
  
  @RestController
  public class DruidStatController {
  
      @GetMapping("/monitor/druid/stat")
      @Deprecated
      public Object druidStat(){
          // `DruidStatManagerFacade#getDataSourceStatDataList()` 方法，可以获取所有数据源的监控数据。
          // 除此之外，DruidStatManagerFacade 还提供了一些其他方法，你可以按需选择使用。
          return DruidStatManagerFacade.getInstance().getDataSourceStatDataList();
      }
  
  }
  ```

  

  - 当然，绝大多数情况下，我们并不需要做这方面的拓展。

# 5. Druid 多数据源

> 示例代码对应仓库：[lab-19-datasource-pool-druid-multiple](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-19/lab-19-datasource-pool-druid-multiple) 。

在本小节，我们会使用配置**两个**数据源的 Druid 连接池。

## 5.1 引入依赖

和 [「4.1 引入依赖」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 是一致。

## 5.2 应用配置

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-druid-multiple/src/main/resources/application.yaml) 中，添加 Druid 配置，如下：



```yml
spring:
  # datasource 数据源配置内容
  datasource:
    # 订单数据源配置
    orders:
      url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password:
      type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
      # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
    # 用户数据源配置
    users:
      url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password:
      type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
      # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
    # Druid 自定已配置
    druid:
      # 过滤器配置
      filter:
        stat: # 配置 StatFilter ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter
          log-slow-sql: true # 开启慢查询记录
          slow-sql-millis: 5000 # 慢 SQL 的标准，单位：毫秒
      # StatViewServlet 配置
      stat-view-servlet: # 配置 StatViewServlet ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
        enabled: true # 是否开启 StatViewServlet
        login-username: yudaoyuanma # 账号
        login-password: javaniubi # 密码
```



- 不同于我们在 [「3.2 应用配置文件」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 中，我们将 Druid 的自定义配置，和 `url`、`driver-class-name` 等数据源的通用配置放在同一级，这样后续我们只需要使用 `@ConfigurationProperties(prefix = "spring.datasource.orders")` 的方式，就能完成 DruidDataSource 的配置属性设置。
- 在 `spring.datasource.druid` 配置项下，我们还是配置了 `filter.stat` 和 `stat-view-servlet` 配置项，用于 Druid 监控功能。

## 5.3 数据源配置类

在 [`cn.iocoder.springboot.lab19.datasourcepool.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-19/lab-19-datasource-pool-druid-multiple/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/config) 包路径下，我们会创建 [DataSourceConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-druid-multiple/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/config/DataSourceConfig.java) 配置类。代码如下：



```java
// DataSourceConfig.java

@Configuration
public class DataSourceConfig {

    /**
     * 创建 orders 数据源
     */
    @Primary
    @Bean(name = "ordersDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.orders") // 读取 spring.datasource.orders 配置到 HikariDataSource 对象
    public DataSource ordersDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    /**
     * 创建 users 数据源
     */
    @Bean(name = "usersDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.users")
    public DataSource usersDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

}
```



- 因为我们在 [「5.2 应用配置」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 中，将 Druid 自定义的配置项，和数据源的通用配置放在了同一级，所以我们只需使用 `@ConfigurationProperties(prefix = "spring.datasource.orders")` 这样的方式即可。
- 当然，[「3.3.2 正确的示例」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 也是可以这么做的。实际情况下，比较推荐本小节的方式。

## 5.4 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-19/lab-19-datasource-pool-druid-multiple/src/main/java/cn/iocoder/springboot/lab19/datasourcepool/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```java
// Application.java

@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Resource(name = "ordersDataSource")
    private DataSource ordersDataSource;

    @Resource(name = "usersDataSource")
    private DataSource usersDataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        // orders 数据源
        logger.info("[run][获得数据源：{}]", ordersDataSource.getClass());

        // users 数据源
        logger.info("[run][获得数据源：{}]", usersDataSource.getClass());
    }

}
```



执行日志如下：



```bash
2019-11-12 21:39:24.063  INFO 49670 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得数据源：class com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceWrapper]
2019-11-12 21:39:24.064  INFO 49670 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得数据源：class com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceWrapper]
```



- 可以看到，两个 DruidDataSource 成功初始化。

## 5.5 监控功能

和 [「4.4 监控功能」](https://www.iocoder.cn/Spring-Boot/datasource-pool/?self#) 一致。

不过呢，我们在监控页面上，可以看到两个 Druid 数据库连接池。

# 666. 彩蛋

> 艿艿：咳咳咳，瞎哔哔了一些内容，可以选择不看。😜

艿艿在星球里，做了一波目前在使用的连接池的调查，大概比例是 Druid : HikariCP 为 2:1 左右。猜测随着 Spring Boot 2.X 逐步普及之后，HikariCP 有一定几率反超 Druid 。

虽然说，HikariCP 没有直接提供监控功能，但是可以使用 [Prometheus](https://prometheus.io/) 采集 Spring Boot Metrics 的数据，后续使用 [Grafana](https://grafana.com/) 制作仪表盘。目前，已经有 [Spring Boot HikariCP / JDBC](https://grafana.com/grafana/dashboards/6083) 可以直接使用。具体怎么做，胖友可以看看 [《Spring Boot 中使用 HikariCP 连接池》](https://blog.frognew.com/2019/02/spring-boot-guides-hikari.html) 文章。

Druid 的 [Issues 3047](https://github.com/alibaba/druid/issues/3047) 中，也有人提出，是否能够提供 Druid 接入 Prometheus 统一监控的诉求。Druid 目前暂时不支持，不过有聪慧的胖友，提出了使用 [Prometheus jmx_exporter](https://github.com/prometheus/jmx_exporter) 的方式，将 Druid 实现的 JMX 格式的指标暴露出来，提供给 Prometheus 采集监控信息。

在编写本文的过程中，无意中看到 Druid 文档中提到，曾经想试验性的提供 [分库分表](https://github.com/alibaba/druid/wiki/分库分表) 的功能，而艿艿的记忆中，[Sharding-JDBC](https://shardingsphere.apache.org/document/current/cn/quick-start/sharding-jdbc-quick-start/) 曾经也想开发数据库连接池的功能。大体在 DataSource 数据源上做拓展的中间件，可能都不甘于仅仅只覆盖一块需求，而是希望成为一站式的数据库中间件。立个 Flag ，[ShardingSphere](https://shardingsphere.apache.org/document/current/cn/overview/) 可能会提供数据库连接池的组件。

> 旁白君：Sharding-JDBC 是 ShardingSphere 在 JDBC 层面提供的分库分表组件。当然，不仅仅提供分库分表的功能，也提供读写分离、数据脱敏、分布式事务等等功能。

如果胖友工作的比较早，一定还接触过其它连接池。例如说，[c3p0](https://github.com/swaldman/c3p0)、[dbcp](https://github.com/apache/commons-dbcp)、[BoneCP](https://github.com/wwadge/bonecp) 等等。数据库连接池的发展过程，是个非常有意思的历史。感兴趣的胖友，可以看看 [《大话数据库连接池简史，你都用过几个？》](http://www.iocoder.cn/Fight/talk-about-database-connection-pool/?self) 一文，江湖味十足~

可能胖友会比较纠结，是否要去自定义连接池的配置呢？一般情况下，默认的配置基本能够满足项目的基本要求，不需要特别刻意去修改。当然，这里推荐看两篇文章：

- [《DruidDataSource 配置》](https://github.com/alibaba/druid/wiki/DruidDataSource配置) ，Druid 官方提供了通用的配置。
- [《Druid 连接池推荐配置》](http://rabbitgyk.com/2018/01/04/druid-configuration/) ，某公司的内部实践。
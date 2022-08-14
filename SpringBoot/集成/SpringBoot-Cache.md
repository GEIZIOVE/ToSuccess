# 1. 概述

在系统访问量越来越大之后，往往最先出现瓶颈的往往是数据库。而为了减少数据库的压力，**我们可以选择让产品砍掉消耗数据库性能的需求**。当然，我们也可以选择如下方式优化：

> 艿艿：在这里，我们暂时不考虑优化数据库的硬件，索引等等手段。

- 读写分离。通过将读操作分流到从节点，避免主节点压力过多。
- 分库分表。通过将读写操作分摊到多个节点，避免单节点压力过多。
- 缓存。相比数据库来说，缓存往往能够提供更快的读性能，减小数据库的压力。关于缓存和数据库的性能情况，可以看看如下两篇文章：
  - [《性能测试 —— Redis 基准测试》](http://www.iocoder.cn/Performance-Testing/Redis-benchmark/?self)
  - [《性能测试 —— MySQL 基准测试》](http://www.iocoder.cn/Performance-Testing/MySQL-benchmark/?self)

那么，在引入缓存之后，我们的读操作的代码，往往代码如下：



```java
// UserService.java

@Autowired
private UserMapper userMapper; // 读取 DB

@Autowired
private UserCacheDao userCacheDao; // 读取 Cache

public UserDO getUser(Integer id) {
    // 从 Cache 中，查询用户信息
    UserDO user = userCacheDao.get(id);
    if (user != null) {
        return user;
    }
    // 如果 Cache 查询不到，从 DB 中读取
    user = userMapper.selectById(id);
    if (user != null) { // 非空，则缓存到 Cache 中
        userCacheDao.put(user);
    }
    // 返回结果
    return user;
}
```



- 这段代码，是比较常用的缓存策略，俗称**“被动写”**。整体步骤如下：
  - 1）首先，从 Cache 中，读取用户缓存。如果存在，则直接返回。
  - 2）然后，从 DB 中，读取用户数据。如果存在，写入 Cache 中。
  - 3）最后，返回 DB 的查询结果。
- 可能会有胖友说，这里没有考虑缓存击穿、缓存穿透、缓存并发写的情况。恩，是的，但是这并不在本文的内容范围。感兴趣的，可以看看我的男神超哥写的 [《缓存穿透、缓存并发、缓存失效之思路变迁》](http://www.iocoder.cn/Fight/Cache-penetration-and-Cache-concurrency-and-Cache-invalidation/?self) 文章。嘿嘿~

虽然说，上述的代码已经挺简洁了，但是我们是热爱“偷懒”的开发者，必然需要寻找更优雅（偷懒）的方式。在 Spring 3.1 版本的时候，它发布了 [Spring Cache](https://github.com/spring-projects/spring-framework/tree/master/spring-context/src/main/java/org/springframework/cache) 。关于它的介绍，如下：

> FROM [《注释驱动的 Spring Cache 缓存介绍》](https://www.ibm.com/developerworks/cn/opensource/os-cn-spring-cache/)
>
> Spring 3.1 引入了激动人心的基于注释（annotation）的缓存（cache）技术，它本质上不是一个具体的缓存实现方案（例如 EHCache 或者 OSCache），而是一个对缓存使用的抽象，通过在既有代码中添加少量它定义的各种 annotation，即能够达到缓存方法的返回对象的效果。
>
> Spring 的缓存技术还具备相当的灵活性，不仅能够使用 SpEL（Spring Expression Language）来定义缓存的 key 和各种 condition，还提供开箱即用的缓存临时存储方案，也支持和主流的专业缓存例如 EHCache 集成。
>
> 其特点总结如下：
>
> - 通过少量的配置 annotation 注释即可使得既有代码支持缓存
> - 支持开箱即用 Out-Of-The-Box，即不用安装和部署额外第三方组件即可使用缓存
> - 支持 Spring Express Language，能使用对象的任何属性或者方法来定义缓存的 key 和 condition
> - 支持 AspectJ，并通过其实现任何方法的缓存支持
> - 支持自定义 key 和自定义缓存管理者，具有相当的灵活性和扩展性

- 介绍有点略长，胖友耐心看看噢。
- 简单来说，我们可以像使用 `@Transactional` 声明式**事务**，使用 Spring Cache 提供的 `@Cacheable` 等注解，😈 声明式**缓存**。而在实现原理上，也是基于 Spring AOP 拦截，实现缓存相关的操作。

下面，我们使用 Spring Cache 将 `#getUser(Integer id)` 方法进行简化。代码如下：



```java
// UserService.java
public UserDO getUser2(Integer id) {
    return userMapper.selectById(id);
}

// UserMapper.java
@Cacheable(value = "users", key = "#id")
UserDO selectById(Integer id);
```



- 在 UserService 的 `#getUser2(Integer id)` 方法上，我们直接调用 UserMapper ，从 DB 中查询数据。
- 在 UserMapper 的 `#selectById(Integer id)` 方法上，有 `@Cacheable` 注解。Spring Cache 会拦截有 `@Cacheable` 注解的方法，实现“**被动写**”的逻辑。

是不是瞬间很清爽。下面，让我们开始愉快的入门吧。

# 2. 注解

在入门 Spring Cache 之前，我们先了解下其提供的所有注解：

- `@Cacheable`
- `@CachePut`
- `@CacheEvict`
- `@CacheConfig`
- `@Caching`
- `@EnableCaching`

## 2.1 @Cacheable

[`@Cacheable`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/annotation/Cacheable.java) 注解，添加在方法上，缓存方法的执行结果。执行过程如下：

- 1）首先，判断方法执行结果的缓存。如果有，则直接返回该缓存结果。
- 2）然后，执行方法，获得方法结果。
- 3）之后，根据是否满足缓存的条件。如果满足，则缓存方法结果到缓存。
- 4）最后，返回方法结果。

`@Cacheable` 注解的**常用属性**，如下：

- `cacheNames` 属性：缓存名。**必填**。`[]` 数组，可以填写多个缓存名。
- `values` 属性：和 `cacheNames` 属性相同，是它的别名。
- `key`属性：缓存的 key 。允许空。
  - 如果为空，则默认方法的所有参数进行组合。
  - 如果非空，则需要按照 [SpEL(Spring Expression Language)](http://itmyhome.com/spring/expressions.html) 来配置。例如说，`@Cacheable(value = "users", key = "#id")` ，使用方法参数 `id` 的值作为缓存的 key 。
- `condition`属性：基于方法入参，判断要缓存的条件。允许空。
  - 如果为空，则不进行入参的判断。
  - 如果非空，则需要按照 [SpEL(Spring Expression Language)](http://itmyhome.com/spring/expressions.html) 来配置。例如说，`@Cacheable(condition="#id > 0")` ，需要传入的 `id` 大于零。
- `unless`属性：基于方法返回，判断不缓存的条件。允许空。
  - 如果为空，则不进行入参的判断。
  - 如果非空，则需要按照 [SpEL(Spring Expression Language)](http://itmyhome.com/spring/expressions.html) 来配置。例如说，`@Cacheable(unless="#result == null")` ，如果返回结果为 `null` ，则不进行缓存。
  - 要注意，`condition` 和 `unless` 都是条件属性，**差别在于前者针对入参，后者针对结果。**

`@Cacheable` 注解的**不常用属性**，如下：

- `keyGenerator` 属性：自定义 key 生成器 [KeyGenerator](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/interceptor/KeyGenerator.java) Bean 的名字。允许空。如果设置，则 `key` 失效。
- `cacheManager` 属性：自定义缓存管理器 [CacheManager](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/CacheManager.java) Bean 的名字。允许空。一般不填写，除非有多个 CacheManager Bean 的情况下。
- `cacheResolver` 属性：自定义缓存解析器 [CacheResolver](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/interceptor/CacheResolver.java) Bean 的名字。允许空。
- `sync`属性，在获得不到缓存的情况下，是否同步执行方法。
  - 默认为 `false` ，表示无需同步。
  - 如果设置为 `true` ，则执行方法时，会进行加锁，保证同一时刻，有且仅有一个方法在执行，其它线程阻塞等待。通过这样的方式，避免重复执行方法。注意，该功能的实现，需要参考第三方缓存的具体实现。

## 2.2 @CachePut

[`@CachePut`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/annotation/CachePut.java) 注解，添加在方法上，缓存方法的执行结果。不同于 `@Cacheable` 注解，它的执行过程如下：

- 1）首先，执行方法，获得方法结果。**也就是说，无论是否有缓存，都会执行方法**。
- 2）然后，根据是否满足缓存的条件。如果满足，则缓存方法结果到缓存。
- 3）最后，返回方法结果。

一般来说，使用方式如下：

- `@Cacheable`：搭配**读**操作，实现缓存的**被动**写。
- `@CachePut`：配置**写**操作，实现缓存的**主动**写。

`@Cacheable` 注解的属性，和 `@Cacheable` 注解的属性，基本**一致**，只少一个 `sync` 属性。

## 2.3 @CacheEvict

[`@CacheEvict`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/annotation/CacheEvict.java) 注解，添加在方法上，删除缓存。

相比 `@CachePut` 注解，它额外多了两个属性：

- `allEntries` 属性，是否删除缓存名( `cacheNames` )下，所有 key 对应的缓存。默认为 `false` ，只删除指定 key 的缓存。
- `beforeInvocation` 属性，是否在方法执行**前**删除缓存。默认为 `false` ，在方法执行**后**删除缓存。

## 2.4 @Caching

[`@Caching`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/annotation/CacheConfig.java) 注解，添加在方法上，可以组合使用多个 `@Cacheable`、`@CachePut`、`@CacheEvict` 注解。不太常用，可以暂时忽略。

## 2.5 @CacheConfig

[`@CacheConfig`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/annotation/CacheConfig.java) 注解，添加在类上，共享如下四个属性的配置：

- `cacheNames`
- `keyGenerator`
- `cacheManager`
- `cacheResolver`

## 2.6 @EnableCaching

[`@EnableCaching`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/annotation/EnableCaching.java) 注解，标记开启 Spring Cache 功能，所以一定要添加。代码如下：



```java
// EnableCaching.java

boolean proxyTargetClass() default false;

AdviceMode mode() default AdviceMode.PROXY;

int order() default Ordered.LOWEST_PRECEDENCE;
```



# 3. Spring Boot 集成

在 Spring Boot 里，提供了 [`spring-boot-starter-cache`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-cache) 库，实现 Spring Cache 的自动化配置，通过 [CacheAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/CacheAutoConfiguration.java) 配置类。

在 Java 后端开发中，常见的缓存工具和框架列举如下：

- 本地缓存：Guava LocalCache、Ehcache、Caffeine 。

  > Ehcache 的功能更加丰富，Caffeine 的性能要比 Guava LocalCache 好。

- 分布式缓存：Redis、Memcached、Tair 。

  > Redis 最为主流和常用。

如果胖友想要了解本地缓存和分布式缓存的优缺点的对比，可以看看 [《进程内缓存与分布式缓存的比较》](http://gocwind.com/blog/2015/04/08/in-processes-vs-distributed-caching/) 文章。

那么，在这些缓存方案当中，`spring-boot-starter-cache` 怎么知道使用哪种呢？在默认情况下，Spring Boot 会按照如下顺序，自动判断使用哪种缓存方案，创建对应的 CacheManager 缓存管理器。



```java
// CacheConfigurations.java

private static final Map<CacheType, Class<?>> MAPPINGS;

static {
	Map<CacheType, Class<?>> mappings = new EnumMap<>(CacheType.class);
	mappings.put(CacheType.GENERIC, GenericCacheConfiguration.class);
	mappings.put(CacheType.EHCACHE, EhCacheCacheConfiguration.class);
	mappings.put(CacheType.HAZELCAST, HazelcastCacheConfiguration.class);
	mappings.put(CacheType.INFINISPAN, InfinispanCacheConfiguration.class);
	mappings.put(CacheType.JCACHE, JCacheCacheConfiguration.class);
	mappings.put(CacheType.COUCHBASE, CouchbaseCacheConfiguration.class);
	mappings.put(CacheType.REDIS, RedisCacheConfiguration.class);
	mappings.put(CacheType.CAFFEINE, CaffeineCacheConfiguration.class);
	mappings.put(CacheType.SIMPLE, SimpleCacheConfiguration.class);
	mappings.put(CacheType.NONE, NoOpCacheConfiguration.class);
	MAPPINGS = Collections.unmodifiableMap(mappings);
}
```



- 最差的情况下，会使用 [SimpleCacheConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/SimpleCacheConfiguration.java) 。

因为自动判断可能和我们希望使用的缓存方案不同，此时我们可以手动配置 `spring.cache.type` 指定类型。

考虑到目前最常使用的是 Ehcache 本地缓存，和 Redis 分布式缓存，所以我们分别在 [「4. Ehcache 示例」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 和 [「5. Redis 示例」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 小节中，一起来遨游下。

# 4. Ehcache 示例

> 示例代码对应仓库：[lab-21-cache-ehcache](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache) 。

可能有些胖友不了解 Ehcache ，所以引入用介绍如下，方便艿艿凑下篇幅：

> EhCache 是一个纯 Java 的进程内缓存框架，具有快速、精干等特点，是 Hibernate 中默认的 CacheProvider 。
>
> 下图是 Ehcache 在应用程序中的位置：
>
> ![img](E:\Development\Typora\images\0086c27bbe11c17acaaf2f5194ec1b91.jpg)
>
> 主要的特性有：
>
> 1. 快速.
> 2. 简单.
> 3. 多种缓存策略
> 4. 缓存数据有两级：内存和磁盘，因此无需担心容量问题
> 5. 缓存数据会在虚拟机重启的过程中写入磁盘
> 6. 可以通过 RMI、可插入 API 等方式进行分布式缓存
> 7. 具有缓存和缓存管理器的侦听接口
> 8. 支持多缓存管理器实例，以及一个实例的多个缓存区域
> 9. 提供 Hibernate 的缓存实现
> 10. 等等

下面，让我们使用 Ehcache 作为 Spring Cache 的缓存方案，开始遨游~

## 4.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-21/lab-21-cache-ehcache/pom.xml) 文件中，引入相关依赖。



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

    <artifactId>lab-21-cache-ehcache</artifactId>

    <dependencies>
        <!-- 实现对数据库连接池的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency> <!-- 本示例，我们使用 MySQL -->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!-- 实现对 MyBatis Plus 的自动化配置 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>

        <!-- 实现对 Caches 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <!-- Ehcache 依赖 -->
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
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

## 4.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/resources/application.yaml) 配置文件。配置如下：



```yml
spring:
  # datasource 数据源配置内容
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/lab-21-cache-demo?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password:
  # cache 缓存配置内容
  cache:
    type: ehcache

# mybatis-plus 配置内容
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true # 虽然默认为 true ，但是还是显示去指定下。
  global-config:
    db-config:
      id-type: auto # ID 主键自增
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
  mapper-locations: classpath*:mapper/*.xml
  type-aliases-package: cn.iocoder.springboot.lab21.cache.dataobject

# logging
logging:
  level:
    # dao 开启 debug 模式 mybatis 输入 sql
    cn:
      iocoder:
        springboot:
          lab21:
            cache:
              mapper: debug
```



- `spring.datasource` 配置项下，设置数据源相关的配置。
- `spring.cache`配置项下，设置 Cache 相关的配置。
  - `type` 属性，设置 Cache 使用方案为 Ehcache 。
- `mybatis-plus` 配置项下，设置 MyBatis-Plus 相关的配置。如果没有使用过 MyBatis-Plus 的胖友，不用慌，照着改就好。当然，也欢迎阅读 [《芋道 Spring Boot MyBatis 入门》](http://www.iocoder.cn/Spring-Boot/MyBatis/?self) 文章。
- `logging` 配置项，设置打印 SQL 日志，方便我们查看是否读取了 DB 。

## 4.3 Ehcache 配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/resources) 目录下，创建 [`ehcache.xml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/resources/ehcache.xml) 配置文件。配置如下：



```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd">

    <!-- users 缓存 -->
    <!-- name：缓存名 -->
    <!-- maxElementsInMemory：最大缓存 key 数量 -->
    <!-- timeToLiveSeconds：缓存过期时长，单位：秒 -->
    <!-- memoryStoreEvictionPolicy：缓存淘汰策略 -->
    <cache name="users"
           maxElementsInMemory="1000"
           timeToLiveSeconds="60"
           memoryStoreEvictionPolicy="LRU"/>  <!-- 缓存淘汰策略 -->

</ehcache>
```



- 我们配置了一个名字为 `users` 的缓存。后续，我们会使用到。

## 4.4 Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-21/lab-21-cache-ehcache/src/main/java/cn/iocoder/springboot/lab21/cache/Application.java) 类，代码如下：



```java
// Application.java

@SpringBootApplication
@EnableCaching
@MapperScan(basePackages = "cn.iocoder.springboot.lab21.cache.mapper")
public class Application {
}
```



- 配置 `@EnableCaching` 注解，开启 Spring Cache 功能。
- 配置 `@MapperScan` 注解，扫描对应 Mapper 接口所在的包路径。

## 4.5 UserDO

在 [`cn.iocoder.springboot.lab21.cache.dataobject`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/java/cn/iocoder/springboot/lab21/cache/dataobject) 包路径下，创建 [UserDO.java](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/java/cn/iocoder/springboot/lab21/cache/dataobject/UserDO.java) 类，用户 DO 。代码如下：



```java
// UserDO.java
@TableName(value = "users")
public class UserDO {

    /**
     * 用户编号
     */
    private Integer id;
    /**
     * 账号
     */
    private String username;
    /**
     * 密码（明文）
     *
     * ps：生产环境下，千万不要明文噢
     */
    private String password;
    /**
     * 创建时间
     */
    private Date createTime;
    /**
     * 是否删除
     */
    @TableLogic
    private Integer deleted;

    // ... 省略 setting/getting 方法

}
```



对应的创建表的 SQL 如下：



```sql
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户编号',
  `username` varchar(64) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '账号',
  `password` varchar(32) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '密码',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `deleted` bit(1) DEFAULT NULL COMMENT '是否删除。0-未删除；1-删除',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

INSERT INTO `users`(`id`, `username`, `password`, `create_time`, `deleted`) VALUES (1, 'yudaoyuanma', 'buzhidao', now(), 0);
```



## 4.6 UserMapper

在 [`cn.iocoder.springboot.lab21.cache.mapper`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/java/cn/iocoder/springboot/lab21/cache/mapper) 包路径下，创建 [UserMapper](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-ehcache/src/main/java/cn/iocoder/springboot/lab21/cache/mapper/UserMapper.java) 接口。代码如下：



```java
// UserMapper.java

@Repository
@CacheConfig(cacheNames = "users")
public interface UserMapper extends BaseMapper<UserDO> {

    @Cacheable(key = "#id")
    UserDO selectById(Integer id);

    @CachePut(key = "#user.id")
    default UserDO insert0(UserDO user) {
        // 插入记录
        this.insert(user);
        // 返回用户
        return user;
    }

    @CacheEvict(key = "#id")
    int deleteById(Integer id);

}
```



- 在类上，我们添加了 `@CacheConfig(cacheNames = "users")` 注解，统一配置该 UserMapper 使用的缓存名为 `users` 。

- `#selectById(Integer id)` 方法，添加了 `@Cacheable(key = "#id")` 注解，优先读缓存。

- `#insert0(UserDO user)` 方法，添加了 `@CachePut(key = "#user.id")` 注解，主动写缓存。

  > 注意，此处我们并没有使用 MyBatis-Plus 自带的插入方法，而是包装了一层，因为原插入方法返回的是 `int` 结果，无法进行缓存。

- `#deleteById(Integer id)` 方法，添加了 `@CacheEvict(key = "#id")` 注解，删除缓存。

## 4.7 UserMapperTest

创建 [UserMapperTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-21/lab-21-cache-ehcache/src/test/java/cn/iocoder/springboot/lab21/cache/UserMapperTest.java) 测试类，我们来测试一下简单的 UserMapper 的每个操作。核心代码如下：



```java
// UserMapperTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserMapperTest {

    private static final String CACHE_NAME_USER = "users";

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private CacheManager cacheManager;

    // ... 省略每个单元测试。
}
```



**① `#testCacheManager()`**



```java
// UserMapperTest.java

@Test
public void testCacheManager() {
    System.out.println(cacheManager);
}
```



- 目的，是查看 `cacheManager` 的类型。执行日志如下：

  

  ```
  org.springframework.cache.ehcache.EhCacheCacheManager@77134e08
  ```

  

  - 可以确认是 EhCache 对应的缓存管理器。

**② `#testSelectById()`**

本测试用例，是为了演示 `@Cacheable` 注解的用途。



```java
// UserMapperTest.java

@Test
public void testSelectById() {
    // 这里，胖友事先插入一条 id = 1 的记录。
    Integer id = 1;

    // <1.1> 查询 id = 1 的记录
    UserDO user = userMapper.selectById(id);
    System.out.println("user：" + user);
    // <1.2> 判断缓存中，是不是存在
    Assert.assertNotNull("缓存为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));

    // <2> 查询 id = 1 的记录
    user = userMapper.selectById(id);
    System.out.println("user：" + user);
}
```



- `<1.1>` 处，首次查询 `id = 1` 的记录。这里，我们是为了触发从 DB 中查询该记录，然后缓存起来。执行日志如下：

  ```bash
  2019-11-16 18:11:21.139 DEBUG 82760 --- [           main] c.i.s.l.c.mapper.UserMapper.selectById   : ==>  Preparing: SELECT id,password,deleted,create_time,username FROM users WHERE id=? AND deleted=0
  2019-11-16 18:11:21.162 DEBUG 82760 --- [           main] c.i.s.l.c.mapper.UserMapper.selectById   : ==> Parameters: 1(Integer)
  2019-11-16 18:11:21.188 DEBUG 82760 --- [           main] c.i.s.l.c.mapper.UserMapper.selectById   : <==      Total: 1
  user：UserDO{id=1, username='yudaoyuanma', password='buzhidao', createTime=Fri Nov 15 07:05:48 CST 2019, deleted=0}
  ```

  

  - 这里，我们执行查询了一次 DB 。

- `<1.2>` 处，我们从 CacheManager 中，查询该记录缓存，然后通过 Assert 断言该记录的缓存存在。

- `<2>` 处，再次查询 `id = 1` 的记录。这里，我们不会从 DB 查询，直接走缓存返回即可。执行日志如下：

  ```
  user：UserDO{id=1, username='yudaoyuanma', password='buzhidao', createTime=Fri Nov 15 07:05:48 CST 2019, deleted=0}
  ```

  

  - 这里，我们只查询了一次 Cache 。

**③ `#testInsert()`**

本测试用例，是为了演示 `@CachePut` 注解的用途。



```java
// UserMapperTest.java

@Test
public void testInsert() {
    // <1> 插入记录
    UserDO user = new UserDO();
    user.setUsername(UUID.randomUUID().toString()); // 随机账号，因为唯一索引
    user.setPassword("nicai");
    user.setCreateTime(new Date());
    user.setDeleted(0);
    userMapper.insert0(user);

    // <2> 判断缓存中，是不是存在
    Assert.assertNotNull("缓存为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));
}
```



- `<1>` 处，插入一条 `users` 的记录。这里，我们是为了触发主动写入该记录到缓存中。
- `<2>` 处，我们从 CacheManager 中，查询该记录缓存，然后通过 Assert 断言该记录的缓存存在。

**④ `#deleteById()`**

本测试用例，是为了演示 `@CacheEvict` 注解的用途。



```java
// UserMapperTest.java

@Test
public void testDeleteById() {
    // <1> 插入记录，为了让缓存里有记录
    UserDO user = new UserDO();
    user.setUsername(UUID.randomUUID().toString()); // 随机账号，因为唯一索引
    user.setPassword("nicai");
    user.setCreateTime(new Date());
    user.setDeleted(0);
    userMapper.insert0(user);
    // <2> 判断缓存中，是不是存在
    Assert.assertNotNull("缓存为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));

    // <3.1> 删除记录，为了让缓存被删除
    userMapper.deleteById(user.getId());
    // <3.2> 判断缓存中，是不是存在
    Assert.assertNull("缓存不为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));
}
```



- `<1>` 和 `<2>` 处，和 `#testInsert()` 方法是一致的。此时，刚插入的一条 `users` 的记录在缓存中。
- `<3.1>` 处，删除刚插入的那条 `users` 的记录。这里，我们是为了触发从 Cache 中删除该记录的。
- `<3.2>` 处，我们从 CacheManager 中，查询该记录缓存，然后通过 Assert 断言该记录的缓存不存在。

# 5. Redis 示例

> 示例代码对应仓库：[lab-21-cache-redis](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-redis) 。

本小节，我们的整体示例，和 [「4. Ehcache」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 是一致的。

## 5.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-21/lab-21-cache-redis/pom.xml) 文件中，引入相关依赖。



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

    <artifactId>lab-21-cache-redis</artifactId>

    <dependencies>
        <!-- 实现对数据库连接池的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency> <!-- 本示例，我们使用 MySQL -->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!-- 实现对 MyBatis Plus 的自动化配置 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.2.0</version>
        </dependency>

        <!-- 实现对 Caches 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>

        <!-- 实现对 Spring Data Redis 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
            <exclusions>
                <!-- 去掉对 Lettuce 的依赖，因为 Spring Boot 优先使用 Lettuce 作为 Redis 客户端 -->
                <exclusion>
                    <groupId>io.lettuce</groupId>
                    <artifactId>lettuce-core</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- 引入 Jedis 的依赖，这样 Spring Boot 实现对 Jedis 的自动化配置 -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
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

Spring Data 使用 Redis 作为缓存的方案的时候，底层使用的是 Spring Data 提供的 [RedisTemplate](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/core/RedisTemplate.java) ，所以我们引入 `spring-boot-starter-data-redis` 依赖，实现对 RedisTemplate 的自动化配置。

## 5.2 应用配置文件

在 [`resources`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-redis/src/main/resources) 目录下，创建 [`application.yaml`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-redis/src/main/resources/application.yaml) 配置文件。配置如下：



```yml
spring:
  # datasource 数据源配置内容
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/lab-21-cache-demo?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password:
  # 对应 RedisProperties 类
  redis:
    host: 127.0.0.1
    port: 6379
    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedisProperties.Jedis 内部类
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数，默认为 8 。使用负数表示没有限制。
        max-idle: 8 # 默认连接数最小空闲的连接数，默认为 8 。使用负数表示没有限制。
        min-idle: 0 # 默认连接池最小空闲的连接数，默认为 0 。允许设置 0 和 正数。
        max-wait: -1 # 连接池最大阻塞等待时间，单位：毫秒。默认为 -1 ，表示不限制。
  # cache 缓存配置内容
  cache:
    type: redis

# mybatis-plus 配置内容
mybatis-plus:
  configuration:
    map-underscore-to-camel-case: true # 虽然默认为 true ，但是还是显示去指定下。
  global-config:
    db-config:
      id-type: auto # ID 主键自增
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
  mapper-locations: classpath*:mapper/*.xml
  type-aliases-package: cn.iocoder.springboot.lab21.cache.dataobject

# logging
logging:
  level:
    # dao 开启 debug 模式 mybatis 输入 sql
    cn:
      iocoder:
        springboot:
          lab21:
            cache:
              mapper: debug
```



- `spring.datasource` 配置项下，设置数据源相关的配置。
- `spring.cache`配置项下，设置 Cache 相关的配置。
  - `type` 属性，设置 Cache 使用方案为 Redis 。
- `spring.redis` 配置项下，设置 Spring Data Redis 相关的配置。如果没有使用过 Spring Data Redis 的胖友，不用慌，照着改就好。当然，也欢迎阅读 [《芋道 Spring Boot Redis 入门》](http://www.iocoder.cn/Spring-Boot/Redis/?self) 文章。
- `mybatis-plus` 配置项下，设置 MyBatis-Plus 相关的配置。如果没有使用过 MyBatis-Plus 的胖友，不用慌，照着改就好。当然，也欢迎阅读 [《芋道 Spring Boot MyBatis 入门》](http://www.iocoder.cn/Spring-Boot/MyBatis/?self) 文章。
- `logging` 配置项，设置打印 SQL 日志，方便我们查看是否读取了 DB 。

## 5.3 Application

和 [「4.4 Application」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 一致。

## 5.4 UserDO

和 [「4.5 UserDO」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 一致。差别在于，需要让 [UserDO](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-21/lab-21-cache-redis/src/main/java/cn/iocoder/springboot/lab21/cache/dataobject/UserDO.java) 实现 Serializable 接口。因为，我们需要将 UserDO 序列化后，才能存储到 Redis 中。

## 5.5 UserMapper

和 [「4.6 UserMapper」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 一致。

## 5.6 UserMapperTest

和 [「4.7 UserMapperTest」](https://www.iocoder.cn/Spring-Boot/Cache/?github#) 基本一致。为了结合 Redis 中的数据一起说，所以这里就再“重复”一遍。

创建 [UserMapperTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-21/lab-21-cache-redis/src/test/java/cn/iocoder/springboot/lab21/cache/UserMapperTest.java) 测试类，我们来测试一下简单的 UserMapper 的每个操作。核心代码如下：



```java
// UserMapperTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserMapperTest {

    private static final String CACHE_NAME_USER = "users";

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private CacheManager cacheManager;

    // ... 省略每个单元测试。
}
```



**① `#testCacheManager()`**



```java
// UserMapperTest.java

@Test
public void testCacheManager() {
    System.out.println(cacheManager);
}
```



- 目的，是查看 `cacheManager` 的类型。执行日志如下：

  

  ```
  org.springframework.data.redis.cache.RedisCacheManager@39ad12b6
  ```

  

  - 可以确认是 Redis 对应的缓存管理器。

> 艿艿：注意，每一轮的测试，都使用 [`FLUSHDB`](https://redis.io/commands/flushdb) 指令，将 Redis 清空下。

**② `#testSelectById()`**

本测试用例，是为了演示 `@Cacheable` 注解的用途。



```java
// UserMapperTest.java

@Test
public void testSelectById() {
    // 这里，胖友事先插入一条 id = 1 的记录。
    Integer id = 1;

    // <1.1> 查询 id = 1 的记录
    UserDO user = userMapper.selectById(id);
    System.out.println("user：" + user);
    // <1.2> 判断缓存中，是不是存在
    Assert.assertNotNull("缓存为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));

    // <2> 查询 id = 1 的记录
    user = userMapper.selectById(id);
    System.out.println("user：" + user);
}
```



- `<1.1>` 处，首次查询 `id = 1` 的记录。这里，我们是为了触发从 DB 中查询该记录，然后缓存起来。执行日志如下：

  ```bash
  2019-11-16 21:26:42.206 DEBUG 84419 --- [           main] c.i.s.l.c.mapper.UserMapper.selectById   : ==>  Preparing: SELECT id,password,deleted,create_time,username FROM users WHERE id=? AND deleted=0
  2019-11-16 21:26:42.228 DEBUG 84419 --- [           main] c.i.s.l.c.mapper.UserMapper.selectById   : ==> Parameters: 1(Integer)
  2019-11-16 21:26:42.302 DEBUG 84419 --- [           main] c.i.s.l.c.mapper.UserMapper.selectById   : <==      Total: 1
  user：UserDO{id=1, username='yudaoyuanma', password='buzhidao', createTime=Fri Nov 15 07:05:48 CST 2019, deleted=0}
  ```

  

  - 这里，我们执行查询了一次 DB 。

- `<1.2>` 处，我们从 CacheManager 中，查询该记录缓存，然后通过 Assert 断言该记录的缓存存在。我们进入 Reids 命令行，查看下缓存情况。如下：

  ```
  # 使用 SCAN 指令，获得当前所有的 key 们
  127.0.0.1:6379> scan 0
  1) "0"
  2) 1) "users::1"
  
  # 使用 GET 指令，获得 `id = 1` 的缓存数据
  127.0.0.1:6379> get users::1
  "\xac\xed\x00\x05sr\x003cn.iocoder.springboot.lab21.cache.dataobject.UserDO.\xe5\xf8\xd67o\"x\x02\x00\x05L\x00\ncreateTimet\x00\x10Ljava/util/Date;L\x00\adeletedt\x00\x13Ljava/lang/Integer;L\x00\x02idq\x00~\x00\x02L\x00\bpasswordt\x00\x12Ljava/lang/String;L\x00\busernameq\x00~\x00\x03xpsr\x00\x0ejava.util.Datehj\x81\x01KYt\x19\x03\x00\x00xpw\b\x00\x00\x01nl*d\xe0xsr\x00\x11java.lang.Integer\x12\xe2\xa0\xa4\xf7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x00\x00sq\x00~\x00\a\x00\x00\x00\x01t\x00\bbuzhidaot\x00\x0byudaoyuanma"
  ```

  

  - 可以看到 `users::1` 这个 Redis key 。说明，RedisCacheManager 使用 `::` 将 `cacheNames` 和 `key` 作为分隔符后，拼接在一起。
  - `users::1` 对应的值，是一串“乱七八糟”的字符，是因为 RedisTemplate 默认使用 [Java 序列化](https://juejin.im/post/5ce3cdc8e51d45777b1a3cdf)方式，所以看到的值才是这样的。实际在使用时，我们可以修改 RedisTemplate 的序列化方式为 JSON 的序列化方式。

- `<2>` 处，再次查询 `id = 1` 的记录。这里，我们不会从 DB 查询，直接走缓存返回即可。执行日志如下：

  ```
  user：UserDO{id=1, username='yudaoyuanma', password='buzhidao', createTime=Fri Nov 15 07:05:48 CST 2019, deleted=0}
  ```

  

  - 这里，我们只查询了一次 Cache 。

**③ `#testInsert()`**

本测试用例，是为了演示 `@CachePut` 注解的用途。



```
// UserMapperTest.java

@Test
public void testInsert() {
    // <1> 插入记录
    UserDO user = new UserDO();
    user.setUsername(UUID.randomUUID().toString()); // 随机账号，因为唯一索引
    user.setPassword("nicai");
    user.setCreateTime(new Date());
    user.setDeleted(0);
    userMapper.insert0(user);

    // <2> 判断缓存中，是不是存在
    Assert.assertNotNull("缓存为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));
}
```



- `<1>` 处，插入一条 `users` 的记录。这里，我们是为了触发主动写入该记录到缓存中。

- `<2>` 处，我们从 CacheManager 中，查询该记录缓存，然后通过 Assert 断言该记录的缓存存在。我们进入 Reids 命令行，查看下缓存情况。如下：

  

  ```
  # 使用 SCAN 指令，获得当前所有的 key 们
  127.0.0.1:6379> scan 0
  1) "0"
  2) 1) "users::14"
  
  # 使用 GET 指令，获得 `id = 14` 的缓存数据
  127.0.0.1:6379> get users::14
  "\xac\xed\x00\x05sr\x003cn.iocoder.springboot.lab21.cache.dataobject.UserDO.\xe5\xf8\xd67o\"x\x02\x00\x05L\x00\ncreateTimet\x00\x10Ljava/util/Date;L\x00\adeletedt\x00\x13Ljava/lang/Integer;L\x00\x02idq\x00~\x00\x02L\x00\bpasswordt\x00\x12Ljava/lang/String;L\x00\busernameq\x00~\x00\x03xpsr\x00\x0ejava.util.Datehj\x81\x01KYt\x19\x03\x00\x00xpw\b\x00\x00\x01nj\x1e\xafLxsr\x00\x11java.lang.Integer\x12\xe2\xa0\xa4\xf7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x00\x00sq\x00~\x00\a\x00\x00\x00\x0et\x00\x05nicait\x00$acbed95f-668d-4d3e-a3aa-37dd05094db3"
  ```

  

**④ `#deleteById()`**

本测试用例，是为了演示 `@CacheEvict` 注解的用途。



```
// UserMapperTest.java

@Test
public void testDeleteById() {
    // <1> 插入记录，为了让缓存里有记录
    UserDO user = new UserDO();
    user.setUsername(UUID.randomUUID().toString()); // 随机账号，因为唯一索引
    user.setPassword("nicai");
    user.setCreateTime(new Date());
    user.setDeleted(0);
    userMapper.insert0(user);
    // <2> 判断缓存中，是不是存在
    Assert.assertNotNull("缓存为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));

    // <3.1> 删除记录，为了让缓存被删除
    userMapper.deleteById(user.getId());
    // <3.2> 判断缓存中，是不是存在
    Assert.assertNull("缓存不为空", cacheManager.getCache(CACHE_NAME_USER).get(user.getId(), UserDO.class));
}
```



- `<1>` 和 `<2>` 处，和 `#testInsert()` 方法是一致的。此时，刚插入的一条 `users` 的记录在缓存中。

- `<3.1>` 处，删除刚插入的那条 `users` 的记录。这里，我们是为了触发从 Cache 中删除该记录的。

- `<3.2>` 处，我们从 CacheManager 中，查询该记录缓存，然后通过 Assert 断言该记录的缓存不存在。我们进入 Reids 命令行，查看下缓存情况。如下：

  

  ```
  # 使用 SCAN 指令，获得当前所有的 key 们。发现不存在任何 key ，说明该记录的缓存，已经被删除
  127.0.0.1:6379> scan 0
  1) "0"
  2) (empty list or set)
  ```

  

## 5.7 过期时间

在 Spring Data 使用 Redis 作为缓存方案时，默认情况下是**永不过期**的。



```
127.0.0.1:6379> ttl users::1
(integer) -1
```



- 在 Redis 命令行中，我们可以看到 `users::1` 的过期时间为 `-1` 永不过期。

虽然说，我们可以通 `spring.cache.redis.time-to-live` 配置项，设置过期时间。但是，它是**全局的统一**的。这样在实际使用时，是无法满足我们希望不同的缓存，使用不同的过期时间。

不过我们如果翻看 [RedisCacheManager](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/cache/RedisCacheManager.java) 的源码，我们又会发现有个 `initialCacheConfiguration` 属性，又是支持每个缓存允许自定义配置。代码如下：



```java
// RedisCacheManager.java

// 默认的全局配置
private final RedisCacheConfiguration defaultCacheConfig;

// 每个缓存的自定义配置
private final Map<String, RedisCacheConfiguration> initialCacheConfiguration;
```



所以，我们可以参考 [《SpringBoot 2.0 的 @Cacheable(Redis) 缓存失效时间解决方案》](https://blog.csdn.net/zyt807/article/details/82428615) 文章，自定义一个 CacheManager Bean 对象。当然，更加系统的解决方式，是按照这个文章的思路，实现一个新的支持使用配置文件自定义每个缓存配置的 [RedisCacheConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/cache/RedisCacheConfiguration.java) 自动化配置类。

当然，还有一个解决方案，就是使用 Redisson 提供的缓存管理器。具体可以看看 [《Redisson 文档 —— Spring Cache整合》](https://github.com/redisson/redisson/wiki/14.-第三方框架整合#142-spring-cache整合) 。

## 5.8 CacheManager

```java
@Bean
public CacheManager cacheManager(RedisConnectionFactory factory) {
    RedisSerializer<String> redisSerializer = new StringRedisSerializer();
    Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

    //解决查询缓存转换异常的问题
    ObjectMapper om = new ObjectMapper();
    om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
    om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
    jackson2JsonRedisSerializer.setObjectMapper(om);

    // 配置序列化（解决乱码的问题）,过期时间600秒
    RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofSeconds(600)) // 设置缓存过期时间
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer)) // 设置key序列化
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer)) // 设置value序列化
            .disableCachingNullValues(); // 不缓存空值

    RedisCacheManager cacheManager = RedisCacheManager.builder(factory) // 初始化RedisCacheManager
            .cacheDefaults(config) // 设置默认缓存配置
            .build(); // 初始化RedisCacheManager
    return cacheManager;
}
```

# 666. 彩蛋

快乐的时光，总是这么短暂。😈 我们已经成功完成了对 Spring Boot 如何集成 Spring Cache 的入门。下面还是进入我们的日常彩蛋环节。

Guava 也提供了本地缓存的功能，但是 `spring-boot-starter-cache` 2.X 的版本，并未提供对它的内置支持。原因我们可以在 [Why did Spring framework deprecate the use of Guava cache?](https://stackoverflow.com/questions/44175085/why-did-spring-framework-deprecate-the-use-of-guava-cache) 看到，Spring 5.X 版本，从 Guava 替换成了 [Caffeine](https://github.com/ben-manes/caffeine) 。如果胖友使用的是 Spring Boot 1.X 的版本，倒是可以看看 [《Spring Boot + Guava Cache + @EnableCaching》](https://blog.csdn.net/u011726984/article/details/79013055) 文章。

在推荐两篇大厂在缓存方面的实践：

- 美团技术团队 [《缓存那些事》](https://tech.meituan.com/2017/03/17/cache-about.html)
- 有赞技术团队 [《有赞透明多级缓存解决方案（TMC）》](http://www.iocoder.cn/Fight/A-nifty-multilevel-cache-implementation-scheme-how-nifty-is-it/self)

除了 Spring Cache 缓存框架之外，我们也可以考虑如下的解决方案：

- [JetCache](https://github.com/alibaba/jetcache)
- [J2Cache](https://gitee.com/ld/J2Cache)

推荐阅读：

- [《性能测试 —— Redis 基准测试》](http://www.iocoder.cn/Performance-Testing/Redis-benchmark/?self)
- [《芋道 Spring Boot Redis 入门》](http://www.iocoder.cn/Spring-Boot/Redis/?self)
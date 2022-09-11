## 初识ShardingSphere

### 了解

 我们先来学两个单词，虽然我知道这两个单词谁都认识。😱

 sharding：  分片；分区；分库分表

 sphere：   范围；球体； 包围；    /sfɪr/

  👉 [ShardingSphere官网](https://shardingsphere.apache.org/index_zh.html)



### 介绍

 `Apache ShardingSphere`是一套开源的分布式数据库中间件解决方案组成的生态圈，它由 `JDBC`、`Proxy` 和 `Sidecar`（规划中）这 3 款相互独立，却又能够混合部署配合使用的产品组成。 它们均提供`标准化的数据分片`、`分布式事务和数据库治理功能`，可适用于如 `Java 同构`、`异构语言`、`云原生`等各种多样化的应用场景。

 `Apache ShardingSphere`定位为关系型数据库中间件，旨在`充分合理地在分布式的场景下利用关系型数据库的计算和存储能力`，而并非实现一个全新的关系型数据库。 它通过关注不变，进而抓住事物本质。关系型数据库当今依然占有巨大市场，是各个公司核心业务的基石，未来也难于撼动，我们目前阶段更加关注在原有基础上的增量，而非颠覆。

 `Apache ShardingSphere 5.x` 版本开始致力于`可插拔架构`，项目的功能组件能够灵活的以可插拔的方式进行扩展。 目前，`数据分片`、`读写分离`、`多数据副本`、`数据加密`、`影子库压测`等功能，以及 `MySQL`、`PostgreSQL`、`SQLServer`、`Oracle` 等 SQL 与协议的支持，均通过插件的方式织入项目。 开发者能够像使用积木一样定制属于自己的独特系统。Apache ShardingSphere 目前已提供数十个 SPI 作为系统的扩展点，仍在不断增加中。

 ShardingSphere 已于2020年4月16日成为 Apache 软件基金会的顶级项目。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70.png)](https://img-blog.csdnimg.cn/20200712000332348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### ShardingSphere-JDBC

 定位为`轻量级 Java 框架`，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为`增强版的 JDBC 驱动`，`完全兼容 JDBC 和各种 ORM 框架`。

 Sharding-JDBC的核心功能为数据分片和读写分离，通过Sharding-JDBC，应用可以透明的使用jdbc访问已经分库分表、读写分离的多个数据源，而不用关心数据源的数量以及数据如何分布。

 Sharding-JDBC的主要目的是`简化对分库分表之后数据相关操作`

- 适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC。
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP 等。
- 支持任意实现 JDBC 规范的数据库，目前支持 MySQL，Oracle，SQLServer，PostgreSQL 以及任何遵循 SQL92 标准的数据库。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16625698615501.png)](https://img-blog.csdnimg.cn/20200705004934174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### ShardingSphere-Proxy

 定位为`透明化的数据库代理端`，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。 目前提供 MySQL 和 PostgreSQL 版本，它可以使用任何兼容 MySQL/PostgreSQL 协议的访问客户端(如：MySQL Command Client, MySQL Workbench, Navicat 等)操作数据，对 DBA 更加友好。

- 向应用程序完全透明，可直接当做 MySQL/PostgreSQL 使用。
- 适用于任何兼容 MySQL/PostgreSQL 协议的的客户端。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16625698615502.png)](https://img-blog.csdnimg.cn/20200705005131804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

## 分库分表介绍

### 什么是分库分表

 数据库中的`数据量不一定是可控的`，在未进行分库分表的情况下，随着时间和业务的发展，库中的表会越来越多，表中的数据量也会越来越大，相应地，数据操作，增删改查的开销也会越来越大；另外，由于无法进行分布式式部署，而一台服务器的资源（CPU、磁盘、内存、IO 等）是有限的，最终`数据库所能承载的数据量、数据处理能力都将遭遇瓶颈`。

 分库分表就是`为了解决由于数据量过大而导致数据库性能降低的问题`，将原来独立的数据库拆分成若干数据库组成，将数据大表拆分成若干数据表组成，使得单一数据库、单一数据表的数据量变小，从而达到提升数据库性能的目的。

 **例子**

   初始的数据库设计

[![img](E:\Development\Typora\images\20200705010755856.png)](https://img-blog.csdnimg.cn/20200705010755856.png)


   要想提高性能，有两种方案

     1. 硬件上的提高(提高cpu，提高内存，提高硬盘…但是指标不治本)

     2. 分库分表(看下图)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16625698615513.png)](https://img-blog.csdnimg.cn/20200705011219985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### **分库分表的方式**

 数据库的切分指的是通过**某种特定的条件**，将我们存放在同一个数据库中的数据分散存放到多个数据库（主机）中，以达到分散单台设备负载的效果，即分库分表。

 数据的切分根据其切分规则的类型，可以分为 垂直切分 和水平切分。

> 1. 垂直切分： 把单一的表拆分成多个表，并分散到不同的数据库（主机）上。
> 2. 水平切分：根据表中数据的逻辑关系，将表中的数据按照某种条件拆分到多台数据库上



### 垂直拆分

#### 简单介绍

 一个数据库有多个表构成，每个表对应不同的业务，垂直切分是只按照业务将表进行分类，将其分布到不同的数据库上，这样就将数据分担到了不同的库上（**专库专用**）



#### 数据库垂直拆分例子

 有如下几张表：

   • 用户信息表（User）

   • 订单信息表（Orders）

 针对以上案例，垂直切分就是根据每个表的不同业务进行切分。

 比如 User 表， Orders 表，将每个表切分到不同的数据库上。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70.png)](https://img-blog.csdnimg.cn/20200316144530900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70)



#### 数据库表垂直拆分例子

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70-16625698615514.png)](https://img-blog.csdnimg.cn/20200316144621894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70)



#### 优点

1. 拆分后业务清晰。
2. 系统之间进行整合或扩展很容易。
3. 按照成本、应用的等级、应用的类型等奖表放到不同的机器上，便于管理，数据维护简单



#### 缺点

1. 部分业务表无法关联(Join), 只能通过接口方式解决，提高了系统的复杂度。
2. 受每种业务的不同限制，存在单库性能瓶颈，不易进行数据扩展和提升性能。
3. 事务处理变得复杂。



### 水平拆分

#### 简单介绍

 与垂直切分对比，水平切分不是将表进行分类，而是将其按照某个字段的某种规则分散到多个库中，在每个表中包含一部分数据，所有表加起来就是全量的数据。

 简单来说，我们可以将对数据的水平切分理解为按照数据行进行切分，就是将表中的某些行切分到一个数据库表中，而将其他行切分到其他数据库表中。

 这种切分方式根据单表的数据量的规模来切分，保证单表的容量不会太大，从而保证了单表的查询等处理能力

 例如将用户的信息表拆分成 User1、User2 等，表结构是完全一样的。我们通常根据某些特定的规则来划分表，比如根据用户的 ID 来取模划分。



#### 数据库水平拆分例子

 在博客类系统中，读取量一般都会很大。当同时有 100 万个用户在浏览时，如果是单表，则单表会进行 100 万次请求，如果是单库，数据库就会承受 100 万次的请求压力。如果采取水平切分来减少每个单表的压力，将其分为 100 个表，并且分布在 10 个数据库中，每个表进行 1 万次请求，则每个数据库会承受 10 万次的请求压力，虽然这不可能绝对平均，但是这样，压力就减少了很多，并且是成倍减少的。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70-16625698615515.png)](https://img-blog.csdnimg.cn/20200316144357868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70)



#### 数据库表水平拆分例子

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70-16625698615526.png)](https://img-blog.csdnimg.cn/20200316144652488.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMzcxMDA0,size_16,color_FFFFFF,t_70)



### 优点

1. 单库单表的数据保持在一定的量级，有助于性能的提高。
2. 切分的表的结构相同，应用层改造较少，只需要增加路由规则即可。
3. 提高了系统的稳定性和负载能力



### 缺点

1. 切分后，数据是分散的，很难利用数据库的 Join 操作，跨库 Join 性能较差。
2. 分片事务的一致性难以解决，数据扩容的难度和维护量极大。



### 拆分前的思考

 数据拆分前其实是要首先做准备工作的，然后才是开始数据拆分

1. 第一步：采用分布式缓存redis、memcached等降低对数据库的读操作。
2. 第二步：如果缓存使用过后，数据库访问量还是非常大，可以考虑数据库读、写分离原则。
3. 第三步：当我们使用读写分离、缓存后，数据库的压力还是很大的时候，这就需要使用到数据库拆分了。

> 数据库拆分原则：就是指通过某种特定的条件，按照某个维度，将我们存放在同一个数据库中的数据分散存放到多个数据库（主机）上面以达到分散单库（主机）负载的效果。



### 思考步骤

1. 优先考虑缓存降低对数据库的读操作。
2. 再考虑读写分离，降低数据库写操作。
3. 最后开始数据拆分,切分模式： **首先垂直（纵向）拆分、再次水平拆分。**
4. 首先考虑按照业务垂直拆分。
5. 再考虑水平拆分：先分库(设置数据路由规则，把数据分配到不同的库中)
6. 最后再考虑分表，单表拆分到数据1000万以内。



### **分库分表带来的问题**

 综上所述，垂直切分和水平切分的共同点如下：

  • 存在跨节点 Join 的问题。

  • 存在跨节点合并排序、分页的问题。

  • 存在多数据源管理的问题



## ShardingSphere-JDBC

### 简单介绍

 定位为`轻量级 Java 框架`，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为`增强版的 JDBC 驱动`，`完全兼容 JDBC 和各种 ORM 框架`。

 Sharding-JDBC的核心功能为`数据分片`和`读写分离`，通过Sharding-JDBC，应用可以透明的使用jdbc访问已经分库分表、读写分离的多个数据源，而不用关心数据源的数量以及数据如何分布。

  Sharding-JDBC的主要目的是`简化对分库分表之后数据相关操作`

- 适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC。
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP 等。
- 支持任意实现 JDBC 规范的数据库，目前支持 MySQL，Oracle，SQLServer，PostgreSQL 以及任何遵循 SQL92 标准的数据库。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16625698615501.png)](https://img-blog.csdnimg.cn/20200705004934174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 环境搭建

#### maven依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.1</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.20</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
        <version>4.0.0-RC1</version>
    </dependency>
</dependencies>
```

#### yml基本配置

```yml
server:
  port: 8081

spring:
  application:
    name: user-service
  main:
    allow-bean-definition-overriding: true
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/manager?characterEncoding=UTF-8&serverTimezone=UTC
    type: com.alibaba.druid.pool.DruidDataSource
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 30000
    validationQuery: select 'x';
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    filters: stat,wall,slf4j
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    useGlobalDataSourceStat: true



mybatis-plus:
  global-config:
    db-config:
      table-prefix: tb_
      id-type: assign_id
      logic-delete-field: deleted 
      logic-delete-value: 1  
      logic-not-delete-value: 0  

logging:
  level:
    test.codekiller.top.testshardingjdbc: debug
```



### 水平分表

#### 建表

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-16625698615527.png)](https://img-blog.csdnimg.cn/20200720170340435.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```mysql
CREATE TABLE `tb_course_1` (
	`id` bigint(64) NOT NULL COMMENT '课程id',
	`name` varchar(50) NOT NULL COMMENT '课程名称',
	`user_id` bigint(20) NOT NULL COMMENT '课程的用户id',
	`status` varchar(10)  NOT NULL COMMENT '状态',
	`version` bigint(20) DEFAULT '0' COMMENT '版本，乐观锁',
    `deleted` tinyint(1) DEFAULT '0' COMMENT '逻辑删除，1删除，0没删除',
	PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

CREATE TABLE `tb_course_2` (
	`id` bigint(64) NOT NULL COMMENT '课程id',
	`name` varchar(50) NOT NULL COMMENT '课程名称',
	`user_id` bigint(20) NOT NULL COMMENT '课程的用户id',
	`status` varchar(10)  NOT NULL COMMENT '状态',
	`version` bigint(20) DEFAULT '0' COMMENT '版本，乐观锁',
    `deleted` tinyint(1) DEFAULT '0' COMMENT '逻辑删除，1删除，0没删除',
	PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```



#### 实体类

```java
/**
 * @author codekiller
 * @date 2020/7/12 0:44
 * @Description 课程实体类
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Course {

    @TableId
    private Long cid;

    private String name;

    private Long userId;

    private String status;

    @Version
    private Long version;

    private Boolean deleted;

}
```



#### mapper

```java
public interface CourseMapper extends BaseMapper<Course1> {
}
```



#### 分片配置

 👉[打开官网](https://shardingsphere.apache.org/index_zh.html)

[![img](E:\Development\Typora\images\20200720155347660.png)](https://img-blog.csdnimg.cn/20200720155347660.png)

```properties
#shardingjdbc分片策略
#配置数据源,给数据源配置名字，可配置多个,eg:ds0,ds1
spring.shardingsphere.datasource.names=ds0

#这里因为只有一个数据源，配置一个数据源的信息就行了
spring.shardingsphere.datasource.ds0.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds0.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds0.url=jdbc:mysql://localhost:3306/sharding_sphere?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds0.username=jiang
spring.shardingsphere.datasource.ds0.password=jiang

#指定tb_course表的分布情况，使用行内表达式配置表在哪个数据库里面，表名称是什么。
# 如果有12个，就是${1..12}。如果有两个数据原ds$->{0..1}.t_course_$->{1..2}
spring.shardingsphere.sharding.tables.tb_course.actual-data-nodes=ds0.tb_course_$->{1..2}

#指定course表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_course.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_course.key-generator.type=SNOWFLAKE

# 指定分片策略，约定id值的偶数加到tb_course_1，奇数加到tb_course_2表
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.sharding-column= id
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.algorithm-expression=tb_course_$->{id%2+1}

#代开sql输出日志
spring.shardingsphere.props.sql.show=true
```

- 我这里只配置了一个数据源，多个数据源配置请看👉[分库](https://blog.codekiller.top/2021/02/03/ShardingSphere/#split_datasource)
- 配置中的ds0是数据源名称，tb_course是表的名称[![img](E:\Development\Typora\images\20200720163842965.png)](https://img-blog.csdnimg.cn/20200720163842965.png)

#### 测试(报错)

##### 插入

```java
@Autowired
private CourseMapper courseMapper;

@Test
void contextLoads() {
    Course course = new Course();
    course.setId(null);
    course.setName("数学");
    course.setStatus("3");
    course.setUserId(123L);
    this.courseMapper.insert(course);

}
```

 点击运行，你会发现报了一个错误

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624732.png)](https://img-blog.csdnimg.cn/20200720164251923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



>  其实报这个错的原因也很好分析，因为我数据库里有两张表，而我们只有一个实体类，造成同一个实体类无法对应两张表。因此需要添加配置`spring.main.allow-bean-definition-overriding=true`。
>
> 注：表示后发现的bean会覆盖之前相同名称的bean

 



 修改好配置后，继续运行。查看结果

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624733.png)](https://img-blog.csdnimg.cn/20200720164845219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

- id为偶数的数据在表1中
- id为奇数的数据在表2中



##### 查询

```java
List<Course> courses  = this.courseMapper.selectList(new QueryWrapper<Course>().lambda().eq(Course::getName, "数学"));
courses.forEach(course -> System.out.println(course));
```

 结果

[![img](E:\Development\Typora\images\2020072016553043.png)](https://img-blog.csdnimg.cn/2020072016553043.png)



##### 删除

```
int result = this.courseMapper.deleteById(1285130867717615617L);
```

 结果

[![img](E:\Development\Typora\images\20200720165808268.png)](https://img-blog.csdnimg.cn/20200720165808268.png)





### 水平分库

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624734.png)](https://img-blog.csdnimg.cn/20200720170418677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```mysql
create DATABASE sharding_sphere_2;
use sharding_sphere_2;
CREATE TABLE `tb_course_1` (
	`id` bigint(64) NOT NULL COMMENT '课程id',
	`name` varchar(50) NOT NULL COMMENT '课程名称',
	`user_id` bigint(20) NOT NULL COMMENT '课程的用户id',
	`status` varchar(10)  NOT NULL COMMENT '状态',
	`version` bigint(20) DEFAULT '0' COMMENT '版本，乐观锁',
    `deleted` tinyint(1) DEFAULT '0' COMMENT '逻辑删除，1删除，0没删除',
	PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

CREATE TABLE `tb_course_2` (
	`id` bigint(64) NOT NULL COMMENT '课程id',
	`name` varchar(50) NOT NULL COMMENT '课程名称',
	`user_id` bigint(20) NOT NULL COMMENT '课程的用户id',
	`status` varchar(10)  NOT NULL COMMENT '状态',
	`version` bigint(20) DEFAULT '0' COMMENT '版本，乐观锁',
    `deleted` tinyint(1) DEFAULT '0' COMMENT '逻辑删除，1删除，0没删除',
	PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

[![img](E:\Development\Typora\images\20200720171219896.png)](https://img-blog.csdnimg.cn/20200720171219896.png)



#### 分片配置

```properties
#shardingjdbc分片策略
#配置数据源,给数据源配置名字，可配置多个,eg:ds0,ds1
spring.shardingsphere.datasource.names=ds1,ds2

#这里因为有两个数据源，需要配置两个数据源的信息
spring.shardingsphere.datasource.ds1.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds1.url=jdbc:mysql://localhost:3306/sharding_sphere_1?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds1.username=jiang
spring.shardingsphere.datasource.ds1.password=jiang

spring.shardingsphere.datasource.ds2.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds2.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds2.url=jdbc:mysql://localhost:3306/sharding_sphere_2?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds2.username=jiang
spring.shardingsphere.datasource.ds2.password=jiang

#指定tb_course表的分布情况，配置表在哪个数据库里面，表名称是什么。
spring.shardingsphere.sharding.tables.tb_course.actual-data-nodes=ds$->{1..2}.tb_course_$->{1..2}

#指定course表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_course.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_course.key-generator.type=SNOWFLAKE

# 指定数据库的分片策略,分库。约定id是偶数添加到ds1，奇数添加到ds2
spring.shardingsphere.sharding.tables.tb_course.database-strategy.inline.sharding-column=id
spring.shardingsphere.sharding.tables.tb_course.database-strategy.inline.algorithm-expression=ds$->{id%2+1}

# 指定表分片策略，分表，约定id值的偶数加到tb_course_1，奇数加到tb_course_2表
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.sharding-column= id
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.algorithm-expression=tb_course_$->{id%2+1}


#代开sql输出日志
spring.shardingsphere.props.sql.show=true
```



#### 测试

##### 插入

```java
Course course = new Course();
course.setId(null);
course.setName("数学");
course.setStatus("3");
course.setUserId(123L);
this.courseMapper.insert(course);
```

 结果

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624735.png)](https://img-blog.csdnimg.cn/20200720172905440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



 查看数据库

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624836.png)](https://img-blog.csdnimg.cn/20200720173406907.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 因为分片策略的原因，只有两个表会有数据

##### 查询

```java
List<Course> courses  = this.courseMapper.selectList(new QueryWrapper<Course>().lambda().eq(Course::getName, "数学"));
```

 结果

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624837.png)](https://img-blog.csdnimg.cn/20200720173506951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

<br?

##### 删除

```
int result = this.courseMapper.deleteById(1285143985013338114L);
```

 结果

[![img](E:\Development\Typora\images\20200720173619928.png)](https://img-blog.csdnimg.cn/20200720173619928.png)



### 垂直分库(专库专表)

#### 建表

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624838.png)](https://img-blog.csdnimg.cn/20200720174717512.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```mysql
CREATE TABLE `tb_user` (
    `id` bigint(64) NOT NULL COMMENT '用户id',
    `username` varchar(100) NOT NULL COMMENT '用户名',
    `status` varchar(50) NOT NULL COMMENT '状态',
    `version` bigint(20) DEFAULT '0' COMMENT '版本，乐观锁',
    `deleted` tinyint(1) DEFAULT '0' COMMENT '逻辑删除，1删除，0没删除',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```



#### 实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Long id;

    private String username;

    private String status;

    @Version
    private Long version;

    private Boolean deleted;
}
```



#### mapper

```
public interface UserMapper extends BaseMapper<User> {
}
```



#### 配置

```properties
#shardingjdbc分片策略
#配置数据源,给数据源配置名字，可配置多个
spring.shardingsphere.datasource.names=ds1,ds2

#这里因为有两个数据源，需要配置两个数据源的信息
spring.shardingsphere.datasource.ds1.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds1.url=jdbc:mysql://localhost:3306/sharding_sphere_1?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds1.username=jiang
spring.shardingsphere.datasource.ds1.password=jiang

spring.shardingsphere.datasource.ds2.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds2.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds2.url=jdbc:mysql://localhost:3306/sharding_sphere_2?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds2.username=jiang
spring.shardingsphere.datasource.ds2.password=jiang

#指定tb_course表的分布情况，配置表在哪个数据库里面，表名称是什么。
spring.shardingsphere.sharding.tables.tb_course.actual-data-nodes=ds$->{1..2}.tb_course_$->{1..2}
#垂直分库，单库单表
spring.shardingsphere.sharding.tables.tb_user.actual-data-nodes=ds1.tb_user

#指定course表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_course.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_course.key-generator.type=SNOWFLAKE
#指定user表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_user.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_user.key-generator.type=SNOWFLAKE

# 指定数据库的分片策略,分库。约定id是偶数添加到ds1，奇数添加到ds2
spring.shardingsphere.sharding.tables.tb_course.database-strategy.inline.sharding-column=id
spring.shardingsphere.sharding.tables.tb_course.database-strategy.inline.algorithm-expression=ds$->{id%2+1}

# 指定表分片策略，分表，约定id值的偶数加到tb_course_1，奇数加到tb_course_2表
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.sharding-column= id
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.algorithm-expression=tb_course_$->{id%2+1}

#代开sql输出日志
spring.shardingsphere.props.sql.show=true
```

> 这里和水平分库进行了一个整合



#### 测试

##### 插入

```java
User user=new User();
user.setId(null);
user.setStatus("管理员");
user.setUsername("李四");
this.userMapper.insert(user);
```



 结果

[![img](E:\Development\Typora\images\20200720180319110.png)](https://img-blog.csdnimg.cn/20200720180319110.png)



##### 查询

[![img](E:\Development\Typora\images\20200720180413385.png)](https://img-blog.csdnimg.cn/20200720180413385.png)



##### 删除

```
int result = this.userMapper.deleteById(1285152631994642434L);
```

 结果

[![img](E:\Development\Typora\images\2020072018052740.png)](https://img-blog.csdnimg.cn/2020072018052740.png)



### 公共表

 (1) 存储固定数据的表，表数据很少发生变化，查询时候经常进行关联

 (2) `在每个数据库中创建出相同结构公共表`

#### 建表

```java
CREATE TABLE `tb_udict` (
    `id` bigint(64) NOT NULL COMMENT '用户id',
    `status` varchar(50) NOT NULL COMMENT '状态',
    `value` varchar(100) NOT NULL COMMENT '值',
    `version` bigint(20) DEFAULT '0' COMMENT '版本，乐观锁',
    `deleted` tinyint(1) DEFAULT '0' COMMENT '逻辑删除，1删除，0没删除',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

[![img](E:\Development\Typora\images\20200720200335478.png)](https://img-blog.csdnimg.cn/20200720200335478.png)



#### 配置文件

```properties
#shardingjdbc分片策略
#配置数据源,给数据源配置名字，可配置多个
spring.shardingsphere.datasource.names=ds1,ds2

#这里因为有两个数据源，需要配置两个数据源的信息
spring.shardingsphere.datasource.ds1.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds1.url=jdbc:mysql://localhost:3306/sharding_sphere_1?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds1.username=jiang
spring.shardingsphere.datasource.ds1.password=jiang

spring.shardingsphere.datasource.ds2.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds2.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds2.url=jdbc:mysql://localhost:3306/sharding_sphere_2?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds2.username=jiang
spring.shardingsphere.datasource.ds2.password=jiang

#指定tb_course表的分布情况，配置表在哪个数据库里面，表名称是什么。
spring.shardingsphere.sharding.tables.tb_course.actual-data-nodes=ds$->{1..2}.tb_course_$->{1..2}
#垂直分库，单库单表
spring.shardingsphere.sharding.tables.tb_user.actual-data-nodes=ds1.tb_user

#指定course表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_course.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_course.key-generator.type=SNOWFLAKE
#指定user表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_user.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_user.key-generator.type=SNOWFLAKE
#配置公共表
spring.shardingsphere.sharding.binding-tables=tb_udict
spring.shardingsphere.sharding.tables.tb_udict.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_udict.key-generator.type=SNOWFLAKE

# 指定数据库的分片策略,分库。约定id是偶数添加到ds1，奇数添加到ds2
spring.shardingsphere.sharding.tables.tb_course.database-strategy.inline.sharding-column=id
spring.shardingsphere.sharding.tables.tb_course.database-strategy.inline.algorithm-expression=ds$->{id%2+1}

# 指定表分片策略，分表，约定id值的偶数加到tb_course_1，奇数加到tb_course_2表
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.sharding-column= id
spring.shardingsphere.sharding.tables.tb_course.table-strategy.inline.algorithm-expression=tb_course_$->{id%2+1}


#代开sql输出日志
spring.shardingsphere.props.sql.show=true
```



#### 测试

##### 插入

```java
Udict udict=new Udict();
udict.setId(null);
udict.setStatus("1");
udict.setValue("普通用户");
this.udictMapper.insert(udict);
```

 结果

[![img](E:\Development\Typora\images\20200720200558156.png)](https://img-blog.csdnimg.cn/20200720200558156.png)

[![img](E:\Development\Typora\images\20200720200626912.png)](https://img-blog.csdnimg.cn/20200720200626912.png)



##### 查询

```
List<Udict> udicts = this.udictMapper.selectList(new QueryWrapper<Udict>().lambda().eq(Udict::getStatus, "1"));
udicts.forEach(udict -> System.out.println(udict));
```

 结果

[![img](E:\Development\Typora\images\20200720200930520.png)](https://img-blog.csdnimg.cn/20200720200930520.png)

> 查询到了两条数据



##### 删除

```
int result = this.udictMapper.deleteById(1285184037734981633L);
```

 结果

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624839.png)](https://img-blog.csdnimg.cn/20200720201634650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 两个表的数据都进行了删除



### 读写分离

#### 读写分离基本概念

 为了确保数据库产品的稳定性,很多数据库拥有双机热备功能。也就是,第一台数据库服务器,是对外提供`增删改业务的生产服务器`;第二台数据库服务器,主要进行`读的操作`。原理:让主数据库( master )处理事务性增、改、删操作,而从数据库( slave )处理SELECT查询操作。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624840.png)](https://img-blog.csdnimg.cn/20200720202719361.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### 原理

 `主从复制`：当主服务器有写入(增删改)语句的时候，从服务器自动获取。

 `读写分离`：增删改语句操作一台服务器，查询语句操作另一台服务器

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624841.png)](https://img-blog.csdnimg.cn/202007202033427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



> Sharding-JDBC 通过 sql 语句语义分析，实现读写分离过程，不会做数据同步。它提供透明化读写分离，让使用方尽量像使用一个数据库一样使用主从数据库集群。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624842.png)](https://img-blog.csdnimg.cn/202007202043572.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



#### 操作

##### 启动两台服务器

1. 复制之前的mysql文件夹

   [![img](E:\Development\Typora\images\20200720204009631.png)](https://img-blog.csdnimg.cn/20200720204009631.png)



1. 修改复制之后mysql配置文件

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624843.png)](https://img-blog.csdnimg.cn/20200720204530475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

- 修改端口

[![img](E:\Development\Typora\images\20200720204653472.png)](https://img-blog.csdnimg.cn/20200720204653472.png)

- 修改文件的路径(安装和数据目录)

[![img](E:\Development\Typora\images\20200720204813163.png)](https://img-blog.csdnimg.cn/20200720204813163.png)

1. 在win上面安装服务

```bash
mysqld install mysqls1 --defaults-file="E:\Mysql\mysql-5.7.27-winx64 - 2\my.ini"
    
#如果想删除服务的话,执行
sc delete 服务名称
```

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624844.png)](https://img-blog.csdnimg.cn/20200720205304324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 注意以管理员的身份运行

1. 启动服务

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624845.png)](https://img-blog.csdnimg.cn/20200720205505589.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



##### 配置mysql中的主从复制

1. 在主服务器配置文件(my.ini)中配置
   - 在最下方增加配置

```properties
#开启日志
log-bin = mysql-bin
#设置服务id，主从不能一致
server-id = 1 
#设置需要同步的数据库
binlog-do-db=user_db 
#屏蔽系统库同步
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
```

1. 在从服务器配置文件(my.ini)中配置
   - 在最下方增加配置

```properties
#开启日志
log-bin = mysql-bin
#设置服务id，主从不能一致
server-id = 2 
#设置需要同步的数据库
replicate_wild_do_table=sharding_sphere.%
#屏蔽系统库同步
replicate_wild_ignore_table=mysql.%
replicate_wild_ignore_table=information_schema.%
replicate_wild_ignore_table=performance_schema.%
```



1. 重启两台服务器的服务

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624946.png)](https://img-blog.csdnimg.cn/20200720211043581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



##### 创建用于主从服务的账号

```properties
#切换至主库bin目录，登录主库
mysql ‐h localhost ‐u root ‐p
#授权主备复制专用账号
GRANT REPLICATION SLAVE ON *.* TO 'db_sync'@'%' IDENTIFIED BY 'db_sync'; 
#刷新权限
FLUSH PRIVILEGES;
#确认位点 记录下文件名以及位点
show master status;
```

[![img](E:\Development\Typora\images\20200720211758665.png)](https://img-blog.csdnimg.cn/20200720211758665.png)



##### 设置从库向主库同步数据、并检查连接

```properties
#切换至从库bin目录，登录从库
mysql ‐h localhost ‐P 3307 ‐u root ‐p
#先停止同步
STOP SLAVE;
#修改从库指向到主库，使用上一步记录的文件名以及位点
CHANGE MASTER TO
master_host = 'localhost',
master_user = 'db_sync',
master_password = 'db_sync',
master_log_file = 'mysql‐bin.000034',
master_log_pos = 154;
#启动同步
#查看从库状态Slave_IO_Runing和Slave_SQL_Runing都为Yes说明同步成功，如果不为Yes，请检查error_log，然后排查相关异常。
show slave status\G 
#注意 如果之前此备库已有主库指向 需要先执行以下命令清空
STOP SLAVE IO_THREAD FOR CHANNEL '';
reset slave all;
```

- master_log_file = ‘mysql‐bin.000034’, 是上方截图的File
- master_log_pos = 154;是上方截图的Position
- show slave status\G查看是否是yes

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624947.png)](https://img-blog.csdnimg.cn/20200720212144850.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)





#### 配置文件

```properties
#shardingjdbc分片策略
#配置数据源,给数据源配置名字，可配置多个
spring.shardingsphere.datasource.names=ds1,ds2,ds0,s0

spring.shardingsphere.datasource.ds0.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.ds0.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds0.url=jdbc:mysql://localhost:3306/sharding_sphere?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.ds0.username=jiang
spring.shardingsphere.datasource.ds0.password=jiang


#ds2数据源的从服务器
spring.shardingsphere.datasource.s0.type=com.alibaba.druid.pool.DruidDataSource
#spring.shardingsphere.datasource.s0.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.s0.url=jdbc:mysql://localhost:3307/sharding_sphere?characterEncoding=UTF-8&serverTimezone=UTC
spring.shardingsphere.datasource.s0.username=root
spring.shardingsphere.datasource.s0.password=jcl5412415845

# 主库从库逻辑数据源定义 (设置主服务器和从服务器,并起名)
spring.shardingsphere.sharding.master‐slave‐rules.ms0.master-data-source-name=ds0
spring.shardingsphere.sharding.master‐slave‐rules.ms0.slave-data-source-names=s0

spring.shardingsphere.sharding.tables.tb_user.actual-data-nodes=ms0.tb_user

#指定user表里面主键id生成策略 雪花算法
spring.shardingsphere.sharding.tables.tb_user.key-generator.column=id
spring.shardingsphere.sharding.tables.tb_user.key-generator.type=SNOWFLAKE
```

> 只显示了主要配置





## ShardingSphere-Proxy

### 介绍

 定位为`透明化的数据库代理端`，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。 目前提供 MySQL 和 PostgreSQL 版本，它可以使用任何兼容 MySQL/PostgreSQL 协议的访问客户端(如：MySQL Command Client, MySQL Workbench, Navicat 等)操作数据，对 DBA 更加友好。

- 向应用程序完全透明，可直接当做 MySQL/PostgreSQL 使用。
- 适用于任何兼容 MySQL/PostgreSQL 协议的的客户端。

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624948.png)](https://img-blog.csdnimg.cn/20200705005131804.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 下载

[https://shardingsphere.apache.org/document/current/cn/downloads/#%E5%85%A8%E9%83%A8%E7%89%88%E6%9C%AC](https://shardingsphere.apache.org/document/current/cn/downloads/#全部版本)





### 修改jar包

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624949.png)](https://img-blog.csdnimg.cn/20200720232747915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> ```
> 将lib下的包后缀全部换成jar
> ```

### 修改文件

- **进入** **conf** **目录，修改文件** **server.yaml**，打开两段内容注释

  ```yaml
  authentication:
    users:
      root:
        password: root
      sharding:
        password: sharding 
        authorizedSchemas: sharding_db
  
  props:
    max.connections.size.per.query: 1
    acceptor.size: 16  # The default value is available processors count * 2.
    executor.size: 16  # Infinite by default.
    proxy.frontend.flush.threshold: 128  # The default value is 128.
      # LOCAL: Proxy will run with LOCAL transaction.
      # XA: Proxy will run with XA transaction.
      # BASE: Proxy will run with B.A.S.E transaction.
    proxy.transaction.type: LOCAL
    proxy.opentracing.enabled: false
    proxy.hint.enabled: false
    query.with.cipher.column: true
    sql.show: false
    allow.range.query.with.inline.sharding: false
  ```

- **进入** **conf** **目录，修改** **config-sharding.yaml**

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624950.png)](https://img-blog.csdnimg.cn/20200720231646513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 需要添加mysq驱动

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624951.png)](https://img-blog.csdnimg.cn/20200720231635778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



```yml
dataSources:
  ds_0:
    url: jdbc:mysql://localhost:3306/sharding_sphere?characterEncoding=UTF-8&serverTimezone=UTC
    username: jiang
    password: jiang
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50

shardingRule:
  tables:
    t_order:
      actualDataNodes: ds_${0}.tb_order_${1..2}
      tableStrategy:
        inline:
          shardingColumn: order_id
          algorithmExpression: tb_order_${order_id % 2+1}
      keyGenerator:
        type: SNOWFLAKE
        column: order_id
  bindingTables:
    - tb_order
  defaultDatabaseStrategy:
    inline:
      shardingColumn: user_id
      algorithmExpression: ds_${0}
  defaultTableStrategy:
    none:
```

> 以水平分表为例

### 启动

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624952.png)](https://img-blog.csdnimg.cn/20200720232901171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

```
#默认是3307端口，可以自行指定
start.bat 3308
```

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624953.png)](https://img-blog.csdnimg.cn/20200720233341412.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

> 如果出现未找到类的情况，应该是jar包问题,请修改lib下所有包的后缀为jar



### 连接数据库

```
mysql  -P3308 -u root -p
```

- 3308是启动端口
- 用户名为root，密码也是root,在配置文件中指定了

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624954.png)](https://img-blog.csdnimg.cn/20200720233457549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



### 代理数据库中的库

[![img](E:\Development\Typora\images\20200720235130687.png)](https://img-blog.csdnimg.cn/20200720235130687.png)

> 与我们配置中设置的一样





### 水平分表

#### 建表

```mysql
CREATE TABLE `tb_order_1` (
  `order_id` bigint(20) NOT NULL COMMENT '订单id',
  `price` decimal(10,2) NOT NULL COMMENT '订单价格',
  `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
  `status` varchar(50) NOT NULL COMMENT '订单状态',
  PRIMARY KEY (`order_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

CREATE TABLE `tb_order_2` (
  `order_id` bigint(20) NOT NULL COMMENT '订单id',
  `price` decimal(10,2) NOT NULL COMMENT '订单价格',
  `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
  `status` varchar(50) NOT NULL COMMENT '订单状态',
  PRIMARY KEY (`order_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

[![img](E:\Development\Typora\images\20200720234932367.png)](https://img-blog.csdnimg.cn/20200720234932367.png)

#### 配置文件

```yml
dataSources:
  ds_0:
    url: jdbc:mysql://localhost:3306/sharding_sphere?characterEncoding=UTF-8&serverTimezone=UTC
    username: jiang
    password: jiang
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50

shardingRule:
  tables:
    t_order:
      actualDataNodes: ds_${0}.tb_order_${1..2}
      tableStrategy:
        inline:
          shardingColumn: order_id
          algorithmExpression: tb_order_${order_id % 2+1}
      keyGenerator:
        type: SNOWFLAKE
        column: order_id
  bindingTables:
    - tb_order
  defaultDatabaseStrategy:
    inline:
      shardingColumn: user_id
      algorithmExpression: ds_${0}
  defaultTableStrategy:
    none:
```



#### 连接数据库

```
mysql  -P3308 -u root -p
```



 查看数据库和数据表

[![img](E:\Development\Typora\images\20200720235130687.png)](https://img-blog.csdnimg.cn/20200720235130687.png)

[![img](E:\Development\Typora\images\20200720235255976.png)](https://img-blog.csdnimg.cn/20200720235255976.png)

> `可以看到我们这里只有一个t_order的表`，而不是tb_order1和tb_order2两个表。至于为什么会是t_order而不是tb_order，看了一眼配置，我在tables里设置的就是t_order



#### 操作

##### 插入

```
insert into t_order values(1,10.0,1123,'管理员');
insert into t_order values(2,10.0,1123,'管理员');
insert into t_order values(3,10.0,1123,'管理员');
insert into t_order values(4,10.0,1123,'管理员');
```

 查看

[![img](E:\Development\Typora\images\20200720235715548.png)](https://img-blog.csdnimg.cn/20200720235715548.png)

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988624955.png)](https://img-blog.csdnimg.cn/20200720235728234.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



##### 查询

[![img](E:\Development\Typora\images\20200720235818168.png)](https://img-blog.csdnimg.cn/20200720235818168.png)



### 水平分库

#### 建表

```
CREATE TABLE `tb_order_1` (
  `order_id` bigint(20) NOT NULL COMMENT '订单id',
  `price` decimal(10,2) NOT NULL COMMENT '订单价格',
  `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
  `status` varchar(50) NOT NULL COMMENT '订单状态',
  PRIMARY KEY (`order_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;

CREATE TABLE `tb_order_2` (
  `order_id` bigint(20) NOT NULL COMMENT '订单id',
  `price` decimal(10,2) NOT NULL COMMENT '订单价格',
  `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
  `status` varchar(50) NOT NULL COMMENT '订单状态',
  PRIMARY KEY (`order_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=DYNAMIC;
```

[![img](E:\Development\Typora\images\20200721000523110.png)](https://img-blog.csdnimg.cn/20200721000523110.png)

#### 配置

```
dataSources:
  ds_0:
    url: jdbc:mysql://localhost:3306/sharding_sphere_1?characterEncoding=UTF-8&serverTimezone=UTC
    username: jiang
    password: jiang
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50
  ds_1:
    url: jdbc:mysql://localhost:3306/sharding_sphere_2?characterEncoding=UTF-8&serverTimezone=UTC
    username: jiang
    password: jiang
    connectionTimeoutMilliseconds: 30000
    idleTimeoutMilliseconds: 60000
    maxLifetimeMilliseconds: 1800000
    maxPoolSize: 50

shardingRule:
  tables:
    tb_order:
      actualDataNodes: ds_${0,1}.tb_order_${1..2}
      tableStrategy:
        inline:
          shardingColumn: order_id
          algorithmExpression: tb_order_${order_id % 2+1}
      keyGenerator:
        type: SNOWFLAKE
        column: order_id
  bindingTables:
    - tb_order
  defaultDatabaseStrategy:
    inline:
      shardingColumn: user_id
      algorithmExpression: ds_${user_id%2}
  defaultTableStrategy:
    none:
```



#### 启动并连接数据库

```
start.bat 3308

mysql  -P3308 -u root -p
```



 查看数据库和数据表

[![img](E:\Development\Typora\images\20200720235130687.png)](https://img-blog.csdnimg.cn/20200720235130687.png)

[![img](E:\Development\Typora\images\20200721001556389.png)](https://img-blog.csdnimg.cn/20200721001556389.png)



#### 操作

##### 插入

```
insert into tb_order values(1,10.0,1123,'管理员');
insert into tb_order values(2,10.0,1123,'管理员');
insert into tb_order values(3,10.0,1123,'管理员');
insert into tb_order values(4,10.0,1123,'管理员');
insert into tb_order values(5,10.0,1124,'管理员');
insert into tb_order values(6,10.0,1124,'管理员');
insert into tb_order values(7,10.0,1124,'管理员');
insert into tb_order values(8,10.0,1124,'管理员');
```

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988625056.png)](https://img-blog.csdnimg.cn/20200721001438735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)



##### 查询

[![img](E:\Development\Typora\images\watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70-166256988625057.png)](https://img-blog.csdnimg.cn/20200721001527603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0NzY2ODgz,size_16,color_FFFFFF,t_70)

-------------本文结束感谢您的阅读-------------

六经蕴籍胸中久，一剑十年磨在手

打赏

- **本文作者：** 迷雾总会解
- **本文链接：** https://blog.codekiller.top/2021/02/03/ShardingSphere/
- **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/) 许可协议。转载请注明出处！
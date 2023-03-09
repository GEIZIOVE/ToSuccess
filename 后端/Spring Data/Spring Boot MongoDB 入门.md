Spring Boot MongoDB 入门
====================================================================================================

* * *

> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 [lab-16-spring-data-mongo](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo) 目录。



# 1. 概述

可能有一些胖友对 MongoDB 不是很了解，这里我们引用一段介绍：

> FROM [《分布式文档存储数据库 MongoDB》](https://www.oschina.net/p/mongodb)
>
> MongoDB 是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
>
> 他支持的数据结构非常松散，是类似 json 的 bjson 格式，因此可以存储比较复杂的数据类型。
>
> Mongo 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

MongoDB 中的许多概念在 MySQL 中具有相近的类比。本表概述了每个系统中的一些常见概念。

> 对于不熟悉的胖友，可以先看下该表，然后开始本文的旅程。

| MySQL       | MongoDB          |
| :---------- | :--------------- |
| 库 Database | 库 Database      |
| 表 Table    | 集合 Collection  |
| 行 Row      | 文档 Document    |
| 列 Column   | 字段 Field       |
| joins       | 嵌入文档或者链接 |

在早期，在项目中 MongoDB 的 ORM 框架使用 [Morphia](https://github.com/MorphiaOrg/morphia) 较多。随着 [Spring Data MongoDB](https://spring.io/projects/spring-data-mongodb) 的日趋完善，更为主流。目前，艿艿手头所有的项目，都从 Morphia 该用 Spring Data MongoDB 。

在 Spring Data MongoDB 中，有两种方式进行 MongoDB 操作：

- **Spring Data Repository 方式**
- **MongoTemplate**

# 2. 快速入门

> 示例代码对应仓库：[lab-16-spring-data-mongodb](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb) 。
>
> - MongoDB 版本号：4.2.1

本小节，我们会使用 [`spring-boot-starter-data-mongodb`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb) 自动化配置 Spring Data MongoDB 主要配置。同时，使用 **Spring Data Repository** 实现的 MongoDB 的 CRUD 操作。

## 2.1 引入依赖

在 [`pom.xml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/pom.xml) 文件中，引入相关依赖。



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

    <artifactId>lab-16-spring-data-mongodb</artifactId>

    <dependencies>
        <!-- 自动化配置 Spring Data Mongodb -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
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

## 2.2 Application

创建 `Application.java`类，配置 `@SpringBootApplication` 注解即可。代码如下：

```java
// Application.java
@SpringBootApplication(exclude = {ElasticsearchAutoConfiguration.class, ElasticsearchDataAutoConfiguration.class})
public class Application {
}
```

## 2.3 MongoDBConfig

在 [`cn.iocoder.springboot.lab16.springdatamongodb.config`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/config) 包路径下，创建 [MongoDBConfig](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/config/MongoDBConfig.java) 配置类。代码如下：



```java
// MongoDBConfig.java

@Configuration
public class MongoDBConfig {

    @Bean // 目的，就是为了移除 _class field 。参考博客 https://blog.csdn.net/bigtree_3721/article/details/82787411
    public MappingMongoConverter mappingMongoConverter(MongoDbFactory factory,
                                                       MongoMappingContext context,
                                                       BeanFactory beanFactory) {
        // 创建 DbRefResolver 对象
        DbRefResolver dbRefResolver = new DefaultDbRefResolver(factory);
        // 创建 MappingMongoConverter 对象
        MappingMongoConverter mappingConverter = new MappingMongoConverter(dbRefResolver, context);
        // 设置 conversions 属性
        try {
            mappingConverter.setCustomConversions(beanFactory.getBean(CustomConversions.class));
        } catch (NoSuchBeanDefinitionException ignore) {
        }
        // 设置 typeMapper 属性，从而移除 _class field 。
        mappingConverter.setTypeMapper(new DefaultMongoTypeMapper(null));
        return mappingConverter;
    }

}
```

- 通过在自定义 MappingMongoConverter Bean 对象，避免实体保存到 MongoDB 中时，会多一个 `_class` 字段，存储实体的全类名。

## 2.4 配置文件

在 [`application.yml`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/resources/application.yaml) 中，添加 MongoDB 配置，如下：



```yml
spring:
  data:
    # MongoDB 配置项，对应 MongoProperties 类
    mongodb:
      host: 127.0.0.1
      port: 27017
      database: yourdatabase
      username: test01
      password: password01
      # 上述属性，也可以只配置 uri

logging:
  level:
    org:
      springframework:
        data:
          mongodb:
            core: DEBUG # 打印 mongodb 操作的具体语句。生产环境下，不建议开启。
```



## 2.5 UserDO

在 [`cn.iocoder.springboot.lab16.springdatamongodb.dataobject`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dataobject) 包路径下，创建 [UserDO](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dataobject/UserDO.java) 类。代码如下：



```java
@Document(collection = "User")
public class UserDO {

    /**
     * 用户信息
     */
    public static class Profile {

        /**
         * 昵称
         */
        private String nickname;
        /**
         * 性别
         */
        private Integer gender;

        // ... 省略 setting/getting 方法

    }

    @Id
    private Integer id;
    /**
     * 账号
     */
    private String username;
    /**
     * 密码
     */
    private String password;
    /**
     * 创建时间
     */
    private Date createTime;
    /**
     * 用户信息
     */
    private Profile profile;

    // ... 省略 setting/getting 方法

}
```



- 在 UserDO 类中，我们内嵌了一个 `profile` 属性，它是 Profile 类。这里仅仅作为示例，实际场景下，还是建议把 User 和 Profile 拆分开。
- 推荐阅读 [《你应该知道的 MongoDB 最佳实践》](http://www.iocoder.cn/MongoDB/MongoDB-best-practices) 文章。对于初用 MongoDB 的开发者，往往错误的使用**内嵌**属性，需要去理解一下。

## 2.6 UserRepository

在 [`cn.iocoder.springboot.lab16.springdatamongodb.repository`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository) 包路径下，创建 [UserRepository](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepository.java) 接口。代码如下：



```java
// ProductRepository.java

public interface UserRepository extends MongoRepository<UserDO, Integer> {
}
```



- 继承 `org.springframework.data.mongodb.repository.MongoRepository` 接口，第一个泛型设置对应的实体是 **UserDO** ，第二个泛型设置对应的**主键类型**是 Integer 。
- 因为实现了 MongoRepository 接口，Spring Data MongoDB 会自动生成对应的 CRUD 等等的代码。😈 是不是很方便。
- MongoRepository 类图如下：

![MongoRepository 类图](E:/Development/Typora/images/01-1676709601593-1.png)



- 每个接口定义的方法，胖友可以点击下面每个链接，自己瞅瞅，简单~
- [`org.springframework.data.repository.CrudRepository`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/CrudRepository.java)
- [`org.springframework.data.repository.PagingAndSortingRepository`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/PagingAndSortingRepository.java)
- [`org.springframework.data.repository.query.QueryByExampleExecutor`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/query/QueryByExampleExecutor.java) ：定义基于 `org.springframework.data.domain.Example.Example` 的查询操作。在 [「4. 基于 Example 查询」](https://www.iocoder.cn/Spring-Boot/MongoDB/?github#) 中，我们再来详细介绍。
- [`org.springframework.data.mongodb.repository.MongoRepository`](https://github.com/spring-projects/spring-data-mongodb/blob/master/spring-data-mongodb/src/main/java/org/springframework/data/mongodb/repository/MongoRepository.java) ：主要重写 PagingAndSortingRepository 和 CrudRepository 接口，将 findXXX 方法返回的结果从 Iterable 放大成 List ，同时增加 insert 插入方法。

> 会发现和 Spring Data JPA 的使用方式，**基本一致**。这就是 Spring Data 带给我们的好处，使用相同的 API ，统一访问不同的数据源。o(￣▽￣)ｄ 点赞。

## 2.7 简单测试

创建 [UserRepositoryTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/test/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepositoryTest.java) 测试类，我们来测试一下简单的 UserRepositoryTest 的每个操作。代码如下：



```java
// UserRepositoryTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test // 插入一条记录
    public void testInsert() {
        // 创建 UserDO 对象
        UserDO user = new UserDO();
        user.setId(1); // 这里先临时写死一个 ID 编号，后面演示自增 ID 的时候，在修改这块
        user.setUsername("yudaoyuanma");
        user.setPassword("buzhidao");
        user.setCreateTime(new Date());
        // 创建 Profile 对象
        UserDO.Profile profile = new UserDO.Profile();
        profile.setNickname("芋道源码");
        profile.setGender(1);
        user.setProfile(profile);
        // 存储到 DB
        userRepository.insert(user);
    }

    // 这里要注意，如果使用 save 方法来更新的话，必须是全量字段，否则其它字段会被覆盖。
    // 所以，这里仅仅是作为一个示例。
    @Test // 更新一条记录
    public void testUpdate() {
        // 查询用户
        Optional<UserDO> userResult = userRepository.findById(1);
        Assert.isTrue(userResult.isPresent(), "用户一定要存在");
        // 更新
        UserDO updateUser = userResult.get();
        updateUser.setUsername("yutou");
        userRepository.save(updateUser);
    }

    @Test // 根据 ID 编号，删除一条记录
    public void testDelete() {
        userRepository.deleteById(1);
    }

    @Test // 根据 ID 编号，查询一条记录
    public void testSelectById() {
        Optional<UserDO> userDO = userRepository.findById(1);
        System.out.println(userDO.isPresent());
    }

    @Test // 根据 ID 编号数组，查询多条记录
    public void testSelectByIds() {
        Iterable<UserDO> users = userRepository.findAllById(Arrays.asList(1, 4));
        users.forEach(System.out::println);
    }

}
```



- 每个测试单元方法，胖友自己看看方法上的注释。

具体的，胖友可以自己跑跑，妥妥的。

# 3. 基于方法名查询

> 示例代码对应仓库：[lab-16-spring-data-mongodb](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb) 。

在 [《芋道 Spring Boot JPA 入门》](http://www.iocoder.cn/Spring-Boot/JPA/?self) 文章的「4. 基于方法名查询」小节中，我们已经提到：

> 在 Spring Data 中，支持根据方法名作生成对应的查询（`WHERE`）条件，进一步进化我们使用 JPA ，具体是方法名以 `findBy`、`existsBy`、`countBy`、`deleteBy` 开头，后面跟具体的条件。具体的规则，在 [《Spring Data JPA —— Query Creation》](https://docs.spring.io/spring-data/jpa/docs/2.2.0.RELEASE/reference/html/#jpa.query-methods.query-creation) 文档中，已经详细提供。如下：

| 关键字                 |                           方法示例                           | JPQL snippet                                                 |
| :--------------------- | :----------------------------------------------------------: | :----------------------------------------------------------- |
| `And`                  |                 `findByLastnameAndFirstname`                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   |                 `findByLastnameOrFirstname`                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`, `Equals`         | `findByFirstname`,`findByFirstnameIs`,<`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              |                   `findByStartDateBetween`                   | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             |                     `findByAgeLessThan`                      | `… where x.age < ?1`                                         |
| `LessThanEqual`        |                   `findByAgeLessThanEqual`                   | `… where x.age <= ?1`                                        |
| `GreaterThan`          |                    `findByAgeGreaterThan`                    | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     |                 `findByAgeGreaterThanEqual`                  | `… where x.age >= ?1`                                        |
| `After`                |                    `findByStartDateAfter`                    | `… where x.startDate > ?1`                                   |
| `Before`               |                   `findByStartDateBefore`                    | `… where x.startDate < ?1`                                   |
| `IsNull`, `Null`       |                     `findByAge(Is)Null`                      | `… where x.age is null`                                      |
| `IsNotNull`, `NotNull` |                    `findByAge(Is)NotNull`                    | `… where x.age not null`                                     |
| `Like`                 |                    `findByFirstnameLike`                     | `… where x.firstname like ?1`                                |
| `NotLike`              |                   `findByFirstnameNotLike`                   | `… where x.firstname not like ?1`                            |
| `StartingWith`         |                `findByFirstnameStartingWith`                 | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`           |                 `findByFirstnameEndingWith`                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`           |                 `findByFirstnameContaining`                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`              |                `findByAgeOrderByLastnameDesc`                | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  |                     `findByLastnameNot`                      | `… where x.lastname <> ?1`                                   |
| `In`                   |                `findByAgeIn(Collection ages)`                | `… where x.age in ?1`                                        |
| `NotIn`                |              `findByAgeNotIn(Collection ages)`               | `… where x.age not in ?1`                                    |
| `True`                 |                     `findByActiveTrue()`                     | `… where x.active = true`                                    |
| `False`                |                    `findByActiveFalse()`                     | `… where x.active = false`                                   |
| `IgnoreCase`           |                 `findByFirstnameIgnoreCase`                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

- 注意，如果我们有排序需求，可以使用 `OrderBy` 关键字。

下面，我们来编写一个简单的示例。

## 3.1 UserRepository02

在 [`cn.iocoder.springboot.lab16.springdatamongodb.repository`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/) 包路径下，创建 [UserRepository02](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepository02.java) 接口。代码如下：



```java
// UserRepository02.java

public interface UserRepository02 extends MongoRepository<UserDO, Integer> {

    UserDO findByUsername(String username);

    Page<UserDO> findByUsernameLike(String username, Pageable pageable);

}
```

- 对于分页操作，需要使用到 `Pageable` 参数，需要作为方法的最后一个参数。
- 注意噢，基于方法名查询，不支持内嵌对象的属性。

## 3.2 简单测试

创建 [UserRepository02Test](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/test/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepository02Test.java) 测试类，我们来测试一下简单的 UserRepository02Test 的每个操作。代码如下：



```java
// UserRepository02Test.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserRepository02Test {

    @Autowired
    private UserRepository02 userRepository;

    @Test // 根据名字获得一条记录
    public void testFindByName() {
        UserDO user = userRepository.findByUsername("yutou");
        System.out.println(user);
    }

    @Test // 使用 username 模糊查询，分页返回结果
    public void testFindByNameLike() {
        // 创建排序条件
        Sort sort = new Sort(Sort.Direction.DESC, "id"); // ID 倒序
        // 创建分页条件。
        Pageable pageable = PageRequest.of(0, 10, sort);
        // 执行分页操作
        Page<UserDO> page = userRepository.findByUsernameLike("yu", pageable);
        // 打印
        System.out.println(page.getTotalElements());
        System.out.println(page.getTotalPages());
    }

}
```



- 每个测试单元方法，胖友自己看看方法上的注释。

具体的，胖友可以自己跑跑，妥妥的。

# 4. 基于 Example 查询

> 示例代码对应仓库：[lab-16-spring-data-mongodb](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb) 。

> 实际场景下，我们并不会**基于 Example 查询**。所以本小节，胖友可以选择性看看即可。

对于大多数胖友，可能不了解 Spring Data Example 。我们先来一起看看官方文档的介绍：

> Query by Example (QBE) is a user-friendly querying technique with a simple interface. 使用 Example 进行查询，是一种友好的查询方式，可以使用便捷的 API 方法。
>
> It allows dynamic query creation and does not require to write queries containing field names. 它允许创建动态查询，而无需编写包含字段名的查询。
>
> In fact, Query by Example does not require to write queries using store-specific query languages at all. 事实上，在使用 Example 进行查询的时候，我们无需使用特定的存储器（数据库）的查询语言。

- 请原谅艿艿蹩脚的翻译。简单来说，我们可以通过 Example 进行编写动态的查询条件，而无需使用每个不同的 Spring Data 实现类的 Query 对象。例如说：

  - Spring Data JPA 提供的 `javax.persistence.criteria.Predicate`
  - Spring Data MongoDB 提供的 `org.springframework.data.mongodb.core.query.Query` 。

- 相当于说，不同 Spring Data 实现框架，会将 Spring Data Example 条件，翻译成对应的查询对象。例如说：

  - Spring Data JPA 将 Example 转换成 Predicate 。
  - Spring Data MongoDB 将 Example 转换成 Query 。

- 当然，并不是所有 Spring Data 实现都支持 Spring Data Example 。例如说，Spring Data Elasticsearch 并不支持。一般情况下，支持 Spring Data Example 的 Spring Data 实现框架，它们的 Repository 都继承了

  `org.springframework.data.repository.query.QueryByExampleExecutor`

  接口。例如说：

  - Spring Data JPA 的 [JpaRepository](https://github.com/spring-projects/spring-data-jpa/blob/master/src/main/java/org/springframework/data/jpa/repository/JpaRepository.java) 接口。
  - Spring Data MongoDB 的 [MongoRepository](https://github.com/spring-projects/spring-data-mongodb/blob/master/spring-data-mongodb/src/main/java/org/springframework/data/mongodb/repository/MongoRepository.java) 接口。

Example API 一共包含三部分：

- Probe ：含有对应字段的实体对象。通过设置该实体对象的字段，作为查询字段。

  > 注意，Probe 并不是一个类，而是实体对象的泛指。

- [ExampleMatcher](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/domain/ExampleMatcher.java) ：ExampleMatcher 可以定义特定字段的匹配模式。例如说，全模糊匹配、前缀模糊匹配等等。

  > 简单来说，通过实体对象的字段作为查询条件，只能满足相等的情况，对于 `!=` 、`LIKE` 等等情况，需要通过 ExampleMatcher 特殊指定。如果不理解，没事，看了示例会更容易明白。

- [Example](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/domain/Example.java) ：是 Probe 和 ExampleMatcher 的组合，构成查询对象。

当然，Example 有它的限制条件：

> No support for nested/grouped property constraints like firstname = ?0 or (firstname = ?1 and lastname = ?2) 不支持嵌套或分组约束。例如说，`firstname = ?0 or (firstname = ?1 and lastname = ?2)` 。
>
> Only supports starts/contains/ends/regex matching for strings and exact matching for other property types 对于字符串，只支持 tarts/contains/ends/regex 匹配方式；对于其它类型，只支持精确匹配。

## 4.1 UserRepository03

在 [`cn.iocoder.springboot.lab16.springdatamongodb.repository`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/) 包路径下，创建 [UserRepository02](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepository02.java) 接口。代码如下：



```java
// UserRepository03.java

public interface UserRepository03 extends MongoRepository<UserDO, Integer> {

    // 使用 username 精准匹配
    default UserDO findByUsername01(String username) {
        // 创建 Example 对象，使用 username 查询
        UserDO probe = new UserDO();
        probe.setUsername(username); // 精准匹配 username 查询
        Example<UserDO> example = Example.of(probe);
        // 执行查询
        return findOne(example)
                .orElse(null); // 如果为空，则返回 null
    }

    // 使用 username 模糊匹配
    default UserDO findByUsernameLike01(String username) {
        // 创建 Example 对象，使用 username 查询
        UserDO probe = new UserDO();
        probe.setUsername(username); // 这里还需要设置
        ExampleMatcher matcher = ExampleMatcher.matching()
                .withMatcher("username", ExampleMatcher.GenericPropertyMatchers.contains()); // 模糊匹配 username 查询
        Example<UserDO> example = Example.of(probe, matcher);
        // 执行查询
        return findOne(example)
                .orElse(null); // 如果为空，则返回 null
    }

}
```



## 4.2 简单测试

创建 [UserRepository03Test](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/test/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepository03Test.java) 测试类，我们来测试一下简单的 UserRepository03Test 的每个操作。代码如下：



```java
// UserRepository03Test.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserRepository03Test {

    @Autowired
    private UserRepository03 userRepository;

    @Test
    public void testFindByUsername01() {
        UserDO user = userRepository.findByUsername01("yutou");
        System.out.println(user);
    }

    @Test
    public void testFindByUsernameLike01() {
        UserDO user = userRepository.findByUsernameLike01("yu");
        System.out.println(user);
    }

}
```



- 每个测试单元方法，胖友自己看看方法上的注释。

具体的，胖友可以自己跑跑，妥妥的。更多示例，可以看看如下文章：

- [《Spring Data JPA Query by Example》](https://www.baeldung.com/spring-data-query-by-example)
- [《Spring Data JPA 使用 Example 快速实现动态查询》](https://blog.csdn.net/long476964/article/details/79677526)

# 5. MongoTemplate

> 示例代码对应仓库：[lab-16-spring-data-mongodb](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb) 。

在 Spring Data MongoDB 中，有一个 [MongoTemplate](https://github.com/spring-projects/spring-data-mongodb/blob/master/spring-data-mongodb/src/main/java/org/springframework/data/mongodb/core/MongoTemplate.java) 类，提供了 MongoDB 操作模板，方便我们操作 MongoDB 。

## 5.1 UserDao

在 [`cn.iocoder.springboot.lab16.springdatamongodb.dao`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dao/) 包路径下，创建 [UserDao](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dao/UserDao.java) 类。代码如下：



```java
// UserDao.java

@Repository
public class UserDao {

    @Autowired
    private MongoTemplate mongoTemplate;

    public void insert(UserDO entity) {
        mongoTemplate.insert(entity);
    }

    public void updateById(UserDO entity) {
        // 生成 Update 条件
        final Update update = new Update();
        // 反射遍历 entity 对象，将非空字段设置到 Update 中
        ReflectionUtils.doWithFields(entity.getClass(), new ReflectionUtils.FieldCallback() {

            @Override
            public void doWith(Field field) throws IllegalArgumentException, IllegalAccessException {
                // 排除指定条件
                if ("id".equals(field.getName()) // 排除 id 字段，因为作为查询主键
                        || field.getAnnotation(Transient.class) != null // 排除 @Transient 注解的字段，因为非存储字段
                        || Modifier.isStatic(field.getModifiers())) { // 排除静态字段
                    return;
                }
                // 设置字段可反射
                if (!field.isAccessible()) {
                    field.setAccessible(true);
                }
                // 排除字段为空的情况
                if (field.get(entity) == null) {
                    return;
                }
                // 设置更新条件
                update.set(field.getName(), field.get(entity));
            }

        });
        // 防御，避免有业务传递空的 Update 对象
        if (update.getUpdateObject().isEmpty()) {
            return;
        }
        // 执行更新
        mongoTemplate.updateFirst(new Query(Criteria.where("_id").is(entity.getId())), update, UserDO.class);
    }

    public void deleteById(Integer id) {
        mongoTemplate.remove(new Query(Criteria.where("_id").is(id)), UserDO.class);
    }

    public UserDO findById(Integer id) {
        return mongoTemplate.findOne(new Query(Criteria.where("_id").is(id)), UserDO.class);
    }

    public UserDO findByUsername(String username) {
        return mongoTemplate.findOne(new Query(Criteria.where("username").is(username)), UserDO.class);
    }

    public List<UserDO> findAllById(List<Integer> ids) {
        return mongoTemplate.find(new Query(Criteria.where("_id").in(ids)), UserDO.class);
    }

}
```



- 使用 MongoTemplate 实现了 CRUD 的功能。其它 findAndModify/findAndRemove/count/upsert 等操作，胖友可以自己尝试下。





- 对于 `#updateById(UserDO entity)` 方法，胖友可以自己抽象成一个通用的 `#updateEntityFieldsById(Object id, Object entity)` 方法，封装在一个 BaseMongoDao 抽象类中。

  > 友情提示：此处暂时有个**问题**，对于 UserDO 内嵌的 `profile` 对象，一旦设置了值，是整个 Profile 对象覆盖更新。所以，使用时需要注意下。
  >
  > 目前艿艿自己项目里，大多数内嵌对象，全量更新不存在问题。如果存在问题的，提供了另外的方法解决。
  >
  > 当然，也可以进一步封装 `#updateEntityFieldsById(Object id, Object entity, String... fields)` 方法，只更新指定字段。

## 5.2 简单测试

创建 [UserDaoTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/test/java/cn/iocoder/springboot/lab16/springdatamongodb/dao/UserDaoTest.java) 测试类，我们来测试一下简单的 UserDaoTest 的每个操作。代码如下：



```java
// UserDaoTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserDaoTest {

    @Autowired
    private UserDao userDao;

    @Test // 插入一条记录
    public void testInsert() {
        // 创建 UserDO 对象
        UserDO user = new UserDO();
        user.setId(1); // 这里先临时写死一个 ID 编号，后面演示自增 ID 的时候，在修改这块
        user.setUsername("yudaoyuanma");
        user.setPassword("buzhidao");
        user.setCreateTime(new Date());
        // 创建 Profile 对象
        UserDO.Profile profile = new UserDO.Profile();
        profile.setNickname("芋道源码");
        profile.setGender(1);
        user.setProfile(profile);
        // 存储到 DB
        userDao.insert(user);
    }

    // 这里要注意，如果使用 save 方法来更新的话，必须是全量字段，否则其它字段会被覆盖。
    // 所以，这里仅仅是作为一个示例。
    @Test // 更新一条记录
    public void testUpdate() {
        // 创建 UserDO 对象
        UserDO updateUser = new UserDO();
        updateUser.setId(1);
        updateUser.setUsername("nicai");

        // 执行更新
        userDao.updateById(updateUser);
    }

    @Test // 根据 ID 编号，删除一条记录
    public void testDelete() {
        userDao.deleteById(1);
    }

    @Test // 根据 ID 编号，查询一条记录
    public void testSelectById() {
        UserDO userDO = userDao.findById(1);
        System.out.println(userDO);
    }

    @Test // 根据 ID 编号数组，查询多条记录
    public void testSelectByIds() {
        List<UserDO> users = userDao.findAllById(Arrays.asList(1, 4));
        users.forEach(System.out::println);
    }

}
```



- 每个测试单元方法，胖友自己看看方法上的注释。

# 6. 自增主键

> 示例代码对应仓库：[lab-16-spring-data-mongodb](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb) 。

MongoDB 自带的主键选择是 ObjectId 类型，需要占用 12 字节。相比 Int 4 字节或者 Long 8 字节占用更多的内存空间。而绝大多数业务场景下，Int 或 Long 足够使用，所以我们更加偏向使用 Int 或 Long 作为自增 ID 主键。

> 当然，我们在日志记录上，我们还是采用 ObjectId 为主。

我们在实现 MongoDB 自增主键时，会创建一个名为 `"sequence"` 的集合。字段如下：

> `"sequence"` 集合名，也可以取其它名字，看自己喜好。

- `_id` ：String 类型，集合的实体类名。
- `value` ：Long 类型，存储每个集合的自增序列。

在程序中，每次插入实体对象到 MongoDB 之前，通过 [$inc](https://docs.mongodb.com/manual/reference/operator/update/inc/) 操作，从 `"sequence"` 自增获得最新的 ID ，然后将该 ID 赋值给实体对象，最终在插入到 MongoDB 之中。

当然，考虑到并不是所有实体都需要使用自增 ID ，所以我们要有方式去标记：

- 方式一：创建自定义 `@AutoIncKey` 注解，添加到 ID 属性上。
- 方式二：创建 IncIdEntity 抽象基类，这样需要自增的实体继承它。

对于方式一，网上博客比较多，胖友可以看看 [《Java 中实现 MongoDB 主键自增》](https://blog.csdn.net/try_try_try/article/details/80612666) 文章。

对于方式二，目前艿艿项目采用这种方式，主要是我们自己的 BaseMongoDao 提供的很多方法，是基于 IncIdEntity 抽象类的。例如说，`#updateEntityFieldsById(final IncIdEntity entity, String... fields)` 方法，这样可以方便的直接获取到 ID 属性。

当然，无论方式一还是方式二，实现原理是一致的，差异仅仅在于获取 ID 属性不同。下面，本小节我们就来实现下方式二。

## 6.1 IncIdEntity

在 [`cn.iocoder.springboot.lab16.springdatamongodb.mongo`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/mongo) 包路径下，创建 [IncIdEntity](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/mongo/IncIdEntity.java) 类。代码如下：



```
// IncIdEntity.java

/**
 * 自增主键实体
 *
 * @param <T> 主键泛型
 */
public abstract class IncIdEntity<T extends Number> {

    @Id
    private T id;

    public T getId() {
        return id;
    }

    public void setId(T id) {
        this.id = id;
    }

}
```



## 6.2 ProductDO

在 [`cn.iocoder.springboot.lab16.springdatamongodb.dataobject`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dataobject) 包路径下，创建 [ProductDO](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dataobject/ProductDO.java) 类。代码如下：



```
// ProductDO.java

@Document(collection = "Product")
public class ProductDO extends IncIdEntity<Integer> {

    /**
     * 商品名
     */
    private String name;

    public String getName() {
        return name;
    }

    public ProductDO setName(String name) {
        this.name = name;
        return this;
    }

}
```



- 继承 IncIdEntity 抽象类，使用 Integer 自增主键。

## 6.3 MongoInsertEventListener

在 [`cn.iocoder.springboot.lab16.springdatamongodb.dataobject`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dataobject) 包路径下，创建 [MongoInsertEventListener](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/dataobject/MongoInsertEventListener.java) 类。代码如下：



```
// MongoInsertEventListener.java

@Component
public class MongoInsertEventListener extends AbstractMongoEventListener<IncIdEntity> {

    /**
     * sequence - 集合名
     */
    private static final String SEQUENCE_COLLECTION_NAME = "sequence";
    /**
     * sequence - 自增值的字段名
     */
    private static final String SEQUENCE_FIELD_VALUE = "value";

    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public void onBeforeConvert(BeforeConvertEvent<IncIdEntity> event) {
        IncIdEntity entity = event.getSource();
        // 判断 id 为空
        if (entity.getId() == null) {
            // 获得下一个编号
            Number id = this.getNextId(entity);
            // 设置到实体中
            // noinspection unchecked
            entity.setId(id);
        }
    }

    /**
     * 获得实体的下一个主键 ID 编号
     *
     * @param entity 实体对象
     * @return ID 编号
     */
    private Number getNextId(IncIdEntity entity) {
        // 使用实体名的简单类名，作为 ID 编号
        String id = entity.getClass().getSimpleName();
        // 创建 Query 对象
        Query query = new Query(Criteria.where("_id").is(id));
        // 创建 Update 对象
        Update update = new Update();
        update.inc(SEQUENCE_FIELD_VALUE, 1); // 自增值
        // 创建 FindAndModifyOptions 对象
        FindAndModifyOptions options = new FindAndModifyOptions();
        options.upsert(true); // 如果不存在时，则进行插入
        options.returnNew(true); // 返回新值
        // 执行操作
        @SuppressWarnings("unchecked")
        HashMap<String, Object> result = mongoTemplate.findAndModify(query, update, options,
                HashMap.class, SEQUENCE_COLLECTION_NAME);
        // 返回主键
        return (Number) result.get(SEQUENCE_FIELD_VALUE);
    }

}
```



- MongoDB 实体对象在插入之前，会发布 BeforeConvertEvent 事件。所以，我们可以通过创建 MongoInsertEventListener 监听器，监听该事件，生成自增主键 ID 主键，设置到实体对象中。
- 如果胖友想要使用集合名作为 `"sequence"` 集合的 `"id"` ，可以使用 `BeforeConvertEvent.collectionName` 属性。

## 6.4 ProductRepository

在 [`cn.iocoder.springboot.lab16.springdatamongodb.repository`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository) 包路径下，创建 [ProductRepository](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/main/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/ProductRepository.java) 接口。代码如下：



```
// ProductRepository.java

public interface ProductRepository extends MongoRepository<ProductDO, Integer> {
}
```



## 6.5 简单测试

创建 [ProductRepositoryTest](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-16-spring-data-mongo/lab-16-spring-data-mongodb/src/test/java/cn/iocoder/springboot/lab16/springdatamongodb/repository/UserRepository03Test.java) 测试类，我们来测试一下简单的 ProductRepositoryTest 的每个操作。代码如下：



```
// ProductRepositoryTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Test
    public void testInsert() {
        // 创建 ProductDO 对象
        ProductDO product = new ProductDO();
        product.setName("芋头");
        // 插入
        productRepository.insert(product);
        // 打印 ID
        System.out.println(product.getId());
    }

}
```



# 666. 彩蛋

艿艿的项目中，**只采用 MongoTemplate** 。主要是我们封装了 BaseMongoDao ，提供了一些我们日常开发需要的功能，例如说：

- `#insertToMongo(Object objectToSave)` 方法：插入实体，使用自增 ID 主键。
- `#updateEntityFieldsById(final IncIdEntity entity, String... fields)` 方法：只更新实体指定字段。

推荐阅读：

- [《性能测试 —— MongoDB 基准测试》](http://www.iocoder.cn/Performance-Testing/MongoDB-benchmark/?self)
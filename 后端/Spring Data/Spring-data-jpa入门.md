Spring-data-jpa入门
========================

1.从入门到入土
======

1.1啥是JPA？
------

全英文名叫`Java Persistence API`，就是java持久化api，是SUN公司推出的一套基于`ORM`的规范。

> ORM呢？全英文名为`Object-Relational Mapping`：对象关系映射，简单来说为了不用JDBC那一套原始方法来操作数据库，ORM框架横空出世（mybatis、hibernate等等）。

然而ORM框架出的太多了，百花齐放，琳琅满目，你一套标准我一套标准，要是想换一套框架实现项目，可能要从头再写。啊这？入土吧。

百度这样介绍SUN的JPA规范：

Sun引入新的JPA ORM规范出于两个原因：

其一，简化现有Java EE和Java SE应用开发工作；

其二，Sun希望整合ORM技术，实现天下归一。

1.2spring-data-jpa
---------------

> 学jpa哪家强？哪家简单学哪家，spring-data-jpa最简单。
>

Spring Data JPA是Spring Data家族的一部分，可以轻松实现基于JPA的存储库。 此模块处理对基于JPA的数据访问层的增强支持。 它使构建使用数据访问技术的Spring驱动应用程序变得更加容易。

​		==总的来说JPA是ORM规范==，Hibernate、TopLink等是JPA规范的具体实现，这样的好处是开发者可以面向JPA规范进行持久层的开发，而底层的实现则是可以切换的。==Spring Data Jpa则是在JPA之上添加另一层抽象（Repository层的实现）==，极大地简化持久层开发及ORM框架切换的成本。

也就是如下图所示：

![image-20230217184235567](E:/Development/Typora/images/image-20230217184235567.png)

2.配置环境
====

话不多说，使用Maven管理包，使用springboot框架，建个空maven项目就行

3.1POM信息
-----

```xml
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <!--web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--spring-data-jpa-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!--druid连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.23</version>
        </dependency>
        <!--oracle桥接器-->
        <dependency>
            <groupId>com.oracle.ojdbc</groupId>
            <artifactId>ojdbc8</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

    </dependencies>
```

3.2application.yml
---------------

```yml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:oracle:thin:@localhost:1521:XE
    username: JPADEMO
    password: oracle
    driver-class-name: oracle.jdbc.OracleDriver
  jpa:
    hibernate:
      ddl-auto: update #自动更新
    show-sql: true  #日志中显示sql语句
  application:
    name: spring-data-jpa-demo
server:
  port: 2333 #端口号
```


3.3文件夹架构
-----

![image-20230217184546337](E:/Development/Typora/images/image-20230217184546337.png)

*   标准的MVC结构，有助于解耦的实现；
    
*   实体类放在 `pojo/entity` 下
    
*   dao（数据访问对象 data access object）在JPA中叫做`repository`，请遵守这个规范，就像mybaits的dao叫`mapper`一样。
    

3.4创建数据库和表
-------

复习一下oracle建数据库和表的操作吧

### 1.创建数据库

Jpa支持mySQL和Oracle数据库，这里使用Oracle做例子

mysql数据库也就`实体类的主键声明`和使用的`桥接器`不同，之后的章节会做具体解释

1.1 建库前先看一下这个库存不存在

```sql
-- 查看当前已有的用户
SELECT Username FROM dba_users;
```

1.2 oracle建数据库语句

```sql
-- 创建用户（schema）账号：JPADEMO 密码：oracle
create user JPADEMO identified by oracle
-- 授权
grant create session to JPADEMO;
grant create table to JPADEMO;
grant create sequence to JPADEMO;
grant unlimited tablespace to JPADEMO;
```


### 2.创建表

2.1 建一张用户表吧

```sql
-- 创建一张表
create table JPA_USER
(
    id                number not null,
    name              varchar2(100),
    object_version    number not null,
    created_by        varchar2(50),
    created_date      date,
    last_updated_by   varchar2(50),
    last_updated_date date

);
-- 给表加主键 单列主键 主键命名为JPA_USER_PK1 
alter table JPA_USER add constraint JPA_USER_PK1 primary key (id);
-- 给表加注释
COMMENT ON table JPA_USER IS '用户信息表';
-- 给字段加注释
comment on column JPA_USER.id  is 'id';
comment on column JPA_USER.name is '用户名称';
-- 创建序列 命名为JPA_USER_S
create sequence JPA_USER_S
    minvalue 1
    maxvalue 9999999999999999999999999999
    start with 1
    increment by 1
    cache 20;
--创建索引 命名为JPA_USER_INDEX1 
create index JPA_USER_INDEX1 on JPA_USER(name);
```

1.4 运行sql，成功！

![image-20230217185329825](E:/Development/Typora/images/image-20230217185329825.png)

环境配好了，开始demo吧！

3.代码
==

1.Springboot启动类
---------------

SpringContextApplication

```java
/**
 * 启动类
 */
@EnableJpaAuditing
@SpringBootApplication
public class SpringContextApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringContextApplication.class, args);
    }
}
```

除了`@SpringBootApplication`启动注解外，还有一个注解`@EnableJpaAuditing`，它是用来启动Jpa的审计功能，比如说在使用建表中经常会加入 **版本号、创建时间、修改时间 、创建者、修改者** 这五个字段。因此为了简化开发， 我们可以将其交给jpa来自动填充。

审计功能的**创建人**和**修改者**的注入方式下一节再讲哦，贪多嚼不烂。

2.entity实体类
-----------

自下而上，先把实体创建

JpaUser

```java
package org.example.jpademo.pojo.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.util.Date;

/**
 * @Classname JpaUser
 * @Description TODO 用户实体类
 * @Date 2020/8/13 14:52
 * @Created by orange
 */
@Data
@Entity
@Table(name = "JPA_USER")
@EntityListeners(AuditingEntityListener.class)
public class JpaUser {

    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "JPA_USER_S")
    @SequenceGenerator(sequenceName = "JPA_USER_S", name = "JPA_USER_S", allocationSize = 1)
    private Long id;

    @Column(name = "NAME")
    private String name;

    @Column(name = "OBJECT_VERSION" )
    @Version
    private Long objectVersion;

    @Column(name = "CREATED_BY")
    @CreatedBy
    private String createdBy;

    @Column(name = "CREATED_DATE")
    @CreatedDate
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createdDate;

    @Column(name = "LAST_UPDATED_BY" )
    @LastModifiedBy
    private String lastUpdatedBy;

    @Column(name = "LAST_UPDATED_DATE" )
    @LastModifiedDate
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date lastUpdatedDate;
}
```



| **注解**           | **作用**                                                  | 常用属性                                                     |
| ------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| @Entity            | 指定当前类是实体类                                        |                                                              |
| @Table             | 指定实体类和表之间的对应关系                              | name：指定数据库表的名称                                     |
| @EntityListeners   | 在实体类增删改的时候监听，为创建人/创建时间等基础字段赋值 | value：指定监听类                                            |
| @Id                | 指定当前字段是主键                                        |                                                              |
| @SequenceGenerator | 指定数据库序列别名                                        | sequenceName：数据库序列名</br> name：取的别名               |
| @GeneratedValue    | 指定主键的生成方式                                        | strategy ：指定主键生成策略 </br> generator：选择主键别名    |
| @Column            | 指定实体类属性和数据库表之间的对应关系                    | name：指定数据库表的**列名称**。 unique：是否唯一  nullable：是否可以为空 nserttable：是否可以插入 updateable：是否可以更新 columnDefinition: 定义建表时创建此列的DDL |
| @CreatedBy         | 自动插入创建人                                            |                                                              |
| @CreatedDate       | 自动插入创建时间                                          |                                                              |
| @LastModifiedBy    | 自动修改更新人                                            |                                                              |
| @LastModifiedDate  | 自动修改更细时间                                          |                                                              |
| @Version           | 自动更新版本号                                            |                                                              |
| @JsonFormat        | 插入/修改/读取的时间转换成想要的格式                      | pattern：展示格式<br/>timezone：国际时间                     |

注意：

有了`@EntityListeners(AuditingEntityListener.class)`这个注解，`@CreatedBy`、`@CreatedDate` 、`@LastModifiedBy` 、`@LastModifiedDate`才生效哦，而且创建人和更新人需要另作注入操作，此篇埋个伏笔。

3.repository 数据访问层
------------------

此处便是整个spring-data-jpa中最令人虎躯一震的地方！

震惊，一个接口居然可以实现常用的所有操作！

JpaUserRepository代码如下：

```java
package org.example.jpademo.repository;

import org.example.jpademo.pojo.entity.JpaUser;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @Classname JpaUserRepository
 * @Description TODO
 * @Date 2020/8/13 15:12
 * @Created by orange
 */
public interface JpaUserRepository extends JpaRepository<JpaUser, Long> {
}
```

可以看到，这个接口继承了`JpaRepository<实体,ID>`，spring-data-jpa只需要这个信息，就可以帮你完成常用的操作：增删查改。

4.Service业务逻辑层
--------------

而且Service层是需要分层的：接口和实现类，这个不必多说，规范！规范！

我们实现最简单的新增、删除、修改、查询功能

接口如下：

JpaUserService

```java
public interface JpaUserService {
    /**
     * 新增用户
     * @param user 用户对象
     */
    JpaUser insertUser(JpaUser user);

    /**
     * 删除用户
     * @param id 删除id
     */
    void deleteUser(Long id);

    /**
     * 修改用户
     * @param user 用户信息
     */
    JpaUser updateUser(JpaUser user);

    /**
     * 查询所有用户
     */
    List<JpaUser> findAllUser();

    /**
     * 通过id查询用户
     * @param id 用户id
     */
    JpaUser findUserById(Long id);
}
```


接口实现：

JpaUserServiceImpl

```java
@Service
public class JpaUserServiceImpl implements JpaUserService {
    @Resource
    private JpaUserRepository jpaUserRepository;

    @Override
    public JpaUser insertUser(JpaUser user) {
        return jpaUserRepository.save(user);
    }

    @Override
    public void deleteUser(Long id) {
        jpaUserRepository.deleteById(id);
    }

    @Override
    public JpaUser updateUser(JpaUser user) {
        return jpaUserRepository.save(user);
    }

    @Override
    public List<JpaUser> findAllUser() {
        return jpaUserRepository.findAll();
    }

    @Override
    public JpaUser findUserById(Long id) {
        return jpaUserRepository.findById(id).orElse(null);
    }
}
```


是滴，你没有看错，一个sql语句也没有见着，直接通过jpaUserRepository接口把方法点了出来。

数据访问层（dao）被确确实实的优化的很简便，这是spring-data-jpa很大的亮点。

> 细心的同学可能发现了，新增和修改都调用的save()方法，jpa靠什么区分是insert还是update呢？
>
> 靠的是主键id有没有赋值判断~id有值为update，id无值为insert。
>

5.Controller控制层
---------------

控制层是前后台交互的层，我采用的是**restful**编写格式的接口，对于资源的具体操作类型，由HTTP动词表示。

简单借用晨瑞大佬文章中的解释：

*   GET（SELECT）：从服务器取出资源（一项或多项）。
*   POST（CREATE）：在服务器新建一个资源。
*   PUT（UPDATE）：在服务器更新完整资源（客户端提供改变后的完整资源）。
*   PATCH（UPDATE）：在服务器更新部分资源（客户端提供改变的属性）。
*   DELETE（DELETE）：从服务器删除资源。

简化一下：

*   GET：查询
*   POST：插入、新建
*   PUT：完全更新
*   PATCH：部分更新
*   DELETE：删除

举个栗子：

*   GET /zoos：获取所有动物园
*   POST /zoos：新建一个动物园
*   GET /zoos/ID：获取此ID的动物园信息
*   PUT /zoos/ID：更新此ID动物园全部信息（提供该动物园的全部信息）
*   PATCH /zoos/ID：更新此ID动物园部分信息（提供该动物园的部分信息）
*   DELETE /zoos/ID：删除此ID的动物园信息
*   GET /zoos/ID/animals：获取此ID动物园的所有动物
*   DELETE /zoos/ID/animals/ID：删除ID(前者)动物园的ID(后者)动物

好，如果你看懂了什么是restful编写格式，那么就看看控制层代码：

JpaUserController

```java
@RestController
@RequestMapping("/user")
public class JpaUserController {
    @Resource
    private JpaUserService jpaUserService;

    /**
     * 新增用户
     */
    @PostMapping("")
    public JpaUser addUser(@RequestBody JpaUser user){
        return jpaUserService.insertUser(user);
    }

    /**
     * 删除用户
     */
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable("id") Long id){
        jpaUserService.deleteUser(id);
    }

    /**
     * 修改用户
     */
    @PutMapping("")
    public JpaUser updateUser(@RequestBody JpaUser user){
        return jpaUserService.updateUser(user);
    }

    /**
     * 全查用户
     */
    @GetMapping("")
    public List<JpaUser> findAll(){
        return jpaUserService.findAllUser();
    }

    /**
     * id查用户
     */
    @GetMapping("/{id}")
    public JpaUser findbyId(@PathVariable("id") Long id){
        return jpaUserService.findUserById(id);
    }
}
```


代码ok，开始测试！

4.测试
==

单单讲spring-data-jpa的话，就没有加swagger注解了，那么测试我们就使用postman来进行

1.用户插入
------

![image-20230218104142775](E:/Development/Typora/images/image-20230218104142775.png)

POST提交

`URL：localhost:2333/user`

body数据：

```json
{
    "name":"orange4"
}
```


返回数据：

```json
{
    "id": 4,
    "name": "orange4",
    "objectVersion": 0,
    "createdBy": null,
    "createdDate": "2020-08-13 16:58:35",
    "lastUpdatedBy": null,
    "lastUpdatedDate": "2020-08-13 16:58:35"
}
```


分析：

id自动通过序列生成，

name是提交的数据，

版本号自动插入为0，

createdBy,lastUpdatedBy由于还未配置完整，暂时没有数据，

createdDate,lastUpdatedDate在插入时皆为当前时间

2.用户删
------

DELETE提交

`URL：localhost:2333/user/4`

返回数据：状态码 200

分析：

状态码200，代表服务器响应正确，删除成功

3.用户查询（全查）
----------

GET提交

> URL：localhost:2333/user
>

返回数据：

```java
[
    {
        "id": 2,
        "name": "banana",
        "objectVersion": 1,
        "createdBy": null,
        "createdDate": "2020-08-13 15:35:44",
        "lastUpdatedBy": null,
        "lastUpdatedDate": "2020-08-13 16:39:55"
    },
    {
        "id": 3,
        "name": "orange2",
        "objectVersion": 0,
        "createdBy": null,
        "createdDate": "2020-08-13 15:36:00",
        "lastUpdatedBy": null,
        "lastUpdatedDate": "2020-08-13 15:36:00"
    }
]
```


分析：

上面插入的id为4的用户此处全查没有，也代表着删除操作的成功

4.用户修改
------

修改的时候需要全部实体数据哦，因为jpa的save()是全部修改，==前端少传一个字段，数据库更新可能就变成null了==，特别注意。之后会讲部分更新的实现。  

PUT提交

`URL：localhost:2333/user`

body数据：

```java
{
    "id": 2,
    "name": "banana-update",
    "objectVersion": 1,
    "createdBy": null,
    "createdDate": "2020-08-13 15:35:44",
    "lastUpdatedBy": null,
    "lastUpdatedDate": "2020-08-13 16:39:55"
}
```


返回数据：

```java
{
    "id": 2,
    "name": "banana-update",
    "objectVersion": 2,
    "createdBy": null,
    "createdDate": "2020-08-13 15:35:44",
    "lastUpdatedBy": null,
    "lastUpdatedDate": "2020-08-13 20:08:18"
}
```


分析：

因为有了id值，save()方法变为了修改方法，

name的值从banana修改成banana-update，

objectVersion版本号因为@Version注解，从1变为了2，

> createdBy和createdDate别看没变，是因为前端传的字段中带了值，如果不传值，数据库会被清成null，切记切记，
>

lastUpdatedBy和lastUpdatedDate不需要管，传不传值都会自动更新。

注意点：

**@Version**注解加上后开启乐观锁，更新必须加上objectVersion字段，且值一定要和数据库中的版本号一致，这样才会触发更新操作。

如果不加objectVersion字段，且后端没有验证操作，id值会被忽略，从更新操作变为新增操作，这是一个坑。

5.用户查询（id查
-----------

GET提交

`URL：localhost:2333/user/2`

返回数据：

```java
{
    "id": 2,
    "name": "banana-update",
    "objectVersion": 2,
    "createdBy": null,
    "createdDate": "2020-08-13 15:35:44",
    "lastUpdatedBy": null,
    "lastUpdatedDate": "2020-08-13 20:08:18"
}
```


分析：

和修改后的数据一样，就是个简单的id查询

5.总结
==

1.`@Version`注解加上后，更新操作一定要带上注解修饰的字段，且要与数据库中的值一致。

2.`@CreatedBy`和`@CreatedDate`会在更新时一并更新，需要主动去维护，或者在`@Column`注解中加上`updatable = false`，比如这样`@Column(name = "CREATED_DATE",updatable = false)`。



填坑1：实体类的主键生成策略详解
===============================================================================

上一节讲到实体类时，介绍了很多注解的作用及其属性，举的例子是`oracle数据库`的实体类。

我们知道，`oracle数据库`的主键不是自增的，而是依靠序列来实现主键增长，要想实现一个表的主键是自增长的，我们首先要新建一个此表的序列，让它从1开始（或者从你想要开始的数字开始），每次调用都让序列＋1，那么也就实现了主键的自增长。

上一节`oracle`的主键自增长的注解是这样的：

1.Oracle自增长主键策略： GenerationType.SEQUENCE
-------------------------------------------------------------------------------------------------------

**使用如下：**

```java
@Id
@Column(name = "ID")
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "JPA_USER_S")
@SequenceGenerator(sequenceName = "JPA_USER_S", name = "JPA_USER_S", allocationSize = 1)
private Long id;

```

**解释一下这几个注解如何理解：**

*   当生成策略为`SEQUENCE`时，`@GeneratedValue`要配合`@SequenceGenerator`使用
    
*   首先`@Id`代表它下方的属性是实体类的主键属性，也就是数据库的主键；
    
*   其次`@Column(name = "ID")`代表此实体类属性对应的数据库的列名是什么，需要进行关系映射；
    
*   再看`@SequenceGenerator(sequenceName = "JPA_USER_S", name = "JPA_USER_S", allocationSize = 1)`，它代表着需要从数据库找到一个序列并在java中映射。
    
    *   `sequenceName`属性值：数据库中的序列名
    *   `name`属性值：这个序列名在java中要映射成的名字
    *   `allocationSize`属性值：这个序列的自增长步长是几
*   最后看`@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "JPA_USER_S")`，它代表着这个主键采取什么样的生成策略。
    
    *   `strategy`属性值：采取的主键策略是什么
    *   `generator`属性值：使用的序列的映射名是什么，这个映射名就是`@SequenceGenerator`中`name`的值

从上到下理解后有没有理顺这几个注解的先后呢？尤其是`@SequenceGenerator`和`@GeneratedValue`的关系。

**至此`oracle`的主键策略注解才算基本讲完，然而并没有结束！**

我们知道以序列作为自增长的数据库有`Oracle、PostgreSQL、DB2`，不过还有一个耳熟能详的数据库`Mysql`是可以不需要序列，直接定义主键自增长的，那么它就需要另一种生成策略。

2.Mysql自增长主键策略：GenerationType.IDENTITY
-----------------------------------------------------------------------------------------------------

**使用如下：**

```
@Id  
@Column(name = "ID")
@GeneratedValue(strategy = GenerationType.IDENTITY)  
private Long id;

```

**解释一下：**

*   当生成策略为`IDENTITY`时，`@GeneratedValue`单独使用
    
*   `@GeneratedValue(strategy = GenerationType.IDENTITY)`，只需要这一个注解就可以实现mysql的主键自增长，我们知道mysql建表的时候可以给主键声明`auto_increment`，这就实现了自增长了，所以注解也相对简单。
    

在`@GeneratedValue`注解中我们只需要生成策略为`IDENTITY`，即可完成mysql数据库的主键自增长。

[](https://blog.csdn.net/qq_42495847/article/details/108801387)3.万能自增长主键策略：GenerationType.TABLE
-----------------------------------------------------------------------------------------------

**使用如下：**

```
    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "sequence_table")
    @TableGenerator(name = "sequence_table",
                    allocationSize = 1, 
                    table = "sequence_table",
                    pkColumnName = "sequence_name",
                    valueColumnName = "sequence_count")
    private Long id;

```

**解释一下：**

*   当生成策略为`TABLE`时，`@GeneratedValue`要配合`@TableGenerator`使用
*   `@TableGenerator(name = "id_sequence", allocationSize = 1, table = "sequence_table",pkColumnName = "sequence_max_id", valueColumnName = "sequence_count")`，它代表着需要在数据库建立一张索引表来帮助我们实现主键自增
    *   `name`属性值：建立的索引表在java中要映射成的名字
    *   `allocationSize`属性值：这个序列的自增长步长是几
    *   `table`属性值：建立的序列表的表名，缺省值：`SEQUENCE`
    *   `pkColumnName`属性值：建立的序列表的第一个列的列名，此列自动填充需要序列作为主键自增长的表的表名，缺省值：`SEQ_NAME`
    *   `valueColumnName`属性值：建立的序列表的第二个列的列名，此列自动填充`pkColumnName`所代表的表的下一个序列值，缺省值：`SEQ_COUNT`
*   `@GeneratedValue(strategy = GenerationType.TABLE, generator = "id_sequence")`，代表着这个主键采取什么样的生成策略，和Oracle中的解释一样
    *   `strategy`属性值：采取的主键策略是什么
    *   `generator`属性值：使用的映射名是什么，这个映射名就是`@SequenceGenerator`中`name`的值

一通解释看下来，可能会有点蒙，用示例来解释会更加直观：

### [](https://blog.csdn.net/qq_42495847/article/details/108801387)TABLE生成策略示例：

**3.1 新建一个部门实体类：JpaDepartment（属性很简单：id、部门名、五个系统字段）**

```
@Data
@Entity
@Table(name = "JPA_DEPARTMENT")
@EntityListeners(AuditingEntityListener.class)
public class JpaDepartment {
    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "sequence_table")
    @TableGenerator(name = "sequence_table",
                    allocationSize = 1, 
                    table = "sequence_table",
                    pkColumnName = "sequence_name",
                    valueColumnName = "sequence_count")
    private Long id;

    @Column(name = "NAME")
    private String name;

    @Column(name = "OBJECT_VERSION" )
    @Version
    private Long objectVersion;

    @Column(name = "CREATED_BY")
    @CreatedBy
    private String createdBy;

    @Column(name = "CREATED_DATE")
    @CreatedDate
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createdDate;

    @Column(name = "LAST_UPDATED_BY" )
    @LastModifiedBy
    private String lastUpdatedBy;

    @Column(name = "LAST_UPDATED_DATE" )
    @LastModifiedDate
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date lastUpdatedDate;
}

```

**3.2 其他的respository、service、controller的代码略过，实现的还是和用户类一样的简单的增删查改；**

这一次我并没有使用sql来新建一张表，仅仅是建立了一个实体类，这时候前面写的yml配置就开始发挥它的作用了，我们对jpa的配置如下：

```
spring:
  jpa:
    hibernate:
      ddl-auto: update #自动更新
    show-sql: true  #日志中显示sql语句

```

**解释一下：**

*   `ddl-auto: create`：启动时删除上一次生成的表，并根据实体类生成表，表中数据会被清空
    
    `ddl-auto: create-drop`：启动时根据实体类生成表，程序关闭时表会被删除
    
    `ddl-auto: update`：启动时会根据实体类生成表，当实体类属性变动的时候，表结构也会更新，在初期开发阶段使用此选项
    
    `ddl-auto: validate`：启动时验证实体类和数据表是否一致，在数据结构稳定时采用此选项
    
    `ddl-auto: none`：不采取任何措施
    
    建议只使用update和none，前者适合初期建表，后者适合建表完成后保护表结构
    
*   `show-sql: true`，这个属性代表是否开启显示sql语句，为`true`我们就可以在每一次对数据库的操作在控制台看到所使用的sql语句了，方便找错，很方便的属性，建议开发时开启，上线后关闭
    

**`ddl-auto: update`时，jpa会根据实体类帮助我们创建表~**

**3.3 有了部门实体类JpaDepartment，让我们启动看一下实际效果：**

**3.3.1 控制台的输出：**

```
Hibernate: create table jpa_department (id number(19,0) not null,
                                        created_by varchar2(255 char),
                                        created_date timestamp, 
                                        last_updated_by varchar2(255 char), 
                                        last_updated_date timestamp, 
                                        name varchar2(255 char), 
                                        object_version number(19,0), 
                                        primary key (id))
                                  
Hibernate: create table sequence_table (sequence_name varchar2(255 char) not null, sequence_count number(19,0), primary key (sequence_name))

Hibernate: insert into sequence_table(sequence_name, sequence_count) values ('jpa_department',0)

```

*   1.创建了一张名为`jpa_department`的表，各个列名皆为实体类属性名，长度取的默认值
    
*   2.创建了一张名为`sequence_table`序列表（`table`属性的值），
    
    *   列名1：`pkColumnName`的值（sequence\_name），代表此行索引值所属表，
    *   列名2：`valueColumnName`的值（sequence\_count），代表下一次插入的索引值（在插入前会提前+1）
*   3.根据`@TableGenerator`所在类映射的表名插入了一行数据，分别为`('jpa_department',0)`，也就是代表`jpa_department`这张表下一次插入的索引值是`1（0+1）`
    

**3.3.2 借助DateBase工具可以看到表全貌：**

![image-20200901220341701](E:/Development/Typora/images/eeeee869ceff07f32f0fb3f690934c98.png)

缺少的表，jpa通过实体类对表的映射补全了。

新增了我们所需的`JPA_DEPARTMENT`表和一张对主键实现自增长的序列表`SEQUENCE_TABLE`

**3.3.3 进行一次插入操作**

postman:

![image-20200818222428281](E:/Development/Typora/images/29bd28227e677fd74865f148dcb19c29.png)

控制台输出：

```
Hibernate: select tbl.sequence_count from sequence_table tbl where tbl.sequence_name=? for update

Hibernate: update sequence_table set sequence_count=?  where sequence_count=? and sequence_max_id=?

Hibernate: insert into jpa_department (created_by, 
                                       created_date,
                                       last_updated_by, 
                                       last_updated_date,
                                       name,
                                       object_version, 
                                       id)
                                       values (?, ?, ?, ?, ?, ?, ?)

```

一次插入拥有三条sql语句

*   1.会等待行锁释放之后，返回`sequence_table`表的查询结果，返回`jpa_department`的当前索引值
*   2.更新操作，把`sequence_table`表中的`sequence_count`的值+1
*   3.把更新后的`sequence_count`值当成主键值插入到`jpa_department`表中作为主键值

在这三条sql语句执行后，显然`jpa_department`这一次插入的主键值为1，postman返回的json数据也证实了这一点

[](https://blog.csdn.net/qq_42495847/article/details/108801387)4.AUTO自动判断主键策略（缺省策略）：
------------------------------------------------------------------------------------

**使用如下：**

```
    @Id
    @Column(name = "ID")
	@GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

```

**解释一下：**

*   当生成策略为`AUTO`时，`@GeneratedValue`单独使用
    
*   `@GeneratedValue(strategy = GenerationType.AUTO)`，JPA会自己从从 `Table 策略`，`Sequence 策略`和 `Identity 策略`三种策略中选择合适的主键生成策略；
    
    *   `strategy`属性值：主键生成策略，什么都不写即缺省值即为`AUTO`

### [](https://blog.csdn.net/qq_42495847/article/details/108801387)AUTO生成策略示例：

**4.1 仍然使用的是刚刚的部门类JPA\_DEPARTMENT，把已创建的表删除，修改生成策略为auto**

```
@Data
@Entity
@Table(name = "JPA_DEPARTMENT")
@EntityListeners(AuditingEntityListener.class)
public class JpaDepartment {
    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(name = "NAME")
    private String name;

    @Column(name = "OBJECT_VERSION" )
    @Version
    private Long objectVersion;

    @Column(name = "CREATED_BY")
    @CreatedBy
    private String createdBy;

    @Column(name = "CREATED_DATE")
    @CreatedDate
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createdDate;

    @Column(name = "LAST_UPDATED_BY" )
    @LastModifiedBy
    private String lastUpdatedBy;

    @Column(name = "LAST_UPDATED_DATE" )
    @LastModifiedDate
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date lastUpdatedDate;
}

```

**4.2 运行程序看一下jpa自动帮助我们做了什么**

```
Hibernate: create table jpa_department (id number(19,0) not null, 
                                        created_by varchar2(255 char), 
                                        created_date timestamp, 
                                        last_updated_by varchar2(255 char), 
                                        last_updated_date timestamp, 
                                        name varchar2(255 char),
                                        object_version number(19,0), 
                                        primary key (id))
Hibernate: create sequence hibernate_sequence start with 1 increment by  1

```

*   创建了`jpa_department`这张表
*   创建了一个名为`hibernate_sequence`的序列

显然jpa识别到了我连接的数据库是oracle数据库，所以采取的是SEQUENCE生成策略，它帮助我们建立了一个名为`hibernate_sequence`的序列，但是这个名字显然是无法一眼看出来是属于谁的序列的，不方便对序列的维护，所以不推荐使用`auto`策略哦

[](https://blog.csdn.net/qq_42495847/article/details/108801387)5.总结-主键策略：
-------------------------------------------------------------------------

1.主键生成策略分为四种：`SEQUENCE`策略、`IDENTITY`策略、`TABLE`策略、`AUTO`策略；

2.`SEQUENCE`策略适合拥有序列的数据库，比如Oracle；

3.`IDENTITY`策略适合拥有主键自增长的数据库，比如Mysql；

4.`TABLE`策略是通过一张序列表来维护主键插入的值的，所以适合所有数据库；

5.`AUTO`策略是jpa自行判断使用上面三个中的哪一个作为主键生成策略；

6.推荐使用`SEQUENCE`和`IDENTITY`策略，开发人员应该自行判断使用的是何种数据库，而不是由jpa进行判断。

[](https://blog.csdn.net/qq_42495847/article/details/108801387)填坑2：系统字段：注入创建人和更新人
=================================================================================

[](https://blog.csdn.net/qq_42495847/article/details/108801387)1.复习
-------------------------------------------------------------------

**1.1 上一节讲到修饰五个系统字段的注解：**

*   `@Version`：版本号；进行update操作时启动乐观锁，@Version修饰的字段值与数据库中字段值一致才能进行修改
*   `@CreatedDate` ：创建时间；进行insert操作时，将当前时间插入到@CreatedDate修饰字段中；进行update操作时，会随实体类中的@CreatedDate修饰的字段值进行修改
*   `@CreatedBy`：创建人；进行insert操作时，将当前用户名插入到@CreatedBy修饰字段中；进行update操作时，会随实体类中的@CreatedBy修饰的字段值进行修改
*   `@LastModifiedDate`：最后一次修改时间；进行update操作时，将当前时间修改进@LastModifiedDate修饰字段中；进行insert操作时，将当前时间插入到@LastModifiedDate修饰字段中
*   `@LastModifiedBy` ：最后一次修改的修改人；进行update操作时，将当前修改人修改进@LastModifiedBy修饰的字段中；进行insert操作时，将当前用户名插入到@LastModifiedBy修饰字段中

**1.2 启动审计：**

1.2.1 在Springboot启动类上加上启动审计注解：`@EnableJpaAuditing`

```
@EnableJpaAuditing
@SpringBootApplication
public class SpringContextApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringContextApplication.class, args);
    }
}

```

1.2.2 在实体类上方加上监听注解：`@EntityListeners(AuditingEntityListener.class)`

```
@Data
@Entity
@Table(name = "JPA_USER")
@EntityListeners(AuditingEntityListener.class)
public class JpaUser {
    //...
}

```

*   `@EntityListeners`该注解用于指定Entity或者superclass上的回调监听类；
*   `AuditingEntityListener`这个类是一个JPA Entity Listener，用于捕获监听信息，当Entity发生新增和更新操作时进行捕获。

当设置完这两个注解后@CreatedDate、@LastModifiedBy这两个注解对于创建时间和修改时间的注入就ok了，但是对创建人、修改人的注入却为null，毕竟jpa并不知道当前是谁在操作数据，需要我们来进行提供；

[](https://blog.csdn.net/qq_42495847/article/details/108801387)2.注入创建人和更新人
--------------------------------------------------------------------------

那么jpa从哪里得到它想要注入的创建人或者更新人信息呢？

我们需要一个配置类，并且实现`AuditorAware<String>`接口

### [](https://blog.csdn.net/qq_42495847/article/details/108801387)2.1 测试

**2.1.1 建立配置文件夹config，创建配置类`UserAuditor`**

![image-20200924131640363](E:/Development/Typora/images/8555d19cbcfb9966057595d6f56a1cb2.png)

```
@Configuration
public class UserAuditor implements AuditorAware<String> {
    /**
     * 获取当前创建或修改的用户
     *
     * @return 获取当前创建或修改的用户Uid
     */
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.of("俺是测试创建者");
    }
}

```

很容易理解的一个配置，它返回一个`Optional<String>`对象，对象内的String值便是创建人和更新人根据实际情况去注入的值。

> 关于`Optional<?>`，不了解的同学可以去度娘，java8很好的一个防止空指针的类

**2.1.2 去postman调用新增用户的接口**

![image-20200924132329729](E:/Development/Typora/images/529bbc6472c527aaf27b0f31ba5b6783.png)

分析：

可以看到，createdBy和lastUpdatedBy都进行了自动注入，全部变成了在`UserAuditor`中设置的返回值

**2.1.3 调用更新用户接口**

我们手动将`UserAuditor`中的返回值改变，重启应用再试试更新用户接口

```
@Configuration
public class UserAuditor implements AuditorAware<String> {
    /**
     * 获取当前创建或修改的用户
     *
     * @return 获取当前创建或修改的用户Uid
     */
    @Override
    public Optional<String> getCurrentAuditor() {
        return Optional.of("俺是测试更新者");
    }
}

```

更新接口调用：

![image-20200924133030604](E:/Development/Typora/images/b4b8c284bae0f6901d47533dc75b0149.png)

分析：

可以看到，我只将`name`的值改变为orange1-update，其他不变，调用接口后`name`、`objectVersion`、`lastUpdatedBy`、`lastUpdatedDate`的值被改变了，系统字段的改变很符合我们的需求；

更新时对更新者的注入也是从`UserAuditor`中拿取的，jpa自动判断是不是更新操作，是就把当前`UserAuditor`获取到的返回值注入更新人字段中；

### [](https://blog.csdn.net/qq_42495847/article/details/108801387)2.2 session保存用户信息并进行审计注入

**2.2.1 具体思路**

正式项目当然不像测试那样可以把`UserAuditor`的返回值给改了再重启项目，我们需要实时获取到用户的uid并注入到创建人和更新人字段中，这时候我们就需要一个东西帮助我们认知当前用户是谁，而http协议是无状态的，不能像服务器一样一直保存一个东西，于是出来了很多对于用户认证的解决方案，比如session，cookie，token等；

我们这里使用最简单session存储用户登录信息，当用户登录完后，进行新增和修改操作时，再从seesion中获取到用户数据，将其通过`UserAuditor`类来返回给jpa的审计功能

**2.2.2 实现登录功能**

登录需要什么呢？账号和密码，这里我们再简单一点，不需要密码，只要输对了用户名就给这个用户一个session。（小声嘀咕：毕竟我数据库设计忘了密码字段了）

2.2.2.1 在JpaUserController添加

```
    /**
     * 登录
     * @param session 拿到session
     * @param jpaUser 前端传来的数据：有效字段只有name
     * @return 有此用户名返回ok，没有则error
     */
    @PostMapping("/login")
    public String login(HttpSession session,@RequestBody JpaUser jpaUser){
        return jpaUserService.login(session,jpaUser);
    }

```

2.2.2.2 JpaUserServiceImpl实现逻辑：从JpaUser表根据name查找用户，有就生产session，返回ok；没有就不生成session，返回error

```
    @Override
    public String login(HttpSession session, JpaUser jpaUser) {
        // 查询
        JpaUser user = jpaUserRepository.findByName(jpaUser.getName());
        if (user !=null){
            session.setAttribute("jpaUser",jpaUser);
            return "ok";
        }else {
            return "error";
        }
    }

```

2.2.2.3 jpaUserRepository.findByName()实现：jpa的单表查询很香，此篇不细讲，自己体会

```
public interface JpaUserRepository extends JpaRepository<JpaUser, Long> {
    // 根据name查询用户，无需sql
    JpaUser findByName(@Param("name") String name);
}

```

**2.2.3 修改`UserAuditor`逻辑**

```
@Configuration
public class UserAuditor implements AuditorAware<String> {
    /**
     * 获取当前创建或修改的用户
     *
     * @return 获取当前创建或修改的用户Uid
     */
    @Override
    public Optional<String> getCurrentAuditor() {
        // request声明
        HttpServletRequest request;
        // 工号
        String username = "anonymous";
        // 拿到Session
        ServletRequestAttributes requestAttributes= (ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
        if (requestAttributes !=null){
            request = requestAttributes.getRequest();
            HttpSession session = request.getSession();
            Object obj = session.getAttribute("jpaUser");
            if (obj instanceof JpaUser){
                JpaUser jpaUser = (JpaUser) obj;
                username = jpaUser.getName();
            }
        }
        return Optional.of(username);
    }
}

```

session大家应该耳熟能详，但一般我们是在controller层拿到request，再拿session，或者直接在controller层拿到session，在其他非controller类中如何拿session呢，使用`RequestContextHolder`上下文容器

我具体解析一下代码：

*   声明一个`HttpServletRequest`对象request，声明一个username字符串，默认值为anonymous（匿名）
*   通过`RequestContextHolder`拿到request的属性，转换成可以得到request对象的`ServletRequestAttributes`
*   判断是否当前线程的请求属性为空，为空则直接返回username默认值，不为空继续
*   不为空就可以拿到我们熟悉的request对象啦，它里面存着session信息
*   用Object类接收约定好的key值(我这里叫jpaUser)，如果存储了session，那么拿到的对象类型一定是`JpaUser`，进行强转后拿到存储在其中的name值
*   再返回拥有真实用户名的username，接下来就交给jpa审计去注入了

就是酱紫~

**2.2.4 升级后的测试**

由于没有前端页面，我懒的写了哈哈，用postman模拟数据的同时还要模拟cookie信息，这样才能让程序知道你是哪个session

2.2.4.1 去数据库看一眼有哪些用户

![image-20200925151622462](E:/Development/Typora/images/aadbd31d9948da84f48e74cfa5197861.png)

2.2.4.2 就用orange2来登录

![image-20200925151745493](E:/Development/Typora/images/f6d1edcc650460d52fdc999f6091a616.png)

2.2.4.3 返回ok代表登录成功，那么服务器此时已经存储了orange2的session信息了，我们需要让服务器知道我们是这个session的拥有者，那么查看这次返回的headers信息

![image-20200925151954578](E:/Development/Typora/images/b33bed330b1c868593f11bde140a5396.png)

2.2.4.4 可以看到cookie附带了`JSESSIONID=2E9DF14E6308D8D6C0EA759FAE587436; Path=/; HttpOnly`这一串信息，`JSESSIONID`表示你这个用户的sessionId，它就是服务器如何知道你是谁的原因，浏览器会自动带上这个cookie去访问服务器，服务器会自动解析，拿到属于你的那个session；

知道了这点，我们就可以模拟浏览器的做法，把JSESSIONID的键值放到cookie就可以模拟了

2.2.4.5 点击右上角的cookies

![image-20200925152452111](E:/Development/Typora/images/7642bca92cfcf9777c937ec0c495f723.png)

2.2.4.6 添加域名信息：本地自然是localhost

![image-20200925153013538](E:/Development/Typora/images/f6e24c06ad551498cd525fa29574ca43.png)

2.2.4.7 在其下添加cookie，只需要改变第一个键值就行

![image-20200925153211555](E:/Development/Typora/images/72ec2503c1cf22988d93d9f99f64e5f1.png)

![image-20200925153138854](E:/Development/Typora/images/d80ef38d1c5349a26e1cd9e2a82a9497.png)

配置一次后，以后会自动设置`JSESSIONID`的值，一劳永逸

2.2.4.8 使用用户插入接口

![image-20200925153542716](E:/Development/Typora/images/1c5bec04478c3e991fe54a2bf7a2f860.png)

可以看到创建者和修改者都是orange2，证明程序正确，至此Jpa审计内容大致结束，这里仅使用了session来存储用户信息，之后可能会对sping-security进行详细讲解（挖坑挖坑！）

[](https://blog.csdn.net/qq_42495847/article/details/108801387)3.总结-注入创建人和更新人:
------------------------------------------------------------------------------

1.通过对配置类`UserAuditor(继承了AuditorAware<String>)`进行返回值的逻辑加工，就可以实现jpa对5个系统字段的审计注入，非常nice

2.通过session、token、cookie等方式可以保存用户的信息，配合jpa审计功能达到自动注入的效果，非常nice

鸽鸽复鸽鸽，鸽子何其多~

 

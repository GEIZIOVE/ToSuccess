# Easy-ES

# 1.快速入门

## 避坑指南

### ES版本及SpringBoot版本

​		由于我们底层用了ES官方的`RestHighLevelClient`,所以对ES版本有要求,要求ES和RestHighLevelClient JAR依赖版本必须为7.14.0,至于es客户端,实际==测下来7.X任意版本都可以很好的兼容.==

​		值得注意的是,由于`Springdata-ElasticSearch`的存在,Springboot它内置了和ES及RestHighLevelClient依赖版本,这导致了不同版本的Springboot实际引入的ES及RestHighLevelClient 版本不同,而ES官方的这两个依赖在不同版本间的兼容性非常差,进一步导致很多用户无法正常使用Easy-Es,抱怨我们框架有缺陷,实际上这是一个依赖冲突的问题. 我们在项目启动时做了依赖校验,==如果您的项目在启动时可以在控制台看到打印出级别为**`Error`**且内容为`"Easy-Es supported elasticsearch and restHighLevelClient jar version is:7.14.0 ,Please resolve the dependency conflict!"` 的日志时,则说明有依赖冲突待您解决.==

​		解决方案其实很简单,可以像下面一样配置maven的exclude移除Springboot或Easy-Es已经声明的ES及RestHighLevelClient依赖,然后重新引入,引入时指定版本号为7.14.0即可解决.

```xml
        <dependency>
            <groupId>cn.easy-es</groupId>
            <artifactId>easy-es-boot-starter</artifactId>
            <version>1.1.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch.client</groupId>
                    <artifactId>elasticsearch-rest-high-level-client</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.elasticsearch</groupId>
                    <artifactId>elasticsearch</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.14.0</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.14.0</version>
        </dependency>
```

​		如果你是个菜的抠脚的懒汉,也可以简单粗暴的把springboot版本调整到2.5.5,其它都不需要调整,也可以勉强正常使用.

### ES索引的keyword类型和text类型以及termQuery,match,match_phrase区别

![compare](https://iknow.hs.net/6b9f24cf-7eb9-43ac-9b65-86c3b759cd69.png)

​		ES中的`keyword`类型,和MySQL中的字段基本上差不多,当我们需要对查询字段进行**精确匹配,左模糊,右模糊,全模糊,排序聚合等操作时**,需要该字段的索引类型为keyword类型,否则你会发现查询没有查出想要的结果,甚至报错. 比如EE中常用的API `eq(),like(),distinct()`等都需要字段类型为keyword类型.

​		当我们需要对字段进行**分词查询**时,需要该字段的类型为`text`类型,并且指定分词器(不指定就用ES默认分词器,效果通常不理想). 比如EE中常用的API `match()`等都需要字段类型为text类型. 

> 当使用match查询时未查询到预期结果时,可以先检查索引类型,然后再检查分词器,因为如果一个词没被分词器分出来,那结果也是查询不出来的.

​		当同一个字段,我们既需要把它当keyword类型使用,又需要把它当text类型使用时,此时我们的索引类型为`keyword_text`类型,EE中可以对字段S添加注解`@TableField(fieldType = FieldType.KEYWORD_TEXT)`,如此该字段就会被创建为keyword+text双类型如下图所示,值得注意的是,当我们把该字段当做keyword类型查询时,ES要求传入的字段名称为"字段名.keyword",当把该字段当text类型查询时,直接使用原字段名即可.

![image2](E:/Development/Typora/images/72818af6-7cc3-4833-b7a7-dbff845ce73e.png)

​		另一种做法是,可以冗余一个字段,值用相同的,一个注解标记为keyword类型,另一个标记为text类型,查询时按规则选择对应字段进行查询.

​		还需要注意的是,如果一个字段的索引类型被创建为仅为keyword类型(如下图所示)查询时,则不需要在其名称后面追加.keyword,直接查询就行.

![image3](https://iknow.hs.net/87335e55-1fe3-44ed-920b-61354383e85a.png)

### 字段id

​		由于框架很多功能都是借助id实现的,比如selectById,update,deleteById...,而且ES中也必须有一列作为数据id,因此我们强制要求用户封装的实体类中包含字段id列,否则框架不少功能无法正常使用.

```java
public class Document {
    /**
     * es中的唯一id,如果你想自定义es中的id为你提供的id,比如MySQL中的id,请将注解中的type指定为customize或直接在全局配置文件中指定,如此id便支持任意数据类型)
     */
    @TableId(type = IdType.CUSTOMIZE)
    private String id;
}
```

> 如果不添加@TableId注解或者添加了注解但未指定type,则id默认为es自动生成的id.
>

​		==在调用insert方法时,如果该id数据在es中不存在,则新增该数据,如果已有该id数据,则即便你调用的是insert方法,实际上的效果也是更新该id对应的数据,这点需要区别于MP和MySQL.==

### 项目中同时使用Mybatis-Plus和Easy-Es

在此场景下,您需要将MP的mapper和EE的mapper分别放在不同的目录下,并在配置扫描路径时各自配各自的扫描路径,如此便可共存使用了,否则两者在SpringBoot启动时都去扫描同一路径,并尝试注册为自己的bean,由于底层实现依赖的类完全不一样,所以会导致其中之一注册失败,整个项目无法正常启动.可参考下图:

![image4](https://iknow.hs.net/30f08bc4-cb07-4ac6-8a52-59e062105238.png)

![image5](https://iknow.hs.net/1b5806d4-6c5b-48e6-a025-7746f89f0f6a.png)

### and和or的使用

需要区别于MySQL和MP,因为ES的查询参数是树形数据结构,和MySQL平铺的不一样,具体可参考[条件构造器-and&or](https://www.easy-es.cn/pages/1cebb8/)章节,有详细节省



## 快速开始

### 初始化工程

创建一个空的 Spring Boot 工程

> 可以使用 [Spring Initializer (opens new window)](https://start.spring.io/)快速初始化一个 Spring Boot 工程

### 添加依赖

**Maven:**

```xml
        <dependency>
            <groupId>cn.easy-es</groupId>
            <artifactId>easy-es-boot-starter</artifactId>
            <version>Latest Version</version>
        </dependency>
```

![image-20230226165958887](E:/Development/Typora/images/image-20230226165958887.png)

### 配置

在 application.yml 配置文件中添加EasyEs必须的相关配置：

```yaml
easy-es:
  enable: true #默认为true,若为false则认为不启用本框架
  address : 127.0.0.1:9200 # es的连接地址,必须含端口 若为集群,则可以用逗号隔开 例如:127.0.0.1:9200,127.0.0.2:9200
  username: elastic #若无 则可省略此行配置
  password: WG7WVmuNMtM4GwNYkyWH #若无 则可省略此行配置
```

其它配置暂可省略,后面有章节详细介绍EasyEs的配置

在 Spring Boot 启动类中添加 @EsMapperScan 注解，扫描 Mapper 文件夹：

```java
@SpringBootApplication
@EsMapperScan("com.xpc.easyes.sample.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 背景

现有一张Document文档表，随着数据量膨胀,其查询效率已经无法满足产品需求,其表结构如下,我们打算将此表内容迁移至Es搜索引擎,提高查询效率

| id   | title | content |
| ---- | ----- | ------- |
| 主键 | 标题  | 内容    |

### 编码

编写实体类Document.java（此处使用了 [Lombok (opens new window)](https://www.projectlombok.org/)简化代码）

```java
@Data
public class Document {
    /**
     * es中的唯一id
     */	
    private String id;
    /**
     * 文档标题
     */
    private String title;
    /**
     * 文档内容
     */
    private String content;
}
```



温馨提示

- **上面字段名称以及下划线转自动驼峰**,字段在ES中的存储类型,分词器等均可配置,在后续章节会有介绍.
- **String类型默认会被EE创建为keyword类型,keyword类型支持精确查询等**
- **如需分词查询,可像上面content一样,在字段上加上@TableField注解并指明字段类型为text,并指定分词器.**

编写Mapper类 DocumentMapper.java

```java
public interface DocumentMapper extends BaseEsMapper<Document> {
}
```

#### 前置操作

> 启动项目,由Easy-Es自动帮您创建索引(相当于MySQL等数据库中的表),有了索引才能进行后续CRUD操作.索引托管成功后,您可在控制台看到:===> Congratulations auto process index by Easy-Es is done !

温馨提示

- 后续如若索引有更新,索引重建,更新,数据迁移等工作默认都由EE自动帮您完成,当然您也可以通过配置关闭索引自动托管,可通过EE提供的API手动维护或es-head等插件维护.
- 自动托管模式(0.9.9+版本支持),相关配置及详细介绍可在后面章节中看到,此处您只管将这些烦人的步骤交给EE去自动处理即可.
- 若您EE版本低于该版本,可通过EE提供的API手动维护索引

### 开始使用(CRUD)

添加测试类，进行功能测试：

> 测试新增: 新增一条数据(相当于MySQL中的Insert操作)

```java
    @Test
    public void testInsert() {
        // 测试插入数据
        Document document = new Document();
        document.setTitle("老汉");
        document.setContent("推*技术过硬");
        int successCount = documentMapper.insert(document);
        System.out.println(successCount);
    }
```

> 测试查询:根据条件查询指定数据(相当于MySQL中的Select操作)

```java
    @Test
    public void testSelect() {
        // 测试查询
        String title = "老汉";
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.eq(Document::getTitle,title);
        Document document = documentMapper.selectOne(wrapper);
        System.out.println(document);
        Assert.assertEquals(title,document.getTitle());
    }
```

> 测试更新:更新数据(相当于MySQL中的Update操作)

```java
    @Test
    public void testUpdate() {
        // 测试更新 更新有两种情况 分别演示如下:
        // case1: 已知id, 根据id更新 (为了演示方便,此id是从上一步查询中复制过来的,实际业务可以自行查询)
        String id = "krkvN30BUP1SGucenZQ9";
        String title1 = "隔壁老王";
        Document document1 = new Document();
        document1.setId(id);
        document1.setTitle(title1);
        documentMapper.updateById(document1);

        // case2: id未知, 根据条件更新
        LambdaEsUpdateWrapper<Document> wrapper = new LambdaEsUpdateWrapper<>();
        wrapper.eq(Document::getTitle,title1);
        Document document2 = new Document();
        document2.setTitle("隔壁老李");
        document2.setContent("推*技术过软");
        documentMapper.update(document2,wrapper);

        // 关于case2 还有另一种省略实体的简单写法,这里不演示,后面章节有介绍,语法与MP一致
    }
```



经过一顿猛如虎的更新操作 我们来看看标题最终变成了什么?

![image.png](E:/Development/Typora/images/bdb9bbeb-70e2-46ac-9229-3a36f1001987.png)

查询结果证实了我们更新没有问题,这里无意冒犯"老李",仅供演示,如有得罪,请海涵.

> 测试删除:删除数据(相当于MySQL中的Delete操作)

```java
    @Test
    public void testDelete() {
        // 测试删除数据 删除有两种情况:根据id删或根据条件删
        // 鉴于根据id删过于简单,这里仅演示根据条件删,以老李的名义删,让老李心理平衡些
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        String title = "隔壁老李";
        wrapper.eq(Document::getTitle,title);
        int successCount = documentMapper.delete(wrapper);
        System.out.println(successCount);
    }
```



上面完整的代码示例请移步：[Easy-Es-Sample(opens new window)](https://gitee.com/dromara/easy-es/tree/master/easy-es-test/src/test/java/cn/easyes/test)



## 配置

### **基础配置:**

如果缺失可导致项目无法正常启动,其中账号密码可缺省.

```yaml
easy-es:
  enable: true # 是否开启EE自动配置
  address : 127.0.0.1:9200 # es连接地址+端口 格式必须为ip:port,如果是集群则可用逗号隔开
  schema: http # 默认为http
  username: elastic #如果无账号密码则可不配置此行
  password: WG7WVmuNMtM4GwNYkyWH #如果无账号密码则可不配置此行
```

### **拓展配置:**

可缺省,不影响项目启动,为了提高生产环境性能,建议您按需配置

> 特别注意
>
> 如果您开启了索引托管-平滑模式(默认开启),并且您需要迁移的数据量很大,建议调大socketTimeout,否则迁移会超时异常 单位是毫秒,我们经过测试发现迁移1万条数据大约需要5秒左右,当然该数值需要综合考虑您的服务器硬件负载等因素,因此建议 您按需配置,尽量给大不给小,跟那玩意一样,大点没事,太小你懂的!

```yaml
easy-es:
  keep-alive-millis: 18000 # 心跳策略时间 单位:ms
  connect-timeout: 5000 # 连接超时时间 单位:ms
  socket-timeout: 5000 # 通信超时时间 单位:ms 
  request-timeout: 5000 # 请求超时时间 单位:ms
  connection-request-timeout: 5000 # 连接请求超时时间 单位:ms
  max-conn-total: 100 # 最大连接数 单位:个
  max-conn-per-route: 100 # 最大连接路由数 单位:个
```

### **全局配置:**

可缺省,不影响项目启动,若缺省则为默认值

```yaml
easy-es:
  banner: false # 默认为true 打印banner 若您不期望打印banner,可配置为false
  global-config:
    process-index-mode: smoothly #索引处理模式,smoothly:平滑模式,默认开启此模式, not_smoothly:非平滑模式, manual:手动模式
    print-dsl: true # 开启控制台打印通过本框架生成的DSL语句,默认为开启,测试稳定后的生产环境建议关闭,以提升少量性能
    distributed: false # 当前项目是否分布式项目,默认为true,在非手动托管索引模式下,若为分布式项目则会获取分布式锁,非分布式项目只需synchronized锁.
    async-process-index-blocking: true # 异步处理索引是否阻塞主线程 默认阻塞 数据量过大时调整为非阻塞异步进行 项目启动更快
    active-release-index-max-retry: 60 # 分布式环境下,平滑模式,当前客户端激活最新索引最大重试次数若数据量过大,重建索引数据迁移时间超过60*(180/60)=180分钟时,可调大此参数值,此参数值决定最大重试次数,超出此次数后仍未成功,则终止重试并记录异常日志
    active-release-index-fixed-delay: 180 # 分布式环境下,平滑模式,当前客户端激活最新索引最大重试次数 若数据量过大,重建索引数据迁移时间超过60*(180/60)=180分钟时,可调大此参数值 此参数值决定多久重试一次 单位:秒
    
    db-config:
      map-underscore-to-camel-case: false # 是否开启下划线转驼峰 默认为false
      table-prefix: daily_ # 索引前缀,可用于区分环境  默认为空 用法和MP一样
      id-type: customize # id生成策略 customize为自定义,id值由用户生成,比如取MySQL中的数据id,如缺省此项配置,则id默认策略为es自动生成
      field-strategy: not_empty # 字段更新策略 默认为not_null
      enable-track-total-hits: true # 默认开启,查询若指定了size超过1w条时也会自动开启,开启后查询所有匹配数据,若不开启,会导致无法获取数据总条数,其它功能不受影响.
      refresh-policy: immediate # 数据刷新策略,默认为不刷新
      enable-must2-filter: false # 是否全局开启must查询类型转换为filter查询类型 默认为false不转换
      batch-update-threshold: 10000 # 批量更新阈值 默认值为1万
```

### **其它配置:**

```yaml
logging:
  level:
   tracer: trace # 开启trace级别日志,在开发时可以开启此配置,则控制台可以打印es全部请求信息及DSL语句,为了避免重复,开启此项配置后,可以将EE的print-dsl设置为false.
```



> 温馨提示
>
> - id-type支持3种类型:
>   - auto: 由ES自动生成,是默认的配置,无需您额外配置 推荐
>   - uuid: 系统生成UUID,然后插入ES (不推荐)
>   - customize: 用户自定义,在此类型下,用户可以将任意数据类型的id存入es作为es中的数据id,比如将mysql自增的id作为es的id,可以开启此模式,或通过@TableId(type)注解指定.
> - field-strategy支持3种类型:
>   - not_null: 非Null判断,字段值为非Null时,才会被更新
>   - not_empty: 非空判断,字段值为非空字符串时才会被更新
>   - ignore: 忽略判断,无论字段值为什么,都会被更新
>   - 在配置了全局策略后,您仍可以通过注解针对个别类进行个性化配置,全局配置的优先级是小于注解配置的
> - refresh-policy支持3种策略
>   - none: 默认策略,不刷新数据
>   - immediate : 立即刷新,会损耗较多性能,对数据实时性要求高的场景下适用
>   - wait_until: 请求提交数据后，等待数据完成刷新(1s)，再结束请求 性能损耗适中



## 注解

### @EsMapperScan

- 描述：mapper扫描注解,功能与MP的@MapperScan一致
- 使用位置：Springboot启动类

```java
@EsMapperScan("cn.easy-es-mapper")
public class Application{
    // 省略其它...
}
```

| 属性  | 类型   | 必须指定 | 默认值 | 描述                     |
| ----- | ------ | -------- | ------ | ------------------------ |
| value | String | 是       | ""     | 自定义mapper所在包全路径 |

> 温馨提示
>
> 由于EE和MP对Mapper的扫描都是采用Springboot的doScan，而且两套系统互相独立，所以在扫描的时候没有办法互相隔离,因此如果您的项目同时有用到EE和MP，您需要将EE的Mapper和MP的Mapper放在不同的包下，否则项目将无法正常启动。

### [@IndexName](https://gitee.com/dromara/easy-es/blob/master/easy-es-annotation/src/main/java/cn/easyes/annotation/IndexName.java)

- 描述：索引名注解，标识实体类对应的索引对应MP的@TableName注解,在v0.9.40之前此注解为@TableName.
- 使用位置：实体类

```java
@IndexName
public class Document {
    // 省略其它字段
}
```



| 属性             | 类型    | 必须指定 | 默认值                  | 描述                                                         |
| ---------------- | ------- | -------- | ----------------------- | ------------------------------------------------------------ |
| value            | String  | 否       | ""                      | 索引名，可简单理解为MySQL表名                                |
| shardsNum        | int     | 否       | 1                       | 索引分片数                                                   |
| replicasNum      | int     | 否       | 1                       | 索引副本数                                                   |
| aliasName        | String  | 否       | ""                      | 索引别名                                                     |
| keepGlobalPrefix | boolean | 否       | false                   | 是否保持使用全局的 tablePrefix 的值，与MP用法一致            |
| child            | boolean | 否       | false                   | 是否子文档                                                   |
| childClass       | Class   | 否       | DefaultChildClass.class | 父子文档-子文档类                                            |
| maxResultWindow  | int     | 否       | 10000                   | 分页返回的最大数据量,默认值为1万条,超出推荐使用searchAfter或滚动查询等方式,详见拓展功能章节 |



**动态索引名称支持** 如果你的索引名称是不固定的,我们提供了两种方式可修改CRUD时的索引名称

- 调用mapper.setCurrentActiveIndex(String indexName)方法,此处的mapper为你自定义的mapper,如documentMapper,通过此API修改索引名称后,全局生效.
- 在对应的参数中指定当前操作作用的索引,例如 wrapper.index(String indexName),通过此API修改索引名称后,仅作用于该wrapper对应的操作,粒度最细.

> 温馨提示
>
> - 当您想直接把类名当作索引名，且并不需要对索引进行其它配置时，可省略此注解
> - 通过注解指定的索引名称优先级最高,指定了注解索引,则全局配置和自动生成索引不生效,采用注解中指定的索引名称. 优先级排序: 注解索引>全局配置索引前缀>自动生成
> - keepGlobalPrefix选项,(0.9.4+版本才支持)默认值为false,是否保持使用全局的 tablePrefix 的值:
>   - 既配置了全局tablePrefix,@TableName注解又指定了value值时,此注解选项才会生效,如果其值为true,则框架最终使用的索引名称为:全局tablePrefix+此注解的value,例如:dev_document.
>   - 此注解选项用法和MP中保持一致.
> - 其中shardNum为分片数,replicasNum为副本数,如果不指定,默认值均为1

### [@IndexId](https://gitee.com/dromara/easy-es/blob/master/easy-es-annotation/src/main/java/cn/easyes/annotation/IndexId.java)

- 描述：==ES主键注解==
- 使用位置：实体类中被作为ES主键的字段, 对应MP的@TableId注解

```java
public class Document {
    @IndexId
    private String id;
    // 省略其它字段
}
```



| 属性  | 类型   | 必须指定 | 默认值      | 描述         |
| ----- | ------ | -------- | ----------- | ------------ |
| value | String | 否       | "_id"       | 主键字段名   |
| type  | Enum   | 否       | IdType.NONE | 指定主键类型 |



> 温馨提示
>
> - **当您字段命名为id且类型为String时，且不需要采用UUID及自定义ID类型时，可省略此注解**
> - **由于es对id的默认名称做了处理(下划线+id):_id,所以EE已为您屏蔽这步操作,您无需在注解中指定,框架也会自动帮您完成映射.**
> - Id的生成类型支持以下几种:
>   - **IdType.NONE:** 由ES自动生成,是默认缺省时的配置,无需您额外配置 推荐
>   - **IdType.UUID:** 系统生成UUID,然后插入ES (不推荐)
>   - **IdType.CUSTOMIZE:** 由用户自定义,用户自己对id值进行set,如果用户指定的id在es中不存在,则在insert时就会新增一条记录,如果用户指定的id在es中已存在记录,则自动更新该id对应的记录.
>
> **优先级:** 注解配置的Id生成策略>全局配置的Id生成策略

### [@IndexField](https://gitee.com/dromara/easy-es/blob/master/easy-es-annotation/src/main/java/cn/easyes/annotation/IndexField.java)

- 描述：==ES字段注解==
- 使用位置：**实体类中被作为ES索引字段的字段**
- 使用场景举例：

1. 实体类中的字段并非ES中实际的字段,比如把实体类直接当DTO用了,加了一些ES中并不存在的无关字段,此时可以标记此字段,以便让EE框架跳过此字段,对此字段不处理.
2. 字段的更新策略,比如在调用更新接口时,实体类的字段非Null或者非空字符串时才更新,此时可以加字段注解,对指定字段标记更新策略.
3. 需要对类型为text或keyword_tex字段聚合时,可指定其fieldData=true,否则es会报错.
4. 对指定字段进行自定义命名,比如该字段在es中叫wu-la,但在实体model中叫ula,此时可以在value中指定value="wu-la".
5. 在自动托管索引模式下,可指定索引分词器及索引字段类型.
6. 在自动托管索引模式下,可指定索引中日期的format格式.
7. ...

使用示例:

```java
public class Document {
    // 此处省略其它字段... 
        
    // 场景一:标记es中不存在的字段
    @IndexField(exist = false)
    private String notExistsField;
        
    // 场景二:更新时,此字段非空字符串才会被更新
    @IndexField(strategy = FieldStrategy.NOT_EMPTY)
    private String creator;
    
    // 场景三: 指定fieldData
    @IndexField(fieldType = FieldType.TEXT, fieldData = true)
    private String filedData;
    
    // 场景四:自定义字段名
    @IndexField("wu-la")    
    private String ula;

    // 场景五:支持日期字段在es索引中的format类型
    @IndexField(fieldType = FieldType.DATE, dateFormat = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
    private String gmtCreate;

    // 场景六:支持指定字段在es索引中的分词器类型
    @IndexField(fieldType = FieldType.TEXT, analyzer = Analyzer.IK_SMART, searchAnalyzer = Analyzer.IK_MAX_WORD)
    private String content;
}
```



| 属性           | 类型    | 必须指定 | 默认值                   | 描述                            |
| -------------- | ------- | -------- | ------------------------ | ------------------------------- |
| value          | String  | 否       | ""                       | 字段名                          |
| exist          | boolean | 否       | true                     | 字段是否存在                    |
| fieldType      | Enum    | 否       | FieldType.NONE           | 字段在es索引中的类型            |
| fieldData      | boolean | 否       | false                    | text类型字段是否支持聚合        |
| analyzer       | String  | 否       | Analyzer.NONE            | 索引文档时用的分词器            |
| searchAnalyzer | String  | 否       | Analyzer.NONE            | 查询分词器                      |
| strategy       | Enum    | 否       | FieldStrategy.DEFAULT    | 字段验证策略                    |
| dateFormat     | String  | 否       | ""                       | es索引中的日期格式,如yyyy-MM-dd |
| nestedClass    | Class   | 否       | DefaultNestedClass.class | 嵌套类                          |
| parentName     | String  | 否       | ""                       | 父子文档-父名称                 |
| childName      | String  | 否       | ""                       | 父子文档-子名称                 |
| joinFieldClass | Class   | 否       | JoinField.class          | 父子文档-父子类型关系字段类     |



温馨提示

- 更新策略一共有3种:
  - **NOT_NULL**: 非Null判断,字段值为非Null时,才会被更新
  - **NOT_EMPTY**: 非空判断,字段值为非空字符串时才会被更新
  - **IGNORE**: 忽略判断,无论字段值为什么,都会被更新

其中场景四和场景五仅在索引自动托管模式下生效,如果开启了手动处理索引模式,则需要用户通过手动调用我提供的API传入相应的分词器及日期格式化参数进行索引的创建/更新.

### [@Score](https://gitee.com/dromara/easy-es/blob/master/easy-es-annotation/src/main/java/cn/easyes/annotation/Score.java)

- 描述：得分注解
- 使用位置：实体类中被作为ES查询得分返回的字段
- 使用场景举例：比如需要知道本次匹配查询得分有多少时,可以在实体类中添加一个类型为Float/float的字段,并在该字段上添加@Score注解,在后续查询中,若es有返回当次查询的得分,则此得分会自动映射至此字段上

| 属性          | 类型 | 必须指定 | 默认值 | 描述                                         |
| ------------- | ---- | -------- | ------ | -------------------------------------------- |
| decimalPlaces | int  | 否       | 0      | 得分保留小数位,默认不处理,保持es返回的得分值 |



### @Distance

- 描述：距离注解
- 使用位置：实体类中被作为ES地理位置排序距离值的返回字段
- 使用场景举例：比如需要知道按距离由近及远查询后的数据,实际距离某一坐标有多远,可以在实体类中添加一个类型为Double/double的字段,并在该字段上添加@Distance注解,在后续查询中,若es有返回距离,则此距离会自动映射至此字段上

| 属性             | 类型 | 必须指定 | 默认值 | 描述                                                         |
| ---------------- | ---- | -------- | ------ | ------------------------------------------------------------ |
| decimalPlaces    | int  | 否       | 0      | 距离保留小数位,默认不处理,保持es返回的距离值                 |
| sortBuilderIndex | int  | 否       | 0      | 排序字段在sortBuilders中的位置, 默认为0,若有多个排序器,则指定为其所在位置 |



### 其它注解

除了上面这几个高频注解，项目中偶尔还会用到一些其它注解，比如高亮注解@HighLight，比如拦截器注解@Intercepts等注解，我们会在后面具体的章节详细介绍，此处仅列出几个必须掌握的注解，其它注解按需学习即可。



# 2.核心功能

## 索引处理

> 前言
>
> ES难用,索引首当其冲,索引的创建不仅复杂,而且难于维护,一旦索引有变动,就必须面对索引重建带来的服务停机和数据丢失等问题... 尽管ES官方提供了索引别名机制来解决问题,但门槛依旧很高,步骤繁琐,在生产环境中由人工操作非常容易出现失误带来严重的问题. 为了解决这些痛点,Easy-Es提供了多种策略,将用户彻底从索引的维护中解放出来,我们提供了多种索引处理策略,来满足不同用户的个性化需求. 通过对索引的初体验,相信您也可以更深体会到EE的成熟度和易用性. 其中全自动平滑模式,首次采用全球领先的"哥哥你不用动,EE我全自动"的模式,索引的创建,更新,数据迁移等所有全生命周期均无需用户介入,由EE全自动完成,过程零停机,连索引类型都可智能自动推断,一条龙服务,包您满意.是全球开源首创,充分借鉴了JVM垃圾回收算法思想,史无前例,尽管网上已有平滑过渡方案,但并非全自动,过程依旧靠人工介入,我为EE代言,请放心将索引托管给EE,索引只有在彻底迁移成功才会删除旧索引,否则均不会对原有索引和数据造成影响,发生任何意外均能保留原索引和数据,所以安全系数很高.

> **温馨提示:** 新手上路可尽量选择自动挡模式,老司机自动挡手动挡您请随意~ 追求生产环境稳定性,建议您采用手动挡模式,我们手动挡也提供了非常友好的一键创建功能,使用起来也是甜甜的. 自动挡模式,建议您在充分了解其运作原理和源码后再上生产,否则不少小白在没弄明白原理和如何正确配置就无脑上生产,容易被自己坑到,弄明白了请随便,我们的平滑模式实际上是非常安全的.

### 模式一:自动托管之平滑模式(自动挡-雪地模式) 默认开启此模式

> 在此模式下,**索引的创建更新数据迁移等全生命周期用户均不需要任何操作即可完成,**过程零停机,用户无感知,可实现在生产环境的平滑过渡,类似汽车的自动档-雪地模式,平稳舒适,彻底解放用户,尽情享受自动驾驶的乐趣! 
>
> 需要值得特别注意的是,在自动托管模式下,系统会自动生成一条名为`ee-distribute-lock`的索引,该索引为框架内部使用,用户可忽略,若不幸因断电等其它因素极小概率下发生死锁,可删除该索引即可.另外,在使用时如碰到索引变更,原索引名称可能会被追加后缀_s0或_s1,不必慌张,这是全自动平滑迁移零停机的必经之路,索引后缀不影响使用,框架会自动激活该新索引.关于_s0和_s1后缀,在此模式下无法避免,因为要保留原索引数据迁移,又不能同时存在两个同名索引,凡是都是要付出代价的，如果您不认可此种处理方式,可继续往下看,总有一种适合您。

其核心处理流程梳理如下图所示,不妨结合源码看,更容易理解:

![平滑模式.png](E:/Development/Typora/images/fbf59164-dd62-4b88-9483-b222a3c3b52b.png)

### 模式二:自动托管之非平滑模式(自动挡-运动模式)

在此模式下,索引额创建及更新由EE全自动异步完成,但不处理数据迁移工作,速度极快类似汽车的自动挡-运动模式,简单粗暴,弹射起步! 适合在开发及测试环境使用,当然如果您使用logstash等其它工具来同步数据,亦可在生产环境开启此模式,在此模式下不会出现_s0和_s1后缀,索引会保持原名称.

![非平滑模式.png](E:/Development/Typora/images/0b1b4d41-cac5-410f-bae1-9a0b3557da75.png)

> 提示
>
> 以上两种自动模式中,索引信息主要依托于实体类,如果用户未对该实体类进行任何配置,EE依然能够根据字段类型智能推断出该字段在ES中的存储类型,此举可进一步减轻开发者负担,对刚接触ES的小白更是福音.
>
> 当然,仅靠框架自动推断是不够的,我们仍然建议您在使用中尽量进行详细的配置,以便框架能自动创建出生产级的索引.举个例子,例如String类型字段,框架无法推断出您实际查询中对该字段是精确查询还是分词查询,所以它无法推断出该字段到底用keyword类型还是text类型,倘若是text类型,用户期望的分词器是什么? 这些都需要用户通过配置告诉框架,否则框架只能按默认值进行创建,届时将不能很好地完成您的期望.
>
> 自动推断类型的优先级 < 用户通过注解指定的类型优先级
>

自动推断映射表:

| JAVA          | ES      |
| ------------- | ------- |
| byte          | byte    |
| short         | short   |
| int           | integer |
| long          | long    |
| float         | float   |
| double        | double  |
| BigDecimal    | keyword |
| char          | keyword |
| String        | keyword |
| boolean       | boolean |
| Date          | date    |
| LocalDate     | date    |
| LocalDateTime | date    |
| List          | text    |
| ...           | ...     |



> "自动挡"模式下的最佳实践示例:

```java
@Data
@IndexName(shardsNum = 3,replicasNum = 2) // 可指定分片数,副本数,若缺省则默认均为1
public class Document {
    /**
     * es中的唯一id,如果你想自定义es中的id为你提供的id,比如MySQL中的id,请将注解中的type指定为customize,如此id便支持任意数据类型)
     */
    @IndexId(type = IdType.CUSTOMIZE)
    private Long id;
    /**
     * 文档标题,不指定类型默认被创建为keyword类型,可进行精确查询
     */
    private String title;
    /**
     * 文档内容,指定了类型及存储/查询分词器
     */
    @HighLight(mappingField="highlightContent")
    @IndexField(fieldType = FieldType.TEXT, analyzer = Analyzer.IK_SMART, searchAnalyzer = Analyzer.IK_MAX_WORD)
    private String content;
    /**
     * 作者 加@TableField注解,并指明strategy = FieldStrategy.NOT_EMPTY 表示更新的时候的策略为 创建者不为空字符串时才更新
     */
    @IndexField(strategy = FieldStrategy.NOT_EMPTY)
    private String creator;
    /**
     * 创建时间
     */
    @IndexField(fieldType = FieldType.DATE, dateFormat = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
    private String gmtCreate;
    /**
     * es中实际不存在的字段,但模型中加了,为了不和es映射,可以在此类型字段上加上 注解@TableField,并指明exist=false
     */
    @IndexField(exist = false)
    private String notExistsField;
    /**
     * 地理位置经纬度坐标 例如: "40.13933715136454,116.63441990026217"
     */
    @IndexField(fieldType = FieldType.GEO_POINT)
    private String location;
    /**
     * 图形(例如圆心,矩形)
     */
    @IndexField(fieldType = FieldType.GEO_SHAPE)
    private String geoLocation;
    /**
     * 自定义字段名称
     */
    @IndexField(value = "wu-la")
    private String customField;

    /**
     * 高亮返回值被映射的字段
     */
    private String highlightContent;
}
```



### 模式三:手动模式(手动挡)

在此模式下,索引的所有维护工作EE框架均不介入,由用户自行处理,EE提供了开箱即用的索引CRUD相关API,您可以选择使用该API手动维护索引,由于API高度完善,尽管是手动挡,但使用起来依旧简单到爆,一行代码搞定索引创建.当然您亦可通过es-head等工具来维护索引,总之在此模式下,您拥有更高的自由度,比较适合那些质疑EE框架的保守用户或追求极致灵活度的用户使用,类似汽车的手动挡,新手不建议使用此模式,老司机请随便.

> 手动挡模式下,EE提供了如下API,供用户进行便捷调用：
>
> - indexName需要用户手动指定
> - 对象 Wrapper 为 条件构造器

```java
    // 获取索引信息
    GetIndexResponse getIndex();
    // 获取指定索引信息
    GetIndexResponse getIndex(String indexName);
    // 是否存在索引
    Boolean existsIndex(String indexName);
    // 根据实体及自定义注解一键创建索引
    Boolean createIndex();
    // 创建索引
    Boolean createIndex(LambdaEsIndexWrapper<T> wrapper);
    // 更新索引
    Boolean updateIndex(LambdaEsIndexWrapper<T> wrapper);
    // 删除指定索引
    Boolean deleteIndex(String indexName);
```



![手动模式.png](E:/Development/Typora/images/3faa18ce-c39f-44d5-b0e5-244b4828df3e.png)

> 上述API,我们仅演示创建索引,其它过于简单,不在这里赘述，如有需要可移步至源码test模块查看. 通过API手动创建索引,我们提供了两种方式

-方式一:根据实体类及自定义注解一键创建(推荐),99.9%场景适用

```java
/**
 * 实体类信息
**/
@Data
@IndexName(shardsNum = 3, replicasNum = 2, keepGlobalPrefix = true)
public class Document {
    /**
     * es中的唯一id,如果你想自定义es中的id为你提供的id,比如MySQL中的id,请将注解中的type指定为customize或直接在全局配置文件中指定,如此id便支持任意数据类型)
     */
    @IndexId(type = IdType.CUSTOMIZE)
    private String id;
    /**
     * 文档标题,不指定类型默认被创建为keyword类型,可进行精确查询
     */
    private String title;
    /**
     * 文档内容,指定了类型及存储/查询分词器
     */
    @HighLight(mappingField = "highlightContent")
    @IndexField(fieldType = FieldType.TEXT, analyzer = Analyzer.IK_SMART, searchAnalyzer = Analyzer.IK_MAX_WORD)
    private String content;
    // 省略其它字段...
}
```



```java
 @Test
    public void testCreateIndexByEntity() {
        // 然后通过该实体类的mapper直接一键创建,非常傻瓜级
        documentMapper.createIndex();
    }
```



> 提示
>
> 实体类中的注解用法可参考注解章节,整体比较傻瓜级,和MP中的注解用法高度相似.
>

-方式二:通过api创建,每个需要被索引的字段都需要处理,比较繁琐,但灵活性最好,支持所有es能支持的所有索引创建,供0.01%场景使用(不推荐)

```java
    @Test
    public void testCreatIndex() {
        LambdaEsIndexWrapper<Document> wrapper = new LambdaEsIndexWrapper<>();
        // 此处简单起见 索引名称须保持和实体类名称一致,字母小写 后面章节会教大家更如何灵活配置和使用索引
        wrapper.indexName(Document.class.getSimpleName().toLowerCase());

        // 此处将文章标题映射为keyword类型(不支持分词),文档内容映射为text类型,可缺省
        // 支持分词查询,内容分词器可指定,查询分词器也可指定,,均可缺省或只指定其中之一,不指定则为ES默认分词器(standard)
        wrapper.mapping(Document::getTitle, FieldType.KEYWORD)
                .mapping(Document::getContent, FieldType.TEXT,Analyzer.IK_MAX_WORD,Analyzer.IK_MAX_WORD);
        
        // 如果上述简单的mapping不能满足你业务需求,可自定义mapping
        // wrapper.mapping(Map);

        // 设置分片及副本信息,3个shards,2个replicas,可缺省
        wrapper.settings(3,2);

        // 如果上述简单的settings不能满足你业务需求,可自定义settings
        // wrapper.settings(Settings);
        
        // 设置别名信息,可缺省
        String aliasName = "daily";
        wrapper.createAlias(aliasName);
        
        // 创建索引
        boolean isOk = documentMapper.createIndex(wrapper);
        Assert.assertTrue(isOk);
    }
```



> 温馨提示
>
> 实体类中,id字段不需要创建索引,否则会报错.
>
> 由于ES索引改动自动重建的特性,因此本接口设计时将创建索引所需的mapping,settings,alias信息三合一了,尽管其中每一项配置都可缺省,但我们仍建议您在创建索引前提前规划好以上信息,可以规避后续修改带来的不必要麻烦,若后续确有修改,您仍可以通过别名迁移的方式(推荐,可平滑过渡),或删除原索引重新创建的方式进行修改.
>

### 配置启用模式

以上三种模式的配置,您只需要在您项目的配置文件application.properties或application.yml中加入一行配置即可:

```yaml
easy-es:
  socketTimeout: 5000 # 请求通信超时时间 单位:ms 在平滑模式下,由于要迁移数据,用户可根据数据量大小调整此参数值大小,否则请求容易超时导致索引托管失败,建议您尽量给大不给小,跟那玩意一样,大点没事,太小你懂的!
  global-config:
    process_index_mode: smoothly #smoothly:平滑模式, not_smoothly:非平滑模式, manual:手动模式
    async-process-index-blocking: true # 异步处理索引是否阻塞主线程 默认阻塞
    distributed: false # 项目是否分布式环境部署,默认为true, 如果是单机运行可填false,将不加分布式锁,效率更高.
```



若缺省此行配置,则默认开启平滑模式.

> 温馨提示
>
> - 自动挡模式下,如果索引托管成功,则会在控制台打印 Congratulations auto process index by Easy-Es is done !
> - 自动挡模式下,如果索引托管失败,则会在控制台打印 Unfortunately, auto process index by Easy-Es failed... 以及异常日志,可根据异常日志信息去排查
> - 如果索引托管失败,此时用户调用了insert相关API插入数据,由于索引不存在,es(非框架)会自动为用户创建默认索引,创建的索引字段类型为keyword_text类型,并非用户通过注解指定的,因此出现这种情况别问我为啥没生效,因为索引托管因为你的配置或环境原有问题失败了.
> - 运行测试模块时强烈建议开启异步处理索引阻塞主线程,否则测试用例跑完后,主线程退出,但异步线程可能还没跑完,可能出现死锁,若不幸出现死锁,删除ee-distribute-lock即可.
> - 生产环境或迁移数据量比较大的情况下,可以配置开启非阻塞,这样服务启动更快.
> - 以上三种模式,用户可根据实际需求灵活选择,自由体验,在使用过程中如有任何意见或建议可反馈给我们,我们将持续优化和改进,
> - EE在索引托管采用了策略+工厂设计模式,未来如果有更多更优模式,可以在不改动原代码的基础上轻松完成拓展,符合开闭原则,也欢迎各路开源爱好者贡献更多模式PR!
> - 我们将持续秉承把复杂留给框架,把易用留给用户这一理念,砥砺前行.
>
> 上述所有API对应代码演示皆可参考源码test模块->test目录->index包下代码
>



## CRUD接口

> 说明
>
> - 通用 CRUD 封装[BaseEsMapper (opens new window)](https://gitee.com/dromara/easy-es/blob/master/easy-es-core/src/main/java/cn/easyes/core/conditions/interfaces/BaseEsMapper.java)接口,为 Easy-Es 启动时自动解析实体对象关系映射转换为 EE 内部对象注入容器
> - 泛型 T 为任意实体对象
> - insert接口需要区别于MP,具体可看下面insert文档
> - 参数 Serializable 为任意类型主键 Easy-Es 不推荐使用复合主键约定每个索引都有自己的唯一 id 主键
> - 对象 Wrapper 为 条件构造器
> - 针对实体对象T中的 `get和set` 方法,我们推荐您使用[Lombok (opens new window)](https://projectlombok.org/)插件生成,若您采用IDEA自带插件生成,通过Lambda风格获取的字段名称时,会导致部分驼峰命名的字段无法获取正确的字段名. 比如有字段名称叫eName,采用Lombok生成的的get方法为getEName(),但IDEA生成的为geteName(),如此框架底层解析字段名称时就会报错,MP也存在同样问题.

### Insert

```java
// 插入一条记录
Integer insert(T entity);

// 批量插入多条记录
Integer insertBatch(Collection<T> entityList)
```

##### 参数说明

| 类型            | 参数名     | 描述         |
| --------------- | ---------- | ------------ |
| `T`             | entity     | 实体对象     |
| `Collection<T>` | entityList | 实体对象集合 |

> 
>
> 特别注意
>
> - 如果您在insert时传入的entity有id并且该id对应数据已存在,==则此次insert实际效果为更新该id对应的数据,并且更新不计入insert接口最后返回的成功总条数.==
> - 当insert接口如上所述,触发了数据更新逻辑,本次更新字段和全局配置的策略（如NOT_NULL/NOT_EMPTY）等均不生效,若您期望策略生效,可以调用update接口而非insert接口.
> - 插入后如需id值可直接从entity中取,用法和MP中一致,批量插入亦可直接从原对象中获取插入成功后的数据id,以上接口返回Integer为成功条数.

### Delete

```java
 // 根据 ID 删除
Integer deleteById(Serializable id);

// 根据 entity 条件，删除记录
Integer delete(LambdaEsQueryWrapper<T> wrapper);

// 删除（根据ID 批量删除）
Integer deleteBatchIds(Collection<? extends Serializable> idList);
```

##### 参数说明

| 类型                                 | 参数名       | 描述                    |
| ------------------------------------ | ------------ | ----------------------- |
| `Wrapper<T>`                         | queryWrapper | 实体包装类 QueryWrapper |
| `Serializable`                       | id           | 主键ID                  |
| `Collection<? extends Serializable>` | idList       | 主键ID列表              |



### [#](https://www.easy-es.cn/pages/c5999a/#update)Update

```java
//根据 ID 更新
Integer updateById(T entity);

// 根据ID 批量更新
Integer updateBatchByIds(Collection<T> entityList);

//  根据动态条件 更新记录
Integer update(T entity, LambdaEsUpdateWrapper<T> updateWrapper);
```



##### 参数说明

| 类型            | 参数名        | 描述                             |
| --------------- | ------------- | -------------------------------- |
| `T`             | entity        | 实体对象                         |
| `Wrapper<T>`    | updateWrapper | 实体对象封装操作类 UpdateWrapper |
| `Collection<T>` | entityList    | 实体对象集合                     |



### Select

```java
	// 获取总数
    Long selectCount(LambdaEsQueryWrapper<T> wrapper);
 	// 根据 ID 查询
    T selectById(Serializable id);
	// 查询（根据ID 批量查询）
    List<T> selectBatchIds(Collection<? extends Serializable> idList);
	// 根据动态查询条件，查询一条记录 若存在多条记录 会报错
    T selectOne(LambdaEsQueryWrapper<T> wrapper);
    // 根据动态查询条件，查询全部记录
    List<T> selectList(LambdaEsQueryWrapper<T> wrapper);
```

##### [#](https://www.easy-es.cn/pages/c5999a/#参数说明-4)参数说明

| 类型                                 | 参数名       | 描述                    |
| ------------------------------------ | ------------ | ----------------------- |
| `Wrapper<T>`                         | queryWrapper | 实体包装类 QueryWrapper |
| `Serializable`                       | id           | 主键ID                  |
| `Collection<? extends Serializable>` | idList       | 主键ID列表              |



> 提示
>
> - CRUD接口用法基本与MP一致
> - 用户需要继承的Mapper为BaseEsMapper，而非BaseMapper
> - EE没有提供Service层,而是把MP中一些Service层的方法直接下沉到Mapper层了,用户用起来会更方便












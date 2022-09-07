# SpringBoot - Elasticsearch 

## 一.快速入门



![image.png](E:\Development\Typora\images\848e9772cf714eb7ae21074f4cfa89d0tplv-k3u1fbpfcp-zoom-in-crop-mark3024000.awebp)

注意springboot的版本一定要和[elasticsearch](https://so.csdn.net/so/search?q=elasticsearch&spm=1001.2101.3001.7020)和spring-boot-starter-data-elasticsearch的版本匹配,不然就会出现问题

7.12.1的es对于的springboot版本为2.5.1！！！！！

### 1.POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.hong</groupId>
    <artifactId>SpringBoot-ES</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBoot-ES</name>
    <description>SpringBoot-ES</description>
    <properties>
        <java.version>1.8</java.version>
        <elasticsearch.version>7.12.1</elasticsearch.version>
    </properties>


    <dependencies>
        <!-- 自动化配置 Spring Data Elasticsearch -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### 2.application.properties

```yml
spring:
  elasticsearch:
    rest:
      uris: 106.53.124.182:9200
```



### 3.ESProductDO

```java
package com.hong.springbootes.dataobject;

import com.hong.springbootes.IKconst.FieldAnalyzer;
import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

@Document(indexName = "product", // 索引名
        shards = 1, // 默认索引分区数
        replicas = 0, // 每个分区的备份数
        refreshInterval = "-1" // 刷新间隔
)
@Data
public class ESProductDO {

    /**
     * ID 主键
     */
    @Id
    private Integer id;

    /**
     * SPU 名字
     */
    @Field(analyzer = FieldAnalyzer.IK_MAX_WORD, type = FieldType.Text)
    private String name;
    /**
     * 卖点
     */
    @Field(analyzer = FieldAnalyzer.IK_MAX_WORD, type = FieldType.Text)
    private String sellPoint;
    /**
     * 描述
     */
    @Field(analyzer = FieldAnalyzer.IK_MAX_WORD, type = FieldType.Text)
    private String description;
    /**
     * 分类编号
     */
    private Integer cid;
    /**
     * 分类名
     */
    @Field(analyzer = FieldAnalyzer.IK_MAX_WORD, type = FieldType.Text)
    private String categoryName;


}
```

- 为了区别关系数据库的实体对象，艿艿习惯以 ES 前缀开头。
- 字段上的 `@Field` 注解的 [FieldAnalyzer](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-15-spring-data-es/lab-15-spring-data-jest/src/main/java/cn/iocoder/springboot/lab15/springdatajest/constant/FieldAnalyzer.java) ，是定义的枚举类，记得自己创建下噢。代码如下：

```java
// FieldAnalyzer.java

public class FieldAnalyzer {

    /**
     * IK 最大化分词
     *
     * 会将文本做最细粒度的拆分
     */
    public static final String IK_MAX_WORD = "ik_max_word";

    /**
     * IK 智能分词
     *
     * 会做最粗粒度的拆分
     */
    public static final String IK_SMART = "ik_smart";

}
```



### 4.ProductRepository

```java
// ProductRepository.java

public interface ProductRepository extends ElasticsearchRepository<ESProductDO, Integer> {
}
```

- 继承 `org.springframework.data.elasticsearch.repository.ElasticsearchRepository` 接口，第一个泛型设置对应的实体是 ESProductDO ，第二个泛型设置对应的主键类型是 Integer 。

- 因为实现了 ElasticsearchRepository 接口，Spring Data Jest 会自动生成对应的 CRUD 等等的代码。😈 是不是很方便。

- ElasticsearchRepository 类图如下：

  

  - 每个接口定义的方法，胖友可以点击下面每个链接，自己瞅瞅，简单~
  - [`org.springframework.data.repository.CrudRepository`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/CrudRepository.java)
  - [`org.springframework.data.repository.PagingAndSortingRepository`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/PagingAndSortingRepository.java)
  - [`org.springframework.data.elasticsearch.repository.ElasticsearchCrudRepository`](https://github.com/spring-projects/spring-data-elasticsearch/blob/master/src/main/java/org/springframework/data/elasticsearch/repository/ElasticsearchCrudRepository.java)
  - [`org.springframework.data.elasticsearch.repository.ElasticsearchRepository`](https://github.com/spring-projects/spring-data-elasticsearch/blob/master/src/main/java/org/springframework/data/elasticsearch/repository/ElasticsearchRepository.java)

### 5.简单测试



```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Test // 插入一条记录
    public void testInsert() {
        ESProductDO product = new ESProductDO();
        product.setId(1); // 一般 ES 的 ID 编号，使用 DB 数据对应的编号。这里，先写死
        product.setName("芋道源码");
        product.setSellPoint("愿半生编码，如一生老友");
        product.setDescription("我只是一个描述");
        product.setCid(1);
        product.setCategoryName("技术");
        productRepository.save(product);
    }

    // 这里要注意，如果使用 save 方法来更新的话，必须是全量字段，否则其它字段会被覆盖。
    // 所以，这里仅仅是作为一个示例。
    @Test // 更新一条记录
    public void testUpdate() {
        ESProductDO product = new ESProductDO();
        product.setId(1);
        product.setCid(2);
        product.setCategoryName("技术-Java");
        productRepository.save(product);
    }

    @Test // 根据 ID 编号，删除一条记录
    public void testDelete() {
        productRepository.deleteById(1);
    }

    @Test // 根据 ID 编号，查询一条记录
    public void testSelectById() {
        Optional<ESProductDO> userDO = productRepository.findById(1);
        System.out.println(userDO.isPresent());
    }

    @Test // 根据 ID 编号数组，查询多条记录
    public void testSelectByIds() {
        Iterable<ESProductDO> users = productRepository.findAllById(Arrays.asList(1, 4));
        users.forEach(System.out::println);
    }

}
```
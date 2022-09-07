Spring

# 第三章 使用数据



## 3.2 使用Spring DAta JPA持久化数据

 JPA （Java Persistence API）Java 持久化 API。

### 	3.2.1 添加Spring Data JPA到项目中

Spring Boot通过JPA starter来添加Spring Data JPA。

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

​	

### 	3.2.2 将领域对象标注为实体

  [**Ingredient类**]()

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@Entity
public class Ingredient {
 @Id
 private final String id;
 private final String name;
 private final Type type;
 public static enum Type {
 WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
 }
}
```

**@Entity** 

​	为了将Ingredient声明为JPA实体，它必须添加@Entity注解

**@Id**

​	它的id属性需要使⽤@Id注解，以便于将其指定为数据库中唯⼀标识该实体的属性。

**@NoArgsConstructor**(access=xxx,force=xxx)

​	<u>JPA需要实体有⼀个⽆参的构造器</u>， Lombok的@NoArgsConstructor注解能够帮助我们实现这⼀点。

​	将access属性设置为 AccessLevel.PRIVATE使其变成私有的。

​	因为这⾥有必须要设置的 final属性，所以我们将force设置为true，这样Lombok⽣成的构造器就会将它们设置为null。

> 　当用final作用于类的成员变量时，成员变量（注意是类的成员变量，局部变量只需要保证在使用之前被初始化赋值即可）必须在定义时或者构造器中进行初始化赋值

**@RequiredArgsConstructor**

​	@Data注 解会为我们添加⼀个有参构造器，但是使⽤@NoArgsConstructor注 解之后，这个构造器就会被移除掉。需要显式添加



[**Taco类**]()

```java
@Data
@Entity
public class Taco {
 @Id
 @GeneratedValue(strategy=GenerationType.AUTO)
 private Long id;
 @NotNull
 @Size(min=5, message="Name must be at least 5 characters long")
 private String name;
 private Date createdAt;
 @ManyToMany(targetEntity=Ingredient.class)
 @Size(min=1, message="You must choose at least 1 ingredient")
 private List<Ingredient> ingredients;
 @PrePersist
 void createdAt() {
 this.createdAt = new Date();
 }
}
```

**@GeneratedValue**

​	strategy设置为 AUTO,数据库⾃动⽣成ID值

 **@PrePersist**

​	在Taco持久化之前，我们会使⽤这个⽅法将 createdAt设置为当前的⽇期和时间



```java
@Data
@Entity
@Table(name="Taco_Order")
public class Order implements Serializable {}
```

**@Table**

​	它表明Order实体应该持久化到 数据库中名为Taco_Order的表中。

​	我们可以将这个注解⽤到所有的实体上，但是只有Order有必要这 样做。**如果没有它，JPA默认会将实体持久化到名为Order的表中**，但 是order是SQL的保留字，这样做的话会产⽣问题。



### 3.2.3 声明JPA repository

```java
public interface IngredientRepository
 extends CrudRepository<Ingredient, String> {
}

```

​	

**CrudRepository**  定义了很多⽤于CRUD（创建、读取、更新、删 除）操作的⽅法。注意，它是参数化的，

- ​	第⼀个参数是repository要持 久化的实体类型，

- ​    第⼆个参数是实体ID属性的类型。

  Spring Data JPA带来的好消息是，我们根本就不⽤编写实现类！当应⽤启动 的时候，Spring Data JPA会在运⾏期⾃动⽣成实现类。**这意味着我们现在就可以使⽤这些repository了。我们只需要像使⽤基于JDBC的 实现那样将它们注⼊控制器中就可以了**

  

### 3.2.4 ⾃定义JPA repository

```java
List<Order> findByDeliveryZip(String deliveryZip);
```

​	当创建repository实现的时候，Spring Data会检查repository接 ⼝的所有⽅法，解析⽅法的名称，并基于被持久化的对象来试图**推测⽅法的⽬的**。

repository⽅法是由⼀个**动词**、⼀个可选的**主 题**（Subject）、关键词**By**以及⼀个**断⾔**所组成的。

findByDeliveryZip()这个样例中，动词是find，断⾔是 DeliveryZip，主题并没有指定，暗含的主题是Order。(因为我们使⽤ Order对CrudRepository进⾏了参数化)

```java
List<Order> readOrdersByDeliveryZipAndPlacedAtBetween(
 String deliveryZip, Date startDate, Date endDate);
```

<img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220429173415128.png" alt="image-20220429173415128" style="zoom:50%;" />

- Spring Data会将get、read和find视为同义词，它们都是⽤来获 取⼀个或多个实体的。
- 在本例中，断⾔指定了Order的两个属性：deliveryZip和placedAt。 deliveryZip属性的值<u>必须要等于⽅法第⼀个参数传⼊的值</u>。关键字 Between表明placedAt属性的值必须要位于<u>⽅法最后两个参数的值之间</u>。

​	

对于一些通过方法命名约定很难甚至无法实现的可以通过使用注解**@Query**

```java
@Query("Order o where o.deliveryCity='Seattle'")
List<Order> readOrdersDeliveredInSeattle();//这里的方法名可以任意取了
```





# 第四章 保护Spring



## 4.1 启⽤Spring Security

​	添加依赖

```java
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



## 4.2配置Spring Security

 首先编写配置类

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```



### 	4.2.1配置⽤户存储

Spring Security为配置⽤户存储提供了多个可选⽅案， 包括：

-  基于内存的⽤户存储；

-  基于JDBC的⽤户存储； 

-  以LDAP作为后端的⽤户存储；

-  ⾃定义⽤户详情服务。

     

     不管使⽤哪种⽤户存储，你都可以通过覆盖 WebSecurityConfigurerAdapter基础配置类中定义的**configure()** ⽅法来进⾏配置。⾸先，我们可以将如下的⽅法添加到SecurityConfig 类中：

  ```java
  @Override protected void configure(AuthenticationManagerBuilder auth) throws Exception { ... }
  ```

  ### 4.2.2 基于内存的⽤户存储

  ### 4.2.3 基于JDBC的⽤户存储

  ```java
  @Autowired DataSource dataSource;
  @Override protected void configure(AuthenticationManagerBuilder auth)
      throws Exception { 
      
      auth.
          jdbcAuthentication().
          	dataSource(dataSource); 
  }
  ```

  **重写默认的用户查询功能**

  ```java
  @Override
  protected void configure(AuthenticationManagerBuilder auth)
   throws Exception {
       auth
           .jdbcAuthentication() //基于JDBC的⽤户存储
               .dataSource(dataSource)
               .usersByUsernameQuery( //认证查询
                   "select username, password, enabled from Users " +
                   "where username=?")
               .authoritiesByUsernameQuery(//权限查询
                   "select username, authority from UserAuthorities " +
                   "where username=?")
               .passwordEncoder(new StandardPasswordEncoder("53cr3t");
                                //将明文密码转码
  }
  
  ```

  

自定义SQL查询时，很重要的⼀点就是要遵循查询的基本协议。

- 所有查询都将<u>⽤户名</u>作为唯⼀的参数。

- **认证查询**会选取⽤户名、密码以及启⽤状态信息。

- **权限查询**会选取零⾏或多⾏包含 该⽤户名及其权限信息的数据。

- **群组权限查询**会选取零⾏或多⾏数据，每⾏数据中都会包含群组ID、群组名称以及权限。

  

**passwordEncoder()**⽅法可以接受Spring Security中 PasswordEncoder接⼝的任意实现。Spring Security的加密模块包 括了多个这样的实现。

- BCryptPasswordEncoder：使⽤bcrypt强哈希加密。
-  NoOpPasswordEncoder：不进⾏任何转码。
-  Pbkdf2PasswordEncoder：使⽤PBKDF2加密。 
- SCryptPasswordEncoder：使⽤scrypt哈希加密。
-  StandardPasswordEncoder：使⽤SHA-256哈希加密。

数据库中的密码是永远不会解码的~~!!



### 4.2.4 以LDAP作为后端的⽤户存储

LDAP（Lightweight Directory Access Protocol，轻量级⽬录访问协议）



[Spring Security 实战干货：如何获取当前用户信息 - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1838816)





# 第五章 使用配置属性



## 5.1 细粒度的⾃动配置

​	在Spring中有两种不同 （但相关）的配置。

- bean装配：声明在Spring应⽤上下⽂中创建哪些应⽤组件以及它 们之间如何互相注⼊的配置。 

- 属性注⼊：设置Spring应⽤上下⽂中**bean的值**的配置。

  当声明了@Bean注解后，方法会同时初始化bean并⽴即为它的属性设置值

  > 配置属性只不过是bean的属性，它们可以从 Spring的环境抽象中接受配置

  ### 5.1.1  理解Spring的环境抽象

  Spring环境会将属性聚合到⼀个<u>源</u>中，通过这个源可以注⼊到**Spring的 bean**中。

  <img src="C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220501165408454.png" alt="image-20220501165408454" style="zoom:50%;" />

  eg:在“src/main/resources/application.yml”中设置server.port的值
  
  ```java
  server:
   port: 9090
  ```

  

  ### 5.1.2 配置数据源

  尽管我们可以显式地配置⾃⼰的DataSource，但通常没有必要这样做。

  相反,通过配置属性设置数据库URL和凭证信息会更加简单。

  例如，如果你想要开始使⽤MySQL数据库,那么可以把如下的配置属性添 加到application.yml中：
  
  ```java
  spring:
   datasource:
       url: jdbc:mysql://localhost/tacocloud
       username: tacodb
       password: tacopassword
       driver-class-name: com.mysql.jdbc.Driver  //指定JDBC的驱动类。也可不写，Spring Boot会根据数据库URL的结构推算出来。
  ```
  
   spring.datasource下有两个属性  
  
  ​               schme   、  data
  
  其中schema为表初始化语句，data为数据初始化，默认加载schema.sql与data.sql。脚本位置可以通过spring.datasource.schema  与spring.datasource.data 来改变
  
  ### 5.1.3 配置嵌⼊式服务器
  
  当设置
  
  ```java
  server:
   port: 0
  ```

​		尽管我们将server.port属性显式设置成了0，但是服务器并不会真的在端⼝0上启动。相反，它会任选⼀个可⽤的端⼝。

# 第6章 创建REST服务

## 6.1 编写RESTful控制器

​				表6.1 Spring MVC的HTTP请求处理注解

![image-20220503220259319](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220503220259319.png)

![image-20220503220328223](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20220503220328223.png)

### 6.1.1 从服务器中检索数据



服务器中的控制器

```java
@RestController
@RequestMapping(path="/design", ⇽--- 处理针对“/design”的请求
  				produces="application/json")  ⇽---这指明DesignTacoController中的所有处										器⽅法只会处理Accept头信息包含“application/json”的请求。
@CrossOrigin(origins="*") ⇽--- 允许跨域请求
public class DesignTacoController {
     private TacoRepository tacoRepo;
     @Autowired
     EntityLinks entityLinks;
     public DesignTacoController(TacoRepository tacoRepo) {
    	 this.tacoRepo = tacoRepo;
     }
     @GetMapping("/recent")
     public Iterable<Taco> recentTacos() { ⇽--- 获取并返回最近设计的taco
    	 PageRequest page = PageRequest
         .of(0, 12, Sort.by("createdAt").descending());
         return tacoRepo.findAll(page).getContent();
     }
}
```



**@RestController**

目的

- 它是⼀个类似于@Controller和@Service 的构造型注解，能够让类被组件扫描功能发现。
- @RestController注解会告诉Spring，控制器中的所有处理器⽅法的返回值都要**直接写⼊响应体中**，⽽不是将值放到模型中并传递给⼀个视图以便于进⾏渲染。

**@CrossOrigin**

​		因为应⽤程序的 Angular部分将会运⾏在与API相独⽴的主机和/或端⼝上（⾄少⽬前是这样的），Web浏 览器会阻⽌Angular客户端消费该API。我们可以在服务端响应中添加CORS（Cross-Origin Resource Sharing，跨域资源共享）头信息来突破这⼀限制





假设我们想要提供⼀个按照ID抓取单个taco的端点。

```java
@GetMapping("/{id}")
public ResponseEntity<Taco> tacoById(@PathVariable("id") Long id) {
     Optional<Taco> optTaco = tacoRepo.findById(id);
     if (optTaco.isPresent()) {
     return new ResponseEntity<>(optTaco.get(), HttpStatus.OK);
     }
     return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
}

```

.findById()返回的是⼀个**Optional**，因为根据给定的ID可能获取不到 taco，所以在返回值的时候我们需要确定该ID是否能够匹配⼀个taco。如果能够匹配，我们 可以调⽤Optional对象的**get()⽅法**返回实际的Taco。



### 6.1.2 发送数据到服务器端

```java
@PostMapping(consumes="application/json")  //处理POST请求 表明该⽅法只会处理Contenttype与application/json相匹配的请求。
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
 return tacoRepo.save(taco);
}
```

@PostMapping的consumes属性⽤于指定请求输⼊，⽽produces⽤于指定请求输出。



# 第十三章 注册和发现服务



## 13.2 搭建服务注册中心

导入依赖

```java
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

添加完依赖后，还需要在应用的主导类添加**@EnableEurekaServer**注解

```java
@SpringBootApplication
@EnableEurekaServer //开启Eureka服务中心
public class ServiceRegistryApplication {
 public static void main(String[] args) {
 SpringApplication.run(ServiceRegistryApplication.class, args);
 }
}
```

### 13.2.1 配置Eureka

reason:

除⾮我们正确配置了Eureka服务 器，否则它会以⽇志⽂件中异常的形式每隔30秒就抱怨孤独状态。这是因 为，每隔30秒，<u>Eureka服务器就会尝试与另外的Eureka服务器建⽴关联</u>， 以注册⾃⼰并共享其注册中⼼中的信息。

**配置Eureka使其接受当前的孤独状态**

```java
eureka:
 instance:
 	hostname: localhost //告诉Eureka它正运⾏在哪个主机（host）上。
 client:
     fetch-registry: false //因为在 开发模式下并没有其他的Eureka服务器，所以我们将它们设置为false
     register-with-eureka: false //这 样Eureka将不会尝试与其他的Eureka服务器建⽴关联。
     service-url:
    	 defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka

```

**eureka.client.fetch-registry**

 **eureka.client.register-with-eureka** 

​		在其他的微服务中，我们可能会通过 这两个属性告诉它们该如何与Eureka服务器进⾏交互。但是，不要忘了， Eureka也是⼀个微服务，所以这些属性也可以⽤到Eureka服务器上，以便 于告诉它该如何与其他Eureka服务器进⾏交互。

​	这两个属性的默认值都是true，表明Eureka应该从其他的Eureka实例 获取注册信息，并且应该将⾃⾝注册为其他Eureka服务器中的服务。因为在 开发模式下并没有其他的Eureka服务器，所以我们将它们设置为false，这 样Eureka将不会尝试与其他的Eureka服务器建⽴关联。



**指定Eureka的服务器端⼝**

```java
server:
 port: 8761 //Eureka客户端默认监听的端⼝。
```



**禁⽤⾃我保护模式**

```java
eureka:
 ...
 server:
 enable-self-preservation: false
//Eureka希望服务实例能够注册上来，并且每隔30秒向它发送⼀次注册更新的请求。通常，如果Eureka在3个更新周期（或者说90秒）内没有收到服务的更新请求，就会将该服务注销。
```

虽然我们在开发环境可以禁⽤⾃我保护模式，但是在投⼊⽣产环境时需 要将其启⽤。



## 13.3 注册和发现服务

将Eureka客户端依赖添加到服务应⽤的构建⽂件中：

```java
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 13.3.1 配置Eureka客户端属性

**更改服务在Eureka中的注册名称**，在 application.yml中

```java
spring:
 application:
 	name: ingredient-service
```

如果我们为配料服务添加多个实例，它们就会**以相同的名称** 出现在注册中⼼，实际上，服务会扩展到多个实例，并假定它们是完全相同 的，服务的消费者可以从中选择。



**设置服务的端口号**

*因为这些服务现在只会通过Eureka进⾏ 查找，所以它们监听什么端⼝也就⽆所谓了，Eureka能够知道它们使⽤的是 什么端⼝。为了避免本地运⾏时潜在的端⼝冲突，我们可以将端⼝设置为0：*

```java
server:
 port: 0
```

> 注意：将端⼝设置成0的话，应⽤会选择任意⼀个可⽤端⼝来启动。



**指定 Eureka服务器的位置**

```java
eureka:
 client:
 	service-url:
 		defaultZone: http://eureka1.tacocloud.com:8761/eureka/   ##客户端会使⽤eureka1.tacocloud.com主机（端⼝8761）上的Eureka服务器进⾏注册。
	
```



### 13.3.2 消费服务

如果消费者请求 ingredient-service服务时得到了多个服务实例，那么它该如何选择正确的 服务呢？

我们有两种⽅ 式可以消费从Eureka中查找到的服务：

- **⽀持负载均衡的RestTemplate；**
-  **Feign⽣成的客户端接⼝。**



#### 使⽤RestTemplate消费服务


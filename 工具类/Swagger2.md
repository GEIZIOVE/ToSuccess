# Swagger2

https://swagger.io/docs

## 1.导入依赖

在项目中使用Swagger需要Springfox

- swagger2
- ui

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```



## 2.配置Swagger

记得添加`@EnableSwagger2`注解开启Swagger2

### 2.1 ApiInfo

​		我们使用Docket来配置swagger的基本信息，他还需要一个ApiInfo对象创建API的基本信息，这些信息会在Swagger UI中进行显示。

​		我们查看源码，可以发现有以下字段可以配置：

```java
/**
 * Builds the api information
 */
public class ApiInfoBuilder {
  private String title; //- - API 的标题
  private String description;//- - api描述
  private String termsOfServiceUrl;//- - 服务条款的 url
  private Contact contact;//- -联系方式
  private String license;//- -许可证字符串
  private String licenseUrl;//- -- - 许可证网址
  private String version;//- -版本——API
  private List<VendorExtension> vendorExtensions = newArrayList();//扩展- - 扩展
}
```

Swagger也对`ApiInfo`做了默认配置

```java
public class ApiInfo {
  public static final Contact DEFAULT_CONTACT = new Contact("", "", "");
  public static final ApiInfo DEFAULT = new ApiInfo("Api Documentation", 
                                                    "Api Documentation", "1.0", "urn:tos",
          DEFAULT_CONTACT, "Apache 2.0", "http://www.apache.org/licenses/LICENSE-2.0", new ArrayList<VendorExtension>());
}
```



对于大部分项目，以下`ApiInfoBuilder`配置可以满足基本需求

```java
@Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo());
    }
private ApiInfo productApiInfo(){ //创建API的基本信息，这些信息会在SwaggerUI中显示
    return new ApiInfoBuilder()
            .title("接口---API文档")
            .description("本文档描述了书籍取件项目后端服务接口定义")
            .version("测试版")
            .contact(new Contact("wqh", "无", "1601709391@qq.com"))
            .extensions(null) // 在线文档的扩展信息
            .build();
}
```

### 	2.2Docket

记得添加`@EnableSwagger2`注解开启Swagger2

```java
/**
 * 概览
 * Swagger2配置信息
 */
@Configuration
@EnableSwagger2
public class Swagger2Config {
     return new Docket() ;
}
```



直接来看Docker的源码



#### ①DocumentationType 文档类型

可以看到 `Docket` 的构造函数需要一个 `DocumentationType` 作为参数，我们点进它的源码看看：

```java
public class DocumentationType extends SimplePluginMetadata {
    public static final DocumentationType SWAGGER_12 = new DocumentationType("swagger", "1.2");
    public static final DocumentationType SWAGGER_2 = new DocumentationType("swagger", "2.0");
    public static final DocumentationType SPRING_WEB = new DocumentationType("spring-web", "1.0");
}
```

​		可以看到，它提供了三个构造好的 `DocumentationType` 常量，设置了使用 Swagger 哪个版本。

我们现在一般使用的是 Swagger2，所以在构造 `Docket` 对象时传 `DocumentationType.SWAGGER_2`

```java
return new Docket(DocumentationType.SWAGGER_2) 
```



#### ②ApiInfo 基本信息

​	见2.1



#### ③ApiSelectorBuilder 扫描接口

构建 `Docket` 时通过 select() 方法配置怎么扫描接口，它会返回一个 `ApiSelectorBuilder` 对象

点开 `ApiSelectorBuilder` 源码：

![image-20220724022956632](E:\Development\Typora\images\image-20220724022956632.png)

可以看到，它需要配置的是 `requestHandlerSelector` 和 `pathSelector`，然后通过build方法统一构造。

默认情况下，swagger的给了我们如下配置：即不扫描所有标有 @ApiIgnore 注解的类和方法，允许所有的请求路径：

```java
public static final ApiSelector DEFAULT = new ApiSelector(Predicates.and(Predicates.not(RequestHandlerSelectors.withClassAnnotation(ApiIgnore.class)), Predicates.not(RequestHandlerSelectors.withMethodAnnotation(ApiIgnore.class))), PathSelectors.any());
```

所以，在一开始，我们才会看到 [basic-error-controller](http://localhost:8080/swagger-ui.html#/basic-error-controller) 中的这些我们自己没配置过的接口

##### 1）requestHandlerSelector

接口扫描方案。通过 ApiSelectorBuilder#apis() 方法配置（也是链式编程），在 RequestHandlerSelectors 提供了配置方案：

![image-20220724023622702](E:\Development\Typora\images\image-20220724023622702.png)

- any()：扫描所有，项目中的所有接口都会被扫描到
- none()：不扫描接口
- withClassAnnotation()：扫描注解
- withMethodAnnotation()：扫描方法上的注解，比如 withMethodAnnotation(GetMapping.class) 只扫描 @GetMapping 标识的 GET 请求
- withClassAnnotation()：扫描类上的注解，比如 withClassAnnotation(Controller.class) 只扫描有 @Controller 注解的类中的接口
- basePackage()：扫描指定路径，比如 basePackage(“com.test.controller”) 只扫描 controller 包的解耦

##### 2）pathSelector

接口过滤方案。

​		有些时候我们并不是希望所有的 Rest API都呈现在文档上，这种情况下 Swagger2 提供给我们了两种方式配置，一种是基于 `@ApiIgnore` 注解（另一种是在 Docket 上增加筛选。两种方式的区别是，Docket 配置的规则，可以对多个接口器过滤作用，而 `@ApiIgnore` 只能作用于单个接口。

​		这里我们先来看第二种，这种方式可以通过筛选 API 的 url 来进行过滤；通过 ApiSelectorBuilder#paths() 方法配置，在 `PathSelectors` 提供了配置方案：

![image-20220724024257438](E:\Development\Typora\images\image-20220724024257438.png)



- any()：任何路径都满足条件
- none()：任何路径都不满足条件
- regex()：通过正则表达式控制
- ant()：通过 ant 控制

好了，现在我们来看一下在 SwaggerConfig 中该怎样配置：

```java
return new Docket(DocumentationType.SWAGGER_2) //
        .groupName("接口Api")
        .apiInfo(productApiInfo())
        .select()
        .apis(RequestHandlerSelectors.any()) // 对所有api进行监控
        .paths(Predicates.not(PathSelectors.regex("/error.*")))// 错误路径不监控
        .paths(PathSelectors.regex("/.*"))// 对根下所有路径进行监控
        .build();
```



#### ④groupName 分组

`groupName` 就是上面说的右上角的分组选项，一般项目中不同的开发人员，可以创建不同的分组，默认的分组是 default。所以，我们可以通过配置多个 `Docket` 去实现分组

```java
@Bean
public Docket docket2() {
    return new Docket(DocumentationType.SWAGGER_2)
        .groupName("打工人2组");
}

@Bean
public Docket docket3() {
    return new Docket(DocumentationType.SWAGGER_2)
        .groupName("打工人3组");
}
```

![image-20220724025105607](E:\Development\Typora\images\image-20220724025105607.png)



#### ⑤useDefaultResponseMessages 默认状态码

点开接口文档中的接口，可以看见，在 Respon 中 Swagger 默认提供了 200,401,403,404 这几个状态码

![image-20220724025440407](E:\Development\Typora\images\image-20220724025440407.png)

但是，在我们实际开发中大多数都是自定义状态码，所以，就可以通过 `useDefaultResponseMessages(false)` 关闭默认状态码。配置如下：

```java
@Bean // 构建并配置 Docket 对象
public Docket docket() {
    return new Docket(DocumentationType.SWAGGER_2)
        .apiInfo(apiInfo())
        .useDefaultResponseMessages(false)
        .select() // 返回 ApiSelectorBuilder
        .apis(RequestHandlerSelectors.basePackage("com.hong.dk.bookcollect.controller")) // 接口扫描方案
        .paths(PathSelectors.any()) // 接口过滤方案
        .build();
```

重启项目后，可以看到，原先的 401,403,404 没有了

![image-20220724025744060](E:\Development\Typora\images\image-20220724025744060.png)

#### ⑥enabled 是否启动 Swagger

我们在开发、测试时候需要启动 Swagger，但是在实际项目发布上线了就要关闭它，因为一旦一些重要的接口暴露是很危险的，而且一直运行着 Swagger 也会浪费系统资源。

所以可以通过 Docket#enable(false) 来关闭 Swagger，但是如果每次都手动操作显得有些笨拙，我们可以根据当前项目的环境来决定是否开启 Swagger。

法一：

将[swagger](https://so.csdn.net/so/search?q=swagger&spm=1001.2101.3001.7020)限制在开发环境中使用，通过`@ConditionalOnProperty`实现。

```java
@Configuration
@EnableSwagger2
@ConditionalOnProperty(prefix = "spring.profiles", name = "active", havingValue = "dev")
public class SwaggerConfig {
    @Bean
    public Docket userApi() {
        ...
    }
}
```

applicaiton.yml配置如下：

```yml
spring:
  profiles:
    active: dev
```



法二：

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {

    @Value("${swagger.enable}")
    private boolean swagger_enable;

    @Bean(value = "userApi")
    @Order(value = 1)
    public Docket groupRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .enable(swagger_enable)
                .apiInfo(groupApiInfo())
                .select()
                //.apis(RequestHandlerSelectors.basePackage(Swagger2Describe.BASE_PACKAGE))
                .apis(RequestHandlerSelectors.withMethodAnnotation(ApiOperation.class))
                .paths(PathSelectors.any())
                .build();
    }
```

application.yml配置文件：

```yml
#swagger是否激活
swagger:
  enable: true
```

你应该懂我意思吧

#### ⑦globalOperationParameters

添加将应用于所有操作的默认参数，将全局应用于所有操作的参数

配置全局token（放在header里面）

```java
    //添加head参数配置
    ParameterBuilder tokenPar = new ParameterBuilder();
    List<Parameter> pars = new ArrayList<Parameter>();
    tokenPar.name("token").description("令牌").modelRef(new ModelRef("string")).parameterType("header").required(false).build();
    pars.add(tokenPar.build());
    return new Docket(DocumentationType.SWAGGER_2) //
            .groupName("郭浩")
            .apiInfo(productApiInfo())
            .useDefaultResponseMessages(false)
            .select()
            .apis(RequestHandlerSelectors.any()) // 对所有api进行监控
            .paths(Predicates.not(PathSelectors.regex("/error.*")))// 错误路径不监控
            .paths(PathSelectors.regex("/.*"))// 对根下所有路径进行监控
            .build()
            .globalOperationParameters(pars); // 增加全局参数
}
```

在上面的`ModelRef`对象里面可以配置的有

![image-20220724031619609](E:\Development\Typora\images\image-20220724031619609.png)



```java
@Bean(value = "publicApi")
public Docket publicApi() {
    return new Docket(DocumentationType.SWAGGER_2)
            .enable(isEnable)
            .apiInfo(apiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("project.api.pub"))
            .paths(PathSelectors.any())
            .build().groupName("无需token验证").ignoredParameterTypes(HttpServletResponse.class, HttpServletRequest.class);
}
```



[【Swagger】常用注解及具体实例（超全）_A minor的博客-CSDN博客_swagger 参数示例](https://blog.csdn.net/weixin_43935927/article/details/113401678)



## 3.使用Swagger注解

### 1.用于类的注解

#### @[Api](https://so.csdn.net/so/search?q=Api&spm=1001.2101.3001.7020)：资源描述

标识这个类是 Swagger 的资源

```java
@Api(tags = "书籍信息接口")
public class BookController {
}
```

可以设置的属性如下：

![image-20220724033402371](E:\Development\Typora\images\image-20220724033402371.png)

- values：字符串；说明该类的作用，在类旁边的小字显示

- **tags**：字符串；标签（也可理解成分类），会替换 Controller 名称；当多个 Controller 的 tags 相同时，它们的方法会在一起显示

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129221144467.png#pic_center)

- consumes：字符串；指定处理请求的提交内容类型(Content-Type)，例如 application/json

- produces：字符串；指定返回的内容类型，即仅当请求头的 Accept 类型中包含该指定类型才返回，例如:application/json

- protocols：字符串；标识当前的请求支持的协议，例如：http`、`https、ws

- hidden：true/false；隐藏整个 Controller，作用与 @ApiIgnore 相似，但没有 @ApiIgnore 功能强大

#### @ApiIgnore：资源过滤

有些时候我们并不是希望所有的 Rest API 都呈现在文档上，这种情况下 Swagger2 提供给我们了两种方式配置，一种是基于 @`ApiIgnore` 注解另一种是在 Docket 上增加筛选。两种方式的区别是，Docket 配置的规则，可以对多个接口器过滤作用，而 @`ApiIgnore` 只能作用于单个接口。

![image-20220724033617922](E:\Development\Typora\images\image-20220724033617922.png)

![image-20220724033630026](E:\Development\Typora\images\image-20220724033630026.png)

如果想在文档中屏蔽掉删除用户的接口（user/delete），那么只需要在删除用户的方法上加上 `@ApiIgnore` 即.可。

### 2.用于方法的注解

#### @ApiOperation：方法描述

```java
@ApiOperation(value = "生成订单信息")
```

![image-20220724033811379](E:\Development\Typora\images\image-20220724033811379.png)

- **value**：字符串；方法摘要，在路径旁显示

- **note**：字符串；方法详细描述
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129230141804.png#pic_center)

- tags：字符串数组；对方法进行分类，一个方法可以有多个分类

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129221244328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzkzNTkyNw==,size_16,color_FFFFFF,t_70#pic_center)

- response：Class；设置当前请求的返回值类型，String.class；会覆盖自动识别的返回类型，一般用不上

- responseContainer：字符串；说明包装的容器，默认情况下，有效值为 List、Set、Map；在定义通用 Response 后，一般用不上

- httpMethod：字符串；指定请求方式，比如 GET、POST、PUT

- consumes：字符串；指定处理请求的提交内容类型(Content-Type)，例如 application/json

- produces：字符串；指定返回的内容类型，即仅当请求头的 Accept 类型中包含该指定类型才返回，例如:application/json

效果如下：![@ApiOperation 示例](E:\Development\Typora\images\03-16594753482047.png)



#### @ApiImplicitParam(s)：参数描述

参数描述，可用在方法头。

ApiImplicitParams 只有一个属性 `ApiImplicitParam[] value();`是 ApiImplicitParam 的数组，比如

```java
@ApiImplicitParams({
        @ApiImplicitParam(name = "id", value = "商户ID")
})
@GetMapping("/{id}")
public Response getMerchantsInfo(@PathVariable Integer id) {
    return null;
}
```

![image-20220724033935126](E:\Development\Typora\images\image-20220724033935126.png)

- **name**：字符串；参数名
- **value**：字符串；参数的汉字说明、解释
- defaultValue：字符串；参数的默认值
- allowableValues：字符串；限制此参数接收的值，可使用的值或值得范围
  - `(`表示是大于，`)`表示是小于
  - `[`表示是大于等于，`]`表示是小于等于
  - `infinity`表示无限大，`-infinity`表示负无限大
- allowEmptyValue：true/false；允许参数为空，默认为 false
- **required**：true/false；参数是否必须传
- **paramType**：字符串；参数放在哪个地方（可以自动识别）
- header --> 参数在request headers 里边提交：@RequestHeader
  - path（用于restful接口）–> 参数以地址的形式提交：@PathVariable
  - query --> 直接跟参数完成自动映射赋值：@RequestParam
  - form --> 以form表单的形式提交，仅支持POST：@RequestParam
  - body --> 以流的形式提交，仅支持POST：@RequestBody
- **dataType**：字符串；参数类型，参数的数据类型（默认String），可以使用类名或原始数据类型（使用不当汇报类型转换异常）
- dataTypeClass：Class；参数的类，如果提供则覆盖 dataType
- example：字符串；非请求体（body）参数的**单个请求**示例
- examples：Example；参数的举例说明，仅适用于 body 类型。

效果如下：![@ApiImplicitParam 示例](E:\Development\Typora\images\04-16594752894263.png)

#### @ApiParam：参数描述

参数描述，用在每个参数前面，比如

```java
@PostMapping("/create")
// 注：这里如果使用 @ApiImplicitParam 会出现无法识别的问题
public Response createMerchants(
        @ApiParam(name = "request", value = "创建商户请求对象") @RequestBody CreateMerchantsRequest request) 
```

![image-20220724034015650](E:\Development\Typora\images\image-20220724034015650.png)

- **name**：字符串；参数名
- **value**：字符串；参数的汉字说明、解释
- defaultValue：字符串；参数默认值
- allowableValues：字符串；限制此参数接收的值，可使用的值或值得范围
  - `(`表示是大于，`)`表示是小于
  - `[`表示是大于等于，`]`表示是小于等于
  - `infinity`表示无限大，`-infinity`表示负无限大
- allowEmptyValue：true/false；允许参数为空，默认为 false
- **required**：true/false；参数是否必须传
- example：字符串；非请求体（body）参数的**单个请求**示例
- examples：Example；参数的举例说明，仅适用于 body 类型。

> 关于 @ApiImplicitParam 和 @ApiParm 它俩都能为方法的请求参数做注释，区别有：
>
> 1. @ApiImplicitParam 可以在方法前使用，而 @ApiParm 只能在参数前使用
> 2. @ApiImplicitParam 提供的可设置属性更多，特别是 **paramType** 很关键
> 3. 对于 @RequestBody 标识的 Json 字符串，不能使用 @ApiImplicitParam，会出现无法识别的情况
>    1. 方案一：通过 @ApiParm 对 @RequestBody 的参数进行说明
>    2. 方案二：直接不对该参数使用注解，将注释的任务交给实体类（@ApiModel）
>
> 另外，两者是可以混搭使用的，一般推荐使用 @ApiImplicitParam(s)，但是注意 @RequestBody 的情况。



#### @ApiResponse(s)

跟上面 @ApiImplicitParam(s) 一样，@ApiResponses 也是只有唯一个属性`ApiResponse[] value();`

```java
@ApiResponses({
    @ApiResponse(code = 1, message = "非HTTP状态码，Response code字段值，描述：成功，返回该商户ID"),
    @ApiResponse(code = 0, message = "非HTTP状态码，Response code字段值，描述：失败")
})
@PostMapping("/create")
public Response createMerchants

   		 -------------------------------------------------
    @ApiResponses({
        @ApiResponse(code = 1, message = "非HTTP状态码，Response code字段值，描述：成功，返回该商户ID"),
        @ApiResponse(code = 0, message = "非HTTP状态码，Response code字段值，描述：失败")
    })
```

![image-20220724034051143](E:\Development\Typora\images\image-20220724034051143.png)



### 3.用于实体类的注解

#### @ApiModel：实体类描述

用于请求类或者响应类上，表示一个返回响应数据的信息

```java
@ApiModel("创建商户的请求对象")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CreateMerchantsRequest 
12345
```

@ApiModel 可以设置的属性有：

![在这里插入图片描述](E:\Development\Typora\images\20210129221522847.png#pic_center)

- **value**：字符串；实体类的备用名，如果不设置，则会采用原类名
- description：字符串；实体类的说明
- parent：Class；父类的一些信息

效果如下：![@ApiModel 示例](E:\Development\Typora\images\05-16594753079885.png)

#### @ApiModelProperty：实体类成员描述

```java
@ApiModel("创建商户的请求对象")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CreateMerchantsRequest {

    @ApiModelProperty("商户名")
    private String name;

    @ApiModelProperty("商户logo的URL")
    private String logoUrl;

    @ApiModelProperty("营业执照URL")
    private String businessLicenseUrl;

    @ApiModelProperty("联系电话")
    private String phone;

    @ApiModelProperty("地址")
    private String address;
}
123456789101112131415161718192021
```

ApiModelProperty 可以设置的属性有：

![在这里插入图片描述](E:\Development\Typora\images\20210129221532910.png#pic_center)

- **value**：字符串；字段说明
- name：字符串；重写字段名称
- dataType：字符串；重写字段类型
- required：true/false；是否必填
- allowableValues：字符串；限制此参数接收的值，可使用的值或值得范围
  - `(`表示是大于，`)`表示是小于
  - `[`表示是大于等于，`]`表示是小于等于
  - `infinity`表示无限大，`-infinity`表示负无限大
- allowEmptyValue：true/false；允许参数为空，默认为 false
- example：String；示例
- hidden：true/false；是否在文档中隐藏该字段

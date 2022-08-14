# knife4j

https://doc.xiaominfo.com/knife4j/

# 1.快速开始

第一步：在maven项目的`pom.xml`中引入Knife4j的依赖包，代码如下：

```xml
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <version>2.0.7</version>
</dependency>
```

第二步：创建Swagger配置依赖，代码如下：

```java
@Configuration
@EnableSwagger2WebMvc
public class Knife4jConfiguration {

    @Bean(value = "defaultApi2")
    public Docket defaultApi2() {
        Docket docket=new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(new ApiInfoBuilder()
                        //.title("swagger-bootstrap-ui-demo RESTful APIs")
                        .description("# swagger-bootstrap-ui-demo RESTful APIs")
                        .termsOfServiceUrl("http://www.xx.com/")
                        .contact("xx@qq.com")
                        .version("1.0")
                        .build())
                //分组名称
                .groupName("2.X版本")
                .select()
                //这里指定Controller扫描包路径
                .apis(RequestHandlerSelectors.basePackage("com.github.xiaoymin.knife4j.controller"))
                .paths(PathSelectors.any())
                .build();
        return docket;
    }
}
```

`IndexController.java`包含一个简单的RESTful接口,代码示例如下：

```java
@Api(tags = "首页模块")
@RestController
public class IndexController {

    @ApiImplicitParam(name = "name",value = "姓名",required = true)
    @ApiOperation(value = "向客人问好")
    @GetMapping("/sayHi")
    public ResponseEntity<String> sayHi(@RequestParam(value = "name")String name){
        return ResponseEntity.ok("Hi:"+name);
    }
}
```

此时，启动Spring Boot工程，在浏览器中访问：`http://localhost:17790/doc.html`



https://doc.xiaominfo.com/knife4j/



# 2.增强特性

以下所有增强功能都需要通过配置yml配置文件开启增强、

```yml
knife4j:
  enable: true
```

## 列举一些我已经用到的

## 2.1访问权限控制

### 2.1.1生产环境屏蔽资源

在开发`Knife4j`功能时,同很多开发者经常讨论的问题就是在生产环境时,屏蔽或者去除Swagger的文档很麻烦

,通常有时候我们碰到的问题如下：

- 系统部署生产环境时,我们想屏蔽Swagger的文档功能,不管是接口或者html文档
- 通常我们有时候需要生产环境部署后,又需要Swagger的文档调试功能,辅助开发者调试,但是存在安全隐患,没有对Swagger的资源接口过滤
- 等等

目前`Springfox-Swagger`以及`Knife4j`提供的资源接口包括如下：

| 资源                                      | 说明                                          |
| ----------------------------------------- | --------------------------------------------- |
| /doc.html                                 | Knife4j提供的文档访问地址                     |
| /v2/api-docs-ext                          | Knife4j提供的增强接口地址,自`2.0.6`版本后删除 |
| /swagger-resources                        | Springfox-Swagger提供的分组接口               |
| /v2/api-docs                              | Springfox-Swagger提供的分组实例详情接口       |
| /swagger-ui.html                          | Springfox-Swagger提供的文档访问地址           |
| /swagger-resources/configuration/ui       | Springfox-Swagger提供                         |
| /swagger-resources/configuration/security | Springfox-Swagger提供                         |

当我们部署系统到生产系统,为了接口安全,需要屏蔽所有Swagger的相关资源

如果使用SpringBoot框架,只需在`application.properties`或者`application.yml`配置文件中配置

```yml
knife4j:
  # 开启增强配置 
  enable: true
　# 开启生产环境屏蔽
  production: true
```

配置此属性后,所有资源都会屏蔽输出。

### 2.1.2访问页面加权控制

不管是官方的`swagger-ui.html`或者`doc.html`,目前接口访问都是无需权限即可访问接口文档的,很多朋友以前问我能不能提供一个登陆界面的功能,开发者输入用户名和密码来控制界面的访问,只有知道用户名和密码的人才能访问此文档

做登录页控制需要有用户的概念,所以相当长一段时间都没有提供此功能

针对Swagger的资源接口,`Knife4j`提供了简单的**Basic认证功能**

效果图如下：

允许开发者在配置文件中配置一个静态的用户名和密码,当对接者访问Swagger页面时,输入此配置的用户名和密码后才能访问Swagger文档页面,如果您使用SpringBoot开发,则只需在相应的`application.properties`或者`application.yml`中配置如下：

效果图如下：

![image-20220729162457578](E:\Development\Typora\images\image-20220729162457578.png)

```yml
knife4j:
  # 开启增强配置 
  enable: true
　# 开启Swagger的Basic认证功能,默认是false
  basic:
      enable: true
      # Basic认证用户名
      username: test
      # Basic认证密码
      password: 123
```

如果用户开启了basic认证功能,但是并未配置用户名及密码,`Knife4j`提供了默认的用户名和密码：

```text
admin/123321
```

## 2.2接口添加作者

有时候在开发接口时,我们希望给该接口添加一个作者,这样前端或者别个团队来对接该接口时,如果该接口返回的数据或者调用有问题,都能准确找到该人,提升效率

添加作者需要使用`knife4j`提供的增强注解`@ApiOperationSupport`

这个是用在方法上的

接口代码示例如下：

```java
@ApiOperationSupport(author = "xiaoymin@foxmail.com")
@ApiOperation(value = "写文档注释我是认真的")
@GetMapping("/getRealDoc")
public Rest<RealDescription> getRealDoc(){
    Rest<RealDescription> r=new Rest<>();
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    r.setData(new RealDescription());
    return r;
}
```

还有一个`@ApiSupport`注解,该注解目前有两个属性,分别是author(作者)和order(排序)

使用代码示例：

```java
@Api(tags = "2.0.3版本-20200312")
@ApiSupport(author = "xiaoymin@foxmail.com",order = 284)
@RestController
@RequestMapping("/api/nxew203")
public class Api203Constroller {
    
    
}
```

![img](https://doc.xiaominfo.com/knife4j/images/2-0-2/debug-3.png)
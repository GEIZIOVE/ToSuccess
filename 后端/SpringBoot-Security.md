## 0. 简介

 **Spring Security** 是 Spring 家族中的一个安全管理[框架](https://so.csdn.net/so/search?q=框架&spm=1001.2101.3001.7020)。相比与另外一个安全框架**Shiro**，它提供了更丰富的功能，社区资源也比[Shiro](https://so.csdn.net/so/search?q=Shiro&spm=1001.2101.3001.7020)丰富。

 一般来说中大型的项目都是使用**SpringSecurity** 来做安全框架。小项目有Shiro的比较多，因为相比与SpringSecurity，Shiro的上手更加的简单。

 一般Web应用的需要进行**认证**和**授权**。

 **认证：验证当前访问系统的是不是本系统的用户，并且要确认具体是哪个用户**

 **授权：经过认证后判断当前用户是否有权限进行某个操作**

 而认证和授权也是SpringSecurity作为安全框架的核心功能。

## 1. 快速入门

### 1.1 准备工作

 我们先要搭建一个简单的SpringBoot工程

① 设置父工程 添加依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

```

② 创建启动类

```java
@SpringBootApplication
public class SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class,args);
    }
}


```

③ 创建Controller

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}


```

### 1.2 引入SpringSecurity

 在SpringBoot项目中使用SpringSecurity我们只需要引入依赖即可实现入门案例。

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

 引入依赖后我们在尝试去访问之前的接口就会自动跳转到一个SpringSecurity的默认登陆页面，默认用户名是user,密码会输出在控制台。

 必须登陆之后才能对接口进行访问。

## 2. 认证

### 2.1 登陆校验流程

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCn5LiK5pyJ5Yac,size_20,color_FFFFFF,t_70,g_se,x_16.png)

### 2.2 原理初探

 想要知道如何实现自己的登陆流程就必须要先知道入门案例中SpringSecurity的流程。

#### 2.2.1 SpringSecurity完整流程

 SpringSecurity的原理其实就是一个过滤器链，内部包含了提供各种功能的过滤器。这里我们可以看看入门案例中的过滤器。

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCn5LiK5pyJ5Yac,size_20,color_FFFFFF,t_70,g_se,x_16-16578743320411.png)

 图中只展示了核心过滤器，其它的非核心过滤器并没有在图中展示。

**UsernamePasswordAuthenticationFilter**:负责处理我们在登陆页面填写了用户名密码后的登陆请求。入门案例的认证工作主要有它负责。

**ExceptionTranslationFilter：** 处理过滤器链中抛出的任何AccessDeniedException和AuthenticationException 。

**FilterSecurityInterceptor：** 负责权限校验的过滤器。



 我们可以通过Debug查看当前系统中SpringSecurity过滤器链中有哪些过滤器及它们的顺序。

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCn5LiK5pyJ5Yac,size_20,color_FFFFFF,t_70,g_se,x_16-16578743320412.png)

#### 2.2.2 认证流程详解

![在这里插入图片描述](E:\Development\Typora\images\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCn5LiK5pyJ5Yac,size_20,color_FFFFFF,t_70,g_se,x_16-16578743320413.png)

概念速查:

Authentication接口: 它的实现类，表示当前访问系统的用户，封装了用户相关信息。

AuthenticationManager接口：定义了认证Authentication的方法

UserDetailsService接口：加载用户特定数据的核心接口。里面定义了一个根据用户名查询用户信息的方法。

UserDetails接口：提供核心用户信息。通过UserDetailsService根据用户名获取处理的用户信息要封装成UserDetails对象返回。然后将这些信息封装到Authentication对象中。
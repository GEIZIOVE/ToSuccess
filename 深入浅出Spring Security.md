深入浅出Spring Security



# 第二章 Spring Security 认证

## 2.1 Spring Security基本认证

### 2.2.1 快速入门

引入Web和Spring Security 依赖即可

```xml
<dependency>
    <groupid>org.springframework.boot</groupid>
    <artifactid>spring-boot-starter-security</artifactid>
</dependency> 
<dependency> 
    <groupid>org.springframework.boot</groupid>
    <artifactid>spring-boot-starter-web</artifactid>
</dependency> 
```



此后，项目的所有接口都被保护起来了。当用户访问任一接口时，都会自动跳转到登录页面，登录成功后才能访问到接口。

![image-20220716144500396](E:\Development\Typora\images\image-20220716144500396.png)

​			默认的登录用户名是user，登录密码则是一个随机生成的UUID字符串，在项目启动日志
中可以看到登录密码（这也意味着项目每次启动时， 密码都会发生变化）：

```
Using generated security password: 8ef9c800-17cf-47a3-9984-8ff936db6dd8
```

​		输入默认的用户名和密码，就可以成功登录了。这就是Spring Security的强大之处， 只需
要引入一个依赖，所有的接口就会被自动保护起来。



### 2.1.2 流程分析

![image-20220716144707203](E:\Development\Typora\images\image-20220716144707203.png)



​	流程图比较清晰地说明了整个请求过程：

​	(1）客户端（浏览器）发起请求去访问/hello 接口， 这个接口默认是需要认证之后才能访问的

​	(2）这个请求会走遍 Spring Security中的过滤器链， 在最后的 FilterSecu1ityInterceptor 过滤器中被拦截下来， 因为系统发现用户未认证 。 请求拦截下来之后， 接下来会抛出AccessDeniedException 异常。

​	(3）抛出的 AccessDeniedException 异常在 ExceptionTranslationFilter 过滤器中被捕获， ExceptionTranslationFilter 过滤器通过调用 LoginUrlAuthenticationEntiyPoint#commence 方法给客户端返回 302，要求客户端重定向到／login 页面。

​	(4）	客户端发送／login 请求。

​	(5)	/login 请求被 DefaultLoginPageGeneratingFilter 过滤器拦截下来， 并在该过滤器中返回登录页面。 所以当用户访问／hello 接口时会首先看到登录页面。



### 2.1.3 原理分析

​	在上面，我们就引入了一个依赖，代码不多，但是SpringBoot背后默默做了很多事情：	

- 开启 Sp1ing Secu1ity 自动化配置， 开启后，会自动创建一个名为 springSecmityFilterChain 的过滤器， 并注入到 Spring 容器中， 这个过滤器将负责所有的安全管理， 包括用户的认证、授权、 重定向到登录页面等（ springSecmityFilterChain 实际上代理了 Spring Secu1ity中的过滤器链
- •	创建一个 UserDetailsSe1vice 实例， UserDetailsSe1vice 负责提供用户数据，默认的用户数据是基于内存的用户， 用户名为 user，密码则是随机生成的 UUID 字符串。
  •	给用户生成一个默认的登录页面。
  •	开启 CSRF 攻击防御。
  •	开启会话固定攻击防御。
  •	集成 X-XSS-Protection。
  •	集成 X-Frame-Options 以防止单击劫持。



#### 2.1.3.1 默认用户生成

​		Spring Security 中定义了 UserDetails接口来规范开发者自定义的用户对象.

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();   //返回当前账户所具备的权限。
    String getPassword();  //返回当前账户的密码。
    String getUsername();  //返回当前账户的用户名。
    boolean isAccountNonExpired();  //返回当前账户是否未过期。
    boolean isAccountNonLocked();  //返回当前账户是否未锁定。
    boolean isCredentialsNonExpired();  //返回当前账户凭证（如密码）是否未过期。
    boolean isEnabled();  //返回当前账户是否可用。
}
```



​	这是用户对 象的定义， 而负责提供用户数据源的接口是 UserDetailsSer叮vice,U serDetailsService 中只有一个用户查询的方法，代码如下：

```java
public interface UserDetailsService {
	UserDetails loadUserByUsername(String username)
				throws UsernameNotFoundException;
}
```



在实际项目中，一般需要开发者自定义UserDetailsService的实现。当然，如果开发者没有自定义UserDetailsService的实现，Spring Security也为UserDetailsService提供了默认实现。

![image-20220716154315804](E:\Development\Typora\images\image-20220716154315804.png)

•	UserDetailsManager 在 UserDetailsSe1vice 的基础上， 继续定义了添加用户、 更新用户、 删除用户、修改密码以及判断用户是否存在共 5 种方法。
•	JdbcDaoImpl 在 UserDetailsService 的基础上， 通过 spring-jdbc 实现了从数据库中查询用 户的方法。
•	InMemoryUserDetailsManager  实现了 UserDetailsManager 中关于用户的增删改查方法，不 过都是基于内存的操作， 数据并没有持久化。
•	JdbcUserDetailsManager 继承自 JdbcDaolmpl 同时又实现了 UserDetailsManager 接口， 因此可以通过 JdbcU serDetailsManager 实现对用户的增删改查操作， 这些操作都会持久化 到数据库中。不过 JdbcUserDetailsManager 有一个局限性，就是操作数据库中用户的 SQL都是提前写好的， 不够灵活， 因此在实际开发中 JdbcUserDetailsManager 使用并不多。
•	CachingUserDeta.ilsSe1vice的特点是会将 UserDetailsService 缓存起来。
•	UserDetailsServiceDelegator 则是提供了 UserDetailsSe1vice 的懒加载功能。
•	ReactiveU serDeta.ilsSe1viceAdapter 是 webflux-web-security 模块定义的 UserDetailsService 实现。



当我们使用 Spring Security 时， 如果仅仅只是引入一个 Spring Security 依赖， 则默认使用的用户就是由 InMemoryUserDetailsManager 提供的。

这里有源码讲解



#### 2.1.3.2 默认页面生成

​		在 1.3.2 小节中， 我们介绍了 Spring Security 中常见的过滤器， 在这些常见的过滤器中就包含两个和页面相关的过滤器： DefaultLoginPageGeneratingFilter 和 DefaultLogoutPageGeneratingFilter。
​		通过过滤器的名字就可以分辨出 DefaultLoginPageGeneratingFilter 过滤器用来生成默认的登录页面， DefaultLogoutPageGeneratingFilter 过滤器则用来生成默认的注销页面。

核心源码









## 2.2 登录表单配置

### 2.2.1快速入门

​		引入依赖web和security

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```



​		项目创建好之后， 为了方便测试， 需要在 application.properties 中添加如下配置， 将登录用户名和密码固定下来：

```properties
spring.security.user.name=wqh
spring.security.user.password=root
```






















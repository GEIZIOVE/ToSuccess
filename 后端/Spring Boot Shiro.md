# Apache Shiro

## 简介

[Apache Shiro](http://shiro.apache.org/)（发音为`shee-roh`，日语堡垒（Castle）的意思）是一个强大易用的Java安全框架，提供了认证、授权、加密和会话管理功能，可为任何应用提供安全保障 - 从命令行应用、移动应用到大型网络及企业应用。相较于Spring Security来说较为简单，易于上手。

Apache Shiro有三个核心的概念Subject，SecurityManager和Realms，如下图所示：

![sdfjasdr3857312-323412.png](E:\Development\Typora\images\sdfjasdr3857312-323412.png)

**1、**[Subject](http://shiro.apache.org/static/1.3.2/apidocs/org/apache/shiro/subject/Subject.html)：主体，代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等，即一个抽象概念。所有Subject 都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager。可以把Subject认为是一个门面，SecurityManager才是实际的执行者。

在shiro中通过`org.apache.shiro.SecurityUtils`类来获取Subject对象：

```java
import org.apache.shiro.subject.Subject;
import org.apache.shiro.SecurityUtils;
...
Subject currentUser = SecurityUtils.getSubject();
```



更多关于Subject的信息可访问http://shiro.apache.org/static/1.3.2/apidocs/org/apache/shiro/subject/Subject.html

**2、**[SecurityManager](http://shiro.apache.org/static/1.3.2/apidocs/org/apache/shiro/mgt/SecurityManager.html)：安全管理器，即所有与安全有关的操作都会与SecurityManager交互，且它管理着所有Subject，可以看出它是Shiro的核心。它负责与后边介绍的其他组件进行 交互，类似于Spring MVC中的DispatcherServlet前端控制器。

**3、**[Realm](http://shiro.apache.org/static/1.3.2/apidocs/org/apache/shiro/realm/Realm.html)：域，Shiro从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法。 也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作。

简而言之，创建一个基本的Shiro应用过程为：

- 应用代码通过Subject来进行认证和授权，而Subject又委托给SecurityManager；
- 我们需要给Shiro的SecurityManager注入Realm，从而让SecurityManager能得到合法的用户及其权限进行判断。

Shiro并没有为我们提供Realm的实现，需要我们手动编写实现。基本过程为继承`org.apache.shiro.realm.AuthorizingRealm`抽象类，实现doGetAuthorizationInfo和doGetAuthenticationInfo方法。

了解了Shiro的核心组件后，接下来看看Shiro为我们带来了哪些功能模块：

![ShiroFeatures.png](https://mrbird.cc/img/ShiroFeatures.png)

Shiro提供了四大基本安全功能：认证，授权，会话管理和加密。

- **身份验证(Authentication)**：也称为登录验证，即验证用户名和密码是否正确；
- **授权(Authorization)**：根据用户的角色和权限来控制用户可访问的资源；
- **会话管理(Session Management)**：即使在非Web或EJB应用程序中，也可以管理用户特定的SESSION会话；
- **密码学(Cryptography)**：使用加密算法保证数据安全，同时易于使用。

除此之外，Shiro也支持以下特性：

- **Web支持(Web Support)**：Shiro提供的web程序API可以帮助轻松保护Web应用程序；
- **缓存(Caching)**：缓存可确保安全验证操作保持快速高效；
- **并发性(Concurrency)**：Apache Shiro支持具有并发功能的多线程应用程序；
- **测试(Testing)**：测试API帮助您编写单元测试和集成测试；
- **运行方式(Run As)**：允许用户以别的用户身份（如果允许）登录；
- **记住我(Remember Me)**：在会话中记住用户的身份，只有在强制登录时才需要登录。

> 参考自:

> http://shiro.apache.org/introduction.html

> http://www.infoq.com/cn/articles/apache-shiro



## 1.Spring Boot Shiro用户认证

原文《Spring-Boot-shiro用户认证》](https://mrbird.cc/Spring-Boot-shiro Authentication.html)

在Spring Boot中集成Shiro进行用户的认证过程主要可以归纳为以下三点：

1、定义一个ShiroConfig，然后配置SecurityManager Bean，SecurityManager为Shiro的安全管理器，管理着所有Subject；

2、在ShiroConfig中配置ShiroFilterFactoryBean，其为Shiro过滤器工厂类，依赖于SecurityManager；

3、自定义Realm实现，Realm包含`doGetAuthorizationInfo()`和`doGetAuthenticationInfo()`方法，因为本文只涉及用户认证，所以只实现`doGetAuthenticationInfo()`方法。



### 引入依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.2.12.RELEASE</version>
</dependency>
<!--mybatis-plus 持久层-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1</version>
</dependency>
<!--mysql驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
</dependency>
<!-- thymeleaf -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.shiro/shiro-spring -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.6</version>
</dependency>
 <!--  lombok      -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```



### ShiroConfig

定义一个Shiro配置类，名称为ShiroConfig：

```java

import com.hong.springbootshiro.realm.ShiroRealm;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;

@Configuration
public class ShiroConfig {
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置securityManager
        shiroFilterFactoryBean.setSecurityManager((SecurityManager) securityManager);
        // 登录的url
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后跳转的url
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 未授权url
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");

        LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<>();

        // 定义filterChain，静态资源不拦截
        filterChainDefinitionMap.put("/css/**", "anon");
        filterChainDefinitionMap.put("/js/**", "anon");
        filterChainDefinitionMap.put("/fonts/**", "anon");
        filterChainDefinitionMap.put("/img/**", "anon");
        // druid数据源监控页面不拦截
        filterChainDefinitionMap.put("/druid/**", "anon");
        // 配置退出过滤器，其中具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/logout", "logout");
        filterChainDefinitionMap.put("/", "anon");
        // 除上以外所有url都必须认证通过才可以访问，未通过认证自动访问LoginUrl
        filterChainDefinitionMap.put("/**", "authc");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }

    @Bean
    public SecurityManager securityManager(){
        // 配置SecurityManager，并注入shiroRealm
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        securityManager.setRealm(shiroRealm());
        return securityManager;
    }

    @Bean
    public ShiroRealm shiroRealm(){
        // 配置Realm，需自己实现
        ShiroRealm shiroRealm = new ShiroRealm();
        return shiroRealm;
    }
}
```



需要注意的是filterChain基于短路机制，即最先匹配原则，如：

```java
/user/**=anon
/user/aa=authc 永远不会执行
```

其中`anon`、`authc`等为Shiro为我们实现的过滤器，具体如下表所示：

| Filter Name       | Class                                                        | Description                                                  |
| :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| anon              | [org.apache.shiro.web.filter.authc.AnonymousFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/AnonymousFilter.html) | 匿名拦截器，即不需要登录即可访问；一般用于静态资源过滤；示例`/static/**=anon` |
| authc             | [org.apache.shiro.web.filter.authc.FormAuthenticationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/FormAuthenticationFilter.html) | 基于表单的拦截器；如`/**=authc`，如果没有登录会跳到相应的登录页面登录 |
| authcBasic        | [org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/BasicHttpAuthenticationFilter.html) | Basic HTTP身份验证拦截器                                     |
| logout            | [org.apache.shiro.web.filter.authc.LogoutFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/LogoutFilter.html) | 退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/），示例`/logout=logout` |
| noSessionCreation | [org.apache.shiro.web.filter.session.NoSessionCreationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/session/NoSessionCreationFilter.html) | 不创建会话拦截器，调用`subject.getSession(false)`不会有什么问题，但是如果`subject.getSession(true)`将抛出`DisabledSessionException`异常 |
| perms             | [org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/PermissionsAuthorizationFilter.html) | 权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例`/user/**=perms["user:create"]` |
| port              | [org.apache.shiro.web.filter.authz.PortFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/PortFilter.html) | 端口拦截器，主要属性`port(80)`：可以通过的端口；示例`/test= port[80]`，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样 |
| rest              | [org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/HttpMethodPermissionFilter.html) | rest风格拦截器，自动根据请求方法构建权限字符串；示例`/users=rest[user]`，会自动拼出user:read,user:create,user:update,user:delete权限字符串进行权限匹配（所有都得匹配，isPermittedAll） |
| roles             | [org.apache.shiro.web.filter.authz.RolesAuthorizationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/RolesAuthorizationFilter.html) | 角色授权拦截器，验证用户是否拥有所有角色；示例`/admin/**=roles[admin]` |
| ssl               | [org.apache.shiro.web.filter.authz.SslFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/SslFilter.html) | SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口443；其他和port拦截器一样； |
| user              | [org.apache.shiro.web.filter.authc.UserFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/UserFilter.html) | 用户拦截器，用户已经身份验证/记住我登录的都可；示例`/**=user` |

配置完ShiroConfig后，接下来对Realm进行实现，然后注入到SecurityManager中。

### Realm

自定义Realm实现只需继承AuthorizingRealm类，然后实现doGetAuthorizationInfo()和doGetAuthenticationInfo()方法即可。这两个方法名乍看有点像，authorization发音[ˌɔ:θəraɪˈzeɪʃn]，为授权，批准的意思，即获取用户的角色和权限等信息；authentication发音[ɔ:ˌθentɪ’keɪʃn]，认证，身份验证的意思，即登录时验证用户的合法性，比如验证用户名和密码。

```java
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.hong.springbootshiro.entity.User;
import com.hong.springbootshiro.mapper.UserMapper;
import org.apache.shiro.authc.*;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;

public class ShiroRealm extends AuthorizingRealm {

    @Autowired
    private UserMapper userMapper;

    /**
     * 获取用户角色和权限
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        return null;
    }

    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        // 获取用户输入的用户名和密码
        String userName = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());

        System.out.println("用户" + userName + "认证-----ShiroRealm.doGetAuthenticationInfo");

        // 通过用户名到数据库查询用户信息
        User user = userMapper.selectOne(new QueryWrapper<User>().eq("user_name", userName));

        if (user == null) {
            throw new UnknownAccountException("用户名或密码错误！");
        }
        if (!password.equals(user.getPassword())) {
            throw new IncorrectCredentialsException("用户名或密码错误！");
        }
        if (user.getStatus().equals("0")) {
            throw new LockedAccountException("账号已被锁定,请联系管理员！");
        }
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, password, getName());
        return info;
    }
```

> 虽然我们可以准确的获取异常信息，并根据这些信息给用户提示具体错误，但最安全的做法是在登录失败时仅向用户显示通用错误提示信息，例如“用户名或密码错误”。这样可以防止数据库被恶意扫描。

### 数据层

#### 表

首先创建一张用户表，用于存储用户的基本信息(mysql)

```mysql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;
-- ----------------------------
-- Table structure for t_user
-- ----------------------------
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user`  (
  `ID` int(11) NOT NULL,
  `user_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户名',
  `password` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '密码',
  `create_time` date NULL DEFAULT NULL COMMENT '创建时间',
  `status` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '是否有效 1：有效  0：锁定',
  PRIMARY KEY (`ID`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
-- ----------------------------
-- Records of t_user
-- ----------------------------
INSERT INTO `t_user` VALUES (1, 'mrbird', '42ee25d1e43e9f57119a00d0a39e5250', '2022-07-14', '0');
INSERT INTO `t_user` VALUES (2, 'test', '123456', '2022-07-13', '0');

SET FOREIGN_KEY_CHECKS = 1;

```

数据源配置略

#### 实体类

```java
@Data
@TableName("t_user")
public class User implements Serializable {

    private static final long serialVersionUID = -5440372534300871944L;

    private Integer id;
    private String userName;
    private String password;
    private Date createTime;
    private String status;
}
```



#### Mapper

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.hong.springbootshiro.entity.User;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```





页面略（这里有问题），可自行通过postman测试

### Controller

LoginController代码如下：

```java
import com.hong.springbootshiro.entity.ResponseBo;
import com.hong.springbootshiro.entity.User;
import com.hong.util.MD5Utils;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.subject.Subject;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @PostMapping("/login")
    @ResponseBody
    public ResponseBo login(String username, String password) {
        // 密码MD5加密
        password = MD5Utils.encrypt(username, password);
        UsernamePasswordToken token = new UsernamePasswordToken(username, password); // 创建用户名和密码的token
        // 获取Subject对象
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
            return ResponseBo.ok();
        } catch (UnknownAccountException e) {
            return ResponseBo.error(e.getMessage());
        } catch (IncorrectCredentialsException e) {
            return ResponseBo.error(e.getMessage());
        } catch (LockedAccountException e) {
            return ResponseBo.error(e.getMessage());
        } catch (AuthenticationException e) {
            return ResponseBo.error("认证失败！");
        }
    }

    @RequestMapping("/")
    public String redirectIndex() {
        return "redirect:/index";
    }

    @RequestMapping("/index")
    public String index(Model model) {
        // 登录成后，即可通过Subject获取登录的用户信息
        User user = (User) SecurityUtils.getSubject().getPrincipal();
        model.addAttribute("user", user);
        return "index";
    }
}
```



登录成功后，根据之前在ShiroConfig中的配置`shiroFilterFactoryBean.setSuccessUrl("/index")`，页面会自动访问/index路径。

此外，代码运用到的工具类和返回类代码如下：

### 工具类

#### MD5Utils

```java
import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.util.ByteSource;

public class MD5Utils {
   private static final String SALT = "mrbird";

   private static final String ALGORITH_NAME = "md5";

   private static final int HASH_ITERATIONS = 2;

   public static String encrypt(String pswd) {
      String newPassword = new SimpleHash(ALGORITH_NAME, pswd, ByteSource.Util.bytes(SALT), HASH_ITERATIONS).toHex();
      return newPassword;
   }

   public static String encrypt(String username, String pswd) {
      String newPassword = new SimpleHash(ALGORITH_NAME, pswd, ByteSource.Util.bytes(username + SALT),
            HASH_ITERATIONS).toHex();
      return newPassword;
   }
   public static void main(String[] args) {
      
      System.out.println(MD5Utils.encrypt("test", "123456"));
   }

}
```

#### ResponseBo

```java
import java.util.HashMap;
import java.util.Map;

public class ResponseBo extends HashMap<String, Object>{
   private static final long serialVersionUID = 1L;

   public ResponseBo() {
      put("code", 0);
      put("msg", "操作成功");
   }

   public static ResponseBo error() {
      return error(1, "操作失败");
   }

   public static ResponseBo error(String msg) {
      return error(500, msg);
   }

   public static ResponseBo error(int code, String msg) {
      ResponseBo ResponseBo = new ResponseBo();
      ResponseBo.put("code", code);
      ResponseBo.put("msg", msg);
      return ResponseBo;
   }

   public static ResponseBo ok(String msg) {
      ResponseBo ResponseBo = new ResponseBo();
      ResponseBo.put("msg", msg);
      return ResponseBo;
   }

   public static ResponseBo ok(Map<String, Object> map) {
      ResponseBo ResponseBo = new ResponseBo();
      ResponseBo.putAll(map);
      return ResponseBo;
   }

   public static ResponseBo ok() {
      return new ResponseBo();
   }

   @Override
   public ResponseBo put(String key, Object value) {
      super.put(key, value);
      return this;
   }
}
```

#### application.yml

```yaml
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/scott?characterEncoding=utf-8&useSSL=false
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource
    #druid数据源配置
    druid:
      # 下面为连接池的补充设置，应用到上面所有数据源中
      # 初始化大小，最小，最大
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 配置获取连接等待超时的时间
      max-wait: 60000
      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      time-between-eviction-runs-millis: 60000
      # 配置一个连接在池中最小生存的时间，单位是毫秒
      min-evictable-idle-time-millis: 300000
      validation-query: SELECT 1 FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: false
      #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
      max-pool-prepared-statement-per-connection-size: 20
      filters: stat,wall
      use-global-data-source-stat: true
      # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
      connect-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
      # 配置监控服务器
      stat-view-servlet:
        login-username: druid
        login-password: druid
        reset-enable: false
        url-pattern: /druid/*
    thymeleaf:
      cache: false
server:
  port: 8080
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```



最终的目录结构

![image-20220714165850987](E:\Development\Typora\images\image-20220714165850987.png)





## 2.Spring Boot Shiro Remember Me

接着[《Spring-Boot-shiro用户认证》](https://mrbird.cc/Spring-Boot-shiro Authentication.html)，当用户成功登录后，关闭浏览器然后再打开浏览器访问http://localhost:8080/web/index，页面会跳转到登录页，之前的登录因为浏览器的关闭已经失效。

Shiro为我们提供了Remember Me的功能，用户的登录状态不会因为浏览器的关闭而失效，直到Cookie过期。

### 更改 ShiroConfig

继续编辑ShiroConfig，加入：

```java
/**
 * cookie对象
 * @return
 */
public SimpleCookie rememberMeCookie() {
    // 设置cookie名称，对应login.html页面的<input type="checkbox" name="rememberMe"/>
    SimpleCookie cookie = new SimpleCookie("rememberMe");
    // 设置cookie的过期时间，单位为秒，这里为一天
    cookie.setMaxAge(86400);
    return cookie;
}

/**
 * cookie管理对象
 * @return
 */
public CookieRememberMeManager rememberMeManager() {
    CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
    cookieRememberMeManager.setCookie(rememberMeCookie());
    // rememberMe cookie加密的密钥 
    cookieRememberMeManager.setCipherKey(Base64.decode("4AvVhmFLUs0KTA3Kprsdag=="));
    return cookieRememberMeManager;
}
```



接下来将cookie管理对象设置到SecurityManager中：

```java
@Bean  
public SecurityManager securityManager(){  
    DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
    securityManager.setRealm(shiroRealm());
    securityManager.setRememberMeManager(rememberMeManager());
    return securityManager;  
}
```



最后修改权限配置，将ShiroFilterFactoryBean的

`filterChainDefinitionMap.put("/**", "authc");`更改为`filterChainDefinitionMap.put("/**", "user");`。`user`指的是用户认证通过或者配置了Remember Me记住用户登录状态后可访问。

### 更改 LoginController

```java
@PostMapping("/login")
@ResponseBody
public ResponseBo login(String username, String password, Boolean rememberMe) {
    password = MD5Utils.encrypt(username, password);
    UsernamePasswordToken token = new UsernamePasswordToken(username, password, rememberMe);
    Subject subject = SecurityUtils.getSubject();
    try {
        subject.login(token);
        return ResponseBo.ok();
    } catch (UnknownAccountException e) {
        return ResponseBo.error(e.getMessage());
    } catch (IncorrectCredentialsException e) {
        return ResponseBo.error(e.getMessage());
    } catch (LockedAccountException e) {
        return ResponseBo.error(e.getMessage());
    } catch (AuthenticationException e) {
        return ResponseBo.error("认证失败！");
    }
}
```

当rememberMe参数为true的时候，Shiro就会帮我们记住用户的登录状态。启动项目即可看到效果。

其实可以不用上传rememberMe，自动为用户设置cookie即可



## 3.Spring Boot Shiro权限控制

在[《Spring-Boot-shiro用户认证》](https://mrbird.cc/Spring-Boot-shiro Authentication.html)中，我们通过继承AuthorizingRealm抽象类实现了`doGetAuthenticationInfo()`方法完成了用户认证操作。接下来继续实现`doGetAuthorizationInfo()`方法完成Shiro的权限控制功能。

授权也称为访问控制，是管理资源访问的过程。即根据不同用户的权限判断其是否有访问相应资源的权限。在Shiro中，权限控制有三个核心的元素：权限，角色和用户。



### 1.库模型设计

在这里，我们使用RBAC（Role-Based Access Control，基于角色的访问控制）模型设计用户，角色和权限间的关系。简单地说，一个用户拥有若干角色，每一个角色拥有若干权限。这样，就构造成“用户-角色-权限”的授权模型。在这种模型中，用户与角色之间，角色与权限之间，一般者是多对多的关系。如下图所示：

![QQ截图20171214123601.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20171214123601.png)

根据这个模型，设计数据库表，并插入一些测试数据：

```mysql
/*
 Navicat Premium Data Transfer

 Source Server         : localhost
 Source Server Type    : MySQL
 Source Server Version : 50737
 Source Host           : localhost:3306
 Source Schema         : scott

 Target Server Type    : MySQL
 Target Server Version : 50737
 File Encoding         : 65001

 Date: 14/07/2022 21:52:17
*/

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for t_permission
-- ----------------------------
DROP TABLE IF EXISTS `t_permission`;
CREATE TABLE `t_permission`  (
  `id` int(11) NOT NULL,
  `url` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'url地址',
  `name` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'url描述',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_permission
-- ----------------------------
INSERT INTO `t_permission` VALUES (1, '/user', 'user:user');
INSERT INTO `t_permission` VALUES (2, '/user/add', 'user:add');
INSERT INTO `t_permission` VALUES (3, '/user/delete', 'user:delete');

-- ----------------------------
-- Table structure for t_role
-- ----------------------------
DROP TABLE IF EXISTS `t_role`;
CREATE TABLE `t_role`  (
  `id` int(11) NOT NULL,
  `name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '角色名称',
  `memo` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '角色描述',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_role
-- ----------------------------
INSERT INTO `t_role` VALUES (1, 'admin', '超级管理员');
INSERT INTO `t_role` VALUES (2, 'test', '测试账户');

-- ----------------------------
-- Table structure for t_role_permission
-- ----------------------------
DROP TABLE IF EXISTS `t_role_permission`;
CREATE TABLE `t_role_permission`  (
  `rid` int(11) NULL DEFAULT NULL COMMENT '角色id',
  `pid` int(11) NULL DEFAULT NULL COMMENT '权限id'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_role_permission
-- ----------------------------
INSERT INTO `t_role_permission` VALUES (1, 2);
INSERT INTO `t_role_permission` VALUES (1, 3);
INSERT INTO `t_role_permission` VALUES (2, 1);
INSERT INTO `t_role_permission` VALUES (1, 1);

-- ----------------------------
-- Table structure for t_user
-- ----------------------------
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user`  (
  `ID` int(11) NOT NULL,
  `user_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '用户名',
  `password` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '密码',
  `create_time` date NULL DEFAULT NULL COMMENT '创建时间',
  `status` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '是否有效 1：有效  0：锁定',
  PRIMARY KEY (`ID`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_user
-- ----------------------------
INSERT INTO `t_user` VALUES (1, 'mrbird', '123456', '2022-07-14', '1');
INSERT INTO `t_user` VALUES (2, 'test', '123456', '2022-07-13', '1');

-- ----------------------------
-- Table structure for t_user_role
-- ----------------------------
DROP TABLE IF EXISTS `t_user_role`;
CREATE TABLE `t_user_role`  (
  `user_id` int(11) NULL DEFAULT NULL COMMENT '用户id',
  `rid` int(11) NULL DEFAULT NULL COMMENT '角色id'
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of t_user_role
-- ----------------------------
INSERT INTO `t_user_role` VALUES (1, 1);
INSERT INTO `t_user_role` VALUES (2, 2);

SET FOREIGN_KEY_CHECKS = 1;

```

### 2.Dao层

创建两个实体类，对应用户角色表Role和用户权限表Permission：

```java
@Data
@TableName("t_role")
public class Role implements Serializable{  
   private static final long serialVersionUID = -227437593919820521L;
   private Integer id;
   private String name;
   private String memo;   
}
```



```java
@Data
@TableName("t_permission")
public class Permission implements Serializable{
   private static final long serialVersionUID = 7160557680614732403L;
   private Integer id;
   private String url;
   private String name; 
}
```



创建两个dao接口，分别用户查询用户的所有角色和用户的所有权限：

UserRoleMapper：

```java
@Mapper
public interface UserRoleMapper extends BaseMapper<Role> {

	List<Role> findByUserName(String userName);
}
	
```

UserPermissionMapper：

```java
@Mapper
public interface UserPermissionMapper extends BaseMapper<Permission> {

    List<Permission> findByUserName(String userName);
}
```

其xml实现：

UserRoleMapper.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hong.springbootshiro.mapper.UserRoleMapper">
<resultMap type="com.hong.springbootshiro.entity.Role" id="role">
   <id column="id" property="id" javaType="java.lang.Integer" jdbcType="NUMERIC"/>
   <id column="name" property="name" javaType="java.lang.String" jdbcType="VARCHAR"/>
   <id column="memo" property="memo" javaType="java.lang.String" jdbcType="VARCHAR"/>
</resultMap>
<select id="findByUserName" resultMap="role">
	select r.id,r.name,r.memo from t_role r
	left join t_user_role ur on(r.id = ur.rid) 
	left join t_user u on(u.id = ur.user_id)
	where u.username = #{userName}
</select>
</mapper>
```



UserPermissionMapper.xml：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.hong.springbootshiro.mapper.UserPermissionMapper">
<resultMap type="com.hong.springbootshiro.entity.Permission" id="permission">
   <id column="id" property="id" javaType="java.lang.Integer" jdbcType="NUMERIC"/>
   <id column="url" property="url" javaType="java.lang.String" jdbcType="VARCHAR"/>
   <id column="name" property="name" javaType="java.lang.String" jdbcType="VARCHAR"/>
</resultMap>
<select id="findByUserName" resultMap="permission">
   select p.id,p.url,p.name from t_role r
   left join t_user_role ur on(r.id = ur.rid) 
   left join t_user u on(u.id = ur.user_id)
   left join t_role_permission rp on(rp.rid = r.id) 
   left join t_permission p on(p.id = rp.pid ) 
   where u.username = #{userName}
</select>
</mapper>
```

数据层准备好后，接下来对Realm进行改造。

在Shiro中，用户角色和权限的获取是在Realm的`doGetAuthorizationInfo()`方法中实现的，所以接下来手动实现该方法：

```java
public class ShiroRealm extends AuthorizingRealm {
    @Autowired
    private UserMapper userMapper;
    @Autowired
    private UserRoleMapper userRoleMapper;
    @Autowired
    private UserPermissionMapper userPermissionMapper;
    /**
     * 获取用户角色和权限
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {

        User user = (User) SecurityUtils.getSubject().getPrincipal(); //获取当前登录用户信息
        String userName = user.getUserName();

        System.out.println("用户" + userName + "获取权限-----ShiroRealm.doGetAuthorizationInfo");
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo(); //创建权限对象

        // 获取用户角色集
        List<Role> roleList = userRoleMapper.findByUserName(userName);
        Set<String> roleSet = new HashSet<String>();
        for (Role r : roleList) {
            roleSet.add(r.getName()); //添加角色
        }
        simpleAuthorizationInfo.setRoles(roleSet); //设置角色集

        // 获取用户权限集
        List<Permission> permissionList = userPermissionMapper.findByUserName(userName);
        Set<String> permissionSet = new HashSet<String>();
        for (Permission p : permissionList) {
            permissionSet.add(p.getName());
        }
        simpleAuthorizationInfo.setStringPermissions(permissionSet);
        return simpleAuthorizationInfo;
    }
    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 登录认证已经实现过，这里不再贴代码
    }
}
```

在上述代码中，我们通过方法`userRoleMapper.findByUserName(userName)`和`userPermissionMapper.findByUserName(userName)`获取了当前登录用户的角色和权限集，然后保存到SimpleAuthorizationInfo对象中，并返回给Shiro，这样Shiro中就存储了当前用户的角色和权限信息了。

除了对Realm进行改造外，我们还需修改ShiroConfig配置。



### 3.ShiroConfig

Shiro为我们提供了一些和权限相关的注解，如下所示：

```java
// 表示当前Subject已经通过login进行了身份验证；即Subject.isAuthenticated()返回true。
@RequiresAuthentication  
 
// 表示当前Subject已经身份验证或者通过记住我登录的。
@RequiresUser  

// 表示当前Subject没有身份验证或通过记住我登录过，即是游客身份。
@RequiresGuest  

// 表示当前Subject需要角色admin和user。  
@RequiresRoles(value={"admin", "user"}, logical= Logical.AND)  

// 表示当前Subject需要权限user:a或user:b。
@RequiresPermissions (value={"user:a", "user:b"}, logical= Logical.OR)
```

要开启这些注解的使用，需要在ShiroConfig中添加如下配置：

```java
...
@Bean
public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
    AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
    authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
    return authorizationAttributeSourceAdvisor;
}
...
```

### 4.Controller

编写一个UserController，用于处理User类的访问请求，并使用Shiro权限注解控制权限：

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequiresPermissions("user:user")
    @RequestMapping("list")
    public String userList(Model model) {
        model.addAttribute("value", "获取用户信息");
        return "user";
    }
    
    @RequiresPermissions("user:add")
    @RequestMapping("add")
    public String userAdd(Model model) {
        model.addAttribute("value", "新增用户");
        return "user";
    }
    
    @RequiresPermissions("user:delete")
    @RequestMapping("delete")
    public String userDelete(Model model) {
        model.addAttribute("value", "删除用户");
        return "user";
    }
}
```



在LoginController中添加一个/403跳转：

```java
@GetMapping("/403")
public String forbid() {
    return "403";
}
```



## 4.Spring Boot Shiro中使用缓存

在Shiro中加入缓存可以使权限相关操作尽可能快，避免频繁访问数据库获取权限信息，因为对于一个用户来说，其权限在短时间内基本是不会变化的。Shiro提供了Cache的抽象，其并没有直接提供相应的实现，因为这已经超出了一个安全框架的范围。在Shiro中可以集成常用的缓存实现，这里介绍基于Redis和Ehcache缓存的实现。

在[《Spring-Boot-shiro权限控制》](https://mrbird.cc/Spring-Boot-Shiro Authorization.html)中，当用户访问”获取用户信息”、”新增用户”和”删除用户”的时候，后台输出了三次打印信息，如下所示：

```
用户mrbird获取权限-----ShiroRealm.doGetAuthorizationInfo
用户mrbird获取权限-----ShiroRealm.doGetAuthorizationInfo
用户mrbird获取权限-----ShiroRealm.doGetAuthorizationInfo
```



说明在这三次访问中，Shiro都会从数据库中获取用户的权限信息，通过Druid数据源SQL监控后台也可以证实这一点：

![QQ截图20171214105048.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20171214105048.png)

这对数据库来说是没必要的消耗。接下来使用缓存来解决这个问题。

### Redis

#### 引入Redis依赖

网络上已经有关于Shiro集成Redis的实现，我们引入即可：

```java
<!-- shiro-redis -->
<dependency>
    <groupId>org.crazycake</groupId>
    <artifactId>shiro-redis</artifactId>
    <version>2.4.2.1-RELEASE</version>
</dependency>
```



#### 配置Redis

我们在application.yml配置文件中加入Redis配置：

```java
spring:
  redis:
    host: localhost
    port: 6379
    pool:
      max-active: 8
      max-wait: -1
      max-idle: 8
      min-idle: 0
    timeout: 0
```



接着在ShiroConfig中配置Redis：

```java
public RedisManager redisManager() {
    RedisManager redisManager = new RedisManager();
    return redisManager;
}

public RedisCacheManager cacheManager() {
    RedisCacheManager redisCacheManager = new RedisCacheManager();
    redisCacheManager.setRedisManager(redisManager());
    return redisCacheManager;
}
```



上面代码配置了RedisManager，并将其注入到了RedisCacheManager中，最后在SecurityManager中加入RedisCacheManager：

```java
@Bean  
public SecurityManager securityManager(){  
    DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
    ...
    securityManager.setCacheManager(cacheManager());
    return securityManager;  
}
```



配置完毕启动项目，分别访问访问”获取用户信息”、”新增用户”和”删除用户”，可发现后台只打印一次获取权限信息：

```java
用户mrbird获取权限-----ShiroRealm.doGetAuthorizationInfo
```



查看Druid数据源SQL监控：

![QQ截图20171225105337.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20171225105337.png)



## 5.Spring Boot Shiro在线会话管理

在Shiro中我们可以通过`org.apache.shiro.session.mgt.eis.SessionDAO`对象的`getActiveSessions()`方法方便的获取到当前所有有效的Session对象。通过这些Session对象，我们可以实现一些比较有趣的功能，比如查看当前系统的在线人数，查看这些在线用户的一些基本信息，强制让某个用户下线等。

为了达到这几个目标，我们在现有的Spring Boot Shiro项目基础上进行一些改造（缓存使用Ehcache）。

### 1.更改ShiroConfig

为了能够在Spring Boot中使用`SessionDao`，我们在ShiroConfig中配置该Bean：

```java
@Bean
public SessionDAO sessionDAO() {
    MemorySessionDAO sessionDAO = new MemorySessionDAO();
    return sessionDAO;
}
```



如果使用的是Redis作为缓存实现，那么SessionDAO则为`RedisSessionDAO`：

```java
@Bean
public RedisSessionDAO sessionDAO() {
    RedisSessionDAO redisSessionDAO = new RedisSessionDAO();
    redisSessionDAO.setRedisManager(redisManager());
    return redisSessionDAO;
}
```



在Shiro中，`SessionDao`通过`org.apache.shiro.session.mgt.SessionManager`进行管理，所以继续在ShiroConfig中配置`SessionManager`：

```java
@Bean
public SessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    Collection<SessionListener> listeners = new ArrayList<SessionListener>();
    listeners.add(new ShiroSessionListener());
    sessionManager.setSessionListeners(listeners);
    sessionManager.setSessionDAO(sessionDAO());
    return sessionManager;
}
```

其中`ShiroSessionListener`为`org.apache.shiro.session.SessionListener`接口的手动实现，所以接下来定义一个该接口的实现：

```java
public class ShiroSessionListener implements SessionListener{

   private final AtomicInteger sessionCount = new AtomicInteger(0); //其维护着一个原子类型的Integer对象，用于统计在线Session的数量。
   
   @Override
   public void onStart(Session session) {
      sessionCount.incrementAndGet(); //每有一个Session创建，就调用incrementAndGet()方法，将其加1。
   }

   @Override
   public void onStop(Session session) {
      sessionCount.decrementAndGet(); //每有一个Session销毁，就调用decrementAndGet()方法，将其减1。
      
   }

   @Override
   public void onExpiration(Session session) {
      sessionCount.decrementAndGet(); //每有一个Session过期，就调用decrementAndGet()方法，将其减1。
   }
}
```



其维护着一个原子类型的Integer对象，用于统计在线Session的数量。

定义完SessionManager后，还需将其注入到SecurityManager中：

```java
@Bean  
public SecurityManager securityManager(){  
    DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
    securityManager.setRealm(shiroRealm());
    ...
    securityManager.setSessionManager(sessionManager());
    return securityManager;  
}
```



### 2.UserOnline

配置完ShiroConfig后，我们可以创建一个UserOnline实体类，用于描述每个在线用户的基本信息：

```java
@Data
public class UserOnline implements Serializable{
   
   private static final long serialVersionUID = 3828664348416633856L;
   
   // session id
    private String id;
    // 用户id
    private String userId;
    // 用户名称
    private String username;
   // 用户主机地址
    private String host;
    // 用户登录时系统IP
    private String systemHost;
    // 状态
    private String status;
    // session创建时间
    private Date startTimestamp;
    // session最后访问时间
    private Date lastAccessTime;
    // 超时时间
    private Long timeout;
 
}
```



### 3.Service

创建一个Service接口，包含查看所有在线用户和根据SessionId踢出用户抽象方法：

```java
public interface SessionService {
    List<UserOnline> list();
    boolean forceLogout(String sessionId);
}
```



其具体实现：

```java
@Service("sessionService")
public class SessionServiceImpl implements SessionService {

   @Autowired
   private SessionDAO sessionDAO;

   /**
    * 获取在线用户列表
    * @return
    */
   @Override
   public List<UserOnline> list() {
      List<UserOnline> list = new ArrayList<>();
      Collection<Session> sessions = sessionDAO.getActiveSessions(); //获取当前已登录的用户session列表
      for (Session session : sessions) { //遍历session列表
         UserOnline userOnline = new UserOnline(); //创建一个新的用户在线对象
         User user = new User(); //创建一个新的用户对象
         SimplePrincipalCollection principalCollection = new SimplePrincipalCollection();  //创建一个新的用户权限集合对象
         if (session.getAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY) == null) { //如果session中没有用户权限集合对象
            continue;
         } else {
            principalCollection = (SimplePrincipalCollection) session
                  .getAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY); //获取session中的用户权限集合对象
            user = (User) principalCollection.getPrimaryPrincipal(); //获取用户对象
            userOnline.setUsername(user.getUserName()); //设置用户名
            userOnline.setUserId(user.getId().toString()); //设置用户id
         }
         userOnline.setId((String) session.getId()); //设置session id
         userOnline.setHost(session.getHost()); //设置session ip
         userOnline.setStartTimestamp(session.getStartTimestamp());     //设置session开始时间
         userOnline.setLastAccessTime(session.getLastAccessTime());     //设置session最后访问时间
         Long timeout = session.getTimeout(); //获取session超时时间
         if (timeout == 0l) { //如果session超时时间为0
            userOnline.setStatus("离线");
         } else {
            userOnline.setStatus("在线");
         }
         userOnline.setTimeout(timeout); //设置session超时时间
         list.add(userOnline);
      }
      return list;
   }
   @Override
   public boolean forceLogout(String sessionId) {
      Session session = sessionDAO.readSession(sessionId); //获取session对象
      sessionDAO.delete(session); //删除session对象
      return true;

   }

}
```

通过SessionDao的`getActiveSessions()`方法，我们可以获取所有有效的Session，通过该Session，我们还可以获取到当前用户的Principal信息。

值得说明的是，当某个用户被踢出后（Session Time置为0），该Session并不会立刻从ActiveSessions中剔除，所以我们可以通过其timeout信息来判断该用户在线与否。

如果使用的Redis作为缓存实现，那么，`forceLogout()`方法需要稍作修改：

```java
@Override
public boolean forceLogout(String sessionId) {
    Session session = sessionDAO.readSession(sessionId);
    sessionDAO.delete(session);
    return true;
}
```



### 4.Controller

定义一个SessionContoller，用于处理Session的相关操作：

```java
@Controller
@RequestMapping("/online")
public class SessionController {
    @Autowired
    SessionService sessionService;
    
    @RequestMapping("index")
    public String online() {
        return "online";
    }

    @ResponseBody
    @RequestMapping("list")
    public List<UserOnline> list() {
        return sessionService.list();
    }

    @ResponseBody
    @RequestMapping("forceLogout")
    public ResponseBo forceLogout(String id) {
        try {
            sessionService.forceLogout(id);
            return ResponseBo.ok();
        } catch (Exception e) {
            e.printStackTrace();
            return ResponseBo.error("踢出用户失败");
        }
    }
}
```


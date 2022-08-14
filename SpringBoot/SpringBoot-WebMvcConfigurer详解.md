# SpringBoot-WebMvcConfigurer详解

### 前置

首先来看一下4个springboot的mvc支持类

`**WebMvcConfigurer**`

### `WebMvcConfigurerAdapter`

`**WebMvcConfigurationSupport**`

`**WebMvcAutoConfiguration**`

1. `WebMvcConfigurer` 为接口
2. `WebMvcConfigurerAdapter` 是 `WebMvcConfigurer` 的**实现类**大部分为**空方法** ；是`WebMvcConfigurer`的子类实现，由于Java8中可以使用default关键字为接口添加默认方法，为在源代码中spring5.0之后就已经弃用本类。**如果需要我接着可以实现`WebMvcConfigurer`类**
3. `WebMvcConfigurationSupport` 是mvc的基本实现并包含了WebMvc`C`onfigurer接口中的方法
4. `WebMvcAutoConfiguration` 是mvc的自动装在类并部分包含了`WebMvcConfigurer`接口中的方法

如果在springboot项目中没有使用到以上类，那么会自动启用（默认）`WebMvcAutoConfiguration`类做自动加载； 项目中的配置都是默认的，比如静态资源文件的访问

​		继承WebMvcConfigurationSupport类是会导致自动配置失效的原因是自动配置类 WebMvcAutoConfiguration 上有条件注解

```java
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
```

这个注解的意思是在项目类路径中 缺少 WebMvcConfigurationSupport类型的bean时该自动配置类才会生效，所以继承 WebMvcConfigurationSupport 后需要自己再重写相应的方法。



Spring Boot 默认为我们提供了静态资源处理，使用 `WebMvcAutoConfiguration` 中的配置各种属性。

如果想要自己完全控制WebMVC，就需要在含有@[Configuration](https://so.csdn.net/so/search?q=Configuration&spm=1001.2101.3001.7020)注解的配置类上增加**@EnableWebMvc**，增加该注解以后`WebMvcAutoConfiguration`中配置就不会生效，你需要自己来配置需要的每一项。这种情况下的配置还是要多看一下`WebMvcAutoConfiguration`类。

## 1. 简介

[WebMvcConfigurer](https://so.csdn.net/so/search?q=WebMvcConfigurer&spm=1001.2101.3001.7020)配置类其实是`Spring`内部的一种配置方式，采用`JavaBean`的形式来代替传统的`xml`配置文件形式进行针对框架个性化定制，可以自定义一些Handler，Interceptor，ViewResolver，MessageConverter。基于java-based方式的spring mvc配置，需要创建一个**配置**类并实现**`WebMvcConfigurer`** 接口；

在Spring Boot 1.5版本都是靠重写**WebMvcConfigurerAdapter**的方法来添加自定义[拦截器](https://so.csdn.net/so/search?q=拦截器&spm=1001.2101.3001.7020)，消息转换器等。SpringBoot 2.0 后，该类被标记为@Deprecated（弃用）。官方推荐直接实现WebMvcConfigurer或者直接继承WebMvcConfigurationSupport，方式一实现WebMvcConfigurer接口（推荐），方式二继承WebMvcConfigurationSupport类，具体实现可看这篇文章。https://blog.csdn.net/fmwind/article/details/82832758

## **2. WebMvcConfigurer接口**

```java
public interface WebMvcConfigurer {
    void configurePathMatch(PathMatchConfigurer var1);
 
    void configureContentNegotiation(ContentNegotiationConfigurer var1);
 
    void configureAsyncSupport(AsyncSupportConfigurer var1);
 
    void configureDefaultServletHandling(DefaultServletHandlerConfigurer var1);
 
    void addFormatters(FormatterRegistry var1);
 
    void addInterceptors(InterceptorRegistry var1);
 
    void addResourceHandlers(ResourceHandlerRegistry var1);
 
    void addCorsMappings(CorsRegistry var1);
 
    void addViewControllers(ViewControllerRegistry var1);
 
    void configureViewResolvers(ViewResolverRegistry var1);
 
    void addArgumentResolvers(List<HandlerMethodArgumentResolver> var1);
 
    void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> var1);
 
    void configureMessageConverters(List<HttpMessageConverter<?>> var1);
 
    void extendMessageConverters(List<HttpMessageConverter<?>> var1);
 
    void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> var1);
 
    void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> var1);
 
    Validator getValidator();
 
    MessageCodesResolver getMessageCodesResolver();
}
```

**常用的方法：**

```java
 /* 拦截器配置 */
void addInterceptors(InterceptorRegistry var1);
/* 视图跳转控制器 */
void addViewControllers(ViewControllerRegistry registry);
/**
     *静态资源处理
**/
void addResourceHandlers(ResourceHandlerRegistry registry);
/* 默认静态资源处理器 */
void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer);
/**
     * 这里配置视图解析器
 **/
void configureViewResolvers(ViewResolverRegistry registry);
/* 配置内容裁决的一些选项*/
void configureContentNegotiation(ContentNegotiationConfigurer configurer);
/** 解决跨域问题 **/
public void addCorsMappings(CorsRegistry registry) ;
```

### 2.1 addInterceptors：拦截器

- `addInterceptor`：需要一个实现`HandlerInterceptor`接口的拦截器实例
- `addPathPatterns`：用于设置拦截器的过滤路径规则；`addPathPatterns("/**")`对所有请求都拦截
- `excludePathPatterns`：用于设置不需要拦截的过滤规则
- 拦截器主要用途：进行用户登录状态的拦截，日志的拦截等。

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    super.addInterceptors(registry);
    registry.addInterceptor(new TestInterceptor()).addPathPatterns("/**").excludePathPatterns("/emp/toLogin","/emp/login","/js/**","/css/**","/images/**");
}
```

来看一下我自己写的除了登录、注册、重置密码的拦截

```java
/**
 * 给接口都配置拦截器,拦截转向到 authHandlerInterceptor
 */
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(authHandlerInterceptor)
            .addPathPatterns("/**") // 拦截所有请求
            .excludePathPatterns("/user/login")   //放行登录地址
            .excludePathPatterns("/user/register")  //放行注册地址
            .excludePathPatterns("/appeal/resetPassword")   //放行重置密码地址
            //放行swagger
            .excludePathPatterns("/swagger-resources/**",
                    "/webjars/**", "/v2/**", "/swagger-ui.html/**");
}
```



#### 小问题

`addPathPatterns("/**")`对所有请求都拦截，但是排除了`/toLogin`和`/login`请求的拦截。

当spring boot版本升级为2.x时，访问静态资源就会被HandlerInterceptor拦截,网上有很多处理办法都是如下写法

```javascript
.excludePathPatterns("/index.html","/","/user/login","/static/**");
```

可惜本人在使用时一直不起作用，查看请求的路径里并没有/static/如图：

![img](E:\Development\Typora\images\70.png)

于是我改成了"/js/**","/css/**","/images/**"这样页面内容就可以正常访问了，我的项目结构如下：

![img](E:\Development\Typora\images\70-16587677984101.png)





### 12.2 addViewControllers：页面跳转

以前写SpringMVC的时候，如果需要访问一个页面，必须要写Controller类，然后再写一个方法跳转到页面，感觉好麻烦，其实重写`WebMvcConfigurer`中的`addViewControllers`方法即可达到效果了

```java
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/toLogin").setViewName("login");
    }
```

值的指出的是，在这里重写`addViewControllers`方法，并不会覆盖**`WebMvcAutoConfiguration`**（Springboot自动配置）中的`addViewControllers`（在此方法中，Spring Boot将“/”映射至index.html），这也就意味着自己的配置和Spring Boot的自动配置同时有效，这也是我们推荐添加自己的MVC配置的方式。

 

### 2.3 addResourceHandlers：静态资源

比如，我们想自定义静态资源映射目录的话，只需重写addResourceHandlers方法即可。

注：如果继承WebMvcConfigurationSupport类实现配置时必须要重写该方法，具体见其它文章

```java
@Configuration
public class MyWebMvcConfigurerAdapter implements WebMvcConfigurer {
    /**
     * 配置静态访问资源
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/my/**").addResourceLocations("classpath:/my/");
    }
}
```

- `addResoureHandler`：指的是对外暴露的访问路径
- `addResourceLocations`：指的是内部文件放置的目录

### 2.4 configureDefaultServletHandling：默认静态资源处理器

```java
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
        configurer.enable("defaultServletName");
    }
```

此时会注册一个默认的[Handler](https://so.csdn.net/so/search?q=Handler&spm=1001.2101.3001.7020)：DefaultServletHttpRequestHandler，这个Handler也是用来处理静态文件的，它会尝试映射/。当DispatcherServelt映射/时（/ 和/ 是有区别的），并且没有找到合适的Handler来处理请求时，就会交给DefaultServletHttpRequestHandler 来处理。注意：这里的静态资源是放置在web根目录下，而非WEB-INF 下。
　　可能这里的描述有点不好懂（我自己也这么觉得），所以简单举个例子，例如：在webroot目录下有一个图片：1.png 我们知道Servelt规范中web根目录（webroot）下的文件可以直接访问的，但是由于DispatcherServlet配置了映射路径是：/ ，它几乎把所有的请求都拦截了，从而导致1.png 访问不到，这时注册一个DefaultServletHttpRequestHandler 就可以解决这个问题。其实可以理解为DispatcherServlet破坏了[Servlet](https://so.csdn.net/so/search?q=Servlet&spm=1001.2101.3001.7020)的一个特性（根目录下的文件可以直接访问），DefaultServletHttpRequestHandler是帮助回归这个特性的。

 

### 2.5 configureViewResolvers：视图解析器

这个方法是用来配置视图解析器的，该方法的参数ViewResolverRegistry 是一个注册器，用来注册你想自定义的视图解析器等。ViewResolverRegistry 常用的几个方法：https://blog.csdn.net/fmwind/article/details/81235401

```java
/** 启用内容裁决视图解析器*/
public void enableContentNegotiation(View... defaultViews) {
    initContentNegotiatingViewResolver(defaultViews);
}
```

### 2.6 configureContentNegotiation：配置内容裁决的一些参数

### 2.7 addCorsMappings：跨域

```java
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        //添加映射路径
        registry.addMapping("/**")
                //是否发送Cookie
                .allowCredentials(true)
                //设置放行哪些原始域   SpringBoot2.4.4下低版本使用.allowedOrigins("*")
                .allowedOrigins("*")
                //放行哪些请求方式
//                .allowedMethods(new String[]{"GET", "POST", "PUT", "DELETE"})
                .allowedMethods("*") //或者放行全部
                //放行哪些原始请求头部信息
                .allowedHeaders("*")
                //暴露哪些原始请求头部信息
                .exposedHeaders("*");
    }
```

### 2.8 configureMessageConverters：信息转换器

```java
 
/**
* 消息内容转换配置
 * 配置fastJson返回json转换
 * @param converters
 */
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    //调用父类的配置
    super.configureMessageConverters(converters);
    //创建fastJson消息转换器
    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
    //创建配置类
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    //修改配置返回内容的过滤
    fastJsonConfig.setSerializerFeatures(
            SerializerFeature.DisableCircularReferenceDetect,
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.WriteNullStringAsEmpty
    );
    fastConverter.setFastJsonConfig(fastJsonConfig);
    //将fastjson添加到视图消息转换器列表内
    converters.add(fastConverter);
 
}
```
# Spring Boot 静态资源处理

原文链接：https://blog.csdn.net/isea533/article/details/50412212

原文链接：https://blog.csdn.net/lvyuanj/article/details/108554170



首先来看一下4个springboot的mvc支持类

**WebMvcConfigurer**

### WebMvcConfigurerAdapter

**WebMvcConfigurationSupport**

**WebMvcAutoConfiguration**

1. WebMvcConfigurer 为接口
2. WebMvcConfigurerAdapter 是 WebMvcConfigurer 的**实现类**大部分为**空方法** ；是WebMvcConfigurer的子类实现，由于Java8中可以使用default关键字为接口添加默认方法，为在源代码中spring5.0之后就已经弃用本类。**如果需要我接着可以实现WebMvcConfigurer类**
3. WebMvcConfigurationSupport 是mvc的基本实现并包含了WebMvcConfigurer接口中的方法
4. WebMvcAutoConfiguration 是mvc的自动装在类并部分包含了WebMvcConfigurer接口中的方法

如果在springboot项目中没有使用到以上类，那么会自动启用（默认）WebMvcAutoConfiguration类做自动加载； 项目中的配置都是默认的，比如静态资源文件的访问

添加自己的拦截器addInterceptors：

##### 方案一：

继承WebMvcConfigurationSupport **会导致WebMvcAutoConfiguration不加载**，所以默认配置丢失导致静态资源等无法访问; 需要@Override 方法



```java
@Configuration
public class MyWebMvc extends WebMvcConfigurationSupport {
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		UserInterceptor userInterceptor = new UserInterceptor();
		registry.addInterceptor(userInterceptor)
				.addPathPatterns("/sys/**");
	}
   //添加静态资源访问
   @Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/**")
				.addResourceLocations("classpath:/resources/")
				.addResourceLocations("classpath:/static/")
				.addResourceLocations("classpath:/public/");
	}
}
```



继承WebMvcConfigurationSupport类是会导致自动配置失效的原因是自动配置类 WebMvcAutoConfiguration 上有条件注解

```java
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
```

这个注解的意思是在项目类路径中 缺少 WebMvcConfigurationSupport类型的bean时该自动配置类才会生效，所以继承 WebMvcConfigurationSupport 后需要自己再重写相应的方法。

##### 方案二:

**实现继承WebMvcConfigurer因为是接口实现所以WebMvcAutoConfiguration还会默认配置**(这个很重要,这个表明若类实现该接口可以不用@EnableWebMvc注解)，所以静态资源是可以访问的

```java
@Configuration
public class MyWebMvc implements WebMvcConfigurer {
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		UserInterceptor userInterceptor = new UserInterceptor();
		registry.addInterceptor(userInterceptor)
				.addPathPatterns("/sys/**");
	}
}
```

Spring Boot 默认为我们提供了静态资源处理，使用 WebMvcAutoConfiguration 中的配置各种属性。

如果想要自己完全控制WebMVC，就需要在含有@[Configuration](https://so.csdn.net/so/search?q=Configuration&spm=1001.2101.3001.7020)注解的配置类上增加**@EnableWebMvc**，增加该注解以后WebMvcAutoConfiguration中配置就不会生效，你需要自己来配置需要的每一项。这种情况下的配置还是要多看一下WebMvcAutoConfiguration类。



## 默认资源映射

我们在启动应用的时候，可以在控制台中看到如下信息：

```java
2016-01-08 09:29:30.362  INFO 24932 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class  org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2016-01-08 09:29:30.362  INFO 24932 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2016-01-08 09:29:30.437  INFO 24932 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
```



其中默认配置的 /** 映射到 /static （或/public、/resources、/META-INF/resources）

其中默认配置的 /webjars/** 映射到 classpath:/META-INF/resources/webjars/
PS：上面的 static、public、resources 等目录都在 classpath: 下面（如 src/main/resources/static）。

```java
classpath:/static
classpath:/public
classpath:/resources
classpath:/META-INF/resources
```

我们在src/main/resources目录下新建 public、resources、static 三个目录，并分别放入 1.jpg 2.jpg 3.jpg 三张图片。然后通过浏览器分别访问：

```java
http://localhost:8080/1.jpg
http://localhost:8080/2.jpg
http://localhost:8080/3.jpg
```

地址均可以正常访问，Spring Boot 默认会从 public resources static 三个目录里面查找是否存在相应的资源。

如果我按如下结构存放相同名称的图片，那么Spring Boot 读取图片的优先级是怎样的呢？

![img](E:\Development\Typora\images\format,png.png)

当我们访问地址 http://localhost:8080/fengjing.jpg 的时候，显示哪张图片？优先级顺序为：

​						**META-INF/resources > resources > static > public**

如果我们想访问pic2.jpg，请求地址 http://localhost:8080/img/pic2.jpg



## 自定义资源映射.

在实际应用中，我们还有很多资源是在管理系统中动态维护的，并不可能在程序包中， 比如当我们的项目有自己上传的文件时，需要自己指定目录资源

### 自定义目录

以增加 /myres/** 映射到 classpath:/myres/** 为例的代码处理为：
实现接口 WebMvcConfigurer 并重写方法 addResourceHandlers

```java
@Configuration
public class MyWebAppConfigurer implements WebMvcConfigurer {
	@Override
	public void addResourceHandlers(ResourceHandlerRegistry registry) {
registry.addResourceHandler("/myres/**").addResourceLocations("classpath:/myres/");
		
	}

}

```



访问myres 文件夹中的fengjing.jpg 图片的地址为 http://localhost:8080/myres/fengjing.jpg
这样使用代码的方式自定义目录映射，并不影响Spring Boot的默认映射，可以同时使用。

**如果我们将/myres/** 修改为 /** 与默认的相同时，则会覆盖系统的配置**，可以多次使用 addResourceLocations 添加目录，优先级先添加的高于后添加的。

```java
// 访问myres根目录下的fengjing.jpg 的URL为 http://localhost:8080/fengjing.jpg （/** 会覆盖系统默认的配置）
registry.addResourceHandler("/**").addResourceLocations("classpath:/myres/").addResourceLocations("classpath:/static/");
```



### 使用外部目录

如果我们要指定一个绝对路径的文件夹（如 H:/myimgs/ ），则只需要使用 addResourceLocations 指定即可。

```java
// 可以直接使用addResourceLocations 指定磁盘绝对路径，同样可以配置多个位置，注意路径写法需要加上 file :
registry.addResourceHandler("/myimgs/**").addResourceLocations("file:E:/myimgs/");
```



### 通过配置文件配置

略...

```
spring:
  mvc:
    static-path-pattern: /resources/**
```










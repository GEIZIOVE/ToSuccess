# Spring Cloud Feign Post 表单请求

# 问题

在使用`SpringCloud`时，发现Feign不能发送POST表单请求

注意：在SpringBoot/Cloud环境中，使用的FeignClient，在不做额外设置的情况下，只能使用MVC的注解，也就是`@RequestMapping`和`@RequestBody`或者`@RequestParam`。

FeignClient 的 接口尝试以 POST表单发送请求的写法：



```java
@RequestMapping(value="/someThing/someMethod", method=RequestMethod.POST)
ApiResponse someThing(@RequestParam("name") String value);
```



在没有改变默认`Encoder`的设置的情况下，FeignClient 对于简单类型的参数，例如String，Integer这些 wrapped primitive types，`这种类型将被编码为form表单参数`，即便写了`method=RequestMethod.POST`，**但是参数没有出现在Body中，而是在URL上，不符合需求。**

# 解决方案



```xml
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.3.0</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.3.0</version>
</dependency>
```



对应FeignClient客户端

```java
@FeignClient(name="remoteService",configuration = IRemoteService.FormSupportConfig.class)
public interface IRemoteService {
@RequestMapping(value="/someThing/someMethod", 
                                method=RequestMethod.POST, 
                                consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
ApiResponse someThing(@RequestBody Map<String, ?> formParams);
}

    class FormSupportConfig {

            @Bean
            public Encoder feignFormEncoder() {
                 return new SpringFormEncoder();
            }
     }
}
```



这样就可以发送表单请求了。

# 另一种不需要引入依赖的方法



```java
List<NameValuePair> nvps = new ArrayList<>();
nvps.add(new BasicNameValuePair("key1", key1.toString()));
nvps.add(new BasicNameValuePair("key2", StringUtils.join(key2, ",")));
nvps.add(new BasicNameValuePair("key3", key3.toString()));
String queryStr = URLEncodedUtils.format(nvps, Consts.UTF_8);

someThing(queryStr);
```

对应

```java
@RequestMapping(value="/someThing/someMethod", 
                                method=RequestMethod.POST, 
                                consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
ApiResponse someThing(@RequestBody String formParam);
```



# 推翻重来，上面的解决方案有问题

在同时使用表单和 RequestBody 的时候，使用上面的配置会抛异常，

```java
is not a type supported by this encoder
```

将上面的configuration 改成这个类就没有问题了



```java
import feign.codec.Encoder;
import feign.form.FormEncoder;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
import org.springframework.cloud.netflix.feign.support.SpringEncoder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Scope;

import static org.springframework.beans.factory.config.BeanDefinition.SCOPE_PROTOTYPE;

public class CoreFeignConfiguration {

    @Autowired
    private ObjectFactory<HttpMessageConverters> messageConverters;

    @Bean
    @Primary
    @Scope(SCOPE_PROTOTYPE)
    Encoder feignFormEncoder() {
        return new FormEncoder(new SpringEncoder(this.messageConverters));
    }
}
```
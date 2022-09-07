# SpringBoot-HandlerInterceptor拦截器

# HandlerInterceptor简介

[拦截器](https://so.csdn.net/so/search?q=拦截器&spm=1001.2101.3001.7020)我想大家都并不陌生，最常用的登录拦截、或是权限校验、或是防重复提交、或是根据业务像12306去校验购票时间,总之可以去做很多的事情。
我仔细想了想
这里我分三篇博客来介绍HandlerInterceptor的使用，从基本的使用、到自定义注解、最后到读取body中的流解决无法多次读取的问题。

## 1、定义实现类

定义一个Interceptor 非常简单方式也有几种，我这里简单列举两种
1、类要实现Spring 的`HandlerInterceptor` 接口
2、类继承实现了`HandlerInterceptor` 接口的类，例如 已经提供的实现了`HandlerInterceptor` 接口的抽象类`HandlerInterceptorAdapter`

> 这里博主用的是第二种方法继承`HandlerInterceptorAdapter`



## 2、HandlerInterceptor方法介绍

```java
boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception;

void postHandle(
    HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
    throws Exception;

void afterCompletion(
    HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
    throws Exception;
```



- preHandle：在业务处理器处理请求之前被调用。预处理，可以进行编码、安全控制、权限校验等处理；
- postHandle：在业务处理器处理请求执行完成后，生成视图之前执行。后处理（调用了Service并返回ModelAndView，但未进行页面渲染），有机会修改ModelAndView （这个博主就基本不怎么用了）；
- afterCompletion：在DispatcherServlet完全处理完请求后被调用，可用于清理资源等。返回处理（已经渲染了页面）；

## 拦截器实现

### ①新建AuthHandlerInterceptor

```java
package com.hong.dk.bookcollect.filter;

import com.hong.dk.bookcollect.handler.Asserts;
import com.hong.dk.bookcollect.result.enmu.ResultCodeEnum;
import com.hong.dk.bookcollect.utils.helper.JwtHelper;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
import java.util.concurrent.TimeUnit;

@Slf4j
@Component
public class AuthHandlerInterceptor  extends HandlerInterceptorAdapter {
    @Autowired
    RedisTemplate<String,String> redisTemplate;
    
    /**
     * 权限认证的拦截操作.
     */
    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest,
                             HttpServletResponse httpServletResponse,
                             @Qualifier("jacksonObjectMapper") Object object) {

//        log.info("=======进入拦截器========");
        // 如果不是映射到方法直接通过,可以访问资源.
        if (!(object instanceof HandlerMethod)) {
            return true;
        }
        //为空就返回错误
        String token = httpServletRequest.getHeader("token");
        System.out.println(token);
        if (null == token || "".equals(token.trim())) {
            Asserts.fail(ResultCodeEnum.LOGIN_AUTH);
        }
        //判断token是否过期
        if (JwtHelper.isTokenExpired(token)) {
            Asserts.fail(ResultCodeEnum.SIGN_EXPIRED);
        }
        //获取token中的用户id
        String userId = JwtHelper.getUserId(token);
        //通过userId查询redis中的token2
        String token_redis = (String) redisTemplate.opsForValue().get(userId+":"+userId);
        //验证token_redis是否为空，如果为空则将token2存入redis中
        if ( null == token_redis || "".equals(token_redis.trim()) ) {
            redisTemplate.opsForValue().set(userId+":"+userId,token,3600 * 3,TimeUnit.SECONDS);
        }
        if(!token.equals(token_redis)){
            Asserts.fail(ResultCodeEnum.REMOTE_LOGIN);
        }

        //token2的过期时间，如果小于1小时，则重新设置过期时间为3小时
        if ( redisTemplate.getExpire(userId+":"+userId, TimeUnit.SECONDS ) < 3600) {
             redisTemplate.expire(userId+":"+userId, 3600 * 3, TimeUnit.SECONDS);
        }
//        log.info("===========放行=========");
        return true;
    }

}
```



这个是上面某段代码的补充

```java
    //如果不是映射到方法，直接通过
    if(!(handler instanceof HandlerMethod)){
        return true;
    }
    //如果是方法探测，直接通过
    if (HttpMethod.OPTIONS.equals(request.getMethod())) {
        response.setStatus(HttpServletResponse.SC_OK);
        return true;
    }
```

### ②新建WebAppConfigurer 实现WebMvcConfigurer接口

```java
@Configuration
public class AuthWebMvcConfig implements WebMvcConfigurer {
    @Autowired
    AuthHandlerInterceptor authHandlerInterceptor;

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
 }
```




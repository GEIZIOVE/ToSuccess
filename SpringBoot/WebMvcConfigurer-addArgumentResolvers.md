# WebMvcConfigurer-addArgumentResolvers

在Springboot中的WebMvcConfigurer接口在Web开发中经常被使用，例如配置拦截器、配置ViewController、配置Cors跨域等。本文主要讲解另一个方法：`addArgumentResolvers()`在实例中的应用。

### 一、方法作用

该方法可以用在对于`Controller`中方法参数传入之前对该参数进行处理。然后将处理好的参数在传给`Controller`方法。

官方API文档解释：添加解析器以支持自定义控制器方法参数类型。

这不会覆盖对解析处理程序方法参数的内置支持。要自定义对参数解析的内置支持，请`RequestMappingHandlerAdapter`直接配置。

### 二、场景描述

在权限场景中，通常会有要求用户登录之后才能访问的场景。对于这些问题可以多种解决方案，如：使用Cookie+Session的会话控制、使用拦截器、使用SpringSecurity或shiro等权限管理框架等。

这里使用Cookie+Session处理。处理的逻辑为：

**用户第一次登录之后会得到一个cookie，在以后每次的访问过程中都会携带Cookie进行访问。在后台的Controller中对于需要登录权限的访问接口都要先获取Cookie中的Token，再使用Token从session中获取用户登录信息来判断用户登录情况决定是否放行。**

### 三、存在问题

如果在每个需要登录权限访问的方法中都要写这个逻辑就会使代码重复，出现了冗余。代码基本如下：

```java
@GetMapping("goods")
public Result showGoods1(HttpSession session, @CookieValue("Token") String token){
    // 你每次都要在这里判断token或session
    if (token == null || MyObjectUtil.isEmpty(session.getAttribute(token))){
        return Result.error();
    }
    return Result.ok();
}
```

所以需要寻找是否有更好的解决方法。

最终找到`addArgumentResolvers()`这个方法。只需要在这个方法中进行处理即可。

### 四、问题解决

该方法的处理流程如下：

![img](E:\Development\Typora\images\2233189-20211115155349691-1287639097.png)

**代码如下：**

- Controller中的方法

  ```java
  @GetMapping("/adminUser/profile")
  public Result profile((@TokenToAdminUser AdminUserToken adminUser){
      // 注意：这里的adminUser参数不是由前端传入的，而是由addArgumentResolvers方法处理之后传进来的
      logger.info("adminUser:{}", adminUser.toString());
      // 根据处理之后传入的参数判断是否登录
      AdminUser adminUserEntity = adminUserService.getUserDetailById(adminUser.getAdminUserId());
      if (adminUserEntity != null) {
          adminUserEntity.setLoginPassword("******");
          Result result = ResultGenerator.genSuccessResult();
          result.setData(adminUserEntity);
          return result;
      }
      return ResultGenerator.genFailResult(ServiceResultEnum.DATA_NOT_EXIST.getResult());
  }
  ```

- WebMvcController实现了类

  ```java
  @Configuration
  public class NeeBeeMallWebMvcConfigurer implements WebMvcConfigurer {
  
      @Autowired
      private TokenToMallUserMethodArgumentResolver tokenToMallUserMethodArgumentResolver;
      @Autowired
      private TokenToAdminUserMethodArgumentResolver tokenToAdminUserMethodArgumentResolver;
  
      /**
       * 该方法作用在调用Controller方法的参数传入之前
       * @param resolvers
       */
      @Override
      @Override
      public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
          argumentResolvers.add(tokenToMallUserMethodArgumentResolver);
          argumentResolvers.add(tokenToAdminUserMethodArgumentResolver);
      }
  		
      // 其他配置
  }
  ```

- TokenToMallUserMethodArgumentResolver自定义实现类

  ```java
  /**
   * 该类用于WebConfig配置类的addArgumentResolvers方法传入
   */
  @Component
  public class TokenToAdminUserMethodArgumentResolver implements HandlerMethodArgumentResolver {
      @Autowired
      private NewBeeAdminUserTokenMapper newBeeAdminUserTokenMapper;
      public TokenToAdminUserMethodArgumentResolver() { // 必须要有无参构造器
      }
      /**
       * 该方法用于判断Controller中方法参数中是否有符合条件的参数：
       *                          有则进入下一个方法resolveArgument；
       *                          没有则跳过不做处理
       * @param parameter
       * @return
       */
      @Override
      public boolean supportsParameter(MethodParameter parameter) {
          // 获取传入参数的类型
    if (parameter.hasParameterAnnotation(TokenToAdminUser.class)) { //如果请求方法上有TokenToAdminUser注解，则返回true 表示需要处理
              return true;
          }
          return false;
      }
  
      /**
       * 该方法在上一个方法同通过之后调用：
       *      在这里可以进行处理，根据情况返回对象——返回的对象将被赋值到Controller的方法的参数中
       * @param parameter
       * @param mavContainer
       * @param webRequest
       * @param binderFactory
       * @return
       * @throws Exception
       */
      @Override //如果需要进行处理在这里操作
      public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                    NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {  //四个参数分别是：方法参数、ModelAndViewContainer、NativeWebRequest、WebDataBinderFactory
          if (parameter.getParameterAnnotation(TokenToAdminUser.class) instanceof TokenToAdminUser) { //获取注解
              String token = webRequest.getHeader("token"); //获取请求头中的token
              if (null != token && !"".equals(token) && token.length() == Constants.TOKEN_LENGTH) { //判断token是否为空，且长度是否为32位
                  AdminUserToken adminUserToken = newBeeAdminUserTokenMapper.selectByToken(token);    //根据token查询用户token信息
                  if (adminUserToken == null) { //如果查询不到，则抛出异常
                      NewBeeMallException.fail(ServiceResultEnum.ADMIN_NOT_LOGIN_ERROR.getResult()); 
                  } else if (adminUserToken.getExpireTime().getTime() <= System.currentTimeMillis()) { //如果查询到，则判断是否过期
                      NewBeeMallException.fail(ServiceResultEnum.ADMIN_TOKEN_EXPIRE_ERROR.getResult());
                  }
                  return adminUserToken;
              } else {
                  NewBeeMallException.fail(ServiceResultEnum.ADMIN_NOT_LOGIN_ERROR.getResult()); //如果token为空，则抛出异常
              }
          }
          return null;
      }
  }
  ```
# Springboot-Hibernate Validator

## 1.常用注解

### 1.1 Bean Validation 定义的约束注解

[`javax.validation.constraints`](https://github.com/eclipse-ee4j/beanvalidation-api/tree/master/src/main/java/javax/validation/constraints) 包下，定义了一系列的约束( constraint )注解。如下：

- 空和非空检查
  - `@NotBlank` ：只能用于字符串不为 `null` ，并且字符串 `#trim()` 以后 length 要大于 0 。
  - `@NotEmpty` ：集合对象的元素不为 0 ，即集合不为空，也可以用于字符串不为 `null` 。
  - `@NotNull` ：不能为 `null` 。
  - `@Null` ：必须为 `null` 。
- 数值检查
  - `@DecimalMax(value)` ：被注释的元素必须是一个数字，其值必须小于等于指定的最大值。
  - `@DecimalMin(value)` ：被注释的元素必须是一个数字，其值必须大于等于指定的最小值。
  - `@Digits(integer, fraction)` ：被注释的元素必须是一个数字，其值必须在可接受的范围内。
  - `@Positive` ：判断正数。
  - `@PositiveOrZero` ：判断正数或 0 。
  - `@Max(value)` ：该字段的值只能小于或等于该值。
  - `@Min(value)` ：该字段的值只能大于或等于该值。
  - `@Negative` ：判断负数。
  - `@NegativeOrZero` ：判断负数或 0 。
- Boolean 值检查
  - `@AssertFalse` ：被注释的元素必须为 `true` 。
  - `@AssertTrue` ：被注释的元素必须为 `false` 。
- 长度检查
  - `@Size(max, min)` ：检查该字段的 `size` 是否在 `min` 和 `max` 之间，可以是字符串、数组、集合、Map 等。
- 日期检查
  - `@Future` ：被注释的元素必须是一个将来的日期。
  - `@FutureOrPresent` ：判断日期是否是将来或现在日期。
  - `@Past` ：检查该字段的日期是在过去。
  - `@PastOrPresent` ：判断日期是否是过去或现在日期。
- 其它检查
  - `@Email` ：被注释的元素必须是电子邮箱地址。
  - `@Pattern(value)` ：被注释的元素必须符合指定的正则表达式。

### 1.2 Hibernate Validator 附加的约束注解

[`org.hibernate.validator.constraints`](https://github.com/hibernate/hibernate-validator/tree/master/engine/src/main/java/org/hibernate/validator/constraints) 包下，定义了一系列的约束( constraint )注解。如下：

- `@Range(min=, max=)` ：被注释的元素必须在合适的范围内。
- `@Length(min=, max=)` ：被注释的字符串的大小必须在指定的范围内。
- `@URL(protocol=,host=,port=,regexp=,flags=)` ：被注释的字符串必须是一个有效的 URL 。
- `@SafeHtml` ：判断提交的 HTML 是否安全。例如说，不能包含 javascript 脚本等等。
- ... 等等，就不一一列举了。

 	@Valid和BindingResult可以配套使用，@Valid用在参数前，BindingResult作为校验结果绑定返回。
 	**注意！**
 	1.@Validated 与BindingResult 需要相邻，否则 变量result 不能接受错误信息
 	
 	2.如果使用了@Validated，那么BeanValidate也会抛出异常而不是之前的封装在BindingResult中

`bindingResult.hasErrors()`判断是否校验通过，校验未通过，`bindingResult.getFieldError().getDefaultMessage()`获取在TestEntity的属性设置的自定义message，如果没有设置，则返回默认值"javax.validation.constraints.XXX.message"。

//具体api可参见自己源码



###  1.3@Valid 和 @Validated

[`@Valid`](https://docs.oracle.com/javaee/7/api/javax/validation/Valid.html) 注解，是 **Bean Validation** 所定义，可以添加在普通方法、构造方法、方法参数、方法返回、成员变量上，表示它们需要进行约束校验。

[`@Validated`](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/validation/annotation/Validated.java) 注解，是 **Spring Validation** 所定义，可以添加在类、方法参数、普通方法上，表示它们需要进行约束校验。同时，`@Validated` 有 `value` 属性，支持分组校验。属性如下：

```
// Validated.java
Class<?>[] value() default {};
```

对于初学的胖友来说，很容易搞混 `@Valid` 和 `@Validated` 注解。

**① 声明式校验**

Spring Validation **仅**对 `@Validated` 注解，实现声明式校验。

**② 分组校验**

Bean Validation 提供的 `@Valid` 注解，因为没有分组校验的属性，所以无法提供分组校验。此时，我们只能使用 `@Validated` 注解。

**③ 嵌套校验**

相比来说，`@Valid` 注解的地方，多了【成员变量】。这就导致，如果有嵌套对象的时候，只能使用 `@Valid` 注解。例如说：

```java
// User.java
public class User { 
    private String id;
    @Valid
    private UserProfile profile;
}
// UserProfile.java
public class UserProfile {
    @NotBlank
    private String nickname;
}
```

- 如果不在 `User.profile` 属性上，添加 `@Valid` 注解，就会导致 `UserProfile.nickname` 属性，不会进行校验。

当然，`@Valid` 注解的地方，也多了【构造方法】和【方法返回】，所以在有这方面的诉求的时候，也只能使用 `@Valid` 注解。

**🔥 总结**

总的来说，绝大多数场景下，我们使用 `@Validated` 注解即可。

而在有嵌套校验的场景，我们使用 `@Valid` 注解添加到成员属性上。

## 2.使用

### 2.1引入依赖

```xml
    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 保证 Spring AOP 相关的依赖包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
        </dependency>
    </dependencies>
```

- [`spring-boot-starter-web`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) 依赖里，已经默认引入 [`hibernate-validator`](https://mvnrepository.com/artifact/org.hibernate.validator/hibernate-validator) 依赖，所以本示例使用的是 Hibernate Validator 作为 Bean Validation 的实现框架。

在 Spring Boot 体系中，也提供了 [`spring-boot-starter-validation`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation) 依赖。在这里，我们并没有引入。为什么呢？该依赖的目的重点也是引入 `hibernate-validator` 依赖，这在 `spring-boot-starter-web` 已经引入，所以无需重复引入。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

- 由于SpringBoot 2.3版本默认移除了校验功能，如果想要开启的话需要添加如下依赖。

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```



### 2.2Application

创建 [`Application.java`](https://github.com/YunaiV/SpringBoot-Labs/blob/master/lab-22/lab-22-validation-01/src/main/java/cn/iocoder/springboot/lab22/validation/Application.java) 类，配置 `@SpringBootApplication` 注解即可。代码如下：



```java
@SpringBootApplication
@EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}0
```

- 添加 `@EnableAspectJAutoProxy` 注解，重点是配置 `exposeProxy = true` ，因为我们希望 Spring AOP 能将当前代理对象设置到 [AopContext](https://github.com/spring-projects/spring-framework/blob/master/spring-aop/src/main/java/org/springframework/aop/framework/AopContext.java) 中。具体用途，我们会在下文看到。想要提前看的胖友，可以看看 [《Spring AOP 通过获取代理对象实现事务切换》](http://www.voidcn.com/article/p-zddcuyii-bpt.html) 文章。

### 3.3UserLoginParam

```java
@Data
public class UserLoginParam implements Serializable {

    @ApiModelProperty("登录名")
    @NotEmpty(message = "学号不能为空")
    private String userId;

    @ApiModelProperty("用户密码(需要MD5加密)")
    @NotEmpty(message = "密码不能为空")
    private String password;
}
```



### 3.4 UserController

```java
@Api(value = "用户表(user)接口", tags = "用户表(user)接口")
@RestController
@RequestMapping("/user")
@Validated
public class UserController {
    private Logger logger = LoggerFactory.getLogger(getClass());
    @Autowired
    private UserService userService;
    @ApiOperation("用户登录")
    @PostMapping("/login")
    public Result login(@RequestBody @Valid UserLoginParam userLoginParam) {//, BindingResult bindingResult
        Map<String, Object> map = userService.login(userLoginParam.getUserId(), userLoginParam.getPassword());
        return Result.ok(map);
    }
    
     @GetMapping("/get")
    public void get(@RequestParam("id") @Min(value = 1L, message = "编号必须大于 0") Integer id) {
        logger.info("[get][id: {}]", id);
    }
}
```

- 在类上，添加 `@Validated` 注解，表示 UserController 是所有接口都需要进行参数校验。

对于这两种方法:

**第一点**，`#get(id)` 方法上，我们并没有给 `id` 添加 `@Valid` 注解，而 `#add(addDTO)` 方法上，我们给 `addDTO` 添加 `@Valid` 注解。这个差异，是为什么呢？

因为 UserController 使用了 `@Validated` 注解，那么 Spring Validation 就会使用 AOP 进行切面，进行参数校验。而该切面的拦截器，使用的是 [MethodValidationInterceptor](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/validation/beanvalidation/MethodValidationInterceptor.java) 。

- 对于 `#get(id)` 方法，需要校验的参数 `id` ，是**平铺**开的，所以无需添加 `@Valid` 注解。
- 对于 `#login(UserLoginParam)` 方法，需要校验的参数 `UserLoginParam` ，实际相当于**嵌套校验**，要校验的参数的都在 `UserLoginParam` 里面，所以需要添加 `@Valid` 注解。

**第二点**，`#get(id)` 方法的返回的结果是 `status = 500` ，而`#login(UserLoginParam)` 方法的返回的结果是 `status = 400` 。

- 对于 `#get(id)` 方法，在 MethodValidationInterceptor 拦截器中，校验到参数不正确，会抛出 [ConstraintViolationException](https://github.com/eclipse-ee4j/beanvalidation-api/blob/master/src/main/java/javax/validation/ConstraintViolationException.java) 异常。
- 对于 `#login(UserLoginParam)` 方法，因为 `UserLoginParam` 是个 POJO 对象，所以会走 SpringMVC 的 [DataBinder](https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/validation.html#validation-binder) 机制，它会调用 `DataBinder#validate(Object... validationHints)` 方法，进行校验。在校验不通过时，会抛出 [BindException](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/validation/BindException.html) 。

在 SpringMVC 中，默认使用 [DefaultHandlerExceptionResolver](https://hyrepo.com/tech/spring-mvc-error-handling/) 处理异常。

- 对于 BindException 异常，处理成 400 的状态码。
- 对于 ConstraintViolationException 异常，没有特殊处理，所以处理成 500 的状态码。

这里，我们在抛个问题，如果 `#login(UserLoginParam)` 方法，如果参数正确，在走完 DataBinder 中的参数校验后，会不会在走一遍 MethodValidationInterceptor 的拦截器呢？思考 100 毫秒...

答案是会。这样，就会导致浪费。所以 Controller 类里，如果**只有**类似的 `#login(UserLoginParam)` 方法的**嵌套校验**，那么我可以不在 Controller 类上添加 `@Validated` 注解。从而实现，仅使用 DataBinder 中来做参数校验。

**第三点**，无论是 `#get(id)` 方法，还是 `#login(UserLoginParam)` 方法，它们的返回提示都非常不友好，那么该怎么办呢？

## 3.处理校验逻辑

### 3.1全局异常处理

记得加`@ResponseBody`！！！ 不然他返回的是一个路径，然后再次进入拦截器！md!!!

> 版本1

```java
/**
 * 全局异常处理类
 * @author wqh
 */
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {

    /**
     * 处理ConstraintViolationException异常
     * @param e
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = ConstraintViolationException.class)
    public Result constraintViolationExceptionHandler(ConstraintViolationException e) {
//        log.debug("[constraintViolationExceptionHandler]", ex);
        // 拼接错误
        StringBuilder detailMessage = new StringBuilder();
        for (ConstraintViolation<?> constraintViolation : e.getConstraintViolations()) {
            // 使用 ; 分隔多个错误
            if (detailMessage.length() > 0) {
                detailMessage.append(";");
            }
            // 拼接内容到其中
            detailMessage.append(constraintViolation.getMessage());
        }
        // 包装 CommonResult 结果
        return Result.build(ResultCodeEnum.PARAM_ERROR.getCode(),
                ResultCodeEnum.PARAM_ERROR.getMessage() + ":" + detailMessage.toString());
    }
    /**
     * 处理BindException异常
     * @param e
     * @return
     */
    @ExceptionHandler(BindException.class) // 处理参数校验异常
    @ResponseBody
    public Result bindException(BindException e) {
//        log.info("BindException异常:{}", e.getMessage());
        // 拼接错误
        StringBuilder detailMessage = new StringBuilder();
        for (ObjectError objectError : e.getAllErrors()) {
            // 使用 ; 分隔多个错误
            if (detailMessage.length() > 0) {
                detailMessage.append(";");
            }
            // 拼接内容到其中
            detailMessage.append(objectError.getDefaultMessage());
        }
        // 包装 CommonResult 结果
        return Result.build(ResultCodeEnum.PARAM_ERROR.getCode(),
                ResultCodeEnum.PARAM_ERROR.getMessage() + ":" + detailMessage.toString());
    }

    /**
     * 处理MethodArgumentNotValidException异常
     * @param e
     * @return
     */
    @ExceptionHandler(MethodArgumentNotValidException.class) // 方法参数校验异常
    @ResponseBody
    public Result bindException(MethodArgumentNotValidException e) {
//        log.info("MethodArgumentNotValidException异常:{}", e.getMessage());
        BindingResult bindingResult = e.getBindingResult(); // 获取BindingResult
        String message = null;
        if (bindingResult.hasErrors()) { // 判断BindingResult是否有错误
            FieldError fieldError = bindingResult.getFieldError(); // 获取错误信息
            if (fieldError != null) {
                message = fieldError.getField()+fieldError.getDefaultMessage();
            }
        }
        return Result.fail(message);
    }
}
```

> 版本2

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ResponseBody
    @ExceptionHandler(value = MethodArgumentNotValidException.class)
    public Result handleValidException(MethodArgumentNotValidException e) {
        BindingResult bindingResult = e.getBindingResult(); // 获取BindingResult
        String message = null;
        if (bindingResult.hasErrors()) { // 判断BindingResult是否有错误
            FieldError fieldError = bindingResult.getFieldError(); // 获取错误信息
            if (fieldError != null) {
                message = fieldError.getField()+fieldError.getDefaultMessage();
            }
        }
        return Result.fail(message);
    }

    @ResponseBody
    @ExceptionHandler(value = BindException.class)
    public Result handleValidException(BindException e) {
        BindingResult bindingResult = e.getBindingResult(); // 获取BindingResult
        String message = null;
        if (bindingResult.hasErrors()) {
            FieldError fieldError = bindingResult.getFieldError();
            if (fieldError != null) {
                message = fieldError.getField()+fieldError.getDefaultMessage();
            }
        }
        return Result.fail(message);
    }
	
}
```

### 3.2使用切面校验

```java
@Aspect
@Component
@Order(2)
public class BindingResultAspect {
    @Pointcut("execution(public * com.hong.MinIO.controller.*.*(..))") // 定义一个切入点，表示要切入的方法
    public void BindingResult() { //方法体为空，只是为了让切入点生效
    }
    @Around("BindingResult()")
    public Object doAround(ProceedingJoinPoint joinPoint) throws Throwable {
        Object[] args = joinPoint.getArgs(); // 获取方法参数
        for (Object arg : args) {
            if (arg instanceof BindingResult) { // 判断参数是否是BindingResult类型
                BindingResult result = (BindingResult) arg;
                if (result.hasErrors()) { // 判断BindingResult是否有错误
                    FieldError fieldError = result.getFieldError(); // 获取错误信息
                    if(fieldError!=null){ // 判断错误信息是否为空
                        System.out.println(fieldError.getDefaultMessage());
                        return Result.ok();
                    }else{
                        return Result.ok();
                    }
                }
            }
        }
        return joinPoint.proceed(); // proceed执行方法并且返回结果
    }
}
```



其实当校验失败时，SpringBoot默认会抛出`MethodArgumentNotValidException`或`BindException`异常，我们只要全局处理该异常依然可以得到校验失败信息。

此时Controller的代码如下:

```java
@ApiOperation("用户登录")
@PostMapping("/login")
public Result login(@RequestBody @Validated UserLoginParam userLoginParam, BindingResult bindingResult) {
    Map<String, Object> map = userService.login(userLoginParam.getUserId(), userLoginParam.getPassword());
    return Result.ok(map);
}
```

没有 BindingResult bindingResult就……

- 由于SpringBoot 2.3版本默认移除了校验功能，如果想要开启的话需要添加如下依赖。

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
  ```




## 4.分组校验

在一些业务场景下，我们需要使用**分组**校验，即相同的 Bean 对象，根据校验分组，使用不同的校验规则。咳咳咳，貌似我们暂时没有这方面的诉求。即使有，也是拆分不同的 Bean 类。当然，作为一篇入门的文章，艿艿还是提供下分组校验的示例。

### 4.1 UserUpdateStatusDTO

在 [`cn.iocoder.springboot.lab22.validation.dto`](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-22/lab-22-validation-01/src/main/java/cn/iocoder/springboot/lab22/validation/dto) 包路径下，创建 [UserUpdateStatusDTO](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-22/lab-22-validation-01/src/main/java/cn/iocoder/springboot/lab22/validation/dto/UserUpdateStatusDTO.java) 类，为用户更新状态 DTO 。代码如下：



```java
// UserUpdateStatusDTO.java

public class UserUpdateStatusDTO {

    /**
     * 分组 01 ，要求状态必须为 true
     */
    public interface Group01 {}

    /**
     * 状态 02 ，要求状态必须为 false
     */
    public interface Group02 {}
    
    /**
     * 状态
     */
    @AssertTrue(message = "状态必须为 true", groups = Group01.class)
    @AssertFalse(message = "状态必须为 false", groups = Group02.class)
    private Boolean status;

    // ... 省略 set/get 方法
}
```



- 创建了 Group01 和 Group02 接口，作为两个校验分组。不一定要定义在 UserUpdateStatusDTO 类中，这里仅仅是为了方便。
- `status` 字段，在 Group01 校验分组时，必须为 `true` ；在 Group02 校验分组时，必须为 `false` 。

### 4.2 UserController

修改 [UserController](https://github.com/YunaiV/SpringBoot-Labs/tree/master/lab-22/lab-22-validation-01/src/main/java/cn/iocoder/springboot/lab22/validation/controller/UserController.java) 类，增加两个修改状态的 API 接口。代码如下：



```java
// UserController.java

@PostMapping("/update_status_true")
public void updateStatusTrue(@Validated(UserUpdateStatusDTO.Group01.class) UserUpdateStatusDTO updateStatusDTO) {
    logger.info("[updateStatusTrue][updateStatusDTO: {}]", updateStatusDTO);
}

@PostMapping("/update_status_false")
public void updateStatusFalse(@Validated(UserUpdateStatusDTO.Group02.class) UserUpdateStatusDTO updateStatusDTO) {
    logger.info("[updateStatusFalse][updateStatusDTO: {}]", updateStatusDTO);
}
```



- 对于 `#updateStatusTrue(updateStatusDTO)` 方法，我们在 `updateStatusDTO` 参数上，添加了 `@Validated` 注解，并且设置校验分组为 Group01 。校验不通过示例如下图：![不通过示例 1](E:\Development\Typora\images\06.jpg)
- 对于 `#updateStatusFalse(updateStatusDTO)` 方法，我们在 `updateStatusDTO` 参数上，添加了 `@Validated` 注解，并且设置校验分组为 Group02 。校验不通过示例如下图：![不通过示例 2](E:\Development\Typora\images\07.jpg)

所以，使用分组校验，核心在于添加上 `@Validated` 注解，并设置对应的校验分组。
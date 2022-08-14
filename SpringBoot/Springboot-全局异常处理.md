

# Springboot 全局异常处理

spring boot 默认情况下会映射到 /error 进行异常处理，但是提示并不十分友好，下面自定义异常处理，提供友好展示。

## @ControllerAdvice 注解

​		使用该注解表示开启了全局异常的捕获，我们只需在自定义一个方法使用`ExceptionHandler`注解然后定义捕获异常的类型即可对这些捕获的异常进行统一的处理。

​		可以用于定义 `@ExceptionHandler 、 @InitBinder 、@ModelAttribute` 

​		简单来说就是，可以通过`@ControllerAdvice` 注解配置一个全局异常处理类，来统一处理 controller 层中的异常，于此同时 controller 中可以不用再写 try/catch，这使得代码既整洁又便于维护。



## 全局异常处理

### 自定义异常类

- 首先我们需要自定义一个异常类`FyException`，当我们校验失败时抛出该异常：

​	**继承 `RuntimeException`**

```java
@Data
@ApiModel(value = "自定义全局异常类")
public class FyException extends RuntimeException {

    @ApiModelProperty(value = "异常状态码")
    private Integer code;

    /**
     * 通过状态码和错误消息创建异常对象
     * @param message
     * @param code
     */
    public FyException(String message, Integer code) {
        super(message);
        this.code = code;
    }

    /**
     * 接收枚举类型对象
     * @param resultCodeEnum
     */
    public FyException(ResultCodeEnum resultCodeEnum) {
        super(resultCodeEnum.getMessage());
        this.code = resultCodeEnum.getCode();
    }
    
    @Override
    public String toString() {
        return "YyghException{" +
                "code=" + code +
                ", message=" + this.getMessage() +
                '}';
    }
```



`ResultCodeEnum`为我们自己定义的枚举类

### 断言处理类Assert

- 然后创建一个断言处理类`Asserts`，用于抛出各种`FyException`；

```java
public class Asserts {
    public static void fail(String message,Integer code) {
        throw new FyException(message,code);
    }

    public static void fail(ResultCodeEnum resultCodeEnum) {
        throw new FyException(resultCodeEnum);
    }
}
```





###  全局异常处理类

- 然后再创建我们的全局异常处理类`GlobalExceptionHandler`，用于处理全局异常，并返回封装好的Result对象；

`@ControllerAdvice + @ExceptionHand` 配置全局异常处理类



```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ResponseBody
    @ExceptionHandler(value = FyException.class)
    public Result handle(FyException e) {
        if (e.getResultCodeEnum() != null) {
            return Result.build(e.getResultCodeEnum().getCode,
        if (e.getResultCodeEnum() != null) {
            return Result.build(e.getResultCodeEnum().getCode(),
                    e.getResultCodeEnum().getMessage());
        }
        return Result.build(e.getCode(),e.getMessage());

}
```



> Result的作用本来就是为了把Service中获取到的数据封装成统一返回结果





Impl层使用：（引用别人的代码）

```java
 //改进后
     @Override
     public void add(Long couponId) {
         UmsMember currentMember = memberService.getCurrentMember();
         //获取优惠券信息，判断数量
         SmsCoupon coupon = couponMapper.selectByPrimaryKey(couponId);
         if(coupon==null){
             Asserts.fail("优惠券不存在");
         }
         if(coupon.getCount()<=0){
             Asserts.fail("优惠券已经领完了");
         }
         Date now = new Date();
         if(now.before(coupon.getEnableTime())){
             Asserts.fail("优惠券还没到领取时间");
         }
         //判断用户领取的优惠券数量是否超过限制
         SmsCouponHistoryExample couponHistoryExample = new SmsCouponHistoryExample();
         couponHistoryExample.createCriteria().andCouponIdEqualTo(couponId).andMemberIdEqualTo(currentMember.getId());
         long count = couponHistoryMapper.countByExample(couponHistoryExample);
         if(count>=coupon.getPerLimit()){
             Asserts.fail("您已经领取过该优惠券");
         }
         //省略领取优惠券逻辑...
     }
```





> **优点**：上面的使用方法克服了只能处理 controller 层抛出的异常这一缺点。
>
> **缺点**：稍微有点复杂。





**方法二：**

**遇到异常时抛出异常即可**

在业务中，遇到业务异常的地方，直接使用 throw 抛出对应的业务异常即可。例如

```cpp
throw new FyException("3000", "账户密码错误");
```



**注意**

1. 不一定必须在 controller 层本身抛出异常才能被 `GlobalExceptionHandler` 处理，只要异常最后是从 contoller 层抛出去的就可以被全局异常处理器处理。
2. 异步方法中的异常不会被全局异常处理。
3. 抛出的异常如果被代码内的 try/catch 捕获了，就不会被 `GlobalExceptionHandler` 处理了。

**缺点**

只能处理 controller 层抛出的异常，对例如 Interceptor（拦截器）层的异常、定时任务中的异常、异步方法中的异常，不会进行处理。










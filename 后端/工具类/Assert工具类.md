# Assert工具类

api

[Assert (Spring Framework 5.3.20 API)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/Assert.html)

## 常用方法

junit中的assert方法总结

```java
1.assertTrue/False(boolean condition); 判断条件是true还是false,如果为false 直接返回报错

2.assertTrue/False(String message,boolean condition); 如果为false 直接返回message错误信息

3.assertEquals([String message,]Object expected,Object actual); 判断是否相等
    可以指定输出错误信息 message ，第一个参数是期望值，第二个参数是实际的值
    需要主意的是float和double最后面多一个delta的值 ，这个delta值其实是一个精度值
    Assert.assertEquals(6.1, 6.0, 0.0);      // pass
    Assert.assertEquals(6.01, 6.0, 0.000);   // error
```


spring自带org.springframework.util.Assert通用类
1. ```java
   1. state(boolean expression, String message); 
          如果为false 直接返回  throw new IllegalStateException(message);
   
   2. isTrue(boolean expression, String message);  与上个一样
   
   3. isNull(@Nullable Object object, String message); 当 object 不为 null 时抛出异常
   
   4. notNull(@Nullable Object object, String message) 当 object 为 null 时抛出异常
   
   5. hasText(String text) / hasText(String text, String message)  text 不能为 null 且必须至少包含一个非空格的字
   ```
   
   
------------------------------------------------

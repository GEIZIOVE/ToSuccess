# Mybatis Plus 乐观锁

## **1.乐观锁使用的场景**

> 当我们要更新一条数据记录的时候，希望这条记录没有被别人更新。
>
> 其实现方式：
>
> 1.取出记录时，我们会获取当前的version；
>
> \2. 当我们更新数据时，带上这个version去数据库执行更新
>
> 3.执行更新时，where version = oldVersion 如果带去的version与数据库中的version不对，就更新失败。

## 2.配置插件的使用

```java
// Spring Boot 方式
@Configuration
@MapperScan("按需修改")
public class MybatisPlusConfig {
    /**
     * 旧版
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }
    
    /**
     * 新版
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```



## 3.在实体类的字段上加上`@Version`注解

```java
@Version
private Integer version;
```

> 说明:
>
> - **支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime**
> - 整数类型下 `newVersion = oldVersion + 1`
> - `newVersion` 会回写到 `entity` 中
> - 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
> - **在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!**

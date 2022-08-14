# SpringBoot-mybatis-plus的事务回滚

事务是一组组合成逻辑工作单元的操作，虽然系统中可能会出错，但事务将控制和维护事务中每个操作的一致性和完整性。

### 首先先开启事务

```java
@EnableTransactionManagement //启动事务
```

### 在OrderServiceImpl的方法中添加事务标记 @Transactional


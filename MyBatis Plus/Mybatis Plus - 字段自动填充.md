# Mybatis Plus - 字段自动填充

## 1实体类修改

实体上增加字段并添加自动填充注解

```java
@TableField(fill = FieldFill.INSERT)
private Date createTime;  //create_time

@TableField(fill = FieldFill.INSERT_UPDATE)
private Date updateTime; //update_time

```



## 1实现元对象处理器接口

注意：不要忘记添加 `@Component` 注解

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    //mp执行添加操作，这个方法执行
    @Override
    public void insertFill(MetaObject metaObject) {
        this.setFieldValByName("createTime",new Date(),metaObject);
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }

    //mp执行修改操作，这个方法执行
    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName("updateTime",new Date(),metaObject);
    }
}

```
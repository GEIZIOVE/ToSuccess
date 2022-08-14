# FastJson

## 1.引入依赖

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>x.x.x</version>
</dependency>
```



## 2. @JSONField 介绍

### 2.1内部实现

```java
public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.42版本之后才支持
    int ordinal() default 0;

     // 指定字段的名称
    String name() default "";

    // 指定字段的格式，对日期格式有用
    String format() default "";

    // 是否序列化
    boolean serialize() default true;

    // 是否反序列化
    boolean deserialize() default true;
}
```

### 2.2使用

注解@JSONField，一般作用在get/set方法上面，常用的使用场景有下面三个：

- 修改和json字符串的字段映射【name】 -->就是指字段序列化后在json字符串的键的名称
- 格式化数据【format】
- 过滤掉不需要序列化的字段【serialize】

#### 2.2.1修改字段映射

```java
	private Integer aid;
　　// 实体类序列化为json字符串的时候，此类的aid字段，序列化为json中的testid字段
　　@JSONField(name="testid") 
　　public Integer getAid() {
　　    return aid;
　　}
　　// json字符串解析为类实体的时候，json中的id字段，写入此类的aid字段
　　@JSONField(name="id")
　　public void setAid(Integer aid) {
　　    this.aid = aid;
　　}
```

#### 2.2.2格式化

```java
 public class A {
      // 配置date序列化和反序列使用yyyyMMdd日期格式
      @JSONField(format="yyyyMMdd")
      public Date date;
 }
```



#### 2.2.3过滤不需要序列化的字段

```java
　　@JSONField(serialize = false)
　　public Integer getProgress() {
    　　return progress;
　　}
```



## 3.java对象转json

Fastjson**将java对象序列化为JSON字符串**，fastjson提供了一个最简单的入口

```java
public abstract class JSON {
    public static String toJSONString(Object object); //静态方法
}
```

eg：

```java
Model model = new Model();
model.id = 1001;

String json = JSON.toJSONString(model);
```

## 4.JSON串转对象 

JSON.parseobject（）

简单用例：

```java
 UserGroup group2 = JSON.parseobject(jsonString, UserGroup.class); //第一个参数是json字符串，第二个参数是待转成对象的类
```

## 5.JSON串转对象数组 

JSON.parseArray（）

```java
    // JSON串转对象数组  
 List<User> users2 = JSON.parseArray(jsonString2, User.class);  
```



## 6.java获取json中某个字段

```java
import com.alibaba.fastjson.JSONObject;
public class JsonTest {
    
    public static void main(String[] args) {
    
        // json串(以自己的为准)
        String str = "{"id":"75","shoppingCartItemList":[{"id":"407","num":"10"}]}";
        JSONObject jsonObject = JSONObject.parseObject(str);
        // 获取到key为shoppingCartItemList的值
        String r = jsonObject.getString("shoppingCartItemList");
        System.out.println(r);
    }
}
```



## 7.JsonObject

[		JSONObject](https://so.csdn.net/so/search?q=JSONObject&spm=1001.2101.3001.7020)只是一种数据结构，可以理解为JSON格式的数据结构（[key-value](https://www.baidu.com/s?wd=key-value&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao) 结构），可以使用put方法给json对象添加元素。JSONObject可以很方便的转换成[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)，也可以很方便的把其他对象转换成JSONObject对象。

## 7.1 JSON字符串转换成JSON对象

```java
String studentString = "{\"id\":1,\"age\":2,\"name\":\"zhang\"}";

//JSON字符串转换成JSON对象
JSONObject jsonObject1 = JSONObject.parseObject(studentString);

System.out.println(jsonObject1);
```


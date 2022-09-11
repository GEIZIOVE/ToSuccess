# 常用开发库 - JSON库

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/develop/package/dev-package-x-json.html

## JSON简介

### [¶](#json是什么) JSON是什么

- JSON 指的是 JavaScript 对象表示法（JavaScript Object Notation）
- JSON 是轻量级的文本数据交换格式
- JSON 独立于语言：JSON 使用 Javascript语法来描述数据对象，但是 JSON 仍然独立于语言和平台。JSON 解析器和 JSON 库支持许多不同的编程语言。 目前非常多的动态（PHP，JSP，.NET）编程语言都支持JSON。
- JSON 具有自我描述性，更易理解

### [¶](#结构与类型) 结构与类型

- 只有两种结构：**对象内的键值对集合结构和数组，对象用`{}`表示、内部是”key”:”value”，数组用`[]`表示，不同值用逗号分开**
- 基本数值有7个： false / null / true / object / array / number / string
- 再加上结构可以嵌套，进而可以用来表达复杂的数据

一个简单实例

```json
{
   "Image": {
       "Width":  800,
       "Height": 600,
       "Title":  "View from 15th Floor",
       "Thumbnail": {
           "Url":    "http://www.example.com/image/481989943",
           "Height": 125,
           "Width":  "100"
       },
       "IDs": [116, 943, 234, 38793]
     }
}
```



### [¶](#json优秀资源) JSON优秀资源

- [awesome-json  (opens new window)](https://github.com/burningtree/awesome-json)

### [¶](#json在线解析工具) JSON在线解析工具

- [JSON 在线解析  (opens new window)](http://c.runoob.com/front-end/53)

![img](E:\Development\Typora\images\dev-package-json-3.png)

## [¶](#json类库) JSON类库

Java中并没有内置JSON的解析，因此使用JSON需要借助第三方类库。

下面是几个常用的 JSON 解析类库：

- `FastJson`: 阿里巴巴开发的 JSON 库，性能优秀。
- `Jackson`: 社区十分活跃且更新速度很快。
- `Gson`: 谷歌开发的 JSON 库，功能十分全面。

### [¶](#性能测试对比) 性能测试对比

从下面的测试结果可以看出，序列化次数比较小的时候，Gson性能最好，当不断增加的时候到了100000，Gson明细弱于Jackson和FastJson， 这时候FastJson性能是真的牛，另外还可以看到不管数量少还是多，Jackson一直表现优异。而那个Json-lib可以直接忽略。

- JSON序列化性能

![img](E:\Development\Typora\images\dev-package-json-1.png)

- JSON反序列化性能

![img](E:\Development\Typora\images\dev-package-json-2.png)

更多请参考： [Java几种常用JSON库性能比较  (opens new window)](https://www.xncoding.com/2018/01/09/java/jsons.html)

## [¶](#fastjson) FastJson

先泼一盆冷水，个人非常不推荐使用FastJson, 为什么？

- FastJson 源码质量较低
- FastJson Bug、漏洞较多
- FastJson 牺牲多数场景下的稳定性而提高的效率

### [¶](#fastjson-简介) Fastjson 简介

Fastjson 是一个 Java 库，可以将 Java 对象转换为 JSON 格式，当然它也可以将 JSON 字符串转换为 Java 对象。

Fastjson 可以操作任何 Java 对象，即使是一些预先存在的没有源码的对象。

- [Fastjson Github  (opens new window)](https://github.com/alibaba/fastjson) 看这里
- [Fastjson 中文 Wiki  (opens new window)](https://github.com/alibaba/fastjson/wiki/Quick-Start-CN)

### [¶](#fastjson-特性) Fastjson 特性

- 提供服务器端、安卓客户端两种解析工具，性能表现较好。
- 提供了 `toJSONString()` 和 `parseObject()` 方法来将 Java 对象与 JSON 相互转换。调用toJSONString方 法即可将对象转换成 JSON 字符串，parseObject 方法则反过来将 JSON 字符串转换成对象。
- 允许转换预先存在的无法修改的对象（只有class、无源代码）。
- Java泛型的广泛支持。
- 允许对象的自定义表示、允许自定义序列化类。
- 支持任意复杂对象（具有深厚的继承层次和广泛使用的泛型类型）。

### [¶](#下载和使用) 下载和使用

你可以在 maven 中央仓库中直接下载：[http://repo1.maven.org/maven2/com/alibaba/fastjson/  (opens new window)](http://repo1.maven.org/maven2/com/alibaba/fastjson/)

配置 maven 依赖:

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>x.x.x</version>
</dependency>
```

其中 x.x.x 是版本号，根据需要使用特定版本，建议使用最新版本。

### [¶](#序列化一个对象成json字符串) 序列化一个对象成JSON字符串

Fastjson**将java对象序列化为JSON字符串**，fastjson提供了一个最简单的入口

```java
public abstract class JSON {
    public static String toJSONString(Object object); //静态方法
}
```

```java
User user = new User();
user.setName("校长");
user.setAge(3);
user.setSalary(new BigDecimal("123456789.0123"));
String jsonString = JSON.toJSONString(user);
System.out.println(jsonString);
// 输出 {"age":3,"name":"校长","old":false,"salary":123456789.0123}
```



### [¶](#反序列化一个json字符串成java对象) 反序列化一个JSON字符串成Java对象

JSON.parseobject（）

JSON.parseArray（）

```java
String jsonString = "{\"age\":3,\"birthdate\":1496738822842,\"name\":\"校长\",\"old\":true,\"salary\":123456789.0123}";
 User u = JSON.parseObject(jsonString ,User.class);
 System.out.println(u.getName());
 // 输出 校长

String jsonStringArray = "[{\"age\":3,\"birthdate\":1496738822842,\"name\":\"校长\",\"old\":true,\"salary\":123456789.0123}]";
List<User> userList = JSON.parseArray(jsonStringArray, User.class);
System.out.println(userList.size());
// 输出 1
```



###  [¶](#)@JSONField 介绍

#### 内部实现

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

#### 2.2使用

注解`@JSONField`，一般作用在get/set方法上面，常用的使用场景有下面三个：

- 修改和json字符串的字段映射【name】 --**>就是指字段序列化后在json字符串的键的名称**
- 格式化数据【format】
- 过滤掉不需要序列化的字段【serialize】

##### 2.2.1修改字段映射

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

##### 2.2.2格式化

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



##### 2.2.4设置字段顺序

```java
public static class VO {
    @JSONField(ordinal = 3)
    private int f0;

    @JSONField(ordinal = 2)
    private int f1;

    @JSONField(ordinal = 1)
    private int f2;
}
```



### [¶](#对于日期的处理) 对于日期的处理

默认序列化Date输出使用”yyyy-MM-dd HH:mm:ss”格式，可以用UseISO8601DateFormat特性换成`”yyyy-MM-dd’T’HH:mm:ss”`格式。

```java
JSON.defaultTimeZone = TimeZone.getTimeZone("Asia/Shanghai");
JSON.defaultLocale = Locale.US;
        
public static class Model {
    @JSONField(format = "MMM dd, yyyy h:mm:ss aa")
    private java.util.Date date;

    public java.util.Date getDate() {
        return date;
    }

    public void setDate(java.util.Date date) {
        this.date = date;
    }

    @JSONField(format = "MMM-dd-yyyy h:mm:ss aa")
    public java.sql.Date date2;
}
  

```

### [¶](#bean和数组转换) Bean和数组转换

- [官方例子 - BeanToArray_cn  (opens new window)](https://github.com/alibaba/fastjson/wiki/BeanToArray_cn)

### [¶](#自定义序列化和反序列化) 自定义序列化和反序列化

- [fastjson SerializerFeature详解  (opens new window)](https://blog.csdn.net/u010246789/article/details/52539576)
- [ObjectDeserializer_cn  (opens new window)](https://github.com/alibaba/fastjson/wiki/ObjectDeserializer_cn)

### [¶](#fastjson漏洞问题) FastJson漏洞问题

> **尽量使用最新版本。**

- [fastjson远程代码执行漏洞技术分析与防护方案  (opens new window)](http://blog.nsfocus.net/analysis-protection-fastjson-remote-code-execution-vulnerability/)

好了，我要开喷了。

DANGER

远离FastJson这个库，老程序员都知道这里有多少坑:

- [alibaba/fastjson  (opens new window)](https://github.com/alibaba/fastjson/issues)
- [fastjson这么快老外为啥还是热衷 jackson?  (opens new window)](https://www.zhihu.com/question/44199956/answer/112224034)

## [¶](#jackson) JackSon

### [¶](#jackson简介) JackSon简介

- [Jackson Github  (opens new window)](https://github.com/FasterXML/jackson) 看这里
- [Jackson Wiki  (opens new window)](http://wiki.fasterxml.com/JacksonHome)
- [Jackson 文档  (opens new window)](https://github.com/FasterXML/jackson-docs)

### [¶](#jackson组件) Jackson组件

#### [¶](#3个核心模块：) 3个核心模块：

- **Streaming**: jackson-core jar，定义了底层的streaming API和实现了Json特性。
- **Annotations**: jackson-annotations jar，包含了标准的Jackson注解。本文暂不介绍。
- **Databind**: jackson-databind jar，实现了数据绑定和对象序列化，它依赖于streaming和annotations的包。

#### [¶](#第三方数据类型模块) 第三方数据类型模块

这些扩展是插件式的Jackson模块，用ObjectMapper.registerModule()注册，并且通过添加serializers和deserializers以便Databind包（ObjectMapper / ObjectReader / ObjectWriter）可以读写这些类型，来增加对各种常用的Java库的数据类型的支持。

#### [¶](#数据格式模块) 数据格式模块

Jackson也有处理程序对JAX-RS标准实现者例如Jersey, RESTeasy, CXF等提供了数据格式支持。处理程序实现了MessageBodyReader和MessageBodyWriter，目前支持的数据格式包括JSON, Smile, XML, YAML和CBOR。

数据格式提供了除了Json之外的数据格式支持，它们绝大部分仅仅实现了streaming API abstractions，以便数据绑定组件可以按照原来的方式使用。另一些（几乎不需要）提供了databind标准功能来处理例如schemas。

### [¶](#jackson的使用) Jackson的使用

引用maven jar包：

```java
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.10.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.10.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-annotations -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.10.1</version>
</dependency>
```

### [¶](#序列化一个对象成json字符串-2) 序列化一个对象成JSON字符串

```java
public void toJson() throws JsonProcessingException {

    ObjectMapper mapper = new ObjectMapper();

    City case1 = new City();
    case1.setCity("SZ");
    case1.setAge(123);

    String jsonStr = mapper.writeValueAsString(case1);
    System.out.println("JSON:" + jsonStr);
}
// 输出：JSON:{"city":"SZ","age":123}
 
```

### [¶](#反序列化一个json字符串成java对象-2) 反序列化一个JSON字符串成Java对象

```java
public void toObj() throws JsonParseException, JsonMappingException, IOException {
    ObjectMapper mapper = new ObjectMapper();
    String inputjsonstr = "{\"city\":\"SZ\",\"age\":123}";
    
    City readcase = mapper.readValue(inputjsonstr, City.class);

    System.out.println("city info:" + readcase);
}
```



> 如果里面有未知属性，比如json中有`desc`字段，但是City中没有相应字段，会报错, 需要设置如下：

```java
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

### [¶](#常用注解) 常用注解

- `@JsonProperty("xxx")`: 将当前的属性名在json字符串中重新命名为当前设置的这个值，比如在示例中，将age-->mAge
- `@JsonIgnore`: 将被标注的属性在生成json字符串的时候，直接忽略
- `@JsonInclude`: 是一个类级别的设置，JsonInclude.Include.NON_EMPTY标识只有非NULL的值才会被纳入json string之中，其余的都被忽略，比如这里的location属性，并没有出现在最终的结果字符串中。
- `@JsonSerialize`: 使用自定义的类来实现自定义的字段转换。写入操作。
- `@JsonDeserialize`: 解析的时候，自定义的转换器；读取操作。
- `@JsonAutoDetect`: 设置类的访问策略，是否所有的属性都可以，还是按照一定的方式来提取。
- `@JsonRawValue`: 无转换的将属性值写入到json 字符串中。 写入操作
- `@JsonValue`: 标注方法，用以替代缺省的方法，由该方法来完成json的字符输出。

## [¶](#gson) GSON

### [¶](#gson简介) Gson简介

Gson是这样一个Java类库，它可以将Java对象转换为相应的JSON形式，也可以将JSON字符串转换为对应的Java对象。 Gson可以使用任意Java对象，包括哪些预先存在的、不在你的源代码中的对象（因此，你并不知道对象的属性）。

- [Gson用户指南（中文翻译）  (opens new window)](https://www.jianshu.com/p/1e20b28c39d1)看这里

### [¶](#gson的目标) Gson的目标

- 提供一种机制，使得将Java对象转换为JSON或相反如使用toString()以及构造器（工厂方法）一样简单。
- 允许预先存在的不可变的对象转换为JSON或与之相反。
- 允许自定义对象的表现形式
- 支持任意复杂的对象
- 输出轻量易读的JSON

### [¶](#gson的使用) Gson的使用

使用Gson的首要类是Gson类，你可以仅仅通过new Gson()的方式创建它。你也可以通过GsonBuilder类去创建Gson实例，这个类允许你进行一系列配置，例如版本控制等等。

Gson实例不会保存任何进行Json操作时的状态。因此，你可以自由的服用相同的Gson对象进行诸多的Json序列化和反序列化操作。

引用maven jar包：

```xml
<!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>28.2-jre</version>
</dependency>

```



### [¶](#序列化) 序列化

```java
// 基础类型
Gson gson = new Gson();
gson.toJson(1);            ==> prints 1
gson.toJson("abcd");       ==> prints "abcd"
gson.toJson(new Long(10)); ==> prints 10
int[] values = { 1 };
gson.toJson(values);       ==> prints [1]

// 对象
BagOfPrimitives obj = new BagOfPrimitives();
Gson gson = new Gson();
String json = gson.toJson(obj);  
==> json is {"value1":1,"value2":"abc"}

// 数组
Gson gson = new Gson();
int[] ints = {1, 2, 3, 4, 5};
String[] strings = {"abc", "def", "ghi"};
gson.toJson(ints);     ==> prints [1,2,3,4,5]
gson.toJson(strings);  ==> prints ["abc", "def", "ghi"]

// 集合
Gson gson = new Gson();
Collection<Integer> ints = Lists.immutableList(1,2,3,4,5);
String json = gson.toJson(ints); ==> json is [1,2,3,4,5]
  
```

其中的对象代码：

```java
class BagOfPrimitives {
  private int value1 = 1;
  private String value2 = "abc";
  private transient int value3 = 3;
  BagOfPrimitives() {
    // no-args constructor
  }
}
```



### [¶](#反序列化) 反序列化

```java
// 基础类型
Gson gson = new Gson();
int one = gson.fromJson("1", int.class);
Integer one = gson.fromJson("1", Integer.class);
Long one = gson.fromJson("1", Long.class);
Boolean false = gson.fromJson("false", Boolean.class);
String str = gson.fromJson("\"abc\"", String.class);
String anotherStr = gson.fromJson("[\"abc\"]", String.class);

// 对象
BagOfPrimitives obj2 = gson.fromJson(json, BagOfPrimitives.class);   
==> obj2 is just like obj

// 数组
Gson gson = new Gson();
int[] ints2 = gson.fromJson("[1,2,3,4,5]", int[].class); 
==> ints2 will be same as ints

// 集合
Gson gson = new Gson();
Type collectionType = new TypeToken<Collection<Integer>>(){}.getType();
Collection<Integer> ints2 = gson.fromJson(json, collectionType);
ints2 is same as ints
    
```



### [¶](#自定义序列化和反序列化机制) 自定义序列化和反序列化机制

有时候，默认的实现并不是你想要的。这在处理类库时常常发生（例如DateTime）。Gson允许你注册自己自定义的序列化器和反序列化器。该过程分为两部分：

- Json序列化器：需要为一个对象自定义序列化机制。
- Json反序列化器：需要为一个类型自定义反序列化机制。

实例构造者：并不需要，如果无参构造器是可用的或者注册了一个反序列化器。

```java
GsonBuilder gson = new GsonBuilder();
gson.registerTypeAdapter(MyType2.class, new MyTypeAdapter());
gson.registerTypeAdapter(MyType.class, new MySerializer());
gson.registerTypeAdapter(MyType.class, new MyDeserializer());
gson.registerTypeAdapter(MyType.class, new MyInstanceCreator());
  
  
```



registerTypeAdapter会检查类型适配器是否实现了上面三个接口中的一个以上并且它们都注册了类型适配器。

更多请参考：[Gson用户指南（中文翻译）](https://www.jianshu.com/p/1e20b28c39d1)
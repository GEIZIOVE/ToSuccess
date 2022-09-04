# Thymeleaf

## **一、Spring boot 集成 Thymeleaf**

### 1、在SpringBoot中使用Thymeleaf[模板引擎](https://so.csdn.net/so/search?q=模板引擎&spm=1001.2101.3001.7020)，添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 2、配置application.properties

```crystal
#开发阶段，建议关闭thymeleaf的缓存
spring.thymeleaf.cache=false
```

### 3、使用Controller去映射到模板页面

```java
@RequestMapping("/index")
public String index (Model model) {
    model.addAttribute("data", "恭喜，Spring boot集成 Thymeleaf成功！");
    //return 中就是你页面的名字（不带.html后缀）
    return "index";
}
```

### 4、新建html

**在src/main/resources的templates下新建一个index.html页面用于展示数据：**

HTML页面的`<html>`元素中加入以下属性：`<html xmlns:th="http://www.thymeleaf.org">`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8"/>
        <title>Spring boot集成 Thymeleaf</title>
    </head>
    <body>
    <p th:text="${data}">Spring boot集成 Thymeleaf</p>
    </body>
</html>
```

Springboot使用thymeleaf作为视图展示，约定将**模板文件**放置在`src/main/resource/templates`目录下，**静态资源**放置在`src/main/resource/static`目录下

## **二、Thymeleaf 的标准表达式**

Thymeleaf 的标准表达式主要有如下几类：

### 1、标准变量表达式

​		语法：`“${...}”`  ，变量表达式用于访问容器（tomcat）上下文环境中的变量，功能和 JSTL 中的 `${}` 相同，Thymeleaf 中的变量表达式使用 `${变量名}` 的方式获取其中的数据，比如在Spring mvc 的 Controllar中使用`model.addAttribute`向前端传输数据，代码如下：

```java
@RequestMapping(value="/userinfo")
public String userinfo (Model model) {
    User user = new User();
    user.setId(1);
    user.setNick("昵称");
    user.setPhone("13700020000");
    user.setAddress("北京朝阳区");
    model.addAttribute("user", user);
    model.addAttribute("hello", "helloworld");
    return "user";
}
```

前端接收代码：

```html
<td th:text="${user.nick}">x</td>
<td th:text="${user.phone}">137xxxxxxxx</td>
<td th:text="${user.email}">xxx@xx.com</td>
<td th:text="${user.address}">北京.xxx</td>
 
<span th:text="${hello}">你好</span>
```

其中`，th:text=""` 是Thymeleaf的一个属性，用于文本的显示；

### 2、选择变量表达式

​		Java代码同上

​		语法：`*{...}，`选择变量表达式，也叫星号变量表达式，使用 `th:object` 属性来绑定对象，比如：

```html
<div th:object="${user}" >
    <p>nick: <span th:text="*{nick}">张</span></p>
    <p>phone: <span th:text="*{phone}" >三</span></p>
    <p>email: <span th:text="*{email}" >北京</span></p>
    <p>address: <span th:text="*{address}" >北京</span></p>
</div>
```

​		选择表达式首先使用`th:object`来邦定后台传来的`User`对象，然后使用 `*` 来**代表这个对象**，后面 `{}` 中的值是此对象中的**属性**，选择变量表达式 `*{...}` 是另一种类似于变量表达式 `${...}` 表示变量的方法；

​		选择变量表达式在执行时**是在选择的对象上**求解，而`${...}`是在上下文的变量Model上求解；通过 `th:object` 属性指明选择变量表达式的求解对象；上述代码等价于：

```html
<div>
    <p>nick: <span th:text="${user.nick}">张</span></p>
    <p>phone: <span th:text="${user.phone}" >三</span></p>
    <p>email: <span th:text="${user.email}" >北京</span></p>
    <p>address: <span th:text="${user.address}" >北京</span></p>
</div>
```

标准变量表达式和选择变量表达式可以**混合一起**使用，比如：

```html
<div th:object="${user}" >
    <p>nick: <span th:text="*{nick}">张</span></p>
    <p>phone: <span th:text="${user.phone}" >三</span></p>
    <p>email: <span th:text="${user.email}" >北京</span></p>
    <p>address: <span th:text="*{address}" >北京</span></p>
</div>
```



也可以不使用 `th:object` 进行对象的选择，而直接使用 `*{...}` 获取数据，比如：

```html
<div>
    <p>nick: <span th:text="*{user.nick}">张</span></p>
    <p>phone: <span th:text="*{user.phone}" >三</span></p>
    <p>email: <span th:text="*{user.email}" >北京</span></p>
    <p>address: <span th:text="*{user.address}" >北京</span></p>
</div>
```



### 3、URL**表达式**

语法：`@{...}`，URL表达式可用于 `<script src="...">、<link href="...">、<a href="...">、<form action="...">`等，

#### ①：绝对URL

```html
<a th:href="@{'http://localhost:8080/boot/user/info?id='+${user.id}}">查看</a>
```

#### ②：相对URL，相对于页面

```html
<a th:href="@{'user/info?id='+${user.id}}">查看</a>
```

#### ③：相对URL，相对于项目上下文

```html
<a th:href="@{'/user/info?id='+${user.id}}">查看</a> （项目的上下文名会被自动添加）/ ${pageContext.request.contextPath}
```



## 三、Thymeleaf 的常见属性

**如下为thymeleaf的常见属性：**

### 1.`th:action`

定义后台控制器的路径，类似`<form>`标签的action属性，比如：`<form id="login" th:action="@{/login}">......</form>`

### 2.`th:each`

#### List类型的循环：  

这个属性非常常用，比如从后台传来一个对象集合那么就可以使用此属性遍历输出，它与JSTL中的<c: forEach>类似，此属性既可以循环遍历集合，也可以循环遍历数组及Map，比如：List类型的循环：

```html
<tr th:each="user, interStat : ${userlist}">
    <td th:text="${interStat.index}"></td>
    <td th:text="${user.id}"></td>
    <td th:text="${user.nick}"></td>
    <td th:text="${user.phone}"></td>
    <td th:text="${user.email}"></td>
    <td th:text="${user.address}"></td>
</tr>
```

  以上代码解读如下：

`th:each="user, iterStat : ${userlist}"` 中的 `${userList}` 是后台传来的Key，
user是`${userList}` 中的一个数据，
iterStat 是 `${userList}` 循环体的信息，
**其中user及iterStat自己可以随便写；**

`interStat`是循环体的信息，通过该变量可以获取如下信息：

`index、size、count、even、odd、first、last`，其含义如下：

- **index**: 当前迭代对象的index（从0开始计算）
- **count**: 当前迭代对象的个数（从1开始计算）
- **size**: 被迭代对象的大小
- **current**: 当前迭代变量
- **even**/**odd**: 布尔值，当前循环是否是偶数/奇数（从0开始计算）
- **first**: 布尔值，当前循环是否是第一个
- **last**: 布尔值，当前循环是否是最后一个

> 注意：循环体信息interStat也可以不定义，则默认采用迭代变量加上Stat后缀，即userStat

#### Map类型的循环：  

```html
<div th:each="myMapVal : ${myMap}">
    <span th:text="${myMapVal.key}"></span>
    <span th:text="${myMapVal.value}"></span>
    <br/>
</div>
```

`${myMapVal.key}` 是获取map中的**key**，`${myMapVal.value}` 是获取map中的**value**；



#### 数组类型的循环：  

```html
<div th:each="myArrayVal : ${myArray}">
    <div th:text="${myArrayVal}"></div>
</div> 
```



### 3.`th:href`

 定义超链接，比如：`<a  class="login" th:href="@{/login}">登录</a>`

### 4.`th:id`

类似html标签中的id属性，比如：`<span th:id="${hello}">aaa</span>`

### 5.`th:if`

条件判断，比如后台传来一个变量，判断该变量的值，1为男，2为女：

```html
<span th:if="${sex} == 1" >
	男：<input type="radio" name="se"  th:value="男" />
</span>
<span th:if="${sex} == 2">
	女：<input type="radio" name="se" th:value="女"  />
</span>
```

### 6.`th:unless`

`th:unless`是`th:if`的一个相反操作，上面的例子可以改写为：

```html
<span th:unless="${sex} == 1" >
	女：<input type="radio" name="se"  th:value="女" />
</span>
<span th:unless="${sex} == 2">
	男：<input type="radio" name="se" th:value="男"  />
</span>
```

### 7.`th:switch/th:case`

switch，case判断语句，比如：

```html
<div th:switch="${sex}">
  <p th:case="1">性别：男</p>
  <p th:case="2">性别：女</p>
  <p th:case="*">性别：未知</p>
</div>
```

​		一旦某个case判断值为true，剩余的case则都当做false，**“*”表示默认的case**，前面的case都不匹配时候，执行默认的case；

### 8.`th:object`

用于数据对象绑定，通常用于选择变量表达式（星号表达式），例如：

```html
<span th:object="${user}">
    <span th:text="*{name}"></span>
    <span th:text="${user.age}"></span>
</span>
```

### 9.`th:src`

用于外部资源引入，比如`<script>`标签的src属性，`<img>`标签的src属性，常与`@{}`表达式结合使用；

```html
<script th:src="@{/js/jquery-2.4.min.js}"></script>
<img th:src="@{/image/logo.png}"/>
```

### 10.`th:text`

用于文本的显示，比如：`<input type="text" id="realName" name="reaName" th:text="${realName}">`

### 11.`th:value`

类似html标签中的value属性，能对某元素的value属性进行赋值，比如：`<input type="hidden" id="userId" name="userId" th:value="${userId}">`

### 12.`th:attr`

该属性也是用于给HTML中某元素的某属性赋值，但该方式写法不够优雅，比如上面的例子可以写成如下形式：`<input type="hidden" id="userId" name="userId" th:attr="value=${userId}" >`

### 13.`th:onclick`

点击事件：`th:οnclick="'getCollect()'"`

### 14.`th:style`

设置样式：`th:style="'display:none;'"`

### 15.`th:method`

设置请求方法，比如：`<form id="login" th:action="@{/login}" th:method="post">......</form>`

### 16.`th:name`

设置表单名称，比如：`<input th:type="text" th:id="userName" th:name="userName">`

### 17.`th:inline` 

`th:inline` 有三个取值类型 (text, javascript 和 none)

该属性使用内联表达式`[[…]]`展示变量数据，比如：

**内联文本**

```html
<span th:inline="text">Hello, [[${user.nick}]]</span>
等同于：
<span>Hello, <span th:text="${user.nick}"></span></span>
```

`th:inline`写在任何父标签都可以，比如如下也是可以的：

```html
<body th:inline="text">
   ...
   <span>[[${user.nick}]]</span>
   ...
</body>
```

**内联脚本**

```html
<script th:inline="javascript" type="text/javascript">
    var user = [[${user.phone}]];
      alert(user);
</script>
<script th:inline="javascript" type="text/javascript">
      var msg  = "Hello," + [[${user.phone}]];
      alert(msg);
</script>
```

## **四、Thymeleaf 字面量**

### 1.文本字面量

用单引号`'...'`包围的字符串为文本字面量，比如：`<a th:href="@{'api/getUser?id=' + ${user.id}}">修改</a>`

### 2.数字字面量

```html
<p>今年是<span th:text="2017">1949</span>年</p>
 
<p>20年后, 将是<span th:text="2017 + 20">1969</span>年</p>
```

### 3.boolean字面量

true和false

```html
<p th:if="${isFlag == true}">
执行操作
</p>
```

### 4.null字面量

```html
<p th:if="${userlist == null}">
userlist为空
</p>
 
 
<p th:if="${userlist != null}">
userlist不为空
</p>
```

## 五、Thymeleaf 字符串拼接

一种是字面量拼接：`<span th:text="'当前是第'+${sex}+'页 ,共'+${sex}+'页'"></span>`

另一种更优雅的方式，使用“|”减少了字符串的拼接：`<span th:text="|当前是第${sex}页，共${sex}页|"></span>`

优雅！！！！！！

## 六、Thymeleaf 三元运算判断

`<span th:text="${sex eq 1} ? '男' : '女'">未知</span>`

## 七、Thymeleaf 运算和关系判断

算术运算：+ , - , * , / , %

关系比较: > , < , >= , <= ( gt , lt , ge , le )

相等判断：== , != ( eq , ne )

[Thymeleaf笔记总结_生出来，我养！的博客-CSDN博客_thymeleaf 多条件判断](https://blog.csdn.net/qq_39669058/article/details/90485532)
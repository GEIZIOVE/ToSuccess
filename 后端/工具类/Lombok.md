# Lombok

[稳定 (projectlombok.org)](https://projectlombok.org/features/)

## 1.Lombok 注解一览

> [`@Getter`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Getter.java) 注解，添加在**类**或**属性**上，生成对应的 get 方法。
>
> [`@Setter`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Setting.java) 注解，添加在**类**或**属性**上，生成对应的 set 方法。
>
> [`@ToString`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/ToString.java) 注解，添加在**类**上，生成 toString 方法。
>
> [`@EqualsAndHashCode`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/EqualsAndHashCode.java) 注解，添加在**类**上，生成 equals 和 hashCode 方法。
>
> `@AllArgsConstructor`、`@RequiredArgsConstructor`、`@NoArgsConstructor` 注解，添加在**类**上，为类自动生成对应参数的构造方法。
>
> [`@Data`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Data.java) 注解，添加在**类**上，是 5 个 Lombok 注解的组合。
>
> - 为所有属性，添加 `@Getter`、`@ToString`、`@EqualsAndHashCode` 注解的效果
> - 为非 `final` 修饰的属性，添加 `@Setter` 注解的效果
> - 为 `final` 修改的属性，添加 `@RequiredArgsConstructor` 注解的效果
>
> `@Value` 注解，添加在**类**上，和 `@Data` 注解类似，区别在于它会把所有属性默认定义为 `private final` 修饰，所以不会生成 set 方法。
>
> [`@CommonsLog`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/apachecommons/CommonsLog.java)、[`@Flogger`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/flogger/Flogger.java)、[`@Log`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/java/Log.java)、[`@JBossLog`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/jbosslog/JBossLog.java)、[@Log4j](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/log4j/Log4j.java)、[@Log4j2](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/log4j/Log4j2.java)、[@Slf4j](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Slf4j.java)、[@Slf4jX](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Slf4jX.java) 注解，添加在**类**上，自动为类添加对应的日志支持。
>
> `@NonNull` 注解，添加在**方法参数**、**类属性**上，用于自动生成 `null` 参数检查。若确实是 `null` 时，抛出 NullPointerException 异常。
>
> `@Cleanup` 注解，添加在方法中的**局部变量**上，在作用域结束时会自动调用 `#close()` 方法，来释放资源。例如说，使用在 Java IO 流操作的时候。
>
> `@Builder` 注解，添加在**类**上，给该类加个构造者模式 Builder 内部类。
>
> `@Synchronized` 注解，添加在**方法**上，添加同步锁。
>
> `@SneakyThrows` 注解，添加在**方法**上，给该方法添加 `try catch` 代码块。
>
> `@Accessors` 注解，添加在**方法**或**属性**上，并设置 `chain = true`，实现链式编程。



## 2.常用注解

### @NonNull

这个注解可以**用在成员方法或者构造方法的参数前面**，会自动产生一个关于此参数的非空检查，如果参数为空，则抛出一个空指针异常，举个例子：

编译前的代码：



```java
//成员方法参数加上@NonNull注解
public String getName(@NonNull Person p) {
    return p.getName();
}
```



编译后的代码：



```
public String getName(@NonNull Person p) {
    if (p == null) {
        throw new NullPointerException("person");
    }
    return p.getName();
}
```



### @Cleanup

这个注解**用在变量前面**，可以保证此变量代表的资源会被自动关闭，默认是调用资源的`close()`方法，如果该资源有其它关闭方法，可使用`@Cleanup("methodName")`来指定要调用的方法，就用输入输出流来举个例子：

编译前的代码：



```java
public static void main(String[] args) throws IOException {
    @Cleanup InputStream in = new FileInputStream(args[0]);
    @Cleanup OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[1024];
    while (true) {
        int r = in.read(b);
        if (r == -1) break;
        out.write(b, 0, r);
    }
 }
```



编译后的代码：



```java
public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
        OutputStream out = new FileOutputStream(args[1]);
        try {
            byte[] b = new byte[10000];
            while (true) {
                int r = in.read(b);
                if (r == -1) break;
                out.write(b, 0, r);
            }
        } finally {
            if (out != null) {
                out.close();
            }
        }
    } finally {
        if (in != null) {
            in.close();
        }
    }
}
```



### @Getter/@Setter

这一对注解从名字上就很好理解，**用在成员变量前面**，相当于为成员变量生成对应的get和set方法，同时还可以为生成的方法指定访问修饰符，当然，默认为public，直接来看下面的简单的例子：

编译前的代码：



```java
public class Programmer {
    @Getter
    @Setter
    private String name;

    @Setter(AccessLevel.PROTECTED)
    private int age;

    @Getter(AccessLevel.PUBLIC)
    private String language;
}
```



编译后的代码：



```
public class Programmer {
    private String name;
    private int age;
    private String language;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    protected void setAge(int age) {
        this.age = age;
    }

    public String getLanguage() {
        return language;
    }
}
```



这两个注解还可以直接**用在类上**，可以为此类里的所有**非静态成员变量**生成对应的get和set方法。

### @Getter(lazy=true)

如果Bean的一个字段的初始化是代价比较高的操作，比如加载大量的数据；同时这个字段并不是必定使用的。那么使用懒加载机制，可以保证节省资源。

懒加载机制，是对象初始化时，该字段并不会真正的初始化，而是第一次访问该字段时才进行初始化字段的操作。

### @ToString/@EqualsAndHashCode

这两个注解也比较好理解，就是生成`toString`，`equals`和`hashcode`方法，同时后者还会生成一个`canEqual`方法，用于判断某个对象是否是当前类的实例，生成方法时只会使用类中的**非静态和非transient成员变量**，这些都比较好理解，就不举例子了。

当然，这两个注解也可以添加限制条件，例如用`@ToString(exclude={"param1","param2"})`来**排除**param1和param2两个成员变量，或者用`@ToString(of={"param1","param2"})`来**指定**使用param1和param2两个成员变量，`@EqualsAndHashCode`注解也有同样的用法。

### @NoArgsConstructor/@RequiredArgsConstructor /@AllArgsConstructor

这三个注解都是**用在类上**的，第一个和第三个都很好理解，就是为该类产生无参的构造方法和包含所有参数的构造方法，第二个注解则使用类中所有**带有@NonNull注解的或者带有final修饰的成员变量**生成对应的构造方法。当然，和前面几个注解一样，**成员变量都是非静态的**，另外，如果类中**含有final修饰的成员变量，是无法使用@NoArgsConstructor注解**的。

三个注解都可以**指定**生成的构造方法的**访问权限**，同时，第二个注解还可以用`@RequiredArgsConstructor(staticName="methodName")`的形式生成一个指定名称的静态方法，返回一个调用相应的构造方法产生的对象，下面来看一个生动鲜活的例子：

编译前的代码：



```
@RequiredArgsConstructor(staticName = "sunsfan")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
@NoArgsConstructor
public class Shape {
    private int x;
    @NonNull
    private double y;
    @NonNull
    private String name;
}
```



编译后的代码：



```
public class Shape {
    private int x;
    private double y;
    private String name;

    public Shape() {
    }

    protected Shape(int x, double y, String name) {
        this.x = x;
        this.y = y;
        this.name = name;
    }

    public Shape(double y, String name) {
        this.y = y;
        this.name = name;
    }

    public static Shape sunsfan(double y, String name) {
        return new Shape(y, name);
    }
}
```



### @Data/@Value

`@Data`注解综合了`@Getter/@Setter`，`@ToString`，`@EqualsAndHashCode`和`@RequiredArgsConstructor`注解，其中`@RequiredArgsConstructor`使用了类中的带有`@NonNull`注解的或者final修饰的成员变量，它可以使用`@Data(staticConstructor="methodName")`来生成一个静态方法，返回一个调用相应的构造方法产生的对象。

`@Value`注解和`@Data`类似，**区别**在于它会**把所有成员变量默认定义为`private final`修饰，并且不会生成set方法**。

### @SneakyThrows

这个注解**用在方法上**，可以将方法中的代码用`try-catch`语句包裹起来，捕获异常并在`catch`中用`Lombok.sneakyThrow(e)`把异常抛出，可以使用`@SneakyThrows(Exception.class)`的形式指定抛出哪种异常，很简单的注解，直接看个例子：

编译前的代码：



```java
public class SneakyThrows implements Runnable {
    @SneakyThrows(UnsupportedEncodingException.class)
    public String utf8ToString(byte[] bytes) {
        return new String(bytes, "UTF-8");
    }

    @SneakyThrows
    public void run() {
        throw new Throwable();
    }
}
```



编译后的代码：



```
public class SneakyThrows implements Runnable {
    @SneakyThrows(UnsupportedEncodingException.class)
    public String utf8ToString(byte[] bytes) {
        try {
            return new String(bytes, "UTF-8");
        } catch(UnsupportedEncodingException uee) {
            throw Lombok.sneakyThrow(uee);
        }
    }

    @SneakyThrows
    public void run() {
        try {
            throw new Throwable();
        } catch(Throwable t) {
            throw Lombok.sneakyThrow(t);
        }
    }
}
```



### @Synchronized

这个注解**用在类方法或者实例方法上**，效果和`synchronized`关键字相同，**区别**在于**锁对象不同**，对于类方法和实例方法，`synchronized`关键字的锁对象分别是`类的class对象`和`this对象`，而`@Synchronized`的锁对象分别是`私有静态final对象LOCK`和`私有final对象lock`，当然，也可以`自己指定锁对象`，例子也很简单，往下看：

编译前的代码：



```
public class Synchronized {
    private final Object readLock = new Object();

    @Synchronized
    public static void hello() {
        System.out.println("world");
    }

    @Synchronized
    public int answerToLife() {
        return 42;
    }

    @Synchronized("readLock")
    public void foo() {
        System.out.println("bar");
    }
}
```



编译后的代码：



```
public class Synchronized {
    private static final Object $LOCK = new Object[0];
    private final Object $lock = new Object[0];
    private final Object readLock = new Object();

    public static void hello() {
        synchronized($LOCK) {
            System.out.println("world");
        }
    }

    public int answerToLife() {
        synchronized($lock) {
            return 42;
        }
    }

    public void foo() {
        synchronized(readLock) {
            System.out.println("bar");
        }
    }
}
```



### @Log

这个注解**用在类上**，可以省去从日志工厂生成日志对象这一步，直接进行日志记录，具体注解根据日志工具的不同而不同，同时，可以在注解中使用`topic`来指定生成log对象时的类名。不同的日志注解总结如下(上面是注解，下面是编译后的代码)：



```java
@CommonsLog
==> private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);

@JBossLog
==> private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);

@Log
==> private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());

@Log4j
==> private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);

@Log4j2
==> private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);

@Slf4j
==> private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);

@XSlf4j
==> private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
```



###  @Accessors 源码

我们打开 @Accessors 的[源码](https://so.csdn.net/so/search?q=源码&spm=1001.2101.3001.7020)可以看到：

（1）该注解主要作用是：当属性字段在生成 getter 和 setter 方法时，做一些相关的设置。

（2）当它可作用于类上时，修饰类中所有字段，当作用于具体字段时，只对该字段有效。

![img](E:\Development\Typora\images\20210830134340960.png)

该字段共有三个属性，分别是 [fluent](https://so.csdn.net/so/search?q=fluent&spm=1001.2101.3001.7020)，chain，prefix，下面我们分别来说明下，他的意思分别是什么？

#### 属性说明

#### fluent 属性

> 不写默认为false，当该值为 true 时，对应字段的 getter 方法前面就没有 get，setter 方法就不会有 set。

![img](E:\Development\Typora\images\20210830145636581.png)

####  chain 属性

> 不写默认为false，当该值为 true 时，对应字段的 setter 方法调用后，会返回当前对象。

![img](E:\Development\Typora\images\20210830151426468.png)

#### prefix 属性

> 该属性是一个字符串数组，当该数组有值时，表示忽略字段中对应的前缀，生成对应的 getter 和 setter 方法。

比如现在有 xxName 字段和 yyAge 字段，xx 和 yy 分别是 name 字段和 age 字段的前缀。

那么，我们在生成的 getter 和 setter 方法如下，它也是带有 xx 和 yy 前缀的。

![img](E:\Development\Typora\images\20210830155916146.png)

如果，我们把它的前缀加到 @Accessors 的属性值中，则可以像没有前缀那样，去调用字段的 getter和 setter 方法。

![img](E:\Development\Typora\images\2021083016013492.png)





## 3.Lombok深入理解

### Lombok的原理

> 会发现在Lombok使用的过程中，只需要添加相应的注解，无需再为此写任何代码。自动生成的代码到底是如何产生的呢？

**核心之处就是对于注解的解析上**。JDK5引入了注解的同时，也提供了两种解析方式。

- **运行时解析**

​	运行时能够解析的注解，必须将`@Retention`设置为RUNTIME, 比如`@Retention(RetentionPolicy.RUNTIME)`，这样就可以通过反射拿到该注解。java.lang,reflect反射包中提供了一个接口`AnnotatedElement`，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。

- **编译时解析**

编译时解析有两种机制，分别简单描述下：

1）Annotation Processing Tool

apt自JDK5产生，JDK7已标记为过期，不推荐使用，JDK8中已彻底删除，自JDK6开始，可以使用`Pluggable Annotation Processing API`来替换它，apt被替换主要有2点原因：

- api都在com.sun.mirror非标准包下
- 没有集成到javac中，需要额外运行

2）Pluggable Annotation Processing API

[JSR 269: Pluggable Annotation Processing API  (opens new window)](https://www.jcp.org/en/jsr/proposalDetails?id=269)自JDK6加入，作为apt的替代方案，它解决了apt的两个问题，javac在执行的时候会调用实现了该API的程序，这样我们就可以对编译器做一些增强，这时javac执行的过程如下：

![img](E:\Development\Typora\images\dev-package-lombok-2.png)

Lombok本质上就是一个实现了“JSR 269 API”的程序。在使用javac的过程中，它产生作用的具体流程如下：

- javac对源代码进行分析，生成了一棵抽象语法树（AST）
- 运行过程中调用实现了“JSR 269 API”的Lombok程序
- 此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和setter方法定义的相应树节点
- javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）

![img](E:\Development\Typora\images\dev-package-lombok-3.png)

从上面的Lombok执行的流程图中可以看出，在Javac 解析成AST抽象语法树之后, Lombok 根据自己编写的注解处理器，动态地修改 AST，增加新的节点（即Lombok自定义注解所需要生成的代码），最终通过分析生成JVM可执行的字节码Class文件。使用Annotation Processing自定义注解是在编译阶段进行修改，而JDK的反射技术是在运行时动态修改，两者相比，反射虽然更加灵活一些但是带来的性能损耗更加大。

### [¶](#lombok类似原理工具有什么) Lombok类似原理工具有什么

> 换言之，我们可以通过Lombok同样的思路解决什么问题？ 

- 第一个问题，我可以通过上述原理，自己实现一个类似Lombok 吗？

可以的，给你找了[一篇文章  (opens new window)](https://www.jianshu.com/p/fc06578e805a)

- 还有一些其它类库使用这种方式实现，比如:
  - [Google Auto  (opens new window)](https://github.com/google/auto)
  - Dagger
  - ...

### [¶](#lombok没有未来---java14-record了解下) Lombok没有未来 - Java14 Record了解下

> Lombok是没有未来的，因为Java完全可以对于这种纯数据载体通过另外一种方式表示, 所以有了[JEP 359: Records  (opens new window)](https://openjdk.java.net/jeps/359), 简单而言就是通过一个语法糖来解决。

![img](E:\Development\Typora\images\dev-package-lombok-1.png)

- 从前

```java
public class Range {
 
    private final int min;
    private final int max;
 
    public Range(int min, int max) {
        this.min = min;
        this.max = max;
    }
 
    public int getMin() {
        return min;
    }
 
    public int getMax() {
        return max;
    }
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Range range = (Range) o;
        return min == range.min && max == range.max;
    }
 
    @Override
    public int hashCode() {
        return Objects.hash(min, max);
    }
 
    @Override
    public String toString() {
        return "Range{" +
          "min=" + min +
          ", max=" + max +
          '}';
    }
}
   
```

- `Java14 record`

```java
public record Range(int min, int max) {}
```

没错就是这个简单！这个语法糖是不是有 “卧槽” 的感觉？我们声明这种类使用record 标识（目前不知道 record 会不会上升到关键字的高度）。当你用record 声明一个类时，该类将自动拥有以下功能：

- 获取成员变量的简单方法，以上面代码为例 min() 和 max() 。注意区别于我们平常getter的写法。
- 一个 equals 方法的实现，执行比较时会比较该类的所有成员属性
- 重写 equals 当然要重写 hashCode
- 一个可以打印该类所有成员属性的 toString 方法。
- 请注意只会有一个构造方法。

因为这个特性是 `preview feature`，默认情况下是无法编译和执行的。同样以上面为例我们需要执行：

```bash
 $ javac -d classes --enable-preview --release 14 Range.java
 $ java -classpath classes --enable-preview Range

```

在 Jshell 中运行

```bash
jshell> Range r = new Range(10, 20);
r ==> Range[min=10, max=20]
jshell> r.min()
$5 ==> 10
jshell> r.toString()
$6 ==> "Range[min=10, max=20]"
jshell> r
r ==> Range[min=10, max=20]
```

虽然 record 声明的类没有 final 关键字，实际上它是一个不可变类。除了一些限制外，它依旧是一个普通的Java 类。因此，我们可以像添加普通类一样添加逻辑。我们可以在构造实例时强制执行前提条件：

```java
public record Range(int min, int max) {
    public Range {
        if (min >= max)
            throw new IllegalArgumentException("min should be less than max");
    }
}  
```



另外我们也可以给 Records 类增加普通方法、静态属性、静态方法，这里不再举例；

**为了简化语法糖的推理，不能在类内声明成员属性**。以下是错误的示范：

```java
public record Range(int min, int max) {
    // 错误的示范
    private String name;
}  
```

## [¶](#lombok有什么坑) Lombok有什么坑

> 谈谈Lombok容易被忽视的坑, 看似代码简洁背后的代价。@pdai

### [¶](#data的坑) `@Data`的坑

在使用Lombok过程中，如果对于各种注解的底层原理不理解的话，很容易产生意想不到的结果。

举一个简单的例子，我们知道，当我们使用`@Data`定义一个类的时候，会自动帮我们生成`equals()`方法 。

但是如果只使用了`@Data`，而不使用`@EqualsAndHashCode(callSuper=true)`的话，会默认是`@EqualsAndHashCode(callSuper=false)`,这时候生成的`equals()`方法只会比较子类的属性，不会考虑从父类继承的属性，无论父类属性访问权限是否开放。

这就可能得到意想不到的结果。

### [¶](#代码可读性可调试性低) 代码可读性，可调试性低

在代码中使用了Lombok，确实可以帮忙减少很多代码，因为Lombok会帮忙自动生成很多代码。但是**这些代码是要在编译阶段才会生成的**，所以在开发的过程中，其实很多代码其实是缺失的。

在代码中大量使用Lombok，就导致代码的可读性会低很多，而且也会给代码调试带来一定的问题。 比如，我们想要知道某个类中的某个属性的getter方法都被哪些类引用的话，就没那么简单了。

### [¶](#lombok有很强的侵入性) Lombok有很强的侵入性

- **强J队友**

因为Lombok的使用要求开发者一定要在IDE中安装对应的插件。如果未安装插件的话，使用IDE打开一个基于Lombok的项目的话会提示找不到方法等错误。导致项目编译失败。也就是说，如果项目组中有一个人使用了Lombok，那么其他人就必须也要安装IDE插件。否则就没办法协同开发。

更重要的是，如果我们定义的一个jar包中使用了Lombok，那么就要求所有依赖这个jar包的所有应用都必须安装插件，这种侵入性是很高的。

- **影响升级**

因为Lombok对于代码有很强的侵入性，就可能带来一个比较大的问题，那就是会影响我们对JDK的升级。按照如今JDK的升级频率，每半年都会推出一个新的版本，但是Lombok作为一个第三方工具，并且是由开源团队维护的，那么他的迭代速度是无法保证的。

所以，如果我们需要升级到某个新版本的JDK的时候，若其中的特性在Lombok中不支持的话就会受到影响。

还有一个可能带来的问题，就是Lombok自身的升级也会受到限制。因为一个应用可能依赖了多个jar包，而每个jar包可能又要依赖不同版本的Lombok，这就导致在应用中需要做版本仲裁，而我们知道，jar包版本仲裁是没那么容易的，而且发生问题的概率也很高。

### [¶](#lombok破坏了封装性) Lombok破坏了封装性

以上几个问题，我认为都是有办法可以避免的。但是有些人排斥使用Lombok还有一个重要的原因，那就是他会破坏封装性。

众所周知，Java的三大特性包括`封装性`、`继承性`和`多态性`。

如果我们在代码中直接使用Lombok，那么他会自动帮我们生成getter、setter 等方法，这就意味着，一个类中的所有参数都自动提供了设置和读取方法。

举个简单的例子，我们定义一个购物车类：

```java
@Data
public class ShoppingCart { 

    //商品数目
    private int itemsCount; 

    //总价格
    private double totalPrice; 

    //商品明细
    private List items = new ArrayList<>();

}

//例子来源于《极客时间-设计模式之美》
  
```



我们知道，购物车中商品数目、商品明细以及总价格三者之前其实是有关联关系的，如果需要修改的话是要一起修改的。

但是，我们使用了Lombok的`@Data`注解，对于itemsCount 和 totalPrice这两个属性。虽然我们将它们定义成 `private` 类型，但是提供了 `public` 的 `getter`、`setter` 方法。

外部可以通过 `setter` 方法随意地修改这两个属性的值。我们可以随意调用 `setter` 方法，来重新设置 itemsCount、totalPrice 属性的值，这也会导致其跟 items 属性的值不一致。

而面向对象封装的定义是：通过访问权限控制，隐藏内部数据，外部仅能通过类提供的有限的接口访问、修改内部数据。所以，暴露不应该暴露的 setter 方法，明显违反了面向对象的封装特性。

好的做法应该是不提供`getter/setter`，而是只提供一个public的addItem方法，同时去修改itemsCount、totalPrice以及items三个属性。

> 以上问题其实也是可以解决的，但这提醒了我们需要理解Lombok，而不是一股脑的用`@Data`注解。 @pdai

## [¶](#总结) 总结

- 优缺点
  - 优点：大大减少了代码量，使代码非常简洁
  - 缺点：可能存在对队友不友好、对代码不友好、对调试不友好、对升级不友好等问题。Lombok还会导致破坏封装性的问题。`@Data`中覆盖`equals`和`hashCode`的坑等。
- 什么样的情况使用Lombok
  - 团队整体的共识，IDE规范，相关代码规范等
  - 对Lombok足够了解，比如知道其中的坑等
- 不推荐使用Lombok的理由
  - 其实我们不缺时间写Getter和Setter的，这些代码通常是由IDE生成的。简化也是有代价的。
  - 对Lombok认知不够，导致带来的坑。
  - Java14中Record了解下。
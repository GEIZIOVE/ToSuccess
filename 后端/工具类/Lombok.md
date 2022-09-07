# Lombok

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
Java - 枚举类详解

[Java面向对象-枚举类详解_NorthCastle的博客-CSDN博客_java枚举对象](https://blog.csdn.net/qq_39505245/article/details/119220298)

# 1.自定义实现枚举类

> 根据枚举类的定义，自定义实现枚举类时需要完成的要求：
>
>  1.对象属性 用 private final修饰 
>
> 2.构造器私有化，private 修饰 
>
> 3.对象在类中创建完成，public static final 修饰
>
>  4.可以存在其他的方法，如 对象属性的getter方法，toString方法等

下面定义了一个枚举类：存在四个季节对象的Season类

```java
public class Season {
    //1.私有化属性
    private final String seasonName;
    private final String seasonDesc;
    //2.私有化构造器
    private Season(String seasonName, String seasonDesc) {
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }
    //3.声明四个对象 春夏秋冬
    public static final Season SPRING = new Season("春天","春暖花开");
    public static final Season SUMMER = new Season("夏天","烈日炎炎");
    public static final Season AUTUMN = new Season("秋天","秋高气爽");
    public static final Season WINTER = new Season("冬天","白雪皑皑");

    //4.其他方法1： 对象属性的getter方法，此时无setter方法的
    public String getSeasonName() {
        return seasonName;
    }
    public String getSeasonDesc() {
        return seasonDesc;
    }

    //4.其他方法2： toString()方法
    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }
}
```

测试使用枚举类

```java
public class Application {
    public static void main(String[] args) {
        // 正常使用自定义的枚举类
        Season spring = Season.SPRING;
        System.out.println(spring); // 默认调用toString方法
        System.out.println(spring.getSeasonName()); // 正常调用方法
        System.out.println("=========================");
    }
}
```

```java
执行结果如下：
Season{seasonName='春天', seasonDesc='春暖花开'}
春天
=========================
```

# 2.使用enum关键字定义枚举类

```
定义的格式要求 ：
    1.使用enum关键字代替class；
    2*.类的枚举对象必须在类的最开始定义；
    3.有多个对象时，对象之间用英文的逗号隔开，在最后一个对象的后面用英文分号结束；
    4.声明枚举对象时的格式为  
    		【枚举对象名称1，枚举名称2.。。。】  或
    		【枚举对象名称1(构造方法参数。。。)，枚举名称2(构造方法参数)。。。】
    5.属性仍然用 private final 进行修饰；
    6.构造方法仍然用 private 修饰；
    7.可以写其他的方法：getter、重写toString()等
```



下面是一个使用enum关键字定义的枚举类

```java
public enum Week {

    // 枚举对象
    MONDAY ("星期一","上一天的语文课"),
    TUESDAY ("星期二","上一天的数学课"),
    WEDNESDAY ("星期三","上一天的英语课"),
    THURSDAY ("星期四","上一天的物理课"),
    FRIDAY ("星期五","上一天的化学课"),
    SATURDAY ("星期六","上一天的生物课"),
    SUNDAY ("星期日","上一天的自习课");

    //声明属性
    private final String weekName;
    private final String weekDesc;
    //构造方法
    Week(String weekName, String weekDesc) {
        this.weekName = weekName;
        this.weekDesc = weekDesc;
    }

    // 其他方法：getter方法，toString()方法等
    public String getWeekName() {
        return weekName;
    }

    public String getWeekDesc() {
        return weekDesc;
    }

    @Override
    public String toString() {
        return "Week{" +
                "weekName='" + weekName + '\'' +
                ", weekDesc='" + weekDesc + '\'' +
                '}';
    }
}

```
















# Java - 日志LoggerFactory.getLogger

**LoggerFactory.getLogger可以在IDE控制台打印日志，便于开发，一般加在最上面：**

## 使用：

```java
//调试日志
    private final static Logger logger = LoggerFactory.getLogger(xxxController.class);
12
```

优点：使用指定类初始化日志对象，在日志输出的时候，可以打印出日志信息所在类

## logger日志的几个方法

logger.debug、logger.info、logger.warn、logger.error、logger.fatal 的区别：

相同处：
它们的作用都是把错误信息写到文本日志里

不同的是它们表示的日志级别不同：
日志级别由高到底是：fatal -> error -> warn -> info -> debug,低级别的会输出高级别的信息，高级别的不会输出低级别的

信息，如等级设为Error的话，warn,info,debug的信息不会输出

修改日志输出的级别要在log4j文件中进行配置
项目正式发布后，一般会把日志级别设置为fatal或者error

## demo例子

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoggerFactoryDemo {
    private static final Logger LOGGER = LoggerFactory.getLogger(LoggerFactoryDemo.class);

    public static void main(String[] args) {
        for (int i=0;i<5;i++){
            LOGGER.info("这是一条数据"+i);
        }
    }
}
```

## 控制台输出：

```java
13:58:52.913 [main] INFO wwfww.warehouse.aaaaa.LoggerFactoryDemo - 这是一条数据0
13:58:52.925 [main] INFO wwfww.warehouse.aaaaa.LoggerFactoryDemo - 这是一条数据1
13:58:52.925 [main] INFO wwfww.warehouse.aaaaa.LoggerFactoryDemo - 这是一条数据2
13:58:52.925 [main] INFO wwfww.warehouse.aaaaa.LoggerFactoryDemo - 这是一条数据3
13:58:52.925 [main] INFO wwfww.warehouse.aaaaa.LoggerFactoryDemo - 这是一条数据4
```
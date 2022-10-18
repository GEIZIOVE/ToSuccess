# SpringBoot-异步调用-@Async

## 1. 概述

### 1.1 简介

**介绍：**

- 异步请求的处理。除了异步请求，一般上我们用的比较多的应该是**异步调用**。通常在开发过程中，会遇到一个方法是和实际业务无关的，没有紧密性的。比如记录日志信息等业务。这个时候正常就是启一个新线程去做一些业务处理，让主线程异步的执行其他业务。

**使用方式：**

在 Spring Framework 的 Spring Task 模块，提供了 `@Async` 注解，可以添加在方法上，自动实现该方法的[异步调用](https://so.csdn.net/so/search?q=异步调用&spm=1001.2101.3001.7020)

需要在启动类或配置类加上`@EnableAsync`使异步调用`@Async`注解生效

```java
@SpringBootApplication
@EnableAsync
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

在需要异步执行的方法上加入此注解即可`@Async("threadPool")`，threadPool为自定义[线程池](https://so.csdn.net/so/search?q=线程池&spm=1001.2101.3001.7020)。(`@Async`默认使用`SimpleAsyncTaskExecutor`线程池。也可以根据Bean Name指定特定线程池)

**注意事项：**

​		在默认情况下，未设置`TaskExecutor`时，默认是使用`SimpleAsyncTaskExecutor`这个线程池，但此线程不是真正意义上的线程池，因为线程不重用，每次调用都会创建一个新的线程。可通过控制台日志输出可以看出，每次输出线程名都是递增的。所以最好我们来自定义一个线程池。

​		调用的异步方法，不能为同一个类的方法（包括同一个类的内部类）

​		**简单来说，因为Spring在启动扫描时会为其创建一个代理类，而同类调用时，还是调用本身的代理类的，所以和平常调用是一样的。**

​		其他的注解如`@Cache`等也是一样的道理，说白了，就是Spring的代理机制造成的。所以在开发中，最好把异步服务单独抽出一个类来管理。



**一些异步使用场景**

- 文章阅读的业务逻辑 = 查询文章详情 + 更新文章阅读量后再响应客户端， 其实也无需等到阅读量更新后才响应文章详情给客户端， 用户查看文章是主要逻辑， 而文章阅读量更新是次要逻辑， 况且阅读量就算更新失败一点数据偏差也不会影响用户阅读因此这两个数据库操作之间的一致性是较弱的，这类都能用异步事件去优化。

### 1.2 @[Async](https://so.csdn.net/so/search?q=Async&spm=1001.2101.3001.7020)失效场景

- 异步方法使用`static`修饰
- 异步类没有使用`@Component`、`@Service`等注解，导致spring无法扫描到异步类
- 调用的异步方法，不能为同一个类的方法（包括同一个类的内部类）。PS：因为Spring在启动扫描时会为其创建一个代理类，而同类调用时，还是调用本身的代理类的，所以和平常调用是一样的
- 类中需要使用`@Autowired`或`@Resource`等注解自动注入，不能自己手动new对象
- 如果使用SpringBoot框架必须在启动类中增加`@EnableAsync`注解
- 在Async 方法上标注`@Transactional`是没用的。 在Async 方法调用的方法上标注`@Transactional` 有效。

## 2. 同步与异步调用示例

> `异步调用`，对应的是`同步调用`。
>
> - 同步调用：指程序按照 定义顺序 依次执行，每一行程序都必须等待上一行程序执行完成之后才能执行；
> - 异步调用：指程序在顺序执行时，不等待异步调用的语句返回结果，就执行后面的程序。

**同步service方法**

```java
/**
 * @Author LiangYiXiang
 * @Description 定义同步任务service
 * @Date 2021/11/13
 **/
@Component
public class Task {

    public static Random random =new Random();

    public void doTaskOne() throws Exception {
        System.out.println("开始做任务一");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
    }

    public void doTaskTwo() throws Exception {
        System.out.println("开始做任务二");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
    }

    public void doTaskThree() throws Exception {
        System.out.println("开始做任务三");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务三，耗时：" + (end - start) + "毫秒");
    }

}

```

**异步service方法**

```java
@Component
public class AsyncTask {

    public static Random random =new Random();

    @Async
    public Future<String> doTaskOne() throws Exception {
        System.out.println("开始做任务一");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务一，耗时：" + (end - start) + "毫秒");
        return new AsyncResult<>("任务一完成");
    }

    @Async
    public Future<String> doTaskTwo() throws Exception {
        System.out.println("开始做任务二");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
        return new AsyncResult<>("任务二完成");
    }

    @Async
    public Future<String> doTaskThree() throws Exception {
        System.out.println("开始做任务三");
        long start = System.currentTimeMillis();
        Thread.sleep(random.nextInt(10000));
        long end = System.currentTimeMillis();
        System.out.println("完成任务三，耗时：" + (end - start) + "毫秒");
        return new AsyncResult<>("任务三完成");
    }

}

```

**测试类**

PS：记得在启动类或配置类加上`@EnableAsync`，使异步调用`@Async`注解生效

```java
@SpringBootTest
public class TaskTest {

    @Autowired
    private Task task;

    @Autowired
    private AsyncTask asyncTask;

    /**
     * 三个task同步调用
     *
     * @author Liangyixiang
     * @date 2021/11/13
     **/
    @Test
    public void taskTest() throws Exception {
        long start = System.currentTimeMillis();
        task.doTaskOne();
        task.doTaskTwo();
        task.doTaskThree();
        long end = System.currentTimeMillis();

        System.out.println("同步调用全部完成，总耗时：" + (end - start) + "毫秒");
    }

    /**
     * 三个task异步调用
     * 与同步区别：
     * 1. 在测试用例一开始记录开始时间
     *
     * 2. 在调用三个异步函数的时候，返回Future类型的结果对象
     *
     * 3. 在调用完三个异步函数之后，开启一个循环，根据返回的Future对象来判断三个异步函数是否都结束了。若都结束，就结束循环；若没有都结束，就等1秒后再判断。
     *
     * @author Liangyixiang
     * @date 2021/11/13
     **/
    @Test
    public void asyncTest() throws Exception {

        long start = System.currentTimeMillis();

        Future<String> task1 = asyncTask.doTaskOne();
        Future<String> task2 = asyncTask.doTaskTwo();
        Future<String> task3 = asyncTask.doTaskThree();

        while(true) {
            if(task1.isDone() && task2.isDone() && task3.isDone()) {
                // 三个任务都调用完成，退出循环等待
                break;
            }
            Thread.sleep(1000);
        }

        long end = System.currentTimeMillis();

        System.out.println("异步调用全部完成，总耗时：" + (end - start) + "毫秒");

    }
}
```

**结果：**

异步处理的耗时还是比同步快很多(这里没有控制变量，但是看结果懂的都懂)

![image-20211114105807939](E:\Development\Typora\images\4a567cee692f68fddc464e306e50d280.png)

![image-20211114105823328](E:\Development\Typora\images\dde044925fe3d51495f0ff8b8044f4b5.png)

泛型指定返回值的类型，`AsyncResult`为Spring实现的`Future`实现类：

![QQ截图20190302140425.png](E:\Development\Typora\images\QQ截图20190302140425.png)



```java
String result = stringFuture.get(60, TimeUnit.SECONDS);
```

## 3. @Async异步调用使用详解及优化

### 3.1 当前使用分析

Spring Task 提供的 `@Async` 注解，声明式**异步** 。在实现原理上，也是基于 Spring AOP 拦截，调用者将在调用时立即返回，方法的实际执行将提交给Spring TaskExecutor的任务中，由指定的线程池中的线程执行，达到异步调用的目的。

Spring应用默认的线程池，指在`@Async`注解在使用时，不指定线程池的名称。查看源码，@Async的默认线程池为SimpleAsyncTaskExecutor。

**默认线程池的弊端**

在线程池应用中，参考阿里巴巴java开发规范：线程池不允许使用Executors去创建，不允许使用系统默认的线程池，推荐通过ThreadPoolExecutor的方式，这样的处理方式让开发的工程师更加明确线程池的运行规则，规避资源耗尽的风险。

Executors各个方法的弊端：

- `newFixedThreadPool`和`newSingleThreadExecutor`：主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
- `newCachedThreadPool`和`newScheduledThreadPool`：要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。

`@Async`默认异步配置使用的是`SimpleAsyncTaskExecuto`r，该线程池默认来一个任务创建一个线程，若系统中不断的创建线程，最终会导致系统占用内存过高，引发`OutOfMemoryErro`r错误。针对线程创建问题，`SimpleAsyncTaskExecutor`提供了限流机制，通过`concurrencyLimit`属性来控制开关，当**concurrencyLimit>=0**时开启限流机制，默认关闭限流机制即**concurrencyLimit=-1**，当关闭情况下，会不断创建新的线程来处理任务。基于默认配置，`SimpleAsyncTaskExecutor`并不是严格意义的线程池，达不到线程复用的功能。

> **Spring 已经实现的线程池**
>
> - SimpleAsyncTaskExecutor：不是真的线程池，这个类不重用线程，默认每次调用都会创建一个新的线程。
> - SyncTaskExecutor：这个类没有实现异步调用，只是一个同步操作。只适用于不需要多线程的地方。
> - ConcurrentTaskExecutor：Executor的适配类，不推荐使用。如果ThreadPoolTaskExecutor不满足要求时，才用考虑使用这个类。
> - SimpleThreadPoolTaskExecutor：是Quartz的SimpleThreadPool的类。线程池同时被quartz和非quartz使用，才需要使用此类。
> - ThreadPoolTaskExecutor ：最常使用，推荐。其实质是对java.util.concurrent.ThreadPoolExecutor的包装。
>
> 异步的方法有
>
> - 最简单的异步调用，返回值为void
> - 带参数的异步调用，异步方法可以传入参数
> - 存在返回值，常调用返回Future

### 3.2 自定义线程池执行异步方法

​		自定义线程池，可对系统中线程池更加细粒度的控制，方便调整线程池大小配置，线程执行异常控制和处理。在设置系统自定义线程池代替默认线程池时，虽可通过多种模式设置，但替换默认线程池最终产生的线程池有且只能设置一个（不能设置多个类继承AsyncConfigurer）自定义线程池有如下模式：

- 重新实现接口`AsyncConfigurer`
- 继承`AsyncConfigurerSupport`
- 配置由自定义的`TaskExecutor`替代内置的任务执行器

通过查看Spring源码关于@Async的默认调用规则，会优先查询源码中实现`AsyncConfigurer`这个接口的类，实现这个接口的类为`AsyncConfigurerSupport`。但默认配置的线程池和异步处理方法均为空，所以，无论是继承或者重新实现接口，都需指定一个线程池。且重新实现`public Executor getAsyncExecutor()`方法

**最简单的实现方式：**

```java
/**
 * @Author LiangYiXiang
 * @Description 配置自定义线程池 bean name方式调用
 * @Date 2021/11/14
 **/
@Configuration
public class AsyncConfig {

    @Bean("customExecutor")
    public ThreadPoolTaskExecutor asyncOperationExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 设置核心线程数
        executor.setCorePoolSize(8);
        // 设置最大线程数
        executor.setMaxPoolSize(20);
        // 设置队列大小
        executor.setQueueCapacity(Integer.MAX_VALUE);
        // 设置线程活跃时间(秒)
        executor.setKeepAliveSeconds(60);
        // 设置线程名前缀+分组名称
        executor.setThreadNamePrefix("AsyncOperationThread-");
        executor.setThreadGroupName("AsyncOperationGroup");
        // 所有任务结束后关闭线程池
        executor.setWaitForTasksToCompleteOnShutdown(true);
        // 初始化
        executor.initialize();
        return executor;
    }
}
```

- `corePoolSize`：线程池核心线程的数量，默认值为1（这就是默认情况下的异步线程池配置使得线程不能被重用的原因）。

- `maxPoolSize`：线程池维护的线程的最大数量，只有当核心线程都被用完并且缓冲队列满后，才会开始申超过请核心线程数的线程，默认值为`Integer.MAX_VALUE`。

- `queueCapacity`：缓冲队列。

- `keepAliveSeconds`：超出核心线程数外的线程在空闲时候的最大存活时间，默认为60秒。

- `threadNamePrefix`：线程名前缀。

- `waitForTasksToCompleteOnShutdown`：是否等待所有线程执行完毕才关闭线程池，默认值为false。

- `awaitTerminationSeconds`：`waitForTasksToCompleteOnShutdown`的等待的时长，默认值为0，即不等待。

- `rejectedExecutionHandler`：当没有线程可以被使用时的处理策略（拒绝任务），默认策略为`abortPolicy`，包含下面四种策略：

  ![QQ截图20190302111014.png](https://mrbird.cc/img/QQ%E6%88%AA%E5%9B%BE20190302111014.png)

  1. `callerRunsPolicy`：用于被拒绝任务的处理程序，它直接在 execute 方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务。
  2. `abortPolicy`：直接抛出`java.util.concurrent.RejectedExecutionException`异常。
  3. `discardOldestPolicy`：当线程池中的数量等于最大线程数时、抛弃线程池中最后一个要执行的任务，并执行新传入的任务。
  4. `discardPolicy`：当线程池中的数量等于最大线程数时，不做任何动作。



要使用该线程池，只需要在`@Async`注解上指定线程池Bean名称即可：

```java
/**
 * 使用自定义线程池异步方法
 *
 * @return void
 * @author Liangyixiang
 * @date 2021/11/14
 **/
@Async("customExecutor")
public void testAsyncTask3() throws InterruptedException {
    System.out.println("内部线程：" + Thread.currentThread().getName());
    System.out.println("开始执行任务2");
    long start = System.currentTimeMillis();
    Thread.sleep(2000);
    long end = System.currentTimeMillis();
    System.out.println("完成任务二，耗时：" + (end - start) + "毫秒");
}
12345678910111213141516
```

PS：比较推荐的配置方式如下3.3

### 3.3 全局处理异步方法中的异常

实现`AsyncConfigurer`接口的`getAsyncExecutor`方法和`getAsyncUncaughtExceptionHandler`方法改造配置类

全局异步异常处理：

```java
/**
 * @Author LiangYiXiang
 * @Description 全局异步调用错误处理
 * @Date 2021/11/14
 **/
public class CustomAsyncExceptionHandler implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(Throwable throwable, Method method, Object... obj) {
        System.out.println("异常捕获---------------------------------");
        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
        System.out.println("异常捕获---------------------------------");
    }

}
```

配置类：

```java
/**
 * @Author LiangYiXiang
 * @Description 实现AsyncConfigurer接口，配置自定义线程池
 * @Date 2021/11/14
 **/
@Configuration
public class AsyncConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 设置核心线程数
        executor.setCorePoolSize(8);
        // 设置最大线程数
        executor.setMaxPoolSize(20);
        // 设置队列大小
        executor.setQueueCapacity(Integer.MAX_VALUE);
        // 设置线程活跃时间(秒)
        executor.setKeepAliveSeconds(60);
        // 设置线程名前缀+分组名称
        executor.setThreadNamePrefix("MyThread-");
        executor.setThreadGroupName("MyAsyncGroup");
        // 所有任务结束后关闭线程池
        executor.setWaitForTasksToCompleteOnShutdown(true);
        // 初始化
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new CustomAsyncExceptionHandler();
    }
}
```

## 4. 异步请求与异步调用的区别

两者的使用场景不同，异步请求用来解决并发请求对服务器造成的压力，从而提高对请求的吞吐量；而异步调用是用来做一些非主线流程且不需要实时计算和响应的任务，比如同步日志到kafka中做日志分析等。

异步请求是会一直等待response相应的，需要返回结果给客户端的；而异步调用我们往往会马上返回给客户端响应，完成这次整个的请求，至于异步调用的任务后台自己慢慢跑就行，客户端不会关心。

## 5. 最后一些思考

- **进程内** 的队列或者线程池，相对**不可靠** 的原因是，队列和线程池中的任务仅仅存储在内存中，如果 JVM 进程被异常关闭，将会导致丢失，未被执行。例如：异步方法执行失败后对Controller前半部分的非异步操作无影响， 异步方法在整个业务逻辑中不是100%可靠的，对于强一致性的业务来说不适用。
- 而分布式消息队列，异步调用会以一个消息的形式，存储在消息队列的服务器上，所以即使 JVM 进程被异常关闭，消息依然在消息队列的服务器上。所以，使用**进程内** 的队列或者线程池来实现异步调用的话，一定要尽可能的保证 JVM 进程的优雅关闭，保证它们在关闭前被执行完成。
- 考虑到异步调用的**可靠性** ，我们一般会考虑引入分布式消息队列，例如说 RabbitMQ、RocketMQ、Kafka 等等。但是在一些时候，我们并不需要这么高的可靠性，可以使用**进程内** 的队列或者线程池。





# 关于@Async线程池配置

## 1、修改@Async默认线程池

关于@Async的原理，可以查看 [Spring原理之@Async](https://blog.csdn.net/weixin_42272869/article/details/116117202?ops_request_misc=%7B%22request%5Fid%22%3A%22164542538916780264096138%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fblog.%22%7D&request_id=164542538916780264096138&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-116117202.nonecase&utm_term=%40Async&spm=1018.2226.3001.4450) 这篇博客，这里不在阐述

关于修改 @Async默认的线程池 ，我们仅仅需要实现一个 **AsyncConfigurer** 类，进行**getAsyncExecutor 方法 **的重写即可，具体例子如下：

```java
@Slf4j
@EnableAsync //对应的@Enable注解，最好写在属于自己的配置文件上，保持内聚性
@Configuration
public class AsyncConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10); //核心线程数
        executor.setMaxPoolSize(20);  //最大线程数
        executor.setQueueCapacity(1000); //队列大小
        executor.setKeepAliveSeconds(300); //线程最大空闲时间
        executor.setThreadNamePrefix("async-Executor-"); 指定用于新创建的线程名称的前缀。
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy()); // 拒绝策略（一共四种，此处省略）

        // 这一步千万不能忘了，否则报错： java.lang.IllegalStateException: ThreadPoolTaskExecutor not initialized
        executor.initialize();
        return executor;
    }
    
    // 异常处理器：当然你也可以自定义的，这里我就这么简单写了~~~
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return return new AsyncUncaughtExceptionHandler() {

            @Override
            public void handleUncaughtException(Throwable arg0, Method arg1, Object... arg2) {
                log.error("=========================="+arg0.getMessage()+"=======================", arg0);
                log.error("exception method:"+arg1.getName());
            }
        };
    }

}
```

具体使用：

```java
@Component 
public class AsyncMyTask {    
    protected final Logger logger = LoggerFactory.getLogger(this.getClass());     
    // 直接使用该注解即可
    @Async    
    public void doTask(int i) throws InterruptedException{        
        logger.info("Task-Native" + i + " started.");    
    } 
}
```

## 2、自定义线程池

我们可以自定义一个线程池，也是写一个配置类，在使用的时候，根据线程池的名称来使用具体的线程池

```java
/**
 * 自定义创建线程池
 */
@Configuration
@EnableAsync
public class TaskExecutePool {
    @Bean
    public Executor myTaskAsyncPool() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10); //核心线程数
        executor.setMaxPoolSize(20);  //最大线程数
        executor.setQueueCapacity(1000); //队列大小
        executor.setKeepAliveSeconds(300); //线程最大空闲时间
        executor.setThreadNamePrefix("async-Executor-"); 指定用于新创建的线程名称的前缀。
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy()); // 拒绝策略（一共四种，此处省略）
        executor.initialize();
        return executor;
    }
}

```

使用：

```java
@Component 
public class AsyncMyTask {    
    protected final Logger logger = LoggerFactory.getLogger(this.getClass());     
    // 注解中标注使用的线程池是哪一个
    // myTaskAsynPool即配置线程池的方法名，此处如果不写自定义线程池的方法名，会使用默认的线程池
    @Async("myTaskAsyncPool")
    public void doTask(int i) throws InterruptedException{        
        logger.info("Task-Native" + i + " started.");    
    } 
}

```

如果想通过配置文件来对线程池中的属性进行方便修改的话：

（1）配置文件中进行配置

```java
task.pool.corePoolSize=20
task.pool.maxPoolSize=40
task.pool.keepAliveSeconds=300
task.pool.queueCapacity=50

```

（2）编写获取配置属性的配置类

```java
/**

线程池配置属性类
*/
@ConfigurationProperties(prefix = "task.pool")
@Data
public class ThreadPoolConfig {
    private int corePoolSize;

    private int maxPoolSize;

    private int keepAliveSeconds;

    private int queueCapacity;
}
```

（3）使用线程池配置属性类

```java
/**
 * 创建线程池
*/
   @Configuration
   @EnableAsync
   public class TaskExecutePool {
       @Autowired
       private ThreadPoolConfig config;

       @Bean
       public Executor myTaskAsyncPool() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           //核心线程池大小
           executor.setCorePoolSize(config.getCorePoolSize());
           //最大线程数
           executor.setMaxPoolSize(config.getMaxPoolSize());
           //队列容量
           executor.setQueueCapacity(config.getQueueCapacity());
           //活跃时间
           executor.setKeepAliveSeconds(config.getKeepAliveSeconds());
           //线程名字前缀
           executor.setThreadNamePrefix("async-Executor-");
           // 拒绝策略
           executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
           executor.initialize();
           return executor;
       }
   }
```


# Java线程池：ExecutorService 的理解与使用

## ExecutorService 

接口 `java.util.concurrent.ExecutorService` 表述了异步执行的机制，并且可以让任务在后台执行。一个 `ExecutorService` 实例因此特别像一个线程池。

事实上，在 java.util.concurrent 包中的 ExecutorService 的实现就是一个线程池的实现。

### **ExecutorService 样例**

这里有一个简单的使用Java 实现的 ExectorService 样例：

```java
ExecutorService executorService = Executors.newFixedThreadPool(10);  
  
executorService.execute(new Runnable() {  
    public void run() {  
        System.out.println("Asynchronous task");  
    }  
});  
  
executorService.shutdown();
```

首先使用 `newFixedThreadPool()` 工厂方法创建一个 `ExecutorService` ，上述代码创建了一个可以容纳10个线程任务的线程池。

其次，向 `execute()` 方法中传递一个异步的 `Runnable` 接口的实现，这样做会让 `ExecutorService` 中的<u>某个线程</u>执行这个 Runnable 线程。

![image-20220915142127810](E:\Development\Typora\images\image-20220915142127810.png)

----

**任务的委托（Task Delegation）**

下方展示了一个线程的把任务委托异步执行的ExecutorService的示意图。

![img](E:\Development\Typora\images\1620.png)

一旦线程把任务委托给 `ExecutorService`，该线程就会继续执行与运行任务无关的其它任务。

### **ExecutorService 的实现**

由于 `ExecutorService` 只是一个接口，你一量需要使用它，那么就需要提供一个该接口的实现。ExecutorService 接口在 java.util.concurrent 包中有如下实现类：

- **ThreadPoolExecutor**
- **ScheduledThreadPoolExecutor**

----

### **实例化 ExecutorService**

你可以根据自己的需要来创建一个 `ExecutorService` ，也可以使用 Executors 工厂方法来创建一个 ExecutorService 实例。这里有几个创建 ExecutorService 的例子：

```java
ExecutorService executorService1 = Executors.newSingleThreadExecutor(); // 单线程
ExecutorService executorService2 = Executors.newFixedThreadPool(10); // 固定线程池
ExecutorService executorService3 = Executors.newScheduledThreadPool(10); // 定时线程池
```

![image-20220915150644991](E:\Development\Typora\images\image-20220915150644991.png)



### **ExecutorService 使用方法**

![image-20220915145209183](E:\Development\Typora\images\image-20220915145209183.png)

这里有几种不同的方式让你将任务委托给一个 ExecutorService：

```java
execute(Runnable)  
submit(Runnable)  
submit(Callable)  
invokeAny(...)  
invokeAll(...)
```

----

#### **execute(Runnable)**

方法 `execute(Runnable)` 

​		接收一个 java.lang.Runnable 对象作为参数，并且以**异步**的方式执行它。如下是一个使用 ExecutorService 执行 Runnable 的例子：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();  
  
executorService.execute(new Runnable() {  
    public void run() {  
        System.out.println("Asynchronous task");  
    }  
});  
      
executorService.shutdown();
```

​		使用这种方式**没有办法**获取执行 `Runnable` 之后的结果，如果你希望获取运行之后的返回值，就必须使用 接收 `Callable` 参数的 execute() 方法，后者将会在下文中提到。

---

#### **submit(Runnable)**

方法 `submit(Runnable)` 

​		同样接收一个 Runnable 的实现作为参数，但是会返回一个 `Future` 对象。这个 Future 对象可以用于判断 Runnable 是否结束执行。如下是一个 ExecutorService 的 submit() 方法的例子：

```java
Future future = executorService.submit(new Runnable() {  
    public void run() {  
        System.out.println("Asynchronous task");  
    }  
});  
//如果任务结束执行则返回 null  
System.out.println("future.get()=" + future.get());
```

---

#### **submit(Callable)**

方法 `submit(Callable)` 和方法 submit(Runnable) 比较类似，但是区别则在于它们接收不同的参数类型。Callable 的实例与 Runnable 的实例很类似，

> 但是 Callable 的 `call()` 方法可以返回一个结果。方法 `Runnable.run()` 则不能返回结果。
>
>  Callable 的返回值可以从方法 `submit(Callable)` 返回的 Future 对象中获取。

如下是一个 ExecutorService Callable 的样例：

```java
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Future future = executorService.submit(new Callable(){
            @Override
            public Object call() throws Exception {
                System.out.println("Asynchronous Callable");
                return "Callable Result";
            }
        });

        System.out.println("future.get() = " + future.get());
    }
```

上述样例代码会输出如下结果：

```java
Asynchronous Callable
future.get() = Callable Result
```



#### **inVokeAny()**

![image-20220915144757772](E:\Development\Typora\images\image-20220915144757772.png)

方法 `invokeAny()` 接收一个包含 `Callable 对象的集合`作为参数。**调用该方法不会返回 Future 对象，而是返回集合中某一个 Callable 对象的结果，**

> 而且无法保证调用之后返回的结果是哪一个 Callable，只知道它是这些 Callable 中一个执行结束的 Callable 对象。 **如果一个任务运行完毕或者抛出异常，方法会取消其它的 Callable 的执行。** ???我不理解

以下是一个样例：

```java
 public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Set<Callable<String>> callables = new HashSet<Callable<String>>();

        callables.add(new Callable<String>() {
            public String call() throws Exception {
                System.out.println("执行了Task 1");
                return "我是随机返回的 Task 1";
            }
        });
        callables.add(new Callable<String>() {
            public String call() throws Exception {
                System.out.println("执行了Task 2");
                return "我是随机返回的 Task 2";
            }
        });
        callables.add(new Callable<String>() {
            public String call() throws Exception {
                System.out.println("执行了Task 3");
                return "我是随机返回的 Task 3";
            }
        });

        String result = executorService.invokeAny(callables);

        System.out.println("result = " + result);

        executorService.shutdown();
    }
```

```java
//本次运行没有运行Task 2
执行了Task 1
执行了Task 3
result = 我是随机返回的 Task 1   
```

```
//本次运行了Task 2
执行了Task 1
执行了Task 3
执行了Task 2
result = 我是随机返回的 Task 1
```



以上样例代码会打印出在给定的集合中的某一个 Callable 的返回结果。我尝试运行了几次，结果都在改变。有时候返回结果是"Task 1"，有时候是"Task 2"，等等。

#### **invokeAll()**

![image-20220915145122405](E:\Development\Typora\images\image-20220915145122405.png)

方法 `invokeAll()` 会调用存在于参数集合中的所有 Callable 对象，并且返回一个包含 Future 对象的集合，你可以通过这个返回的集合来管理每个 Callable 的执行结果。 

> 需要注意的是，任务有可能因为异常而导致运行结束，所以它可能并不是真的成功运行了。但是我们没有办法通过 Future 对象来了解到这个差异。 以下是一个代码样例：



```java
//这里省略的代码同上
List<Future<String>> futures = executorService.invokeAll(callables);

for (Future<String> future : futures) {
    System.out.println(future.get());
}
executorService.shutdown();
```

```java
执行了Task 1
执行了Task 3
执行了Task 2
我是随机返回的 Task 1
我是随机返回的 Task 3
我是随机返回的 Task 2
```

---

### **ExecuteService 服务的关闭**

​		当使用 ExecutorService 完毕之后，我们应该关闭它，这样才能保证线程不会继续保持运行状态。

 	 举例来说，如果你的程序通过 `main()` 方法启动，并且主线程退出了你的程序，如果你还有一个活动的 ExecutorService 存在于你的程序中，那么程序将会继续保持运行状态。存在于 ExecutorService 中的活动线程会阻止JVM关闭。

​		 为了关闭在 ExecutorService 中的线程，你需要调用 `shutdown()` 方法。`ExecutorService` 并不会马上关闭，而是不再接收新的任务，一但所有的线程结束执行当前任务，ExecutorServie 才会真的关闭。所有在调用 `shutdown()` 方法之前提交到 ExecutorService 的任务都会执行。 如果你希望立即关闭 ExecutorService，你可以调用 `shutdownNow()` 方法。这个方法会尝试马上关闭所有正在执行的任务，并且跳过所有已经提交但是还没有运行的任务。但是对于正在执行的任务，是否能够成功关闭它是无法保证的，有可能他们真的被关闭掉了，也有可能它会一直执行到任务结束。这是一个最好的尝试。

要正确的关闭 ExecutorService，可以调用实例的 `shutdown()` 或 `shutdownNow()` 方法。

1. `shutdown()` 方法：

   ```java
   executorService.shutdown();
   ```

   `shutdown()` 方法并不会立即销毁 ExecutorService 实例，而是首先让 ExecutorService 停止接受新任务，并在所有正在运行的线程完成当前工作后关闭。

2. `shutdownNow()` 方法：

   ```java
   List<Runnable> notExecutedTasks = executorService.shutDownNow();
   ```

   `shutdownNow()` 方法会尝试立即销毁 ExecutorService 实例，所以并不能保证所有正在运行的线程将同时停止。该方法会返回等待处理的任务列表，由开发人员自行决定如何处理这些任务。

因为提供了两个方法，因此关闭 ExecutorService 实例的最佳实战 （ 也是 Oracle 所推荐的 ）就是同时使用这两种方法并结合 `awaitTermination()` 方法。

![image-20220915145526934](E:\Development\Typora\images\image-20220915145526934.png)

使用这种方式，`ExecutorService` 首先停止执行新任务，等待指定的时间段完成所有任务。如果该时间到期，则立即停止执行。

```java
executorService.shutdown();
try {
    if (!executorService.awaitTermination(800, TimeUnit.MILLISECONDS)) {
        executorService.shutdownNow();
    } 
} catch (InterruptedException e) {
    executorService.shutdownNow();
}
```



### ScheduledExecutorService 接口

`ScheduledExecutorService` 接口用于在一些预定义的延迟之后运行任务和（ 或 ）定期运行任务。

同样的，实例化 `ScheduledExecutorService` 的最佳方式是使用 Executors 类的工厂方法。

> Executors 类为很多类都提供了工厂方法，简直就是工厂方法的集大成者。

![image-20220915150654896](E:\Development\Typora\images\image-20220915150654896.png)

 ScheduledExecutorService 实例

```java
ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
```

有了实例之后，事情就好办了，比如，要在固定延迟后安排单个任务的执行，可以使用 ScheduledExecutorService 实例的 `scheduled()` 方法

```
Future<String> resultFuture = executorService.schedule(callableTask, 1, TimeUnit.SECONDS);
```

上面这个实例中的代码在执行 callableTask 之前延迟了一秒钟。

`scheduled()` 方法有两个重载，分别用于执行 Runnable 任务或 Callable 任务。

另外，ScheduledExecutorService 实例还提供了另一个重要方法 `scheduleAtFixedRate()` ，它允许在固定延迟后定期执行任务。

```java
Future<String> resultFuture = service.scheduleAtFixedRate(runnableTask, 100, 450, TimeUnit.MILLISECONDS);
```

上面的代码块将在 100 毫秒的初始延迟后执行任务，之后，它将每 450 毫秒执行相同的任务。

如果处理器需要更多时间来执行分配的任务，那么可以使用 `scheduleAtFixedRate()` 方法的 period 参数，ScheduledExecutorService 将等到当前任务完成后再开始下一个任务。

如果任务迭代之间必须具有固定长度的延迟，那么可以使用 `scheduleWithFixedDelay()` 方法 。例如，以下代码将保证当前执行结束与另一个执行结束之间的暂停时间为 150 毫秒。

```
service.scheduleWithFixedDelay(task, 100, 150, TimeUnit.MILLISECONDS);
```

根据 `scheduleAtFixedRate()` 和 `scheduleWithFixedDelay()` 方法契约，在任务执行期间，如果 ExecutorService 终止了或任务抛出了异常，那么任务将自动结束。



## Future 接口

`submit()` 方法和 `invokeAll()` 方法返回一个 Future 接口的对象或 Future 类型的对象集合。这些 Future 接口的对象允许我们获取任务执行的结果或检查任务的状态 ( 是正在运行还是执行完毕 ）。

### Future 接口 get() 方法

Future 接口提供了一个特殊的阻塞方法 `get()`，它返回 Callable 任务执行的实际结果，但如果是 Runnable 任务，则只会返回 null。

因为 `get()` 方法是阻塞的。如果调用 `get()` 方法时任务仍在运行，那么调用将会一直被执阻塞，直到任务正确执行完毕并且结果可用时才返回。

而且更重要的是，正在被执行的任务随时都可能抛出异常或中断执行。因此我们要将 `get()` 调用放在 `try catch` 语句块中，并捕捉 `InterruptedException` 或 `ExecutionException` 异常。

```java
Future<String> future = executorService.submit(callableTask);
String result = null;
try {
    result = future.get();
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
```

因为 `get()` 方法是阻塞的，而且并不知道要阻塞多长时间。因此可能导致应用程序的性能降低。如果结果数据并不重要，那么我们可以使用超时机制来避免长时间阻塞。

```java
String result = future.get(200, TimeUnit.MILLISECONDS);
```

这个 `get()` 的重载，第一个参数为超时的时间，第二个参数为时间的单位。上面的实例所表示就的就是等待 200 毫秒。

注意，这个 `get()` 重载方法，如果在超时时间内正常结束，那么返回的是 Future 类型的结果，如果超时了还没结束，那么将抛出 TimeoutException 异常。

除了 get() 方法之外，Future 还提供了其它很多方法，我们将几个重要的方法罗列在此

| 方法          | 说明                       |
| ------------- | -------------------------- |
| isDone()      | 检查已分配的任务是否已处理 |
| cancel()      | 取消任务执行               |
| isCancelled() | 检查任务是否已取消         |

这些方法的使用方式如下

```java
boolean isDone      = future.isDone();
boolean canceled    = future.cancel(true);
boolean isCancelled = future.isCancelled();
```



##  Fork/Join

Fork/Join 是 Java 7 提供的新框架，在 Java 7 发布之后，许多开发人员都作出了将 ExecutorService 框架替换为 fork/join 框架的决定。

但，这并不总是正确的决定。尽管 fork/join 使用起来更加简单且频繁使用时更带来更快的性能，但开发人员对并发执行的控制量也有所减少。

使用 ExecutorService ，开发人员能够控制生成的线程数以及应由不同线程执行的任务粒度。ExecutorService 的最佳用例是处理独立任务，例如根据 「 一个任务的一个线程 」 方案的事务或请求。

而相比之下，根据 [Oracle 文档](https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html)，fork/join 旨在简化和加速工作，可以将任务递归地分成更小的部分。

## 后记

尽管 ExecutorService 相对简单，但仍有一些常见的陷阱。我们罗列于此

1. 保持未使用的 `ExecutorService` 存活

   本文中对如何关闭 ExecutorService 已经做出了详细解释。

2. 使用固定长度的线程池时设置了错误的线程池容量

   使用 ExecutorService 最重要的一件事，就是确定应用程序有效执行任务所需的线程数

   - 太大的线程池只会产生不必要的开销，只会创建大多数处于等待模式的线程。
   - 太少的线程池会让应用程序看起来没有响应，因为队列中的任务等待时间很长。

3. 在取消任务后调用 Future 的 get() 方法

   尝试获取已取消任务的结果将触发 CancellationException 异常。

4. 使用 Future 的 get() 方法意外地阻塞了很长时间

   应该使用超时来避免意外的等待。
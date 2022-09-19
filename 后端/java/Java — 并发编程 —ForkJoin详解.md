# Java — 并发编程 —ForkJoin详解

# 概念

Fork/Join [框架](https://so.csdn.net/so/search?q=框架&spm=1001.2101.3001.7020)是 Java7 提供了的一个用于并行执行任务的框架， 是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。类似于Java 8中的parallel Stream。

> 【**只能将任务1个切分为两个，不能切分为3个或其他数量**】

# 简单使用

```java
public class ForkJoinExample {

    //针对一个数字，做计算。
    private static final Integer MAX = 200;

    static class CalcForJoinTask extends RecursiveTask<Integer> {
        private Integer startValue; //子任务的开始计算的值
        private Integer endValue; //子任务结束计算的值

        public CalcForJoinTask(Integer startValue, Integer endValue) {
            this.startValue = startValue;
            this.endValue = endValue;
        }

        @Override
        protected Integer compute() {
            //如果当前的数据区间已经小于MAX了，那么接下来的计算不需要做拆分
            if (endValue - startValue < MAX) {
                System.out.println("开始计算：startValue:" + startValue + " ; endValue:" + endValue);
                Integer totalValue = 0;
                for (int i = this.startValue; i <= this.endValue; i++) {
                    totalValue += i;
                }
                return totalValue;
            }
            //任务拆分，拆分成两个任务
            CalcForJoinTask subTask = new CalcForJoinTask(startValue, (startValue + endValue) / 2);
            subTask.fork();

            CalcForJoinTask calcForJoinTask = new CalcForJoinTask((startValue + endValue) / 2 + 1, endValue);
            calcForJoinTask.fork();

            return subTask.join() + calcForJoinTask.join();
        }
    }

    public static void main(String[] args) {
        CalcForJoinTask calcForJoinTask = new CalcForJoinTask(1, 1000);

        // 这是Fork/Join框架的线程池
        ForkJoinPool pool = new ForkJoinPool();
        ForkJoinTask<Integer> taskFuture = pool.submit(calcForJoinTask);
        try {
            Integer result = taskFuture.get();
            System.out.println("result:" + result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

}
```

运行结果：

```java
开始计算：startValue:501 ; endValue:625
开始计算：startValue:251 ; endValue:375
开始计算：startValue:126 ; endValue:250
开始计算：startValue:376 ; endValue:500
开始计算：startValue:876 ; endValue:1000
开始计算：startValue:1 ; endValue:125
开始计算：startValue:626 ; endValue:750
开始计算：startValue:751 ; endValue:875
result:500500
```

# 工作流程图

为了更清晰的了解fork/join的原理，我们通过一个图形来理解。整体思想其实就是拆分与合并。

图中最顶层的任务使用submit方式被提交到Fork/Join框架中，Fork/Join把这个任务放入到某个线程中运行，工作任务中的compute方法的代码开始对这个任务T1进行分析。如果当前任务需要累加的数字范围过大（代码中设定的是大于200），则将这个计算任务拆分成两个子任务（T1.1和T1.2），每个子任务各自负责计算一半的数据累加，请参见代码中的fork方法。如果当前子任务中需要累加的数字范围足够小（小于等于200），就进行累加然后返回到上层任务中。

![image-20211113181046263](E:\Development\Typora\images\4ac663fa4b8e9878b02d56685d84c78b.png)

# 代码分析

简单给大家解释一下Fork/Join的相关api，在刚刚的案例中，涉及到几个重要的API， ForkJoinTask ，ForkJoinPool .

## ForkJoinTask

ForkJoinTask : 基本任务，使用fork、join框架必须创建的对象，提供fork,join操作，常用的三个子类如下：

- RecursiveAction

  无结果返回的任务

- RecursiveTask

  有返回结果的任务

- CountedCompleter

  无返回值任务，完成任务后可以触发回调

ForkJoinTask提供了两个重要的方法：

- fork

  让task异步执行

- join

  让task同步执行，可以获取返回值

## ForkJoinPool

ForkJoinPool ： 专门用来运行 ForkJoinTask 的线程池，（在实际使用中，也可以接收Runnable/Callable 任务，但在真正运行时，也会把这些任务封装成 ForkJoinTask 类型的任务）。

他是 fork/join 框架的核心，是 ExecutorService 的一个实现，用于管理工作线程，并提供了一些工具来帮助获取有关线程池状态和性能的信息。

工作线程一次只能执行一个任务。

`ForkJoinPool` 线程池并不会为每个子任务创建一个单独的线程，相反，池中的每个线程都有自己的双端队列用于存储任务 （ double-ended queue ）。

这种架构使用了一种名为工作窃取（ work-stealing ）算法来平衡线程的工作负载。

### 工作窃取（ work-stealing ）算法

要怎么解释 「 工作窃取算法 」 呢 ？

简单来说，就是空闲的线程试图从繁忙线程的 deques 中 窃取工作。

默认情况下，每个工作线程从其自己的双端队列中获取任务。但如果自己的双端队列中的任务已经执行完毕，双端队列为空时，工作线程就会从另一个忙线程的双端队列尾部或全局入口队列中获取任务，因为这是最大概率可能找到工作的地方。

这种方法最大限度地减少了线程竞争任务的可能性。它还减少了工作线程寻找任务的次数，因为它首先在最大可用的工作块上工作。

### 执行方法

| 方法名                | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| invoke(ForkJoinTask)  | 提交任务并一直阻塞直到任务执行完成返回合并结果。             |
| execute(ForkJoinTask) | 异步执行任务，无返回值。                                     |
| submit(ForkJoinTask)  | 异步执行任务，返回task本身，可以通过task.get()方法获取合并之后的结果。 |

ForkJoinTask 在不显示使用 ForkJoinPool.execute/invoke/submit() 方法进行执行的情况下，也可以使用自己的fork/invoke方法进行执行。

# 源码分析

## ForkJoinPool

```java
ForkJoinPool forkJoinPool = new ForkJoinPool();

//Runtime.getRuntime().availableProcessors()当前操作系统可以使用的CPU内核数量
public ForkJoinPool() {
    this(Math.min(MAX_CAP, Runtime.getRuntime().availableProcessors()),
         defaultForkJoinWorkerThreadFactory, null, false);
}

//this调用到下面这段代码
public ForkJoinPool(int parallelism,
                    ForkJoinWorkerThreadFactory factory,
                    UncaughtExceptionHandler handler,
                    boolean asyncMode) {
    this(checkParallelism(parallelism), //并行度
         checkFactory(factory), //工作线程创建工厂
         handler, //异常处理handler
         asyncMode ? FIFO_QUEUE : LIFO_QUEUE, //任务队列出队模式 异步：先进先出，同步：后进先出
         "ForkJoinPool-" + nextPoolId() + "-worker-");
    checkPermission();
}
//上面的this最终调用到下面这段代码
private ForkJoinPool(int parallelism,
                     ForkJoinWorkerThreadFactory factory,
                     UncaughtExceptionHandler handler,
                     int mode,
                     String workerNamePrefix) {
    this.workerNamePrefix = workerNamePrefix;
    this.factory = factory;
    this.ueh = handler;
    this.config = (parallelism & SMASK) | mode;
    long np = (long)(-parallelism); // offset ctl counts
    this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
}
123456789101112131415161718192021222324252627282930313233
```

- **parallelism**：可并行数量，fork/join框架将依据这个并行数量的设定，决定框架内并行执行的线程数量。并行的每一个任务都会有一个线程进行处理；
- **factory**：当fork/join创建一个新的线程时，同样会用到线程创建工厂。它实现了ForkJoinWorkerThreadFactory接口，使用默认的的接口实现类DefaultForkJoinWorkerThreadFactory来实现newThread方法创建一个新的工作线程；
- **handler**：异常捕获处理器。当执行的任务出现异常，并从任务中被抛出时，就会被handler捕获；
- **asyncMode**：fork/join为每一个独立的工作线程准备了对应的待执行任务队列，这个任务队列是使用数组进行组合的双向队列。即可以使用先进先出的工作模式，也可以使用后进先出的工作模式；

![image-20211113191234959](E:\Development\Typora\images\e856caf3c05fab81320bc617f17fe57b.png)

## Submit

提交：ForkJoinPool.submit(ForkJoinTask task) -> externalPush(task) -> externalSubmit(task)

```java
public <T> ForkJoinTask<T> submit(ForkJoinTask<T> task) {
    if (task == null)
        throw new NullPointerException();
    externalPush(task);
    return task;
}

//将任务添加到随机选取的队列中或新创建的队列中；
final void externalPush(ForkJoinTask<?> task) {
    WorkQueue[] ws; WorkQueue q; int m;
    int r = ThreadLocalRandom.getProbe();//当前线程的一个随机数
    int rs = runState;//当前容器的状态
    //如果随机选取的队列还有空位置可以存放、队列加锁锁定成功，任务就放入队列中
    if ((ws = workQueues) != null && (m = (ws.length - 1)) >= 0 &&
        (q = ws[m & r & SQMASK]) != null && r != 0 && rs > 0 &&
        U.compareAndSwapInt(q, QLOCK, 0, 1)) {
        ForkJoinTask<?>[] a; int am, n, s;
        if ((a = q.array) != null &&
            (am = a.length - 1) > (n = (s = q.top) - q.base)) {
            int j = ((am & s) << ASHIFT) + ABASE;
            U.putOrderedObject(a, j, task);//任务加入队列中
            U.putOrderedInt(q, QTOP, s + 1);//挪动下次任务存放的槽的位置
            U.putIntVolatile(q, QLOCK, 0);//队列解锁
            if (n <= 1)//当前数组元素少时，进行唤醒当前线程；或者当没有活动线程或线程数较少时，添加新的线程
                signalWork(ws, q);
            return;
        }
        U.compareAndSwapInt(q, QLOCK, 1, 0);//队列解锁
    }
    externalSubmit(task);//升级版的externalPush
}


volatile int runState;               // lockable status锁定状态
// runState: SHUTDOWN为负数，其他的为2的次幂
private static final int  RSLOCK     = 1;
private static final int  RSIGNAL    = 1 << 1;//唤醒
private static final int  STARTED    = 1 << 2;//启动
private static final int  STOP       = 1 << 29;//停止
private static final int  TERMINATED = 1 << 30;//结束
private static final int  SHUTDOWN   = 1 << 31;//关闭


//队列添加任务失败，进行升级版操作，即创建队列数组和创建队列后，将任务放入新创建的队列中；
private void externalSubmit(ForkJoinTask<?> task) {
    int r;                                    // initialize caller's probe
    if ((r = ThreadLocalRandom.getProbe()) == 0) {
        ThreadLocalRandom.localInit();
        r = ThreadLocalRandom.getProbe();
    }
    for (;;) {//自旋
        WorkQueue[] ws; WorkQueue q; int rs, m, k;
        boolean move = false;
        /**
        *ForkJoinPool执行器停止工作了，抛出异常
        *ForkJoinPool extends AbstractExecutorService
        *abstract class AbstractExecutorService implements ExecutorService
        *interface ExecutorService extends Executor
        *interface Executor执行提交的对象Runnable任务
        */
        if ((rs = runState) < 0) {
            tryTerminate(false, false);    // help terminate
            throw new RejectedExecutionException();
        }
        //第一次遍历，队列数组未创建，进行创建
        else if ((rs & STARTED) == 0 ||     // initialize初始化
                 ((ws = workQueues) == null || (m = ws.length - 1) < 0)) {
            int ns = 0;
            rs = lockRunState();
            try {
                if ((rs & STARTED) == 0) {
                    U.compareAndSwapObject(this, STEALCOUNTER, null,
                                           new AtomicLong());
                    // create workQueues array with size a power of two
                    int p = config & SMASK; // ensure at least 2 slots，config是CPU核数
                    int n = (p > 1) ? p - 1 : 1;
                    n |= n >>> 1; n |= n >>> 2;  n |= n >>> 4;
                    n |= n >>> 8; n |= n >>> 16; n = (n + 1) << 1;
                    workQueues = new WorkQueue[n];//创建
                    ns = STARTED;
                }
            } finally {
                unlockRunState(rs, (rs & ~RSLOCK) | ns);
            }
        }
        //第三次遍历，把任务放入队列中
        else if ((q = ws[k = r & m & SQMASK]) != null) {
            if (q.qlock == 0 && U.compareAndSwapInt(q, QLOCK, 0, 1)) {
                ForkJoinTask<?>[] a = q.array;
                int s = q.top;
                boolean submitted = false; // initial submission or resizing
                try {                      // locked version of push
                    if ((a != null && a.length > s + 1 - q.base) ||
                        (a = q.growArray()) != null) {
                        int j = (((a.length - 1) & s) << ASHIFT) + ABASE;
                        U.putOrderedObject(a, j, task);
                        U.putOrderedInt(q, QTOP, s + 1);
                        submitted = true;
                    }
                } finally {
                    U.compareAndSwapInt(q, QLOCK, 1, 0);
                }
                if (submitted) {
                    signalWork(ws, q);
                    return;
                }
            }
            move = true;                   // move on failure
        }
        //第二次遍历，队列数组为空，创建队列
        else if (((rs = runState) & RSLOCK) == 0) { // create new queue
            q = new WorkQueue(this, null);
            q.hint = r;
            q.config = k | SHARED_QUEUE;
            q.scanState = INACTIVE;
            rs = lockRunState();           // publish index
            if (rs > 0 &&  (ws = workQueues) != null &&
                k < ws.length && ws[k] == null)
                ws[k] = q;                 // else terminated
            unlockRunState(rs, rs & ~RSLOCK);
        }
        else
            move = true;                   // move if busy
        if (move)
            r = ThreadLocalRandom.advanceProbe(r);
    }
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445464748495051525354555657585960616263646566676869707172737475767778798081828384858687888990919293949596979899100101102103104105106107108109110111112113114115116117118119120121122123124125126127
```

## Fork

fork()用于将新创建的子任务放入当前线程的workQueue队列中，fork/join框架将根据当前正在[并发](https://so.csdn.net/so/search?q=并发&spm=1001.2101.3001.7020)执行ForkJoinTask任务的ForkJoinWorkerThread线程状态，决定是让这个任务在队列中等待，还是创建一个新的ForkJoinWorkedThread线程运行它，又或者是唤起其他正在等待任务的ForkJoinWorkerThread线程运行它。

流程：ForkJoinTask.fork() -> ForkJoinWorkerThread.workQueue.push(task)/ForkJoinPool.common.externalPush(task) -> ForkJoinPool.push(task)/externalPush(task)；

```java
public final ForkJoinTask<V> fork() {
    Thread t;
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)//当前线程是workerThread，任务直接放入workerThread当前的workQueue
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
        ForkJoinPool.common.externalPush(this);//将任务添加到随机选取的队列中或新创建的队列中
    return this;
}

public class ForkJoinPool extends AbstractExecutorService {
    static final class WorkQueue {
        final void push(ForkJoinTask<?> task) {
            ForkJoinTask<?>[] a; ForkJoinPool p;
            int b = base, s = top, n;
            if ((a = array) != null) {    // ignore if queue removed，队列被移除忽略
                int m = a.length - 1;     // fenced write for task visibility
                U.putOrderedObject(a, ((m & s) << ASHIFT) + ABASE, task);//任务加入队列中
                U.putOrderedInt(this, QTOP, s + 1);//挪动下次任务存放的槽的位置
                if ((n = s - b) <= 1) {//当前数组元素少时，进行唤醒当前线程；或者当没有活动线程或线程数较少时，添加新的线程
                    if ((p = pool) != null)
                        p.signalWork(p.workQueues, this);
                }
                else if (n >= m)//数组所有元素都满了进行2倍扩容
                    growArray();
            }
        }
        final ForkJoinTask<?>[] growArray() {
            ForkJoinTask<?>[] oldA = array;
            int size = oldA != null ? oldA.length << 1 : INITIAL_QUEUE_CAPACITY;//2倍扩容或初始化
            if (size > MAXIMUM_QUEUE_CAPACITY)
                throw new RejectedExecutionException("Queue capacity exceeded");
            int oldMask, t, b;
            ForkJoinTask<?>[] a = array = new ForkJoinTask<?>[size];
            if (oldA != null && (oldMask = oldA.length - 1) >= 0 &&
                (t = top) - (b = base) > 0) {
                int mask = size - 1;
                do { // emulate poll from old array, push to new array遍历从旧数组中取出放到新数组中
                    ForkJoinTask<?> x;
                    int oldj = ((b & oldMask) << ASHIFT) + ABASE;
                    int j    = ((b &    mask) << ASHIFT) + ABASE;
                    x = (ForkJoinTask<?>)U.getObjectVolatile(oldA, oldj);//从旧数组中取出
                    if (x != null &&
                        U.compareAndSwapObject(oldA, oldj, x, null))//将旧数组取出的位置的对象置为null
                        U.putObjectVolatile(a, j, x);//放入新数组
                } while (++b != t);
            }
            return a;
        }
    }
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950
```

## doExec 执行

任务的消费的执行链路是ForkJoinTask.doExec() -> RecursiveTask.exec()/RecursiveAction.exec() -> 覆盖重写的compute()。

```java
final int doExec() {
    int s; boolean completed;
    if ((s = status) >= 0) {
        try {
            completed = exec();//消费任务
        } catch (Throwable rex) {
            return setExceptionalCompletion(rex);
        }
        if (completed)
            s = setCompletion(NORMAL);//任务执行完设置状态为NORMAL，并唤醒其他等待任务
    }
    return s;
}
protected abstract boolean exec();
private int setCompletion(int completion) {
    for (int s;;) {
        if ((s = status) < 0)
            return s;
        if (U.compareAndSwapInt(this, STATUS, s, s | completion)) {//任务状态修改为NORMAL
            if ((s >>> 16) != 0)//状态不是SMASK
                synchronized (this) { notifyAll(); }//唤醒其他等待任务
            return completion;
        }
    }
}
/** The run status of this task 任务的运行状态*/
volatile int status; // accessed directly by pool and workers由ForkJoinPool池或ForkJoinWorkerThread控制
static final int DONE_MASK   = 0xf0000000;  // mask out non-completion bits
static final int NORMAL      = 0xf0000000;  // must be negative
static final int CANCELLED   = 0xc0000000;  // must be < NORMAL
static final int EXCEPTIONAL = 0x80000000;  // must be < CANCELLED
static final int SIGNAL      = 0x00010000;  // must be >= 1 << 16
static final int SMASK       = 0x0000ffff;  // short bits for tags
123456789101112131415161718192021222324252627282930313233
```

## 处理逻辑

任务提交到ForkJoinPool，最终真正的是由继承Thread的ForkJoinWorkerThread的run方法来执行消费任务的，ForkJoinWorkerThread处理哪个任务是由join来出队的；

### Join

join()用于让当前线程阻塞，直到对应的子任务完成运行并返回执行结果。或者，如果这个子任务存在于当前线程的任务等待队列workQueue中，则取出这个子任务进行”递归“执行，其目的是尽快得到当前子任务的运行结果，然后继续执行。

```java
public final V join() {
    int s;
    if ((s = doJoin() & DONE_MASK) != NORMAL)
        reportException(s);
    return getRawResult();//得到返回结果
}
private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
    /**
         * (s = status) < 0 判断任务是否已经完成，完成直接返回s
         * 任务未完成：
         *          1）线程是ForkJoinWorkerThread，tryUnpush任务出队然后消费任务doExec
         *              1.1）出队或消费失败，执行awaitJoin进行自旋，如果任务状态是完成就退出，否则继续尝试出队，直到任务完成或超时为止；
         *          2）如果线程不是ForkJoinWorkerThread，执行externalAwaitDone进行出队消费
         */
    return (s = status) < 0 ? s :
    ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
        tryUnpush(this) && (s = doExec()) < 0 ? s :
    wt.pool.awaitJoin(w, this, 0L) :
    externalAwaitDone();
}
private void reportException(int s) {
    if (s == CANCELLED)//取消
        throw new CancellationException();
    if (s == EXCEPTIONAL)//异常
        rethrow(getThrowableException());
}
12345678910111213141516171819202122232425262728
public class ForkJoinPool{
    final int awaitJoin(WorkQueue w, ForkJoinTask<?> task, long deadline) {
        int s = 0;
        if (task != null && w != null) {
            ForkJoinTask<?> prevJoin = w.currentJoin;
            U.putOrderedObject(w, QCURRENTJOIN, task);
            CountedCompleter<?> cc = (task instanceof CountedCompleter) ?
                (CountedCompleter<?>)task : null;
            for (;;) {
                if ((s = task.status) < 0)//任务完成退出
                    break;
                if (cc != null)//当前任务即将完成，检查是否还有其他的等待任务，如果有
                    //运行当前队列的其他任务，若当前的队列中没有任务了，则窃取其他队列的任务并运行
                    helpComplete(w, cc, 0);
                //当前队列没有任务了，或遍历当前队列有没有任务，如果有且在top端取出来运行，或在队列中间使用EmptyTask替代原位置取出来运行，如果没有，执行helpStealer
                else if (w.base == w.top || w.tryRemoveAndExec(task))
                    helpStealer(w, task);//窃取其他队列的任务
                if ((s = task.status) < 0)
                    break;
                long ms, ns;
                if (deadline == 0L)
                    ms = 0L;
                else if ((ns = deadline - System.nanoTime()) <= 0L)//超时退出
                    break;
                else if ((ms = TimeUnit.NANOSECONDS.toMillis(ns)) <= 0L)
                    ms = 1L;
                if (tryCompensate(w)) {//当前队列阻塞了
                    task.internalWait(ms);//进行等待
                    U.getAndAddLong(this, CTL, AC_UNIT);
                }
            }
            U.putOrderedObject(w, QCURRENTJOIN, prevJoin);
        }
        return s;
    }
}
123456789101112131415161718192021222324252627282930313233343536
private int externalAwaitDone() {
    /**
        *   当前任务是CountedCompleter
        *   1）是则执行ForkJoinPool.common.externalHelpComplete()
        *   2）否则执行ForkJoinPool.common.tryExternalUnpush(this)进行任务出队
        *       2.1）出队成功，进行doExec()消费，否则进行阻塞等待
        */
    int s = ((this instanceof CountedCompleter) ? // try helping
             ForkJoinPool.common.externalHelpComplete(
                 (CountedCompleter<?>)this, 0) :
             ForkJoinPool.common.tryExternalUnpush(this) ? doExec() : 0);
    if (s >= 0 && (s = status) >= 0) {//任务未完成
        boolean interrupted = false;
        do {
            if (U.compareAndSwapInt(this, STATUS, s, s | SIGNAL)) {//任务状态标记为SIGNAL
                synchronized (this) {
                    if (status >= 0) {
                        try {
                            wait(0L);//阻塞等待
                        } catch (InterruptedException ie) {//有中断异常
                            interrupted = true;//设置中断标识为true
                        }
                    }
                    else
                        notifyAll();//任务完成唤醒其他任务
                }
            }
        } while ((s = status) >= 0);
        if (interrupted)
            Thread.currentThread().interrupt();//当前线程进行中断
    }
    return s;
}
final int externalHelpComplete(CountedCompleter<?> task, int maxTasks) {
    WorkQueue[] ws; int n;
    int r = ThreadLocalRandom.getProbe();
    //没有任务直接结束，有任务则执行helpComplete
    //helpComplete：运行随机选取的队列的任务，若选取的队列中没有任务了，则窃取其他队列的任务并运行
    return ((ws = workQueues) == null || (n = ws.length) == 0) ? 0 :
    helpComplete(ws[(n - 1) & r & SQMASK], task, maxTasks);
}
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### run和工作窃取

任务是由workThread来窃取的，workThread是一个线程。线程的所有逻辑都是由run()方法执行：

```java
public class ForkJoinWorkerThread extends Thread {
    public void run() {
        if (workQueue.array == null) { // only run once
            Throwable exception = null;
            try {
                onStart();//初始化状态
                pool.runWorker(workQueue);//处理任务队列
            } catch (Throwable ex) {
                exception = ex;//记录异常
            } finally {
                try {
                    onTermination(exception);
                } catch (Throwable ex) {
                    if (exception == null)
                        exception = ex;
                } finally {
                    pool.deregisterWorker(this, exception);//注销工作线程
                }
            }
        }
    }
}
public class ForkJoinPool{
    final void runWorker(WorkQueue w) {
        w.growArray();                   // allocate queue，队列初始化
        int seed = w.hint;               // initially holds randomization hint
        int r = (seed == 0) ? 1 : seed;  // avoid 0 for xorShift
        for (ForkJoinTask<?> t;;) {//自旋
            if ((t = scan(w, r)) != null)//从队列中窃取任务成功，scan()进行任务窃取
                w.runTask(t);//执行任务，内部方法调用了doExec()进行任务的消费
            else if (!awaitWork(w, r))//队列没有任务了则结束
                break;
            r ^= r << 13; r ^= r >>> 17; r ^= r << 5; // xorshift
        }
    }
}
123456789101112131415161718192021222324252627282930313233343536
private ForkJoinTask<?> scan(WorkQueue w, int r) {
    WorkQueue[] ws; int m;
    if ((ws = workQueues) != null && (m = ws.length - 1) > 0 && w != null) {
        int ss = w.scanState;                     // initially non-negative
        for (int origin = r & m, k = origin, oldSum = 0, checkSum = 0;;) {
            WorkQueue q; ForkJoinTask<?>[] a; ForkJoinTask<?> t;
            int b, n; long c;
            if ((q = ws[k]) != null) {   //随机选中了非空队列 q
                if ((n = (b = q.base) - q.top) < 0 &&
                    (a = q.array) != null) {      // non-empty
                    long i = (((a.length - 1) & b) << ASHIFT) + ABASE;  //从尾部出队,b是尾部下标
                    if ((t = ((ForkJoinTask<?>)
                              U.getObjectVolatile(a, i))) != null &&
                        q.base == b) {
                        if (ss >= 0) {
                            if (U.compareAndSwapObject(a, i, t, null)) { //利用cas出队
                                q.base = b + 1;
                                if (n < -1)       // signal others
                                    signalWork(ws, q);
                                return t;  //出队成功，成功窃取一个任务！
                            }
                        }
                        else if (oldSum == 0 &&   // try to activate 队列没有激活，尝试激活
                                 w.scanState < 0)
                            tryRelease(c = ctl, ws[m & (int)c], AC_UNIT);
                    }
                    if (ss < 0)                   // refresh
                        ss = w.scanState;
                    r ^= r << 1; r ^= r >>> 3; r ^= r << 10;
                    origin = k = r & m;           // move and rescan
                    oldSum = checkSum = 0;
                    continue;
                }
                checkSum += b;
            }　　     //k = k + 1表示取下一个队列 如果（k + 1） & m == origin表示已经遍历完所有队列了
            if ((k = (k + 1) & m) == origin) {    // continue until stable 
                if ((ss >= 0 || (ss == (ss = w.scanState))) &&
                    oldSum == (oldSum = checkSum)) {
                    if (ss < 0 || w.qlock < 0)    // already inactive
                        break;
                    int ns = ss | INACTIVE;       // try to inactivate
                    long nc = ((SP_MASK & ns) |
                               (UC_MASK & ((c = ctl) - AC_UNIT)));
                    w.stackPred = (int)c;         // hold prev stack top
                    U.putInt(w, QSCANSTATE, ns);
                    if (U.compareAndSwapLong(this, CTL, c, nc))
                        ss = ns;
                    else
                        w.scanState = ss;         // back out
                }
                checkSum = 0;
            }
        }
    }
    return null;
}
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556
volatile int scanState;    // versioned, <0: inactive; odd:scanning，版本标记，小于0暂停，奇数进行扫描其他任务
static final int SCANNING     = 1;             // false when running tasks，有任务执行是false
/**
         * Executes the given task and any remaining local tasks.
         * 执行给定的任务和任何剩余的本地任务
         */
final void runTask(ForkJoinTask<?> task) {
    if (task != null) {
        scanState &= ~SCANNING; // mark as busy，暂停扫描，当前有任务执行
        (currentSteal = task).doExec();//执行窃取的任务
        U.putOrderedObject(this, QCURRENTSTEAL, null); // release for GC，窃取的任务执行完置为null
        execLocalTasks();//执行本地的任务，即自己workQueue的任务，调用doExec执行到workQueue空为止
        ForkJoinWorkerThread thread = owner;
        if (++nsteals < 0)      // collect on overflow，窃取计数溢出
            transferStealCount(pool);//重置窃取计数
        scanState |= SCANNING;//继续扫描队列
        if (thread != null)
            thread.afterTopLevelExec();
    }
}
static final class InnocuousForkJoinWorkerThread extends ForkJoinWorkerThread {
    @Override // to erase ThreadLocals，清除threadLocals
    void afterTopLevelExec() {
        eraseThreadLocals();
    }
    /**
                 * Erases ThreadLocals by nulling out Thread maps.
                 */
    final void eraseThreadLocals() {
        U.putObject(this, THREADLOCALS, null);//threadLocals置为null
        U.putObject(this, INHERITABLETHREADLOCALS, null);//inheritablethreadlocals置为null
    }
}
123456789101112131415161718192021222324252627282930313233
```

# 总结

对于fork/join来说，在使用时还是存在下面的一些问题的：

- 在使用JVM的时候我们要考虑OOM的问题，如果我们的任务处理时间非常耗时，并且处理的数据非常大的时候，会造成OOM；
- ForkJoin是通过多线程的方式进行处理任务，那么我们不得不考虑是否应该使用ForkJoin。因为当数据量不是特别大的时候，我们没有必要使用ForkJoin。因为多线程会涉及到上下文的切换，所以数据量不大的时候使用串行比使用多线程快；
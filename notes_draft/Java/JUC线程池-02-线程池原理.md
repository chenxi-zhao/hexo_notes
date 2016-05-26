### ThreadPoolExecutor

ThreadPoolExecutor是线程池类。对于线程池，可以通俗的将它理解为"存放一定数量线程的一个线程集合。线程池允许若个线程同时允许，允许同时运行的线程数量就是线程池的容量；当添加的到线程池中的线程超过它的容量时，会有一部分线程阻塞等待。线程池会通过相应的调度策略和拒绝策略，对添加到线程池中的线程进行管理.

#### ThreadPoolExecutor数据结构
![](http://images.cnitblog.com/blog/497634/201401/07233232-2d5fb63cdb064112b90c2d0ec12b60a4.jpg)

1. workers
workers是HashSet<Work>类型，即它是一个Worker集合。而一个Worker对应一个线程，也就是说线程池通过workers包含了"一个线程集合"。当Worker对应的线程池启动时，它会执行线程池中的任务；当执行完一个任务后，它会从线程池的阻塞队列中取出一个阻塞的任务来继续运行。
wokers的作用是，线程池通过它实现了"允许多个线程同时运行"。

2. workQueue
workQueue是BlockingQueue类型，即它是一个阻塞队列。当线程池中的线程数超过它的容量的时候，线程会进入阻塞队列进行阻塞等待。
通过workQueue，线程池实现了阻塞功能。

3. mainLock
mainLock是互斥锁，通过mainLock实现了对线程池的互斥访问。

4. corePoolSize和maximumPoolSize
corePoolSize是"核心池大小"，maximumPoolSize是"最大池大小"。它们的作用是调整"线程池中实际运行的线程的数量"。
例如，当新任务提交给线程池时(通过execute方法)。
    -- 如果此时，线程池中运行的线程数量< corePoolSize，则创建新线程来处理请求。
    -- 如果此时，线程池中运行的线程数量> corePoolSize，但是却< mumPoolSize；则仅当阻塞队列满时才创建新线程。
如果设置的 corePoolSize 和 maximumPoolSize 相同，则创建了固定大小的线程池。如果将 maximumPoolSize 设置为基本的无界值（如 Integer.MAX_VALUE），则允许池适应任意数量的并发任务。在大多数情况下，核心池大小和最大池大小的值是在创建线程池设置的；但是，也可以使用 setCorePoolSize(int) 和 setMaximumPoolSize(int) 进行动态更改。

5. poolSize
poolSize是当前线程池的实际大小，即线程池中任务的数量。

6. allowCoreThreadTimeOut和keepAliveTime
allowCoreThreadTimeOut表示是否允许"线程在空闲状态时，仍然能够存活"；而keepAliveTime是当线程池处于空闲状态的时候，超过keepAliveTime时间之后，空闲的线程会被终止。

7. threadFactory
threadFactory是ThreadFactory对象。它是一个线程工厂类，"线程池通过ThreadFactory创建线程"。

8. handler
handler是RejectedExecutionHandler类型。它是"线程池拒绝策略"的句柄，也就是说"当某任务添加到线程池中，而线程池拒绝该任务时，线程池会通过handler进行相应的处理"。

#### 总结
综上所说，线程池通过workers来管理"线程集合"，每个线程在启动后，会执行线程池中的任务；当一个任务执行完后，它会从线程池的阻塞队列中取出任务来继续运行。阻塞队列是管理线程池任务的队列，当添加到线程池中的任务超过线程池的容量时，该任务就会进入阻塞队列进行等待。


### 线程池原理分析
在分析之前我们先看一个示例
```java
ExecutorService pool = Executors.newFixedThreadPool(2);
// 创建实现了Runnable接口对象，Thread对象当然也实现了Runnable接口
Thread ta = new MyThread();
Thread tb = new MyThread();
Thread tc = new MyThread();
Thread td = new MyThread();
Thread te = new MyThread();
// 将线程放入池中进行执行
pool.execute(ta);
pool.execute(tb);
pool.execute(tc);
pool.execute(td);
pool.execute(te);
// 关闭线程池
pool.shutdown();
```
#### 创建线程池
##### newFixedThreadPool()
newFixedThreadPool()在Executors中，源码如下：
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
LinkedBlockingQueue是一个单向阻塞队列，用于存储被阻塞的线程
##### ThreadPoolExecutor构造函数
```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue) {
        this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
             Executors.defaultThreadFactory(), defaultHandler);
    }
```
**ThreadFactory**的作用就是提供创建线程的功能的线程工厂，它是通过newThread()提供创建线程功能的。
newThread()创建的线程对应的任务是Runnable对象，它创建的线程都是“非守护线程”而且“线程优先级都是Thread.NORM_PRIORITY”。
<br />
**handler**是ThreadPoolExecutor中拒绝策略的处理句柄。所谓拒绝策略，是指将任务添加到线程池中时，线程池拒绝该任务所采取的相应策略。
线程池默认会采用的是defaultHandler策略，即AbortPolicy策略。在AbortPolicy策略中，线程池拒绝任务时会抛出异常！


#### 添加任务到“线程池”
##### execute()
execute()定义在ThreadPoolExecutor中，源码如下：
```java
public void execute(Runnable command) {
    // 如果任务为null，则抛出异常。
    if (command == null)
        throw new NullPointerException();
    // 获取ctl对应的int值。该int值保存了"线程池中任务的数量"和"线程池状态"信息
    int c = ctl.get();
    // 当线程池中的任务数量 < "核心池大小"时，即线程池中少于corePoolSize个任务。
    // 则通过addWorker(command, true)新建一个线程，并将任务(command)添加到该线程中；然后，启动该线程从而执行任务。
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 当线程池中的任务数量 >= "核心池大小"时，
    // 而且，"线程池处于允许状态"时，则尝试将任务添加到阻塞队列中。
    if (isRunning(c) && workQueue.offer(command)) {
        // 再次确认“线程池状态”，若线程池异常终止了，则删除任务；然后通过reject()执行相应的拒绝策略的内容。
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 否则，如果"线程池中任务数量"为0，则通过addWorker(null, false)尝试新建一个线程，新建线程对应的任务为null。
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 通过addWorker(command, false)新建一个线程，并将任务(command)添加到该线程中；然后，启动该线程从而执行任务。
    // 如果addWorker(command, false)执行失败，则通过reject()执行相应的拒绝策略的内容。
    else if (!addWorker(command, false))
        reject(command);
}
```
execute()的作用是将任务添加到线程池中执行。它会分为3种情况进行处理：
>情况1 -- 如果"线程池中任务数量" < "核心池大小"时，即线程池中少于corePoolSize个任务；此时就新建一个线程，并将该任务添加到线程中进行执行。
>情况2 -- 如果"线程池中任务数量" >= "核心池大小"，并且"线程池是允许状态"；此时，则将任务添加到阻塞队列中阻塞等待。在该情况下，会再次确认"线程池的状态"，如果"第2次读到的线程池状态"和"第1次读到的线程池状态"不同，则从阻塞队列中删除该任务。
>情况3 -- 非以上两种情况。在这种情况下，尝试新建一个线程，并将该任务添加到线程中进行执行。如果执行失败，则通过reject()拒绝该任务。

##### addWorker()
addWorker(Runnable firstTask, boolean core) 的作用是将任务(firstTask)添加到线程池中，并启动该任务。

1. core为true的话，则以corePoolSize为界限，若"线程池中已有任务数量>=corePoolSize"，则返回false；core为false的话，则以maximumPoolSize为界限，若"线程池中已有任务数量>=maximumPoolSize"，则返回false。

2. addWorker()会先通过for循环不断尝试更新ctl状态，ctl记录了"线程池中任务数量和线程池状态"。
更新成功之后，再通过try模块来将任务添加到线程池中，并启动任务所在的线程。

3. 从addWorker()中，我们能清晰的发现：
线程池在添加任务时，会创建任务对应的Worker对象；而一个Workder对象包含一个Thread对象。
- 通过将Worker对象添加到"线程的workers集合"中，从而实现将任务添加到线程池中。
- 通过启动Worker对应的Thread线程，则执行该任务。

#####  submit()
submit()实际上也是通过调用execute()实现的，源码如下：
```java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}
```

#### 关闭“线程池”
```java
public void shutdown() {
    final ReentrantLock mainLock = this.mainLock;
    // 获取锁
    mainLock.lock();
    try {
        // 检查终止线程池的“线程”是否有权限。
        checkShutdownAccess();
        // 设置线程池的状态为关闭状态。
        advanceRunState(SHUTDOWN);
        // 中断线程池中空闲的线程。
        interruptIdleWorkers();
        // 钩子函数，在ThreadPoolExecutor中没有任何动作。
        onShutdown(); // hook for ScheduledThreadPoolExecutor
    } finally {
        // 释放锁
        mainLock.unlock();
    }
    // 尝试终止线程池
    tryTerminate();
}
```

### 拒绝策略
线程池的拒绝策略，是指当任务添加到线程池中被拒绝，而采取的处理措施。
当任务添加到线程池中之所以被拒绝，可能是由于：第一，线程池异常关闭。第二，任务数量超过线程池的最大限制。

线程池共包括4种拒绝策略，它们分别是：AbortPolicy, CallerRunsPolicy, DiscardOldestPolicy和DiscardPolicy。

- AbortPolicy,当任务添加到线程池中被拒绝时，它将抛出 RejectedExecutionException 异常。
- CallerRunsPolicy,当任务添加到线程池中被拒绝时，会在线程池当前正在运行的Thread线程池中处理被拒绝的任务。
- DiscardOldestPolicy,当任务添加到线程池中被拒绝时，线程池会放弃等待队列中最旧的未处理任务，然后将被拒绝的任务添加到等待队列中。
- DiscardPolicy,当任务添加到线程池中被拒绝时，线程池将丢弃被拒绝的任务。

**线程池默认的处理策略是AbortPolicy！**

### Callable && Future
Callable用于产生结果，Future用于获取结果。
#### Callable
Callable 是一个接口，它只包含一个call()方法。Callable是一个返回结果并且可能抛出异常的任务。
为了便于理解，我们可以将Callable比作一个Runnable接口，而Callable的call()方法则类似于Runnable的run()方法。
```java
public interface Callable<V> {
    V call() throws Exception;
}
```
Callable支持泛型
#### Future

Future 是一个接口。它用于表示异步计算的结果。提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。
```java
public interface Future<V> {
    // 试图取消对此任务的执行。
    boolean     cancel(boolean mayInterruptIfRunning)

    // 如果在任务正常完成前将其取消，则返回 true。
    boolean isCancelled()

    // 如果任务已完成，则返回 true。
    boolean isDone()

    // 如有必要，等待计算完成，然后获取其结果。
    V get() throws InterruptedException, ExecutionException;

    // 如有必要，最多等待为使计算完成所给定的时间之后，获取其结果（如果结果可用）。
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
Future用于表示异步计算的结果。它的实现类是FutureTask，FutureTask继承自RunnableFuture，而RunableFuture继承自Runable和Future。

#### 示例代码
```java
class MyCallable implements Callable {

    @Override
    public Integer call() throws Exception {
        int sum    = 0;
        // 执行任务
        for (int i=0; i<100; i++)
            sum += i;
        //return sum;
        return Integer.valueOf(sum);
    }
}

public class CallableTest1 {

    public static void main(String[] args)
        throws ExecutionException, InterruptedException{
        //创建一个线程池
        ExecutorService pool = Executors.newSingleThreadExecutor();
        //创建有返回值的任务
        Callable c1 = new MyCallable();
        //执行任务并获取Future对象
        Future f1 = pool.submit(c1);
        // 输出结果
        System.out.println(f1.get());
        //关闭线程池
        pool.shutdown();
    }
}
```













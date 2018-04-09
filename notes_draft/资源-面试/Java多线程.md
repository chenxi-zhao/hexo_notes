#### Java创建线程之后，直接调用start()方法和run()的区别
- start():它的作用是启动一个新线程，新线程会执行相应的run()方法。start()不能被重复调用。
- run(): run()等同于可重复调用的普通方法。单独调用run()，会在当前线程中执行run()方法，但是 并不会启动新线程！


#### 常用的线程池模式以及不同线程池的使用场景
- newCachedThreadPool 在newCachedThreadPool中如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
- newFixedThreadPool 创建一个定长，任务队列无界的线程池，可控制线程最大并发数，超出的线程会在队列中等待。
- newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
- newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。使用装饰模式增强了ScheduledExecutorService（1）的功能，不仅确保只有一个线程顺序执行任务，也保证线程意外终止后会重新创建一个线程继续执行任务。
- newWorkStealingPool创建一个拥有多个任务队列（以便减少连接数）的线程池。

- 对于需要保证所有提交的任务都要被执行的情况，使用FixedThreadPool
- 如果限定只能使用一个线程进行任务处理，使用SingleThreadExecutor
- 如果希望提交的任务尽快分配线程执行，使用CachedThreadPool
- 如果业务上允许任务执行失败，或者任务执行过程可能出现执行时间过长进而影响其他业务的应用场景，可以通过使用限定池线程数量的线程以及限定长度的队列进行容错处理。

![](http://static.tmaczhao.cn/images/f6d67838ff5b192ec4d48035c54bc47a.jpg)

![](http://static.tmaczhao.cn/images/36375c74e1d06bf988650f34c7baacf4.jpg)

![Java线程池执行器ThreadPoolExecutor工作原理](http://blog.csdn.net/tomatomas/article/details/51131576)

#### newFixedThreadPool此种线程池如果线程数达到最大值后会怎么办，底层原理。
创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 nThreads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务，则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在。


#### 多线程之间通信的同步问题，synchronized锁的是对象，衍伸出和synchronized相关很多的具体问题，例如同一个类不同方法都有synchronized锁，一个对象是否可以同时访问。或者一个类的static构造方法加上synchronized之后的锁的影响。

#### 了解可重入锁的含义，以及ReentrantLock和synchronized的区别
synchronized：
在资源竞争不是很激烈的情况下，偶尔会有同步的情形下，synchronized是很合适的。原因在于，编译程序通常会尽可能的进行优化synchronize，另外可读性非常好，不管用没用过5.0多线程包的程序员都能理解。
ReentrantLock:
ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。
Atomic:
和上面的类似，不激烈情况下，性能比synchronized略逊，而激烈的时候，也能维持常态。激烈的时候，Atomic的性能会优于ReentrantLock一倍左右。但是其有一个缺点，就是只能同步一个值，一段代码中只能出现一个Atomic的变量，多于一个同步无效。因为他不能在多个Atomic之间同步。

所以，我们写同步的时候，优先考虑synchronized，如果有特殊需要，再进一步优化。ReentrantLock和Atomic如果用的不好，不仅不能提高性能，还可能带来灾难。

#### 同步的数据结构，例如concurrentHashMap的源码理解以及内部实现原理，为什么他是同步的且效率高
1.7锁段、1.8cas

#### atomicinteger和volatile等线程安全操作的关键字的理解和使用
内存可见性，操作原子性问题

#### 线程间通信，wait和notify

#### 定时线程的使用

#### 场景：在一个主线程中，要求有大量(很多很多)子线程执行完之后，主线程才执行完成。多种方式，考虑效率。
1. 共享变量、同步修改共享变量，满足条件执行主线程
2. join变量，主线程调用，等待子线程执行完继续执行，此方法只能感知一个线程的过程不能判断多个线程的情况
3. CountDownLatch，子线程执行finally中技术减一，主线程通过await堵塞，知道所有子线程剪完执行。
4. FutureTask，FutureTask的get方法堵塞当前线程知道获取到结果为止。知道执行结果。还可以于线程池一起执行利用多cpu计算


#### 进程和线程的区别
![](http://static.tmaczhao.cn/images/f0bbd85eb86f457a9439b4e7768487a0.jpg)

#### 什么叫线程安全？举例说明

#### 线程的几种状态
新建、就绪、运行、堵塞、死亡

#### 并发、同步的接口或方法

15. HashMap 是否线程安全，为何不安全。 ConcurrentHashMap，线程安全，为何安全。底层实现是怎么样的。

16. J.U.C下的常见类的使用。ThreadPool的深入考察；BlockingQueue的使用。（take，poll的区别，put，offer的区别）；原子类的实现。
![BlockingQueue][http://blog.csdn.net/chenchaofuck1/article/details/51660119]

- ArrayBlockingQueue：ArrayBlockingQueue 是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了(译者注：因为它是基于数组实现的，也就具有数组的特性：一旦初始化，大小就无法修改)。

- DelayQueue：DelayQueue 对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 java.util.concurrent.Delayed 接口。

- LinkedBlockingQueue：LinkedBlockingQueue 内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。

- PriorityBlockingQueue：PriorityBlockingQueue 是一个无界的并发队列。它使用了和类 java.util.PriorityQueue 一样的排序规则。你无法向这个队列中插入 null 值。所有插入到 PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。因此该队列中元素的排序就取决于你自己的 Comparable 实现。

- SynchronousQueue：SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。

![](http://static.tmaczhao.cn/images/6c0f2184df961eaa9210f2d6bc07e801.jpg)


>offer: 将指定元素插入此队列中（如果立即可行且不会违反容量限制），成功时返回 true，如果当前没有可用的空间，则返回 false，不会抛异常；add会抛出异常IllegalStateException
put: 将指定元素插入此队列中，将等待可用的空间.通俗点说就是>maxSize 时候，阻塞，直到能够有空间插入元素

>take: 获取并移除此队列的头部，在元素变得可用之前一直等待 。queue的长度 == 0 的时候，一直阻塞
poll: 获取并移除队列头部，元素可用之前等待单位时间，返回头部可用元素或者null


17. 简单介绍下多线程的情况，从建立一个线程开始。然后怎么控制同步过程，多线程常用的方法和结构

18. volatile的理解

19. 实现多线程有几种方式，多线程同步怎么做，说说几个线程里常用的方法

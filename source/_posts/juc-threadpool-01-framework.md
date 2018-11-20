---
title: J.U.C.线程池 - 框架
date: 2016-05-13 15:04:18
categories: java-multi-thread
tags: [java, 多线程, 线程池]
---
Java里面线程池的顶级接口是Executor，但是严格意义上讲Executor并不是一个线程池，而只是一个执行线程的工具。真正的线程池接口是ExecutorService。

下面这张图完整描述了线程池的类体系结构。

![](https://static.tmaczhao.cn/images/java_multi_thread/Executor-class_thumb.png)

<!-- more -->

### 接口详解
#### Executor
execute方法只是执行一个Runnable的任务，当然了从某种角度上将最后的实现类也是在线程中启动此任务的。根据线程池的执行策略最后这个任务可能在新的线程中执行，或者线程池中的某个线程，甚至是调用者线程中执行（相当于直接运行Runnable的run方法）。这点在后面会详细说明。

#### ExecutorService
ExecutorService在Executor的基础上增加了一些方法，其中有两个核心的方法：
- Future<?> submit(Runnable task)
- <T> Future<T> submit(Callable<T> task)
这两个方法都是向线程池中提交任务，它们的区别在于Runnable在执行完毕后没有结果，Callable执行完毕后有一个结果。这在多个线程中传递状态和结果是非常有用的。另外他们的相同点在于都返回一个Future对象。Future对象可以阻塞线程直到运行完毕（获取结果，如果有的话），也可以取消任务执行，当然也能够检测任务是否被取消或者是否执行完毕。

在没有Future之前我们检测一个线程是否执行完毕通常使用Thread.join()或者用一个死循环加状态位来描述线程执行完毕。现在有了更好的方法能够阻塞线程，检测任务执行完毕甚至取消执行中或者未开始执行的任务。

#### ScheduledExecutorService
和Timer/TimerTask类似，解决那些需要任务重复执行的问题。这包括延迟时间一次性执行、延迟时间周期性执行以及固定延迟时间周期性执行等。当然了继承ExecutorService的ScheduledExecutorService拥有ExecutorService的全部特性。

#### ThreadPoolExecutor
ThreadPoolExecutor是ExecutorService的默认实现，其中的配置、策略也是比较复杂的。

#### ScheduledThreadPoolExecutor
继承ThreadPoolExecutor并实现接口ScheduledExecutorService，周期性任务调度的类实现。

这里需要稍微提一下的是CompletionService接口，它是用于描述顺序获取执行结果的一个线程池包装器。它依赖一个具体的线程池调度，但是能够根据任务的执行先后顺序得到执行结果，这在某些情况下可能提高并发效率。



#### Executors
Executors是个静态工厂类。它通过静态工厂方法返回ExecutorService、ScheduledExecutorService、ThreadFactory和Callable等类的对象。

newSingleThreadExecutor：创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。
newFixedThreadPool：创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。
newCachedThreadPool：创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。
newScheduledThreadPool：创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。
newSingleThreadScheduledExecutor：创建一个单线程的线程池。此线程池支持定时以及周期性执行任务的需求。


### 线程池生命周期
#### 线程池的四种状态
![](https://images.cnitblog.com/blog/497634/201401/08000847-0a9caed4d6914485b2f56048c668251a.jpg)

1. RUNNING
- 状态说明：线程池处在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理。
- 状态切换：线程池的初始化状态是RUNNING。换句话说，线程池被一旦被创建，就处于RUNNING状态！
道理很简单，在ctl的初始化代码中(如下)，就将它初始化为RUNNING状态，并且"任务数量"初始化为0。

2. SHUTDOWN
- 状态说明：线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务。
- 状态切换：调用线程池的shutdown()接口时，线程池由RUNNING -> SHUTDOWN。

3. STOP
- 状态说明：线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务。
- 状态切换：调用线程池的shutdownNow()接口时，线程池由(RUNNING or SHUTDOWN ) -> STOP。

4. TIDYING
- 状态说明：当所有的任务已终止，ctl记录的"任务数量"为0，线程池会变为TIDYING状态。当线程池变为TIDYING状态时，会执行钩子函数terminated()。terminated()在ThreadPoolExecutor类中是空的，若用户想在线程池变为TIDYING时，进行相应的处理；可以通过重载terminated()函数来实现。
- 状态切换：当线程池在SHUTDOWN状态下，阻塞队列为空并且线程池中执行的任务也为空时，就会由SHUTDOWN -> TIDYING。
当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP -> TIDYING。

5. TERMINATED
- 状态说明：线程池彻底终止，就变成TERMINATED状态。
- 状态切换：线程池处在TIDYING状态时，执行完terminated()之后，就会由TIDYING -> TERMINATED。

#### 线程池的API
![](https://static.tmaczhao.cn/images/java_multi_thread/ExecutorService-LifeCycle_thumb.png)

1. 平缓关闭线程池使用shutdown()
2. 立即关闭线程池使用shutdownNow()，同时得到未执行的任务列表
3. 检测线程池是否正处于关闭中，使用isShutdown()
4. 检测线程池是否已经关闭使用isTerminated()
5. 定时或者永久等待线程池关闭结束使用awaitTermination()操作
















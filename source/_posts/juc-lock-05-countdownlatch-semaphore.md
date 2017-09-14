---
title: J.U.C.锁机制 - CountDownLatch & 信号量
date: 2016-05-10 10:47:08
categories: java-multi-thread
tags: [java, 多线程]
---
闭锁（Latch）：一种同步方法，可以延迟线程的进度直到线程到达某个终点状态。通俗的讲就是，一个闭锁相当于一扇大门，在大门打开之前所有线程都被阻断，一旦大门打开所有线程都将通过，但是一旦大门打开，所有线程都通过了，那么这个闭锁的状态就失效了，门的状态也就不能变了，只能是打开状态。也就是说闭锁的状态是一次性的，它确保在闭锁打开之前所有特定的活动都需要在闭锁打开之后才能完成。

<!-- more -->

CountDownLatch是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

CountDownLatch和CyclicBarrier的区别
1. CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待。
2. CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。
关于CyclicBarrier的原理，后面一章再来学习。


### CountDownLatch函数列表
```java
CountDownLatch(int count)
构造一个用给定计数初始化的 CountDownLatch。
// 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。
void await()
// 使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断或超出了指定的等待时间。
boolean await(long timeout, TimeUnit unit)
// 递减锁存器的计数，如果计数到达零，则释放所有等待的线程。
void countDown()
// 返回当前计数。
long getCount()
// 返回标识此锁存器及其状态的字符串。
String toString()
```


### CountDownLatch数据结构
CountDownLatch的UML类图如下：
![](https://static.tmaczhao.cn/images/5283cff8e03e4fa5aed86282e9bca233.jpg)

CountDownLatch的数据结构很简单，它是通过"共享锁"实现的。它包含了sync对象，sync是Sync类型。Sync是实例类，它继承于AQS


### CountDownLatch原理解析
CountDownLatch是通过共享锁实现。其中主要有3个核心函数: CountDownLatch(int count), await(), countDown()。

#### CountDownLatch(int count)
```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```
该函数主要是创建一个Sync对象，而Sync是继承于AQS类。Sync构造函数如下：
```java
Sync(int count) {
    setState(count);
}
```
setState()在AQS中实现，源码如下：
```java
protected final void setState(long newState) {
    state = newState;
}
```
在AQS中，state是一个private volatile long类型的对象。
对于CountDownLatch而言，state表示的”锁计数器“。CountDownLatch中的getCount()最终是调用AQS中的getState()，返回的state对象，即”锁计数器“。

#### await()
```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
在多个线程中调用该方法可以将线程节点加入到CLH队列当中堵塞，等待countDown()后依次获取共享锁执行，获取共享锁的方式与ReadLock相似。

#### countDown()
```java
public void countDown() {
    sync.releaseShared(1);
}
```
该函数实际上调用releaseShared(1)释放共享锁。
释放共享锁后，等待在CLH队列中的线程被唤醒执行

#### 总结
CountDownLatch是通过“共享锁”实现的。在创建CountDownLatch中时，会传递一个int类型参数count，该参数是“锁计数器”的初始状态，表示该“共享锁”最多能被count给线程同时获取。当某线程调用该CountDownLatch对象的await()方法时，该线程会等待“共享锁”可用时，才能获取“共享锁”进而继续运行。而“共享锁”可用的条件，就是“锁计数器”的值为0！而“锁计数器”的初始值为count，每当一个线程调用该CountDownLatch对象的countDown()方法时，才将“锁计数器”-1；通过这种方式，必须有count个线程调用countDown()之后，“锁计数器”才为0，而前面提到的等待线程才能继续运行！

### 示例代码
```java
public class PerformanceTestTool {
    public long timecost(final int times, final Runnable task) throws InterruptedException {
        if (times <= 0) throw new IllegalArgumentException();
        final CountDownLatch startLatch = new CountDownLatch(1);
        final CountDownLatch overLatch = new CountDownLatch(times);
        for (int i = 0; i < times; i++) {
            new Thread(new Runnable() {
                public void run() {
                    try {
                        startLatch.await();
                        //
                        task.run();
                    } catch (InterruptedException ex) {
                        Thread.currentThread().interrupt();
                    } finally {
                        overLatch.countDown();
                    }
                }
            }).start();
        }
        //
        long start = System.nanoTime();
        startLatch.countDown();
        overLatch.await();
        return System.nanoTime() - start;
    }
}
```


### Semaphore信号量
信号量维护了一个信号量许可集。线程可以通过调用acquire()来获取信号量的许可；
当信号量中有可用的许可时，线程能获取该许可；否则线程必须等待，直到有可用的许可为止。
线程可以通过release()来释放它所持有的信号量许可。

说白了，Semaphore是一个计数器，在计数器不为0的时候对线程就放行，一旦达到0，那么所有请求资源的新线程都会被阻塞，包括增加请求到许可的线程，也就是说Semaphore不是可重入的。每一次请求一个许可都会导致计数器减少1，同样每次释放一个许可都会导致计数器增加1，一旦达到了0，新的许可请求线程将被挂起。

缓存池就是使用此思想来实现，比如链接池、对象池等。

将信号量初始化为 1，使得它在使用时最多只有一个可用的许可，从而可用作一个相互排斥的锁。这通常也称为二进制信号量，因为它只能有两种状态：一个可用的许可，或零个可用的许可。按此方式使用时，二进制信号量具有某种属性（与很多 Lock 实现不同），即可以由线程释放“锁”，而不是由所有者（因为信号量没有所有权的概念）。在某些专门的上下文（如死锁恢复）中这会很有用。

上面这段话的意思是说当某个线程A持有信号量数为1的信号量时，其它线程只能等待此线程释放资源才能继续，这时候持有信号量的线程A就相当于持有了“锁”，其它线程的继续就需要这把锁，于是线程A的释放才能决定其它线程的运行，相当于扮演了“锁”的角色。

**Semaphore的acquire方法实际上访问的是AQS的acquireSharedInterruptibly(arg)方法**

#### "公平信号量"和"非公平信号量"的区别
"公平信号量"和"非公平信号量"的释放信号量的机制是一样的！
不同的是它们获取信号量的机制：
对于公平信号量而言，如果当前线程不在CLH队列的头部，则排队等候；
而对于非公平信号量而言，无论当前线程是不是在CLH队列的头部，它都会直接获取信号量。该差异具体的体现在，它们的tryAcquireShared()函数的实现不同。

所以非公平信号量的吞吐量总是要比公平信号量的吞吐量要大，但是需要强调的是非公平信号量和非公平锁一样存在“饥渴死”的现象，也就是说活跃线程可能总是拿到信号量，而非活跃线程可能难以拿到信号量。而对于公平信号量由于总是靠请求的线程的顺序来获取信号量，所以不存在此问题。













































---
title: J.U.C.锁机制 - Cyclicbarrier
date: 2016-05-10 10:49:09
categories: java-multi-thread
tags: [java, 多线程]
---
CyclicBarrier是一个同步辅助类，允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。因为该barrier在释放等待线程后可以重用，所以称它为循环的barrier。

<!-- more -->

注意比较CountDownLatch和CyclicBarrier：
1. CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待。
2. CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。

### CyclicBarrier函数列表
```java
//创建一个新的CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动
//但它不会在启动barrier时执行预定义的操作。
CyclicBarrier(int parties)
//创建一个新的CyclicBarrier，它将在给定数量的参与者（线程）处于等待状态时启动，并在启动barrier时执行给定的屏障操作，该操作由最后一个进入barrier的线程执行。
CyclicBarrier(int parties, Runnable barrierAction)

//在所有参与者都已经在此barrier上调用await方法之前，将一直等待。
int await()
//在所有参与者都已经在此屏障上调用await方法之前将一直等待,或者超出了指定的等待时间。
int await(long timeout, TimeUnit unit)
//返回当前在屏障处等待的参与者数目。
int getNumberWaiting()
//返回要求启动此barrier的参与者数目。
int getParties()
//查询此屏障是否处于损坏状态。
boolean isBroken()
//将屏障重置为其初始状态。
void reset()
```

### CyclicBarrier数据结构
CyclicBarrier的UML类图如下：
![](https://static.tmaczhao.cn/images/4f97fc32b879d97c01e13692fd99c99c.jpg)
CyclicBarrier是包含了"ReentrantLock对象lock"和"Condition对象trip"，它是通过独占锁实现的。

### CyclicBarrier原理分析
#### 构造函数
CyclicBarrier的构造函数共2个：CyclicBarrier和CyclicBarrier(int parties, Runnable barrierAction)。第1个构造函数是调用第2个构造函数来实现的，下面第2个构造函数的源码。
```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    // parties表示“必须同时到达barrier的线程个数”。
    this.parties = parties;
    // count表示“处在等待状态的线程个数”。
    this.count = parties;
    // barrierCommand表示“parties个线程到达barrier时，会执行的动作”。
    this.barrierCommand = barrierAction;
}
```

#### 等待函数
CyclicBarrier.java中await()方法如下：
```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen;
    }
}
```
await()是通过dowait()实现的。
```java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    // 获取“独占锁(lock)”
    lock.lock();
    try {
        // 保存“当前的generation”
        final Generation g = generation;

        // 若“当前generation已损坏”，则抛出异常。
        if (g.broken)
            throw new BrokenBarrierException();

        // 如果当前线程被中断，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

       // 将“count计数器”-1
       int index = --count;
       // 如果index=0，则意味着“有parties个线程到达barrier”。
       if (index == 0) {  // tripped
           boolean ranAction = false;
           try {
               // 如果barrierCommand不为null，则执行该动作。
               final Runnable command = barrierCommand;
               if (command != null)
                   command.run();
               ranAction = true;
               // 唤醒所有等待线程，并更新generation。
               nextGeneration();
               return 0;
           } finally {
               if (!ranAction)
                   breakBarrier();
           }
       }

        // 当前线程一直阻塞，直到“有parties个线程到达barrier” 或 “当前线程被中断” 或 “超时”这3者之一发生，
        // 当前线程才继续执行。
        for (;;) {
            try {
                // 如果不是“超时等待”，则调用awati()进行等待；否则，调用awaitNanos()进行等待。
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                // 如果等待过程中，线程被中断，则执行下面的函数。
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    Thread.currentThread().interrupt();
                }
            }

            // 如果“当前generation已经损坏”，则抛出异常。
            if (g.broken)
                throw new BrokenBarrierException();

            // 如果“generation已经换代”，则返回index。
            if (g != generation)
                return index;

            // 如果是“超时等待”，并且时间已到，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        // 释放“独占锁(lock)”
        lock.unlock();
    }
}
```
dowait()的作用就是让当前线程阻塞，直到“有parties个线程到达barrier”或“当前线程被中断”或“超时”这3者之一发生，当前线程才继续执行。
1. generation是CyclicBarrier的一个成员遍历，它的定义如下：
```java
private Generation generation = new Generation();
private static class Generation {
    boolean broken = false;
}
```
在CyclicBarrier中，同一批的线程属于同一代，即同一个Generation；CyclicBarrier中通过generation对象，记录属于哪一代。当有parties个线程到达barrier，generation就会被更新换代。

2. 如果当前线程被中断，即Thread.interrupted()为true；则通过breakBarrier()终止CyclicBarrier。
```java
private void breakBarrier() {
    generation.broken = true;
    count = parties;
    trip.signalAll();
}
```
breakBarrier()会设置当前中断标记broken为true，意味着“将该Generation中断”；同时，设置count=parties，即重新初始化count；最后，通过signalAll()唤醒CyclicBarrier上所有的等待线程。

3. 将“count计数器”-1，即--count；然后判断是不是“有parties个线程到达barrier”，即index是不是为0。
当index=0时，如果barrierCommand不为null，则执行该barrierCommand，barrierCommand就是我们创建CyclicBarrier时，传入的Runnable对象。然后，调用nextGeneration()进行换代工作.
```java
private void nextGeneration() {
    trip.signalAll();
    count = parties;
    generation = new Generation();
}
```
首先，它会调用signalAll()唤醒CyclicBarrier上所有的等待线程；接着，重新初始化count；最后，更新generation的值。

4. 在for(;;)循环中。timed是用来表示当前是不是“超时等待”线程。如果不是，则通过trip.await()进行等待；否则，调用awaitNanos()进行超时等待。



### 示例代码
```java
public class CyclicBarrierDemo {
    final CyclicBarrier barrier;
    final int MAX_TASK;

    public CyclicBarrierDemo(int cnt) {
        barrier = new CyclicBarrier(cnt + 1);
        MAX_TASK = cnt;
    }

    public void doWork(final Runnable work) {
        new Thread() {
            public void run() {
                work.run();
                try {
                    //返回线程执行完毕的索引，任务数-1
                    int index = barrier.await();
                    doWithIndex(index);
                } catch (InterruptedException e) {
                    return;
                } catch (BrokenBarrierException e) {
                    return;
                }
            }
        }.start();
    }

    private void doWithIndex(int index) {
        if (index == MAX_TASK / 3) {
            System.out.println("Left 30%.");
        } else if (index == MAX_TASK / 2) {
            System.out.println("Left 50%");
        } else if (index == 0) {
            System.out.println("run over");
        }
    }

    public void waitForNext() {
        try {
            doWithIndex(barrier.await());
        } catch (InterruptedException e) {
            return;
        } catch (BrokenBarrierException e) {
            return;
        }
    }

    public static void main(String[] args) {
        final int count = 10;
        CyclicBarrierDemo demo = new CyclicBarrierDemo(count);
        for (int i = 0; i < 100; i++) {
            demo.doWork(new Runnable() {

                public void run() {
                    //do something
                    try {
                        Thread.sleep(1000L);
                    } catch (Exception e) {
                        return;
                    }
                }
            });
            //挂起主线程
            if ((i + 1) % count == 0) {
                demo.waitForNext();
            }
        }
    }

}
```





















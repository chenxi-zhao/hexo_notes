---
title: J.U.C.锁机制 - 框架
date: 2016-05-09 09:41:08
categories: java-multi-thread
tags: [java, 多线程]
---
相比同步锁，JUC包中的锁的功能更加强大，它为锁提供了一个框架，该框架允许更灵活地使用锁，只是它的用法更难。

<!-- more -->

### 框架
先看看锁的框架图，如下所示。
![](http://static.tmaczhao.cn/images/5eff15dd9676dc6ddffbf6ed3fa972c5.jpg)
#### Lock接口
>JUC包中的 Lock 接口支持那些语义不同(重入、公平等)的锁规则。
所谓语义不同，是指锁可是有"公平机制的锁"、"非公平机制的锁"、"可重入的锁"等等。
"公平机制"是指"不同线程获取锁的机制是公平的"，
而"非公平机制"则是指"不同线程获取锁的机制是非公平的"，"可重入的锁"是指同一个锁能够被一个线程多次获取。

#### ReadWriteLock
>接口以和Lock类似的方式定义了一些读取者可以共享而写入者独占的锁。JUC包只有一个类实现了该接口，即 ReentrantReadWriteLock，因为它适用于大部分的标准用法上下文。但程序员可以创建自己的、适用于非标准要求的实现。

#### AbstractOwnableSynchronizer/AbstractQueuedSynchronizer/AbstractQueuedLongSynchronizer
>AbstractQueuedSynchronizer就是被称之为AQS的类，它是一个非常有用的超类，可用来定义锁以及依赖于排队阻塞线程的其他同步器；
ReentrantLock，ReentrantReadWriteLock，CountDownLatch，CyclicBarrier和Semaphore等这些类都是基于AQS类实现的。
AbstractQueuedLongSynchronizer类提供相同的功能但扩展了对同步状态的64位的支持。两者都扩展了类 AbstractOwnableSynchronizer（一个帮助记录当前保持独占同步的线程的简单类）。

#### LockSupport
>LockSupport提供“创建锁”和“其他同步类的基本线程阻塞原语”。
>LockSupport的功能和"Thread中的Thread.suspend()和Thread.resume()有点类似"，LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程。但是park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。

#### Condition
>Condition需要和Lock联合使用，它的作用是代替Object监视器方法，可以通过await(),signal()来休眠/唤醒线程。
Condition 接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用 Object.wait 访问的隐式监视器类似，但提供了更强大的功能。
需要特别指出的是，单个Lock可能与多个Condition对象关联。为了避免兼容性问题，Condition方法的名称与对应的Object版本中的不同。

#### ReentrantLock
>ReentrantLock是独占锁。所谓独占锁，是指只能被独自占领，即同一个时间点只能被一个线程锁获取到的锁。
ReentrantLock锁包括"公平的ReentrantLock"和"非公平的ReentrantLock"。
"公平的ReentrantLock"是指"不同线程获取锁的机制是公平的"
"非公平的ReentrantLock"则是指"不同线程获取锁的机制是非公平的"，ReentrantLock是"可重入的锁"。

#### ReentrantReadWriteLock
>ReentrantReadWriteLock是读写锁接口ReadWriteLock的实现类，它包括子类ReadLock和WriteLock。
ReentrantLock是共享锁，而WriteLock是独占锁。

#### CountDownLatch
>CountDownLatch是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

#### CyclicBarrier
>CyclicBarrier是一个同步辅助类，允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。因为该 barrier 在释放等待线程后可以重用，所以称它为循环的barrier。

<font color="red">CyclicBarrier和CountDownLatch的区别是：</font>
1. CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许N个线程相互等待。
2. CountDownLatch的计数器无法被重置；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier。

#### Semaphore
>Semaphore是一个计数信号量，它的本质是一个"共享锁"。
>信号量维护了一个信号量许可集。线程可以通过调用acquire()来获取信号量的许可；当信号量中有可用的许可时，线程能获取该许可；否则线程必须等待，直到有可用的许可为止。 线程可以通过release()来释放它所持有的信号量许可。


### 基本概念
#### AQS -- 指AbstractQueuedSynchronizer类。
>AQS是java中管理“锁”的抽象类，锁的许多公共方法都是在这个类中实现。AQS是独占锁(例如，ReentrantLock)和共享锁(例如，Semaphore)的公共父类。

#### AQS锁的类别 -- 分为“独占锁”和“共享锁”两种。
>**`独占锁`** -- 锁在一个时间点只能被一个线程锁占有。根据锁的获取机制，它又划分为“公平锁”和“非公平锁”。公平锁，是按照通过CLH等待线程按照先来先得的规则，公平的获取锁；而非公平锁，则当线程要获取锁时，它会无视CLH等待队列而直接获取锁。独占锁的典型实例子是ReentrantLock，此外，ReentrantReadWriteLock.WriteLock也是独占锁。
>**`共享锁`** -- 能被多个线程同时拥有，能被共享的锁。JUC包中的ReentrantReadWriteLock.ReadLock，CyclicBarrier，CountDownLatch和Semaphore都是共享锁。这些锁的用途和原理，在以后的章节再详细介绍。

#### CLH队列 -- Craig, Landin, and Hagersten lock queue
>CLH队列是AQS中“等待锁”的线程队列。在多线程中，为了保护竞争资源不被多个线程同时操作而起来错误，我们常常需要通过锁来保护这些资源。在独占锁中，竞争资源在一个时间点只能被一个线程锁访问；而其它线程则需要等待。CLH就是管理这些“等待锁”的线程的队列。
>CLH是一个非阻塞的 FIFO 队列。也就是说往里面插入或移除一个节点的时候，在并发条件下不会阻塞，而是通过自旋锁和 CAS 保证节点插入和移除的原子性。

#### CAS函数 -- Compare And Swap
>CAS函数，是比较并交换函数，是原子操作函数；即，通过CAS操作的数据都是以原子方式进行的。例如，compareAndSetHead(),compareAndSetTail(),等函数。它们共同的特点是，这些函数所执行的动作是以原子的方式进行的。





---
title: Java多线程基础
date: 2016-05-06 15:11:49
categories: java-multi-thread
tags: [java, 多线程]
---
### 什么是多线程

线程状态图
![](http://images.cnitblog.com/blog/288799/201409/061046391107893.jpg)

<!-- more -->

说明：
线程共包括以下5种状态。
1. **新建状态(New)**: 线程对象被创建后，就进入了新建状态。例如，Thread thread = new Thread()。
2. **就绪状态(Runnable)**: 也被称为“可执行状态”。线程对象被创建后，其它线程调用了该对象的start()方法，从而来启动该线程。例如，thread.start()。处于就绪状态的线程，随时可能被CPU调度执行。
3. **运行状态(Running)**: 线程获取CPU权限进行执行。需要注意的是，线程只能从就绪状态进入到运行状态。
4. **阻塞状态(Blocked)**: 阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：
>等待阻塞 -- 通过调用线程的wait()方法，让线程等待某工作的完成。
同步阻塞 -- 线程在获取synchronized同步锁失败(因为锁被其它线程所占用)，它会进入同步阻塞状态。
其他阻塞 -- 通过调用线程的sleep()或join()或发出了I/O请求时，线程会进入到阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

5. **死亡状态(Dead)**: 线程执行完了或者因异常退出了run()方法，该线程结束生命周期。



这5种状态涉及到的内容包括Object类, Thread和synchronized关键字。这些内容我们会在后面的章节中逐个进行学习。
**`Object类`**， 定义了wait(), notify(), notifyAll()等休眠/唤醒函数。
**`Thread类`**， 定义了一些列的线程操作函数。例如，sleep()休眠函数, interrupt()中断函数, getName()获取线程名称等。
**`synchronized`**关键字, 区分为synchronized代码块和synchronized方法。synchronized让线程获取对象的同步锁。
在后面详细介绍wait(),notify()等方法时，我们会分析为什么wait(), notify()等方法要定义在Object类，而不是Thread类中.
>volatile修饰的变量，jvm虚拟机保证从主内存加载到线程工作内存的值是最新,多线程过程中保证每一次函数栈使用该值时都会去堆内存中加载最新的值处理，但很有可能出现脏读

### Thread和Runable
java中常常用继承Thread类和实现Runable接口来实现多线程，Thread本身也实现了Runable接口。
```java
class MyThread extends Thread{
     public void run(){
     }
}

class MyThread implements Runnable{
    public void run(){
    }
}
```
在实际开发中通常建议多使用Runable接口，扩展性相较于Thread更好，同时Runable更加有利于共享资源

### Thread中的start和run
>start():它的作用是启动一个新线程，新线程会执行相应的run()方法。start()不能被重复调用。
run(): run()等同于可重复调用的普通方法。单独调用run()，会在当前线程中执行run()方法，但是 并不会启动新线程！

start通过**`native void start0()`**方法启动一个新线程来调用run()方法，run()方法中直接调用Runable对象的run()方法，不建立新线程，Thread源码（1.7）如下
```java
public synchronized void start() {
    ...
    boolean started = false;
    try {
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
              it will be passed up the call stack */
        }
    }
}
private native void start0();

@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

### synchronized 关键字
#### synchronized原理
>**java中，每一个对象有且仅有一个同步锁。这也意味着，同步锁是依赖于对象而存在。**
**当我们调用某对象的synchronized方法时，就获取了该对象的同步锁。**synchronized(obj)获取了“obj”的同步锁。
**不同线程对同步锁的访问是互斥的**。也就是说，某时间点，对象的同步锁只能被一个线程获取到！通过同步锁，我们就能在多线程中，实现对“对象/方法”的互斥访问。

#### synchronized基本规则
 - 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的该“synchronized方法”或者“synchronized代码块”的访问将被阻塞。
 - 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程仍然可以访问“该对象”的非同步代码块。
 - 当一个线程访问“某对象”的“synchronized方法”或者“synchronized代码块”时，其他线程对“该对象”的其他的“synchronized方法”或者“synchronized代码块”的访问将被阻塞。

“synchronized方法”是用synchronized修饰方法，而 “synchronized代码块”则是用synchronized修饰代码块。
```java
public synchronized void foo1() {
    System.out.println("synchronized method");
}
public void foo2() {
    synchronized (this) {
        System.out.println("synchronized method");
    }
}
```
#### 实例锁和全局锁
实例锁 -- 锁在某一个实例对象上。如果该类是单例，那么该锁也具有全局锁的概念。实例锁对应的就是synchronized关键字。
全局锁 -- 该锁针对的是类，无论实例多少个对象，那么线程都共享该锁。全局锁对应的就是static synchronized（或者是锁在该类的class或者classloader对象上）。
关于“实例锁”和“全局锁”有一个很形象的例子：
```java
pulbic class Something {
    public synchronized void isSyncA(){}
    public synchronized void isSyncB(){}
    public static synchronized void cSyncA(){}
    public static synchronized void cSyncB(){}
}
```
假设，Something有两个实例x和y。分析下面4组表达式获取的锁的情况。
1. x.isSyncA()与x.isSyncB() 不能同时被访问。因为isSyncA()和isSyncB()都是访问同一个对象的同步锁
2. x.isSyncA()与y.isSyncA() 可以同时被访问。因为访问的不是同一个对象的同步锁
3. x.cSyncA()与y.cSyncB() 不能同时被访问。cSyncA()和cSyncB()都是static类型，两个方法共用Something同步锁
4. x.isSyncA()与Something.cSyncA 可以同时被访问。因为isSyncA()是实例方法，x.isSyncA()使用的是对象x的锁；而cSyncA()是静态方法，Something.cSyncA()可以理解对使用的是“类的锁”


### 线程等待和唤醒
#### Object类中的等待/唤醒
>notify() -- 唤醒在此对象监视器上等待的单个线程。

>notifyAll() -- 唤醒在此对象监视器上等待的所有线程。

>wait() -- 让当前线程处于“等待(阻塞)状态”，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，当前线程被唤醒(进入“就绪状态”)。

>wait(long timeout) -- 让当前线程处于“等待(阻塞)状态”，直到其他线程调用此对象的 notify() 方法或
notifyAll() 方法，或者超过指定的时间量，当前线程被唤醒(进入“就绪状态”)。

>wait(long timeout, int nanos) -- 让当前线程处于“等待(阻塞)状态”，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量，当前线程被唤醒(进入就绪状态)。

#### 为什么notify(), wait()等函数定义在Object中，而不是Thread中
Object中的wait(), notify()等函数，和synchronized一样，会对“对象的同步锁”进行操作。

wait()会使“当前线程”等待，因为线程进入等待状态，所以线程应该释放它锁持有的“同步锁”，否则其它线程获取不到该“同步锁”而无法运行！
OK，线程调用wait()之后，会释放它锁持有的“同步锁”；而且，根据前面的介绍，我们知道：等待线程可以被notify()或notifyAll()唤醒。现在，请思考一个问题：notify()是依据什么唤醒等待线程的？或者说，**wait()等待线程和notify()之间是通过什么关联起来的？答案是：依据“对象的同步锁”**。

负责唤醒等待线程的那个线程(我们称为“唤醒线程”)，它只有在获取“该对象的同步锁”(这里的同步锁必须和等待线程的同步锁是同一个)，并且调用notify()或notifyAll()方法之后，才能唤醒等待线程。虽然，等待线程被唤醒；但是，它不能立刻执行，因为唤醒线程还持有“该对象的同步锁”。必须等到唤醒线程释放了“对象的同步锁”之后，等待线程才能获取到“对象的同步锁”进而继续运行。

总之，notify(), wait()依赖于“同步锁”，而“同步锁”是对象锁持有，并且每个对象有且仅有一个！这就是为什么notify(), wait()等函数定义在Object类，而不是Thread类中的原因。


### 线程让步
#### yield
yield()的作用是让步。它能让当前线程由“运行状态”进入到“就绪状态”，从而让其它具有相同优先级的等待线程获取执行权；但是，并不能保证在当前线程调用yield()之后，其它具有相同优先级的线程就一定能获得执行权；也有可能是当前线程又进入到“运行状态”继续运行！
```java
class ThreadA extends Thread{
    public ThreadA(String name){
        super(name);
    }
    public synchronized void run(){
        for(int i=0; i <10; i++){
            System.out.printf("%s [%d]:%d\n", this.getName(), this.getPriority(), i);
            // i整除4时，调用yield
            if (i%4 == 0)
                Thread.yield();
        }
    }
}
```
#### yield()与wait()的比较
我们知道，wait()的作用是让当前线程由“运行状态”进入“等待(阻塞)状态”的同时，也会释放同步锁。而yield()的作用是让步，它也会让当前线程离开“运行状态”。它们的区别是：
 - wait()是让线程由“运行状态”进入到“等待(阻塞)状态”，而不yield()是让线程由“运行状态”进入到“就绪状态”。
 - wait()是会线程释放它所持有对象的同步锁，而yield()方法不会释放锁。


### sleep()
sleep() 定义在Thread.java中。
sleep() 的作用是让当前线程休眠，即当前线程会从“运行状态”进入到“休眠(阻塞)状态”。sleep()会指定休眠时间，线程休眠的时间会大于/等于该休眠时间；在线程重新被唤醒时，它会由“阻塞状态”变成“就绪状态”，从而等待cpu的调度执行。
```java
class ThreadA extends Thread{
    public ThreadA(String name){
        super(name);
    }
    public synchronized void run() {
        try {
            for(int i=0; i <10; i++){
                System.out.printf("%s: %d\n", this.getName(), i);
                // i能被4整除时，休眠100毫秒
                if (i%4 == 0)
                    Thread.sleep(100);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
#### sleep()与wait()的比较
我们知道，wait()的作用是让当前线程由“运行状态”进入“等待(阻塞)状态”的同时，也会释放同步锁。而sleep()的作用是也是让当前线程由“运行状态”进入到“休眠(阻塞)状态”。
但是，wait()会释放对象的同步锁，而sleep()则不会释放锁。


### join()
join() 定义在Thread.java中。join() 的作用：让“主线程”等待“子线程”结束之后才能继续运行（必须线程启动）。
```java
// 主线程
public class Father extends Thread {
    public void run() {
        Son s = new Son();
        s.start();
        s.join();
        ...
    }
}
// 子线程
public class Son extends Thread {
    public void run() {
        ...
    }
}
```
#### join原理
Son是在Father中创建并启动的，所以，Father是主线程类，Son是子线程类,在调用s.join()之后，Father主线程会一直等待，直到“子线程s”运行完毕；在“子线程s”运行完毕之后，Father主线程才能接着运行。这也就是我们所说的“join()的作用，是让主线程会等待子线程结束之后才能继续运行“！
```java
//方法是个同步的，而且会抛出InterruptedException异常
public final synchronized void join(long millis) throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;
    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }
    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        //如果是等待的特定时间
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}

/**
 * Tests if this thread is alive. A thread is alive if it has
 * been started and has not yet died.
 *
 * @return  <code>true</code> if this thread is alive;
 *          <code>false</code> otherwise.
 */
public final native boolean isAlive();
```
isAlive()和wait(0)都是native方法。isAlive判断**`this thread`**是否存活，而wait(0)堵塞**`current thread`**
join方法由主线程触发，则该方法内执行时当前线程是主线程，所以子线程执行结束之前主线程一直被休眠


### interrupt()和线程终止方式
**interrupt()的作用是中断本线程**。
其它线程调用本线程的interrupt()方法时，会通过checkAccess()检查权限。这有可能抛出SecurityException异常。
<font color="red">Thread中的stop()和suspend()方法，由于固有的不安全性，已经建议不再使用！</font>
#### 终止处于“阻塞状态”的线程
当线程由于被调用了sleep(), wait(), join()等方法而进入阻塞状态；若此时调用线程的interrupt()将线程的中断标记设为true。由于处于阻塞状态，中断标记会被清除，同时产生一个InterruptedException异常。
```java
public void run() {
    while (true) {
        try {
            // 执行任务...
        } catch (InterruptedException ie) {
            // InterruptedException在while(true)循环体内。
            // 可以将while循环直接放到try语句块中执行则不需要手动退出
            // 当线程产生了InterruptedException异常时，while(true)仍能继续运行！需要手动退出
            break;
        }
    }
}
```
#### 终止处于“运行状态”的线程
通常，我们通过“标记”方式终止处于“运行状态”的线程。其中，包括“中断标记”和“额外添加标记”。
##### 通过“中断标记”终止线程。
```java
public void run() {
    while (!isInterrupted()) {
        // 执行任务...
    }
}
```
`注意：interrupt()并不会终止处于“运行状态”的线程！它会将线程的中断标记设为true。`

##### 通过“额外添加标记”。
```java
private volatile boolean flag= true;
protected void stopTask() {
    flag = false;
}
public void run() {
    while (flag) {
        // 执行任务...
    }
}
```
说明：线程中有一个flag标记，它的默认值是true；并且我们提供stopTask()来设置flag标记。当我们需要终止该线程时，调用该线程的stopTask()方法就可以让线程退出while循环。
注意：将flag定义为volatile类型，是为了保证flag的可见性。即其它线程通过stopTask()修改了flag之后，本线程能看到修改后的flag的值

**`综合线程处于“阻塞状态”和“运行状态”的终止方式，比较通用的终止线程的形式如下`**：
```java
public void run() {
    try {
        // 1. isInterrupted()保证，只要中断标记为true就终止线程。
        while (!isInterrupted()) {
            // 执行任务...
        }
    } catch (InterruptedException ie) {
        // 2. InterruptedException异常保证，当InterruptedException异常产生时，线程被终止。
    }
}
```
#### interrupted() 和 isInterrupted()的区别
interrupted() 和 isInterrupted()都能够用于检测对象的“中断标记”。
区别是，interrupted()除了返回中断标记之外，它还会清除中断标记(即将中断标记设为false)；而isInterrupted()仅仅返回中断标记。
>PS. 如果该线程在一个Selector（java.nio.channels.Selector）中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，并可能带有一个非零值，就好像调用了选择器的 wakeup 方法一样。


### 线程优先级和守护线程
java中的线程优先级的范围是1～10，默认的优先级是5。“高优先级线程”会优先于“低优先级线程”执行。
java中有两种线程：**`用户线程`**和**`守护线程`**。可以通过isDaemon()(setDaemon()可以设置是否守护线程)方法来区别它们：如果返回false，则说明该线程是“用户线程”；否则就是“守护线程”。
守护线程的优先级比较低，用于为系统中的其它对象和线程提供服务。
用户线程一般用户执行用户级任务，而守护线程也就是“后台线程”，一般用来执行后台任务。需要注意的是：Java虚拟机在只有守护进程运行时退出。
**当设置了某几个线程的优先级后，几个不同优先级的线程根据时间片轮循调度并发执行**

### 生产消费者问题
```java
// 仓库
class Depot {
    private int capacity;    // 仓库的容量
    private int size;        // 仓库的实际数量

    public Depot(int capacity) {
        this.capacity = capacity;
        this.size = 0;
    }

    public synchronized void produce(int val) {
        try {
             // left 表示“想要生产的数量”(有可能生产量太多，需多此生产)
            int left = val;
            while (left > 0) {
                // 库存已满时，等待“消费者”消费产品。
                while (size >= capacity)
                    wait();
                // 获取“实际生产的数量”(即库存中新增的数量)
                // 如果“库存”+“想要生产的数量”>“总的容量”，则“实际增量”=“总的容量”-“当前容量”。(此时填满仓库)
                // 否则“实际增量”=“想要生产的数量”
                int inc = (size+left)>capacity ? (capacity-size) : left;
                size += inc;
                left -= inc;
                System.out.printf("%s produce(%3d) --> left=%3d, inc=%3d, size=%3d\n",
                        Thread.currentThread().getName(), val, left, inc, size);
                // 通知“消费者”可以消费了。
                notifyAll();
            }
        } catch (InterruptedException e) {
        }
    }

    public synchronized void consume(int val) {
        try {
            // left 表示“客户要消费数量”(有可能消费量太大，库存不够，需多此消费)
            int left = val;
            while (left > 0) {
                // 库存为0时，等待“生产者”生产产品。
                while (size <= 0)
                    wait();
                // 获取“实际消费的数量”(即库存中实际减少的数量)
                // 如果“库存”<“客户要消费的数量”，则“实际消费量”=“库存”；
                // 否则，“实际消费量”=“客户要消费的数量”。
                int dec = (size<left) ? size : left;
                size -= dec;
                left -= dec;
                System.out.printf("%s consume(%3d) <-- left=%3d, dec=%3d, size=%3d\n",
                        Thread.currentThread().getName(), val, left, dec, size);
                notifyAll();
            }
        } catch (InterruptedException e) {
        }
    }

    public String toString() {
        return "capacity:"+capacity+", actual size:"+size;
    }
}

// 生产者
class Producer {
    private Depot depot;

    public Producer(Depot depot) {
        this.depot = depot;
    }

    // 消费产品：新建一个线程向仓库中生产产品。
    public void produce(final int val) {
        new Thread() {
            public void run() {
                depot.produce(val);
            }
        }.start();
    }
}

// 消费者
class Customer {
    private Depot depot;

    public Customer(Depot depot) {
        this.depot = depot;
    }

    // 消费产品：新建一个线程从仓库中消费产品。
    public void consume(final int val) {
        new Thread() {
            public void run() {
                depot.consume(val);
            }
        }.start();
    }
}

public class Demo1 {
    public static void main(String[] args) {
        Depot mDepot = new Depot(100);
        Producer mPro = new Producer(mDepot);
        Customer mCus = new Customer(mDepot);

        mPro.produce(60);
        mPro.produce(120);
        mCus.consume(90);
        mCus.consume(150);
        mPro.produce(110);
    }
}
```














































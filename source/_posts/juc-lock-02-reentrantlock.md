---
title: J.U.C.锁机制 - Reentrantlock
date: 2016-05-10 10:45:44
categories: java-multi-thread
tags: [java, 多线程]
---

ReentrantLock是一个可重入的互斥锁，又被称为“独占锁”。

ReentrantLock锁在同一个时间点只能被一个线程锁持有；而可重入的意思是，ReentrantLock锁可以被单个线程多次获取。
ReentrantLock分为“公平锁”和“非公平锁”。它们的区别体现在获取锁的机制上是否公平。“锁”是为了保护竞争资源，防止多个线程同时操作线程而出错，ReentrantLock在同一个时间点只能被一个线程获取；ReentraantLock是通过一个FIFO的等待队列来管理获取该锁所有线程的。在“公平锁”的机制下，线程依次排队获取锁；而“非公平锁”在锁是可获取状态时，不管自己是不是在队列的开头都会获取锁。

<!-- more -->

### ReentrantLock函数如下：
```java
// 创建一个 ReentrantLock ，默认是“非公平锁”。
ReentrantLock()
// 创建策略是fair的 ReentrantLock。fair为true表示是公平锁，fair为false表示是非公平锁。
ReentrantLock(boolean fair)
// 查询当前线程保持此锁的次数。
int getHoldCount()
// 返回目前拥有此锁的线程，如果此锁不被任何线程拥有，则返回 null。
protected Thread getOwner()
// 返回一个 collection，它包含可能正等待获取此锁的线程。
protected Collection<Thread> getQueuedThreads()
// 返回正等待获取此锁的线程估计数。
int getQueueLength()
// 返回一个 collection，它包含可能正在等待与此锁相关给定条件的那些线程。
protected Collection<Thread> getWaitingThreads(Condition condition)
// 返回等待与此锁相关的给定条件的线程估计数。
int getWaitQueueLength(Condition condition)
// 查询给定线程是否正在等待获取此锁。
boolean hasQueuedThread(Thread thread)
// 查询是否有些线程正在等待获取此锁。
boolean hasQueuedThreads()
// 查询是否有些线程正在等待与此锁有关的给定条件。
boolean hasWaiters(Condition condition)
// 如果是“公平锁”返回true，否则返回false。
boolean isFair()
// 查询当前线程是否保持此锁。
boolean isHeldByCurrentThread()
// 查询此锁是否由任意线程保持。
boolean isLocked()
// 获取锁。如果锁不可用，出于线程调度目的，将禁用当前线程，并且在获得锁之前，该线程将一直处于休眠状态。
void lock()
// 如果当前线程未被中断，则获取锁。
void lockInterruptibly()
// 返回用来与此 Lock 实例一起使用的 Condition 实例。
Condition newCondition()
// 仅在调用时锁未被另一个线程保持的情况下，才获取该锁。
boolean tryLock()
// 如果锁在给定等待时间内没有被另一个线程保持，且当前线程未被中断，则获取该锁。
boolean tryLock(long timeout, TimeUnit unit)
// 试图释放此锁。
void unlock()
```


### ReentrantLock数据结构
ReentrantLock的UML类图
![](https://static.tmaczhao.cn/images/0dc838691dcbb1cc7880fcf152467314.jpg)
从图中可以看出：
- ReentrantLock实现了Lock接口。
- ReentrantLock与sync是组合关系。ReentrantLock中，包含了Sync对象；而且，Sync是AQS的子类；更重要的是，Sync有两个子类FairSync(公平锁)和NonFairSync(非公平锁)(都在ReentrantLock中)。ReentrantLock是一个独占锁，至于它到底是公平锁还是非公平锁，就取决于sync对象是"FairSync的实例"还是"NonFairSync的实例"。


### 公平锁-获取锁--lock()
```java
final void lock() {
    acquire(1);
}
```
**当前线程**是通过acquire(1)获取锁的。
>ps. 这里说明一下“1”的含义，它是设置“锁的状态”的参数。对于“独占锁”而言，锁处于可获取状态时，它的状态值是0；锁被线程初次获取到了，它的状态值就变成了1。

由于ReentrantLock(公平锁/非公平锁)是可重入锁，所以“独占锁”可以被单个线程多此获取，每获取1次就将锁的状态+1。也就是说，初次获取锁时，通过acquire(1)将锁的状态值设为1；再次获取锁时，将锁的状态值设为2；依次类推...这就是为什么获取锁时，传入的参数是1的原因了。
<font color="red">可重入就是指锁可以被单个线程多次获取。</font>

#### acquire()
acquire()是在AQS中实现的，源码如下：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
1. **当前线程**首先通过tryAcquire()尝试获取锁。获取成功的话，直接返回；尝试失败的话，进入到等待队列排序等待(前面还有可能有需要线程在等待该锁)。
2. **当前线程**尝试失败的情况下，先通过addWaiter(Node.EXCLUSIVE)来将“当前线程”加入到"CLH队列(非阻塞的FIFO队列)"末尾。CLH队列就是线程等待队列。
3. 然后调用acquireQueued()来获取锁。由于此时ReentrantLock是公平锁，它会根据<font color="red">公平性原则</font>来获取锁。
4. **当前线程**在执行acquireQueued()时，会进入到CLH队列中休眠等待，直到获取锁返回！如果**当前线程**在休眠等待过程中被中断过，acquireQueued会返回true，此时"当前线程"会调用selfInterrupt()来自己给自己产生一个中断。产生中断的原因见下文。

上面是对acquire()的概括性说明。下面，我们将该函数分为4部分来逐步解析。


### acquire()方法逐步分析
#### tryAcquire()
##### 公平锁的中tryAcquire()
在ReentrantLock.java的FairSync类中实现，源码如下：
```java
protected final boolean tryAcquire(int acquires) {
    // 获取“当前线程”
    final Thread current = Thread.currentThread();
    // 获取“独占锁”的状态
    int c = getState();
    // c=0意味着“锁没有被任何线程锁拥有”，
    if (c == 0) {
        // 若“锁没有被任何线程锁拥有”，
        // 则判断“当前线程”是不是CLH队列中的第一个线程线程，
        // 若是的话，则获取该锁，设置锁的状态，并切设置锁的拥有者为“当前线程”。
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 如果“独占锁”的拥有者已经为“当前线程”，
        // 则将更新锁的状态。
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```
##### hasQueuedPredecessors()
```java
public final boolean hasQueuedPredecessors() {
    Node t = tail;
    Node h = head;
    Node s;
    return h != t && ((s = h.next) == null || s.thread != Thread.currentThread());
}
```
hasQueuedPredecessors()是通过判断"当前线程"是不是在CLH队列的队首，来返回AQS中是不是有比“当前线程”等待更久的线程。下面对head、tail和Node进行说明。

##### Node
<font color="red">Node是CLH队列的节点，代表“等待锁的线程队列”。</font>
1. 每个Node都会一个线程对应。
2. 每个Node会通过prev和next分别指向上一个节点和下一个节点，这分别代表上一个等待线程和下一个等待线程。
3. Node通过属性waitStatus保存线程的等待状态。
4. Node通过nextWaiter来区分线程是“独占锁”线程还是“共享锁”线程。如果是“独占锁”线程，则nextWaiter的值为EXCLUSIVE；如果是“共享锁”线程，则nextWaiter的值是SHARED。


##### compareAndSetState()
compareAndSetState()在AQS中实现。它的源码如下：
```java
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```
compareAndSwapInt()是sun.misc.Unsafe类中的一个本地方法。对此，我们需要了解的是 compareAndSetState(expect, update)是以原子的方式操作当前线程；若当前线程的状态为expect，则设置它的状态为update。

##### setExclusiveOwnerThread()
setExclusiveOwnerThread()在AbstractOwnableSynchronizer.java中实现，它的源码如下：
```java
// exclusiveOwnerThread是当前拥有“独占锁”的线程
private transient Thread exclusiveOwnerThread;
protected final void setExclusiveOwnerThread(Thread t) {
    exclusiveOwnerThread = t;
}
```
setExclusiveOwnerThread()的作用是，设置线程t为当前拥有“独占锁”的线程。

##### getState(), setState()
getState()和setState()都在AQS中实现，源码如下：
```java
// 锁的状态
private volatile int state;
// 设置锁的状态
protected final void setState(int newState) {
    state = newState;
}
// 获取锁的状态
protected final int getState() {
    return state;
}
```
state表示锁的状态，对于“独占锁”，state=0表示锁是可获取状态(即，锁没有被任何线程锁持有)
>由于java中的独占锁是可重入的，state的值可以>1。

**tryAcquire()的作用就是让“当前线程”尝试获取锁。获取成功返回true，失败则返回false。**

#### addWaiter(Node.EXCLUSIVE)
addWaiter()在AQS中实现。
```java
private Node addWaiter(Node mode) {
    // 新建一个Node节点，节点对应的线程是“当前线程”，“当前线程”的锁的模型是mode。
    Node node = new Node(Thread.currentThread(), mode);
    Node pred = tail;
    // 若CLH队列不为空，则将“当前线程”添加到CLH队列末尾
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    // 若CLH队列为空，则调用enq()新建CLH队列，然后再将“当前线程”添加到CLH队列中。
    enq(node);
    return node;
}
```
>对于“公平锁”而言，addWaiter(Node.EXCLUSIVE)会首先创建一个Node节点，节点的类型是“独占锁”(Node.EXCLUSIVE)类型。然后，再将该节点添加到CLH队列的末尾。

##### compareAndSetTail()
compareAndSetTail()在AQS中实现，源码如下：
```java
private final boolean compareAndSetTail(Node expect, Node update) {
    //CAS函数，使用java native方法实现
    return unsafe.compareAndSwapObject(this, tailOffset, expect, update);
}
```
##### enq()
enq()在AQS中实现，源码如下：
```java
//如果CLH队列为空，则新建一个CLH表头；然后将node添加到CLH末尾。否则，直接将node添加到CLH末尾。
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

**addWaiter()的作用，就是将当前线程添加到CLH队列中。**


#### acquireQueued()
前面，我们已经将当前线程添加到CLH队列中了。acquireQueued()的作用就是逐步的去执行CLH队列的线程，如果当前线程获取到了锁，则返回；否则，当前线程进行休眠，直到唤醒并重新获取锁了才返回。
##### acquireQueued()
acquireQueued()在AQS中实现，源码如下：
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        // interrupted表示在CLH队列的调度中，
        // “当前线程”在休眠时，有没有被中断过。
        boolean interrupted = false;
        for (;;) {
            // 获取上一个节点。
            // node是“当前线程”对应的节点，这里就意味着“获取上一个等待锁的线程”。
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```
acquireQueued()的目的是从队列中获取锁。

##### shouldParkAfterFailedAcquire()
shouldParkAfterFailedAcquire()在AQS中实现，源码如下：
```java
// 返回“当前线程是否应该阻塞”
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 前继节点的状态
    int ws = pred.waitStatus;
    // 如果前继节点是SIGNAL状态，则意味这当前线程需要被unpark唤醒。此时，返回true。
    if (ws == Node.SIGNAL)
        return true;
    // 如果前继节点是“取消”状态，则设置 “当前节点”的“当前前继节点”为“‘原前继节点’的前继节点”。
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 如果前继节点为“0”或者“共享锁”状态，则设置前继节点为SIGNAL状态。
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```
关于waitStatus请参考下表(中扩号内为waitStatus的值)
>CANCELLED[1]  -- 当前线程已被取消
SIGNAL[-1]    -- “当前线程的后继线程需要被unpark(唤醒)”。一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
CONDITION[-2] -- 当前线程(处在Condition休眠状态)在等待Condition唤醒
PROPAGATE[-3] -- (共享锁)其它线程获取到“共享锁”
[0]           -- 当前线程不属于上面的任何一种状态。

shouldParkAfterFailedAcquire()通过以下规则，判断“当前线程”是否需要被阻塞

> 规则1：如果前继节点状态为SIGNAL，表明当前节点需要被unpark(唤醒)，此时则返回true。
规则2：如果前继节点状态为CANCELLED(ws>0)，说明前继节点已经被取消，则通过先前回溯找到一个有效(非CANCELLED状态)的节点，并返回false。
规则3：如果前继节点状态为非SIGNAL、非CANCELLED，则设置前继的状态为SIGNAL，并返回false。

如果“规则1”发生，即“前继节点是SIGNAL”状态，则意味着“当前线程”需要被阻塞。接下来会调用parkAndCheckInterrupt()阻塞当前线程，直到当前先被唤醒才从parkAndCheckInterrupt()中返回。

##### parkAndCheckInterrupt()
parkAndCheckInterrupt()在AQS中实现，源码如下：
```java
private final boolean parkAndCheckInterrupt() {
    // 通过LockSupport的park()阻塞“当前线程”。
    //unpark()方式唤醒,中断唤醒
    LockSupport.park(this);
    // 返回线程的中断状态。
    return Thread.interrupted();
}
```
parkAndCheckInterrupt()的作用是阻塞当前线程，并且返回“线程被唤醒之后”的中断状态。
它会先通过LockSupport.park()阻塞“当前线程”，然后通过Thread.interrupted()返回线程的中断状态。

##### 再次tryAcquire()
```java
final Node p = node.predecessor();
if (p == head && tryAcquire(arg)) {
    setHead(node);
    p.next = null; // help GC
    failed = false;
    return interrupted;
}
```
1. 通过node.predecessor()获取前继节点。predecessor()就是返回node的前继节点.
2. p == head && tryAcquire(arg)
>首先，判断“前继节点”是不是CHL表头。如果是的话，则通过tryAcquire()尝试获取锁。
其实，这样做的目的是为了“让当前线程获取锁”，先判断p==head是为了保证公平性！
>>(a) 首先，shouldParkAfterFailedAcquire()中判断了“当前线程”是否需要阻塞；
>>(b) 接着，“当前线程”阻塞的话，会调用parkAndCheckInterrupt()来阻塞线程。当线程被解除阻塞的时候，我们会返回线程的中断状态。而线程被解决阻塞，可能是由于线程被中断，也可能是由于其它线程调用了该线程的unpark()函数；
>>(c) 再回到p==head。如果当前线程是因为其它线程调用了unpark()函数而被唤醒，那么唤醒它的线程，应该是它的前继节点所对应的线程。此时，再来理解p==head就很简单了：当前继节点是CLH队列的头节点，并且它释放锁之后；就轮到当前节点获取锁了。然后，当前节点通过tryAcquire()获取锁；获取成功的话，通过setHead(node)设置当前节点为头节点，并返回。
如果当前线程是因为“线程被中断”而唤醒，且不做判断直接获取，则不满足公平性原则了。

**acquireQueued()会根据公平性原则进行阻塞等待，直到获取锁为止；并且返回当前线程在等待过程中有没有并中断过。**

#### selfInterrupt()
selfInterrupt()是AQS中实现，源码如下：
```java
private static void selfInterrupt() {
    Thread.currentThread().interrupt();
}
```
selfInterrupt()的代码很简单，就是“当前线程”自己产生一个中断。但是，为什么需要这么做呢？

这必须结合acquireQueued()进行分析。如果在acquireQueued()中，当前线程被中断过，则执行selfInterrupt()；否则不会执行。

在acquireQueued()中，即使是线程在阻塞状态被中断唤醒而获取到cpu执行权利；但是，如果该线程的前面还有其它等待锁的线程，根据公平性原则，该线程依然无法获取到锁。它会再次阻塞！该线程再次阻塞，直到该线程被它的前面等待锁的线程锁唤醒；线程才会获取锁，然后“真正执行起来”！
在该线程“成功获取锁并真正执行起来”之前，它的中断会被忽略并且中断标记会被清除！因为在parkAndCheckInterrupt()中，我们线程的中断状态时调用了Thread.interrupted()。interrupted()返回并且清除中断状态。
所以现在通过给自己加上中断标识处理中断。



### 公平锁-释放锁--unlock()
#### unlock()
unlock()在ReentrantLock中实现，源码如下：
```java
public void unlock() {
    sync.release(1);
}
```
unlock()是解锁函数，它是通过AQS的release()函数来实现的。
在这里，“1”的含义和“获取锁的函数acquire(1)的含义”一样，它是设置“释放锁的状态”的参数。由于“公平锁”是可重入的，所以对于同一个线程，每释放锁一次，锁的状态-1。

关于AQS, ReentrantLock 和 sync的关系如下：
```java
public class ReentrantLock implements Lock, java.io.Serializable {
    ...
    private final Sync sync;
    ...
    abstract static class Sync extends AbstractQueuedSynchronizer {
        ...
    }
    ...
}
```
sync是ReentrantLock.java中的成员对象，而Sync是AQS的子类。

#### sync.release()
release()在AQS中实现，源码如下：
```java
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
调用tryRelease()来尝试释放当前线程锁持有的锁。成功，则唤醒后继等待线程，并返回true。否则，直接返回false。

#### tryRelease()
tryRelease()在ReentrantLock.的Sync类中实现，源码如下：
```java
protected final boolean tryRelease(int releases) {
    // c是本次释放锁之后的状态
    int c = getState() - releases;
    // 如果“当前线程”不是“锁的持有者”，则抛出异常！
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 如果“锁”已经被当前线程彻底释放，则设置“锁”的持有者为null，即锁是可获取状态。
    if (c == 0) {
        free = true;
        //AbstractOwnableSynchronizer中定义
        setExclusiveOwnerThread(null);
    }
    // 设置当前线程的锁的状态。
    setState(c);
    return free;
}
```
#### unparkSuccessor()
在release()中“当前线程”释放锁成功的话，会唤醒当前线程的后继线程。
根据CLH队列的FIFO规则，“当前线程”即已经获取锁的线程肯定是head；如果CLH队列非空的话，则唤醒锁的下一个等待线程。
下面看看unparkSuccessor()的源码，它在AQS中实现。
```java
private void unparkSuccessor(Node node) {
    // 获取当前线程的状态
    int ws = node.waitStatus;
    // 如果状态<0，则设置状态=0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);
    //获取当前节点的“有效的后继节点”，无效的话，则通过for循环进行获取。
    //这里的有效，是指“后继节点对应的线程状态<=0”
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 唤醒“后继节点对应的线程”
    if (s != null)
        LockSupport.unpark(s.thread);
}
```
unparkSuccessor()的作用是“唤醒当前线程的后继线程”。后继线程被唤醒之后，就可以获取该锁并恢复运行了。


#### 总结
“释放锁”的过程相对“获取锁”的过程比较简单。释放锁时，主要进行的操作，是更新当前线程对应的锁的状态。如果当前线程对锁已经彻底释放，则设置“锁”的持有线程为null，设置当前线程的状态为空，然后唤醒后继线程。

### 非公平锁
**`lock()`**
```java
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```
lock()先通过compareAndSet()判断锁是不是空闲状态。是的话，当前线程直接获取锁；否则，调用acquire(1)获取锁。
>1. compareAndSetState()是CAS函数，它的作用是比较并设置当前锁的状态。若锁的状态值为0，则设置锁的状态值为1。
2. setExclusiveOwnerThread(Thread.currentThread())的作用是，设置“当前线程”为“锁”的持有者。

“公平锁”和“非公平锁”关于lock()的对比

- 公平锁的lock()函数，会直接调用acquire(1)。
- 非公平锁先判断当前锁的状态是不是空闲，是的话，就不排队，而是直接获取锁。

**`公平锁和非公平锁的区别`**，是在获取锁的机制上的区别。
- 公平锁，只有在当前线程是CLH等待队列的表头时，才获取锁；
- 非公平锁，只要当前锁处于空闲状态，则直接获取锁，而不管CLH等待队列中的顺序。
- 只有当非公平锁尝试获取锁失败的时候，它才会像公平锁一样，进入CLH等待队列排序等待。

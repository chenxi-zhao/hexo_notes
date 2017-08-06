JUC中的共享锁有CountDownLatch, CyclicBarrier, Semaphore, ReentrantReadWriteLock等
ReadWriteLock是读写锁。它维护了一对相关的锁-“读取锁”和“写入锁”，一个用于读取操作，另一个用于写入操作。
- “读取锁”用于只读操作，它是“共享锁”，能同时被多个线程获取。
- “写入锁”用于写入操作，它是“独占锁”，写入锁只能被一个线程锁获取。
注意：不能同时存在读取锁和写入锁！
ReadWriteLock是一个接口。ReentrantReadWriteLock是它的实现类，
**ReentrantReadWriteLock包括子类ReadLock和WriteLock。**

<!-- more -->

### ReadWriteLock函数列表
```java
// 返回用于读取操作的锁。
Lock readLock()
// 返回用于写入操作的锁。
Lock writeLock()
```
### ReentrantReadWriteLock函数列表
```java
// 创建一个新的 ReentrantReadWriteLock，默认是采用“非公平策略”。
ReentrantReadWriteLock()
// 创建一个新的 ReentrantReadWriteLock，fair是“公平策略”。fair为true，意味着公平策略；否则，意味着非公平策略。
ReentrantReadWriteLock(boolean fair)
// 返回当前拥有写入锁的线程，如果没有这样的线程，则返回 null。
protected Thread getOwner()
// 返回一个 collection，它包含可能正在等待获取读取锁的线程。
protected Collection<Thread> getQueuedReaderThreads()
// 返回一个 collection，它包含可能正在等待获取读取或写入锁的线程。
protected Collection<Thread> getQueuedThreads()
// 返回一个 collection，它包含可能正在等待获取写入锁的线程。
protected Collection<Thread> getQueuedWriterThreads()
// 返回等待获取读取或写入锁的线程估计数目。
int getQueueLength()
// 查询当前线程在此锁上保持的重入读取锁数量。
int getReadHoldCount()
// 查询为此锁保持的读取锁数量。
int getReadLockCount()
// 返回一个 collection，它包含可能正在等待与写入锁相关的给定条件的那些线程。
protected Collection<Thread> getWaitingThreads(Condition condition)
// 返回正等待与写入锁相关的给定条件的线程估计数目。
int getWaitQueueLength(Condition condition)
// 查询当前线程在此锁上保持的重入写入锁数量。
int getWriteHoldCount()
// 查询是否给定线程正在等待获取读取或写入锁。
boolean hasQueuedThread(Thread thread)
// 查询是否所有的线程正在等待获取读取或写入锁。
boolean hasQueuedThreads()
// 查询是否有些线程正在等待与写入锁有关的给定条件。
boolean hasWaiters(Condition condition)
// 如果此锁将公平性设置为 ture，则返回 true。
boolean isFair()
// 查询是否某个线程保持了写入锁。
boolean isWriteLocked()
// 查询当前线程是否保持了写入锁。
boolean isWriteLockedByCurrentThread()
// 返回用于读取操作的锁。
ReentrantReadWriteLock.ReadLock readLock()
// 返回用于写入操作的锁。
ReentrantReadWriteLock.WriteLock writeLock()
```

### ReentrantReadWriteLock数据结构
ReentrantReadWriteLock的UML类图如下：
![](http://static.tmaczhao.cn/images/5655a50f8ef3986bb52ded1bab729276.jpg)
从中可以看出：
1. ReentrantReadWriteLock实现了ReadWriteLock接口。ReadWriteLock是一个读写锁的接口，提供了"获取读锁的readLock()函数" 和 "获取写锁的writeLock()函数"。
2. ReentrantReadWriteLock中包含：**sync对象**，**读锁readerLock**和**写锁writerLock**。读锁ReadLock和写锁WriteLock都实现了Lock接口。读锁ReadLock和写锁WriteLock中也都分别包含了"Sync对象"，它们的Sync对象和ReentrantReadWriteLock的Sync对象 是一样的，就是通过sync，读锁和写锁实现了对同一个对象的访问。
3. 和"ReentrantLock"一样，sync是Sync类型；而且，Sync也是一个继承于AQS的抽象类。Sync也包括"公平锁"FairSync和"非公平锁"NonfairSync。sync对象是"FairSync"和"NonfairSync"中的一个，默认是"NonfairSync"。



### ReadLock/WriteLock
WriteLock就是一个独占锁，这和ReentrantLock里面的实现几乎相同，都是使用了AQS的acquire/release操作。当然了在内部处理方式上与ReentrantLock还是有一点不同的。
之前介绍到AQS中有一个state字段（int类型，32位）用来描述有多少线程获持有锁。在独占锁的时代这个值通常是0或者1（如果是重入的就是重入的次数），在共享锁的时代就是持有锁的数量。
ReadWriteLock的读、写锁是相关但是又不一致的，所以需要两个数来描述读锁（共享锁）和写锁（独占锁）的数量。显然现在一个state就不够用了。于是在ReentrantReadWrilteLock里面将这个字段一分为二，高位16位表示共享锁的数量，低位16位表示独占锁的数量（或者重入数量）。2^16-1=65536，这就是上节中提到的为什么共享锁和独占锁的数量最大只能是65535的原因了。

#### WriteLock
写入锁获取处理流程与独占锁类似，需要注意的是写入锁数量需要做特殊处理
>这里有一段逻辑是当前写线程是否需要阻塞writerShouldBlock(current)。对于非公平锁而言总是不阻塞当前线程，而对于公平锁而言如果AQS队列不为空或者当前线程不是在AQS的队列头那么就阻塞线程，直到队列前面的线程处理完锁逻辑。

写入锁释放时其实就是检测下剩下的写入锁数量，如果是0就将独占锁线程清空（意味着没有线程获取锁），否则就是说当前是重入锁的一次释放，所以不能将独占锁线程清空。然后将剩余线程状态数写回AQS。

#### ReadLock
读取锁获取锁时首先得到**`写入锁数量`**，如果不为0，返回-1，然后获取读取锁数量，如果“不需要阻塞等待”并且“读取锁的共享计数小于MAX_COUNT”，则直接通过CAS函数更新“读取锁的共享计数”，以及将“当前线程获取读取锁的次数+1”。如果CAS修改失败，会通过调用fullTryAcquireShared()方法循环执行一直到CAS计数成功或者被写入线程占有锁。

当上述方法中返回-1时会调用doAcquireShared()的作用是获取共享锁。
- 它会首先创建线程对应的CLH队列的节点，然后将该节点添加到CLH队列中。
- 如果“当前线程”是CLH队列的表头，则尝试获取共享锁；否则，则需要通过shouldParkAfterFailedAcquire()判断是否阻塞等待，需要的话，则通过parkAndCheckInterrupt()进行阻塞等待。
- doAcquireShared()会通过for循环，不断的进行上面的操作；目的就是获取共享锁。需要注意的是：doAcquireShared()在每一次尝试获取锁时，是通过tryAcquireShared()来执行的！

>ReadLock公平锁和非公平锁性通过是否被阻塞方法readerShouldBlock()方法来实现
公平共享锁中，如果在当前线程的前面有其他线程在等待获取共享锁，则返回true；否则，返回false。
非公平共享锁中，无视当前线程的前面是否有其他线程在等待获取共享锁。只要该共享锁对应的线程为null，则返回false。


### 示例代码
```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockTest1 {

    public static void main(String[] args) {
        // 创建账户
        MyCount myCount = new MyCount("4238920615242830", 10000);
        // 创建用户，并指定账户
        User user = new User("Tommy", myCount);

        // 分别启动3个“读取账户金钱”的线程 和 3个“设置账户金钱”的线程
        for (int i=0; i<3; i++) {
            user.getCash();
            user.setCash((i+1)*1000);
        }
    }
}

class User {
    private String name;            //用户名
    private MyCount myCount;        //所要操作的账户
    private ReadWriteLock myLock;   //执行操作所需的锁对象

    User(String name, MyCount myCount) {
        this.name = name;
        this.myCount = myCount;
        this.myLock = new ReentrantReadWriteLock();
    }

    public void getCash() {
        new Thread() {
            public void run() {
                myLock.readLock().lock();
                try {
                    System.out.println(Thread.currentThread().getName() +" getCash start");
                    myCount.getCash();
                    Thread.sleep(1);
                    System.out.println(Thread.currentThread().getName() +" getCash end");
                } catch (InterruptedException e) {
                } finally {
                    myLock.readLock().unlock();
                }
            }
        }.start();
    }

    public void setCash(final int cash) {
        new Thread() {
            public void run() {
                myLock.writeLock().lock();
                try {
                    System.out.println(Thread.currentThread().getName() +" setCash start");
                    myCount.setCash(cash);
                    Thread.sleep(1);
                    System.out.println(Thread.currentThread().getName() +" setCash end");
                } catch (InterruptedException e) {
                } finally {
                    myLock.writeLock().unlock();
                }
            }
        }.start();
    }
}

class MyCount {
    private String id;         //账号
    private int    cash;       //账户余额

    MyCount(String id, int cash) {
        this.id = id;
        this.cash = cash;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public int getCash() {
        System.out.println(Thread.currentThread().getName() +" getCash cash="+ cash);
        return cash;
    }

    public void setCash(int cash) {
        System.out.println(Thread.currentThread().getName() +" setCash cash="+ cash);
        this.cash = cash;
    }
}
```























































































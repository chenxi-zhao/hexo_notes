
**java.util.concurrent包完全建立在CAS之上的，没有CAS就不会有此包。可见CAS的重要性。**

## CAS
>CAS:Compare and Swap, 翻译成比较并交换。
java.util.concurrent包中借助CAS实现了区别于synchronouse同步锁的一种乐观锁。

本文先从CAS的应用说起，再深入原理解析。

### CAS应用
<font color="red">CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。</font>

**非阻塞算法 （nonblocking algorithms）**
>一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。

现代的CPU提供了特殊的指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而compareAndSet()就用这些代替了锁定。拿出AtomicInteger来研究在没有锁的情况下是如何做到数据正确性的。
```java
private volatile int value;
```

首先毫无以为，在没有锁的机制下可能需要借助volatile原语，保证线程间的数据是可见的（共享的）。

这样才获取变量的值的时候才能直接读取。

public final int get() {
        return value;
    }

然后来看看++i是怎么做到的。

public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}

在这里采用了CAS操作，每次从内存中读取数据然后将此数据和+1后的结果进行CAS操作，如果成功就返回结果，否则重试直到成功为止。

而compareAndSet利用JNI来完成CPU指令的操作。

public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }

整体的过程就是这样子的，利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。

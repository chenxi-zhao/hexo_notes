---
title: J.U.C. 原子操作
date: 2016-05-09 09:31:51
categories: java-multi-thread
tags: [java, 多线程]
---

根据修改的数据类型，可以将JUC包中的原子操作类可以分为4类。

1.**`基本类型`**: AtomicInteger, AtomicLong, AtomicBoolean ;
2.**`数组类型`**: AtomicIntegerArray, AtomicLongArray, AtomicReferenceArray ;
3.**`引用类型:`** AtomicReference, AtomicStampedRerence, AtomicMarkableReference ;
4.**`对象的属性修改类型`**: AtomicIntegerFieldUpdater, AtomicLongFieldUpdater, AtomicReferenceFieldUpdater 。

这些类存在的目的是对相应的数据进行原子操作。

<font color="red">**原子操作的描述是：多个线程执行一个操作时，其中任何一个线程要么完全执行完此操作，要么没有执行此操作的任何步骤，那么这个操作就是原子的。**</font>

**这些类主要都是使用CAS机制以及volatile关键字来保证操作原子性**

<!-- more -->

### AtomicInteger, AtomicLong和AtomicBoolean
这3个基本类型的原子类的原理和用法相似。以下以AtomicLong为例对基本类型的原子类进行介绍

AtomicLong是作用是对长整形进行原子操作。在32位操作系统中，64位的long和double变量由于会被JVM当作两个分离的32位来进行操作，所以不具有原子性。而使用AtomicLong能让long的操作保持原子型。

AtomicLong函数列表
```java
// 构造函数
AtomicLong()
// 创建值为initialValue的AtomicLong对象
AtomicLong(long initialValue)
// 以原子方式设置当前值为newValue。
final void set(long newValue)
// 获取当前值
final long get()
// 以原子方式将当前值减 1，并返回减1后的值。等价于“--num”
final long decrementAndGet()
// 以原子方式将当前值减 1，并返回减1前的值。等价于“num--”
final long getAndDecrement()
// 以原子方式将当前值加 1，并返回加1后的值。等价于“++num”
final long incrementAndGet()
// 以原子方式将当前值加 1，并返回加1前的值。等价于“num++”
final long getAndIncrement()
// 以原子方式将delta与当前值相加，并返回相加后的值。
final long addAndGet(long delta)
// 以原子方式将delta添加到当前值，并返回相加前的值。
final long getAndAdd(long delta)
// 如果当前值 == expect，则以原子方式将该值设置为update。成功返回true，否则返回false，并且不修改原值。
final boolean compareAndSet(long expect, long update)
// 以原子方式设置当前值为newValue，并返回旧值。
final long getAndSet(long newValue)
// 返回当前值对应的int值
int intValue()
// 获取当前值对应的long值
long longValue()
// 以 float 形式返回当前值
float floatValue()
// 以 double 形式返回当前值
double doubleValue()
// 最后设置为给定值。延时设置变量值，这个等价于set()方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能延时（尽管可以忽略），所以如果不是想立即读取设置的新值，允许在“后台”修改值，那么此方法就很有用。如果还是难以理解，这里就类似于启动一个后台线程如执行修改新值的任务，原线程就不等待修改结果立即返回（这种解释其实是不正确的，但是可以这么理解）。
final void lazySet(long newValue)
// 如果当前值 == 预期值，则以原子方式将该设置为给定的更新值。JSR规范中说：以原子方式读取和有条件地写入变量但不 创建任何 happen-before 排序，因此不提供与除 weakCompareAndSet 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是等效的，都调用了unsafe.compareAndSwapInt()完成操作。
final boolean weakCompareAndSet(long expect, long update)
```


### AtomicLongArray
AtomicLongArray的作用是对"长整形数组"进行原子操作。
```java
// 创建给定长度的新 AtomicLongArray。
AtomicLongArray(int length)
// 创建与给定数组具有相同长度的新 AtomicLongArray，并从给定数组复制其所有元素。
AtomicLongArray(long[] array)
// 以原子方式将给定值添加到索引 i 的元素。
long addAndGet(int i, long delta)
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean compareAndSet(int i, long expect, long update)
// 以原子方式将索引 i 的元素减1。
long decrementAndGet(int i)
// 获取位置 i 的当前值。
long get(int i)
// 以原子方式将给定值与索引 i 的元素相加。
long getAndAdd(int i, long delta)
// 以原子方式将索引 i 的元素减 1。
long getAndDecrement(int i)
// 以原子方式将索引 i 的元素加 1。
long getAndIncrement(int i)
// 以原子方式将位置 i 的元素设置为给定值，并返回旧值。
long getAndSet(int i, long newValue)
// 以原子方式将索引 i 的元素加1。
long incrementAndGet(int i)
// 最终将位置 i 的元素设置为给定值。
void lazySet(int i, long newValue)
// 返回该数组的长度。
int length()
// 将位置 i 的元素设置为给定值。
void set(int i, long newValue)
// 返回数组当前值的字符串表示形式。
String toString()
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean    weakCompareAndSet(int i, long expect, long update)
```

### AtomicReference
AtomicReference是作用是对"对象"进行原子操作。
```java
// volatile类型
private volatile V value;

// 使用 null 初始值创建新的 AtomicReference。
AtomicReference()
// 使用给定的初始值创建新的 AtomicReference。
AtomicReference(V initialValue)
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean compareAndSet(V expect, V update)
// 获取当前值。
V get()
// 以原子方式设置为给定值，并返回旧值。
V getAndSet(V newValue)
// 最终设置为给定值。
void lazySet(V newValue)
// 设置为给定值。
void set(V newValue)
// 返回当前值的字符串表示形式。
String toString()
// 如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。
boolean weakCompareAndSet(V expect, V update)
```
V是泛型，将对象使用AtomicReference包装在进行修改将具有原子性
```java
Object obj = new Object();
Object obj2 = new Object();
AtomicReference ar = new AtomicReference(obj);
ar.compareAndSet(obj, obj2);
```

### AtomicLongFieldUpdater
AtomicLongFieldUpdater可以对指定"类的'volatile long'类型的成员"进行原子更新。它是基于反射原理实现的。
#### 函数列表
```java
// 受保护的无操作构造方法，供子类使用。
protected AtomicLongFieldUpdater()
// 以原子方式将给定值添加到此更新器管理的给定对象的字段的当前值。
long addAndGet(T obj, long delta)
// 如果当前值 == 预期值，则以原子方式将此更新器所管理的给定对象的字段设置为给定的更新值。
abstract boolean compareAndSet(T obj, long expect, long update)
// 以原子方式将此更新器管理的给定对象字段当前值减 1。
long decrementAndGet(T obj)
// 获取此更新器管理的在给定对象的字段中保持的当前值。
abstract long get(T obj)
// 以原子方式将给定值添加到此更新器管理的给定对象的字段的当前值。
long getAndAdd(T obj, long delta)
// 以原子方式将此更新器管理的给定对象字段当前值减 1。
long getAndDecrement(T obj)
// 以原子方式将此更新器管理的给定对象字段的当前值加 1。
long getAndIncrement(T obj)
// 将此更新器管理的给定对象的字段以原子方式设置为给定值，并返回旧值。
long getAndSet(T obj, long newValue)
// 以原子方式将此更新器管理的给定对象字段当前值加 1。
long incrementAndGet(T obj)
// 最后将此更新器管理的给定对象的字段设置为给定更新值。
abstract void lazySet(T obj, long newValue)
// 为对象创建并返回一个具有给定字段的更新器。
static <U> AtomicLongFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName)
// 将此更新器管理的给定对象的字段设置为给定更新值。
abstract void set(T obj, long newValue)
// 如果当前值 == 预期值，则以原子方式将此更新器所管理的给定对象的字段设置为给定的更新值。
abstract boolean weakCompareAndSet(T obj, long expect, long update)
```
#### AtomicLongFieldUpdater示例
```java
// LongTest.java的源码
import java.util.concurrent.atomic.AtomicLongFieldUpdater;

public class LongFieldTest {
    public static void main(String[] args) {
        // 获取Person的class对象
        Class cls = Person.class;
        // 新建AtomicLongFieldUpdater对象，传递参数是“class对象”和“long类型在类中对应的名称”
        AtomicLongFieldUpdater mAtoLong = AtomicLongFieldUpdater.newUpdater(cls, "id");
        Person person = new Person(12345678L);
        // 比较person的"id"属性，如果id的值为12345678L，则设置为1000。
        mAtoLong.compareAndSet(person, 12345678L, 1000);
        System.out.println("id="+person.getId());
    }
}
class Person {
    volatile long id;
    public Person(long id) {
        this.id = id;
    }
    public void setId(long id) {
        this.id = id;
    }
    public long getId() {
        return id;
    }
}
```
#### newUpdater()方法源代码分析
```java
public static <U> AtomicLongFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName) {
    Class<?> caller = Reflection.getCallerClass();
    if (AtomicLong.VM_SUPPORTS_LONG_CAS)
        return new CASUpdater<U>(tclass, fieldName, caller);
    else
        return new LockedUpdater<U>(tclass, fieldName, caller);
}
```
该方法主要返回一个AtomicLongFieldUpdater类型的对象，用于设置对象中的long型attribute。
他返回CASUpdater还是LockedUpdater取决于JVM是否支持long型的CAS函数，这两个类都是AtomicLongFieldUpdater的子类。

从上述源代码我们可以知道，在JUC的原子类操作中，主要会使用<font color="red">CAS以及volatile</font>关键字来保证操作的原子性。


### 指令重排序

Java语言规范规定了JVM线程内部维持顺序化语义，也就是说只要程序的最终结果等同于它在严格的顺序化环境下的结果，那么指令的执行顺序就可能与代码的顺序不一致。这个过程通过叫做指令的重排序。指令重排序存在的意义在于：JVM能够根据处理器的特性（CPU的多级缓存系统、多核处理器等）适当的重新排序机器指令，使机器指令更符合CPU的执行特点，最大限度的发挥机器的性能。

程序执行最简单的模型是按照指令出现的顺序执行，这样就与执行指令的CPU无关，最大限度的保证了指令的可移植性。这个模型的专业术语叫做顺序化一致性模型。但是现代计算机体系和处理器架构都不保证这一点（因为人为的指定并不能总是保证符合CPU处理的特性）。

>在两个线程交替执行的情况下数据的结果就不确定了，在机器压力大，多核CPU并发执行的情况下，数据的结果将更加不确定。

### Happens-before法则

Java存储模型有一个happens-before原则，就是如果动作B要看到动作A的执行结果（无论A/B是否在同一个线程里面执行），那么A/B就需要满足happens-before关系。

在介绍happens-before法则之前介绍一个概念：JMM动作（Java Memeory Model Action），Java存储模型动作。一个动作（Action）包括：变量的读写、监视器加锁和释放锁、线程的start()和join()。

happens-before完整规则：

1. 同一个线程中的每个Action都happens-before于出现在其后的任何一个Action。
2. 对一个监视器的解锁happens-before于每一个后续对同一个监视器的加锁。
3. 对volatile字段的写入操作happens-before于每一个后续的同一个字段的读操作。
4. Thread.start()的调用会happens-before于启动线程里面的动作。
5. Thread中的所有动作都happens-before于其他线程检查到此线程结束或者Thread.join（）中返回或者Thread.isAlive()==false。
6. 一个线程A调用另一个另一个线程B的interrupt（）都happens-before于线程A发现B被A中断（B抛出异常或者A检测到B的isInterrupted（）或者interrupted()）。
7. 一个对象构造函数的结束happens-before与该对象的finalizer的开始
8. 如果A动作happens-before于B动作，而B动作happens-before与C动作，那么A动作happens-before于C动作。


### CAS 操作
>所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。

乐观锁用到的机制主要就有CAS，Compare and Swap。

CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

<font color="red">**非阻塞算法 （nonblocking algorithms）**</font>
>一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。

现代的CPU提供了特殊的指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。

拿出AtomicInteger来研究在没有锁的情况下是如何做到数据正确性的。

```java
private volatile int value;
//首先毫无以为，在没有锁的机制下可能需要借助volatile原语，保证线程间的数据是可见的（共享的）。
//这样才获取变量的值的时候才能直接读取。
public final int get() {
        return value;
    }
//然后来看看++i是怎么做到的。
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
//在这里采用了CAS操作，每次从内存中读取数据然后将此数据和+1后的结果进行CAS操作，如果成功就返回结果，否则重试直到成功为止。
//而compareAndSet利用JNI来完成CPU指令的操作。
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

整体的过程就是这样子的，利用CPU的CAS指令，同时借助JNI来完成Java的非阻塞算法。其它原子操作都是利用类似的特性完成的。

而整个J.U.C都是建立在CAS之上的，因此对于synchronized阻塞算法，J.U.C在性能上有了很大的提升。参考资料的文章中介绍了如果利用CAS构建非阻塞计数器、队列等数据结构。

<font color="red">CAS看起来很爽，但是会导致“ABA问题”。</font>

CAS算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。

>一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的。如果链表的头在变化了两次后恢复了原值，但是不代表链表就没有变化。因此前面提到的原子操作AtomicStampedReference/AtomicMarkableReference就很有用了。这允许一对变化的元素进行原子操作。




























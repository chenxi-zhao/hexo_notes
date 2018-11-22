---
title: Disruptor高性能无锁消息队列源码阅读
date: 2018-11-22 15:40:39
tags: [disruptor, 消息队列, 高性能, 并发]
categories: java
---

### disruptor 结构
![disruptor](https://static.tmaczhao.cn/notes/disruptor/20181122143846.jpg)

<!-- more -->

### 伪共享
![false sharing](http://static.tmaczhao.cn/images/180314/EKBHhjh9ic.png)

`value值始终独占一个缓存行，极限条件下为第一个或者最后一个元素，防止更新其他值时重刷导致性能下降`

![false sharing](https://static.tmaczhao.cn/notes/disruptor/2018112214396.jpg)



### Sequence - AtomicLong增强（缓存行填充）
```java
public class Sequence extends RhsPadding{
    static final long INITIAL_VALUE = -1L;
    private static final Unsafe UNSAFE;
    private static final long VALUE_OFFSET;

    static{
        UNSAFE = Util.getUnsafe();
        try{
            VALUE_OFFSET = UNSAFE.objectFieldOffset(Value.class.getDeclaredField("value"));
        }catch (final Exception e){
            throw new RuntimeException(e);
        }
    }

    /**
     * 默认初始value为-1
     */
    public Sequence(){
        this(INITIAL_VALUE);
    }

    public Sequence(final long initialValue){
        UNSAFE.putOrderedLong(this, VALUE_OFFSET, initialValue);
    }

    public long get(){
        return value;
    }

    /**
     * 利用Unsafe更新value的地址内存上的值从而更新value的值
     */
    public void set(final long value){
        UNSAFE.putOrderedLong(this, VALUE_OFFSET, value);
    }

    /**
     * 利用Unsafe原子更新value
     */
    public void setVolatile(final long value){
        UNSAFE.putLongVolatile(this, VALUE_OFFSET, value);
    }

    /**
     * 利用Unsafe CAS
     */
    public boolean compareAndSet(final long expectedValue, final long newValue){
        return UNSAFE.compareAndSwapLong(this, VALUE_OFFSET, expectedValue, newValue);
    }


    public long incrementAndGet(){
        return addAndGet(1L);
    }

    public long addAndGet(final long increment){
        long currentValue;
        long newValue;

        // java1.8可以做进一步优化 UNSAFE.getAndAddLong-native fetch-and-add
        do{
            currentValue = get();
            newValue = currentValue + increment;
        }
        while (!compareAndSet(currentValue, newValue));

        return newValue;
    }

    @Override
    public String toString(){
        return Long.toString(get());
    }
}
```


### Producer
![mark](http://static.tmaczhao.cn/images/180314/g72FkH5dfa.png)

#### Cursored接口
实现此接口的类，记录sequence位置的类。例如，生产者在生产消息时通过访问getCursor来定位当前ringBuffer下一个生产的位置，这个位置需要实时更新。

#### Sequenced接口
实现此接口类，实现一个有序的存储结构，也就是RingBuffer的一个特性。
 - getBufferSize 获取ringBuffer的大小
 - hasAvailableCapacity 判断空间是否足够
 - remainingCapacity 获取ringBuffer的剩余空间
 - next 申请下一个或者n个sequence(value)作为生产event的位置
 - tryNext 尝试申请下一个或者n个sequence(value)作为生产event的位置，容量不足会抛出InsufficientCapacityException
 - publish 发布Event

#### Sequencer接口
Sequencer接口，扩展了Cursored和Sequenced接口。在前两者的基础上，增加了消费与生产相关的方法。
 - INITIAL_CURSOR_VALUE： -1 为 sequence的起始值
 - claim： 申请一个特殊的Sequence，只有设定特殊起始值的ringBuffer时才会使用（一般是多个生产者时才会使用）
 - isAvailable：非阻塞，验证一个sequence是否已经被published并且可以消费
 - addGatingSequences：将这些sequence加入到需要跟踪处理的gatingSequences中
 - removeGatingSequence：移除某个sequence
 - newBarrier：给定一串需要跟踪的sequence，创建SequenceBarrier。SequenceBarrier是用来给多消费者确定消费位置是否可以消费用的
 - getMinimumSequence：获取这个ringBuffer的gatingSequences中最小的一个sequence
 - getHighestPublishedSequence：获取最高可以读取的Sequence
 - newPoller：目前没用，不讲EventPoller相关的内容（没有用到）

>**GatingSequence**：RingBuffer的头由一个名字为cursor的Sequence对象维护，用来协调生产者向RingBuffer中填充数据。表示队列尾的Sequence并没有在RingBuffer中，而是由消费者维护。这样的话，队列尾的维护就是无锁的。但是，在生产者方确定RingBuffer是否已满就需要跟踪更多信息。为此，GatingSequence用来跟踪相关Sequence。

#### AbstractSequencer
AbstractSequencer实现了Sequencer接口，有五个Field：

```java
// 原子更新下面的gatingSequences
private static final AtomicReferenceFieldUpdater<AbstractSequencer, Sequence[]> SEQUENCE_UPDATER =AtomicReferenceFieldUpdater.newUpdater(AbstractSequencer.class, Sequence[].class, "gatingSequences");
// 记录生产目标RingBuffer大小，bufferSize在构造方法构造时必须大于1且为2的n次方
protected final int bufferSize;
// 该生产者的等待策略，当队列满了以后
protected final WaitStrategy waitStrategy;
// 生产定位，即可以消费的Sequence，初始值为 INITIAL_CURSOR_VALUE：-1
protected final Sequence cursor = new Sequence(Sequencer.INITIAL_CURSOR_VALUE);
// 生产队列的队列头有RingBuffer维护，下面的对象用于维护消费者的队列尾
protected volatile Sequence[] gatingSequences = new Sequence[0];
```

该类实现了上面接口中的一些方法：

```java
/**
 * 获取当前生产者的定位
 */
@Override
public final long getCursor(){
    return cursor.get();
}

/**
 * 拿到生产队列的大小
 */
@Override
public final int getBufferSize(){
    return bufferSize;
}

/**
 * 原子添加Sequence到gatingSequences数组中
 */
@Override
public final void addGatingSequences(Sequence... gatingSequences){
    SequenceGroups.addSequences(this, SEQUENCE_UPDATER, this, gatingSequences);
}

/**
 * 原子删除Sequence到gatingSequences数组中
 */
@Override
public boolean removeGatingSequence(Sequence sequence){
    return SequenceGroups.removeSequence(this, SEQUENCE_UPDATER, sequence);
}

/**
 * 获取到正在消费的最小的序列值，即消费的最小的Sequence value
 */
@Override
public long getMinimumSequence(){
    return Util.getMinimumSequence(gatingSequences, cursor.get());
}

/**
 * 返回的是一个ProcessingSequenceBarrier，用来协调消费者消费的对象。
 * 例如消费者A依赖于消费者B，就是消费者A一定要后于消费者B消费，也就是A只能消费B消费过的，也就是A的sequence一定要小于B的。
 * 这个Sequence的协调，通过A和B设置在同一个SequenceBarrier上实现。同时，我们还要保证所有的消费者只能消费被Publish过的。
 */
@Override
public SequenceBarrier newBarrier(Sequence... sequencesToTrack){
    return new ProcessingSequenceBarrier(this, waitStrategy, cursor, sequencesToTrack);
}

/**
 * Creates an event poller for this sequence that will use the supplied data provider and
 * gating sequences.
 */
@Override
public <T> EventPoller<T> newPoller(DataProvider<T> dataProvider, Sequence... gatingSequences){
    return EventPoller.newInstance(dataProvider, this, new Sequence(), cursor, gatingSequences);
}
```

#### SingleProducerSequencer（线程不安全）
```java
// 下一个生产的位置， nextValue在next后更改，cursor在publish更改
protected long nextValue = Sequence.INITIAL_VALUE;
// 缓存之前所有gatingSequences最小的那个，这样不用每次都遍历一遍gatingSequences，影响效率
protected long cachedValue = Sequence.INITIAL_VALUE;
```

```java
/**
 * @see Sequencer#hasAvailableCapacity(int)
 */
@Override
public boolean hasAvailableCapacity(int requiredCapacity){
    return hasAvailableCapacity(requiredCapacity, false);
}

private boolean hasAvailableCapacity(int requiredCapacity, boolean doStore){
    // 下一个生产位置
    long nextValue = this.nextValue;

    // 后退一圈拿到上一个同一位置，用于判断是否消费
    long wrapPoint = (nextValue + requiredCapacity) - bufferSize;
    // 拿到缓存到的gatingSequences中最小的value
    long cachedGatingSequence = this.cachedValue;

    //只要wrapPoint大于缓存的所有gatingSequences最小的那个，就重新检查更新缓存
    if (wrapPoint > cachedGatingSequence || cachedGatingSequence > nextValue){
        if (doStore)        {
            cursor.setVolatile(nextValue);  // StoreLoad fence
        }
        // 检查缓存
        long minSequence = Util.getMinimumSequence(gatingSequences, nextValue);
        this.cachedValue = minSequence;

        // 回退一圈的元素没有被消费，不能覆盖，所以没有足够可用容量
        if (wrapPoint > minSequence){
            return false;
        }
    }

    return true;
}

/**
 * @see Sequencer#next()
 */
@Override
public long next(){ return next(1); }

/**
 * 申请n个Sequence(value)用于生产event的位置
 */
@Override
public long next(int n){
    if (n < 1){
        throw new IllegalArgumentException("n must be > 0");
    }
    long nextValue = this.nextValue;

    // 方法逻辑与hasAvailableCapacity大致相同，这里主要是需要阻塞等待可用的sequence
    long nextSequence = nextValue + n;
    long wrapPoint = nextSequence - bufferSize;
    long cachedGatingSequence = this.cachedValue;

    if (wrapPoint > cachedGatingSequence || cachedGatingSequence > nextValue){
        cursor.setVolatile(nextValue);  // StoreLoad fence

        long minSequence;
        while (wrapPoint > (minSequence = Util.getMinimumSequence(gatingSequences, nextValue))){
            LockSupport.parkNanos(1L); // TODO: Use waitStrategy to spin? 根据等待策略自旋
        }

        this.cachedValue = minSequence;
    }

    this.nextValue = nextSequence;

    return nextSequence;
}

/**
 * @see Sequencer#tryNext()
 */
@Override
public long tryNext() throws InsufficientCapacityException { return tryNext(1); }

/**
 * 尝试申请n个sequence(value)，容量不足会抛出InsufficientCapacityException。
 * 容量检查，使用hasAvailableCapacity方法检查
 */
@Override
public long tryNext(int n) throws InsufficientCapacityException{
    if (n < 1){
        throw new IllegalArgumentException("n must be > 0");
    }

    if (!hasAvailableCapacity(n, true)){
        throw InsufficientCapacityException.INSTANCE;
    }

    long nextSequence = this.nextValue += n;

    return nextSequence;
}

/**
 * 获取剩余的容量大小， 为负数表示消费被生产套圈
 */
@Override
public long remainingCapacity(){
    long nextValue = this.nextValue;

    long consumed = Util.getMinimumSequence(gatingSequences, nextValue);
    long produced = nextValue;
    return getBufferSize() - (produced - consumed);
}

/**
 * Claim a specific sequence.  Only used if initialising the ring buffer to a specific value.
 */
@Override
public void claim(long sequence){ this.nextValue = sequence; }

/**
 * 发布Event
 */
@Override
public void publish(long sequence){
    // cursor代表可以消费的sequence
    cursor.set(sequence);
    waitStrategy.signalAllWhenBlocking();
}
@Override
public void publish(long lo, long hi){ publish(hi); }

@Override
public boolean isAvailable(long sequence){
    return sequence <= cursor.get();
}

@Override
public long getHighestPublishedSequence(long lowerBound, long availableSequence){
    return availableSequence;
}
```




#### MultiProducerSequencer
```java
// 用于Unsafe类获取和设置值
private static final Unsafe UNSAFE = Util.getUnsafe();
private static final long BASE = UNSAFE.arrayBaseOffset(int[].class);
private static final long SCALE = UNSAFE.arrayIndexScale(int[].class);

// 是gatingSequence的缓存，和之前的单一生产者的cacheValue类似，由于是Sequence对象，所以不需要做缓存填充
private final Sequence gatingSequenceCache = new Sequence(Sequencer.INITIAL_CURSOR_VALUE);
// 每个槽存储ringbuffer中当前槽消费了几次，也就是Sequence转了多少圈，下标就是槽
// availableBuffer数组初始化每个值都为-1
private final int[] availableBuffer;
// indexMask=bufferSize - 1 >> sequence % 2^n = sequence & （2^n - 1）
private final int indexMask;
// 上面的n，用来定位某个sequence到底转了多少圈  sequence / 2^n == sequence >>> n
private final int indexShift;
```

```java
// 检查是否有足够容量的逻辑与单生产类似
public boolean hasAvailableCapacity(final int requiredCapacity){
    return hasAvailableCapacity(gatingSequences, requiredCapacity, cursor.get());
}

private boolean hasAvailableCapacity(Sequence[] gatingSequences, final int requiredCapacity, long cursorValue){
    // 加上期望容量后回退一圈的位置
    long wrapPoint = (cursorValue + requiredCapacity) - bufferSize;
    // 当前缓存的最小待消费的位置
    long cachedGatingSequence = gatingSequenceCache.get();
    // 如果还没消费到
    if (wrapPoint > cachedGatingSequence || cachedGatingSequence > cursorValue){
        // 从实时数据中拿出最小消费的sequence value 并更新缓存
        long minSequence = Util.getMinimumSequence(gatingSequences, cursorValue);
        gatingSequenceCache.set(minSequence);

        if (wrapPoint > minSequence){
            return false;
        }
    }
    return true;
}

/**
 * 申请Sequence
 */
@Override
public long next(){ return next(1); }

@Override
public long next(int n){
    if (n < 1){
        throw new IllegalArgumentException("n must be > 0");
    }

    long current;
    long next;

    // 逻辑与单生产类似，但是这里多线程竞争资源时，通过自旋CAS设置期望值，cursor表示请求到的sequence
    do{
        current = cursor.get();
        next = current + n;

        long wrapPoint = next - bufferSize;
        long cachedGatingSequence = gatingSequenceCache.get();

        if (wrapPoint > cachedGatingSequence || cachedGatingSequence > current){
            long gatingSequence = Util.getMinimumSequence(gatingSequences, current);

            if (wrapPoint > gatingSequence){
                LockSupport.parkNanos(1); // TODO, should we spin based on the wait strategy?
                continue;
            }
            // 更新最小消费到的sequence缓存
            gatingSequenceCache.set(gatingSequence);
        }else if (cursor.compareAndSet(current, next)){
            break;
        }
    }while (true);

    return next;
}

/**
 * 发布指定sequence的event
 */
@Override
public void publish(final long sequence){
    setAvailable(sequence);
    waitStrategy.signalAllWhenBlocking();
}

/**
 * 发布范围内的sequence
 */
@Override
public void publish(long lo, long hi){
    for (long l = lo; l <= hi; l++){
        setAvailable(l);
    }
    waitStrategy.signalAllWhenBlocking();
}

/**
 * 后面的方法用于设置availableBuffer中的标记
 * <p>
 * 使用该标记数组的主要原因是在多个发布线程中共享一个sequence（比如说单生产者中的nextValue）
 * calculateIndex(sequence) 计算槽的index
 * calculateAvailabilityFlag(sequence) 计算已经被生产多少圈
 * <p>
 */
private void setAvailable(final long sequence){
    setAvailableBufferValue(calculateIndex(sequence), calculateAvailabilityFlag(sequence));
}
private void setAvailableBufferValue(int index, int flag){
    long bufferAddress = (index * SCALE) + BASE;
    UNSAFE.putOrderedInt(availableBuffer, bufferAddress, flag);
}

/**
 * @see Sequencer#isAvailable(long)
 */
@Override
public boolean isAvailable(long sequence){
    int index = calculateIndex(sequence);
    int flag = calculateAvailabilityFlag(sequence);
    long bufferAddress = (index * SCALE) + BASE;
    return UNSAFE.getIntVolatile(availableBuffer, bufferAddress) == flag;
}

@Override
public long getHighestPublishedSequence(long lowerBound, long availableSequence){
    for (long sequence = lowerBound; sequence <= availableSequence; sequence++)    {
        if (!isAvailable(sequence)){
            return sequence - 1;
        }
    }

    return availableSequence;
}

private int calculateAvailabilityFlag(final long sequence){
    return (int) (sequence >>> indexShift);
}

private int calculateIndex(final long sequence){
    // x / 2^n == x >>> n
    return ((int) sequence) & indexMask;
}
```




### 环形无锁队列RingBuffer
![mark](http://static.tmaczhao.cn/images/180314/kI2D3GH0d6.png)

>RingBuffer类中保存了整个RingBuffer每个槽（entry或者slot）的Event对象，对应的field是private final Object[] entries；这些对象只在RingBuffer初始化时被建立，之后就是修改这些对象（初始化Event和填充Event），并不会重新建立新的对象。RingBuffer可以有多生产者和消费者，所以这个entries会被多线程访问频繁的，但不会修改（因为不会重新建立新的对象，这个数组保存的是对对象的具体引用，所以不会变）

#### 运算取模
>m % 2^n = m & ( 2^n - 1 )   //RingBuffer中数组大小必须为2的倍数

#### 源码
```java
abstract class RingBufferFields<E> extends RingBufferPad{
    // 数组前或后需要填充的对象数量
    private static final int BUFFER_PAD;
    //Buffer数组起始基址
    private static final long REF_ARRAY_BASE;
    //2^n=每个数组对象引用所占空间，这个n就是REF_ELEMENT_SHIFT
    private static final Unsafe UNSAFE = Util.getUnsafe();

    static{
        // Object数组引用长度，32位为4字节，64位为8字节
        final int scale = UNSAFE.arrayIndexScale(Object[].class);
        if (4 == scale){
            REF_ELEMENT_SHIFT = 2;
        } else if (8 == scale){
            REF_ELEMENT_SHIFT = 3;
        } else {
            throw new IllegalStateException("Unknown pointer size");
        }

        //需要填充128字节，缓存行长度一般是128字节
        BUFFER_PAD = 128 / scale;
        // Including the buffer pad in the array base offset， 偏移128个字节
        REF_ARRAY_BASE = UNSAFE.arrayBaseOffset(Object[].class) + (BUFFER_PAD << REF_ELEMENT_SHIFT);
    }

    private final long indexMask;
    private final Object[] entries;
    protected final int bufferSize;
    protected final Sequencer sequencer;

    RingBufferFields(EventFactory<E> eventFactory, Sequencer sequencer){
        this.sequencer = sequencer;
        this.bufferSize = sequencer.getBufferSize();

        if (bufferSize < 1){
            throw new IllegalArgumentException("bufferSize must not be less than 1");
        }
        if (Integer.bitCount(bufferSize) != 1){
            throw new IllegalArgumentException("bufferSize must be a power of 2");
        }

        this.indexMask = bufferSize - 1;
        /**
         * 结构：缓存行填充，避免频繁访问的任一entry与另一被修改的无关变量写入同一缓存行
         * --------------
         * *   数组头   * BASE
         * *   Padding  * 128字节
         * * reference1 * SCALE
         * * reference2 * SCALE
         * * reference3 * SCALE
         * ..........
         * *   Padding  * 128字节
         * --------------
         */
        this.entries = new Object[sequencer.getBufferSize() + 2 * BUFFER_PAD];
        fill(eventFactory);
    }

    // 从填充偏移位置开始初始化数据
    private void fill(EventFactory<E> eventFactory){
        for (int i = 0; i < bufferSize; i++)
        {
            entries[BUFFER_PAD + i] = eventFactory.newInstance();
        }
    }

    // 从偏移位置开始获取数据
    @SuppressWarnings("unchecked")
    protected final E elementAt(long sequence){
        return (E) UNSAFE.getObject(entries, REF_ARRAY_BASE + ((sequence & indexMask) << REF_ELEMENT_SHIFT));
    }
}
```

- RingBuffer中方法大部分都是在包装EventTranslator以及Sequencer中的方法。
- 构造一个RingBuffer需要如下元素：实现EventFactory的Event的工厂，实现Sequencer的生产者，等待策略waitStrategy还有bufferSize。

> EventTranslator的作用主要是发布Event，还有EventTranslatorTwoArg，EventTranslatorVararg等有类似作用



### WaitStrategy
![waitStrategy](https://static.tmaczhao.cn/notes/disruptor/20181122153916.png)

- BlockingWaitStrategy：通过线程阻塞的方式，等待生产者唤醒
- BusySpinWaitStrategy：线程一直自旋等待，比较耗CPU。
- LiteBlockingWaitStrategy：通过线程阻塞的方式，等待生产者唤醒，比BlockingWaitStrategy要轻，某些情况下可以减少阻塞的次数。
- PhasedBackoffWaitStrategy：根据指定的时间段参数和指定的等待策略决定采用哪种等待策略。
- SleepingWaitStrategy：可通过参数设置，使线程通过Thread.yield()主动放弃执行，通过线程调度器重新调度；或一直自旋等待。
- TimeoutBlockingWaitStrategy：通过参数设置阻塞时间，如果超时则抛出异常。
- YieldingWaitStrategy： 通过Thread.yield()主动放弃执行，通过线程调度器重新调度。

#### waitStrategy接口
```java
public interface WaitStrategy {
    /**
     * @param sequence 需要等待available的sequence（需要消费的sequence）
     * @param cursor 对应RingBuffer的Cursor（生产的sequence）
     * @param dependentSequence 需要等待（依赖）的Sequence（sequence依赖）
     * @param barrier 多消费者注册的SequenceBarrier
     * @return 已经available的sequence
     * @throws AlertException
     * @throws InterruptedException
     * @throws TimeoutException
     */
    long waitFor(long sequence, Sequence cursor, Sequence dependentSequence, SequenceBarrier barrier)
            throws AlertException, InterruptedException, TimeoutException;

    /**
     * 唤醒所有等待的消费者
     */
    void signalAllWhenBlocking();
}
```


#### BlockingWaitStrategy
```java
public final class BlockingWaitStrategy implements WaitStrategy{
    private final Lock lock = new ReentrantLock();
    private final Condition processorNotifyCondition = lock.newCondition();

    @Override
    public long waitFor(long sequence, Sequence cursorSequence, Sequence dependentSequence, SequenceBarrier barrier)
        throws AlertException, InterruptedException{
        long availableSequence;
        // 生产位置小于消费位置，阻塞等待
        if (cursorSequence.get() < sequence){
            lock.lock();
            try{
                while (cursorSequence.get() < sequence){
                    //检查是否Alert，如果Alert，则抛出AlertException
                    //Alert在这里代表对应的消费者被halt停止了
                    barrier.checkAlert();
                    //在processorNotifyCondition上等待唤醒
                    processorNotifyCondition.await();
                }
            } finally {
                lock.unlock();
            }
        }

        while ((availableSequence = dependentSequence.get()) < sequence){
            barrier.checkAlert();
        }

        return availableSequence;
    }

    // BusySpinWaitStrategy等待策略，一直自旋
    @Override
    public long waitFor(final long sequence, Sequence cursor, final Sequence dependentSequence, final SequenceBarrier barrier)
            throws AlertException, InterruptedException {

        long availableSequence;
        //一直while自旋检查
        while ((availableSequence = dependentSequence.get()) < sequence) {
            barrier.checkAlert();
        }
        return availableSequence;
    }

    @Override
    public void signalAllWhenBlocking(){
        lock.lock();
        try{
            //生产者生产消息后，会唤醒所有等待的消费者线程
            processorNotifyCondition.signalAll();
        }finally{
            lock.unlock();
        }
    }
}
```


### SequenceBarrier
![mark](http://static.tmaczhao.cn/images/180314/f3329eC4jf.png)

SequenceBarrier只有一个实现类，就是ProcessingSequenceBarrier。
ProcessingSequenceBarrier由生产者Sequencer，生产定位cursorSequence，等待策略waitStrategy还有一组依赖sequence：dependentSequence组成：


```java
final class ProcessingSequenceBarrier implements SequenceBarrier{
    private final WaitStrategy waitStrategy;
    private final Sequence dependentSequence;
    private volatile boolean alerted = false;
    private final Sequence cursorSequence;
    private final Sequencer sequencer;

    ProcessingSequenceBarrier(final Sequencer sequencer, final WaitStrategy waitStrategy,
        final Sequence cursorSequence,
        final Sequence[] dependentSequences){
        this.sequencer = sequencer;
        this.waitStrategy = waitStrategy;
        this.cursorSequence = cursorSequence;
        // 如果没有依赖，则直接等于消费的sequence
        if (0 == dependentSequences.length){
            dependentSequence = cursorSequence;
        }else{
            dependentSequence = new FixedSequenceGroup(dependentSequences);
        }
    }

    @Override
    public long waitFor(final long sequence) throws AlertException, InterruptedException, TimeoutException {
        checkAlert();

        //通过等待策略获取下一个可消费的sequence，这个sequence通过之前的讲解可以知道，
        // 需要大于cursorSequence和dependentSequence，我们可以通过dependentSequence实现先后消费
        long availableSequence = waitStrategy.waitFor(sequence, cursorSequence, dependentSequence, this);
        //等待可能被中断，所以检查下availableSequence是否小于sequence
        if (availableSequence < sequence) {
            return availableSequence;
        }
        //如果不小于，返回所有sequence（可能多生产者）和availableSequence中最大的
        return sequencer.getHighestPublishedSequence(sequence, availableSequence);
    }

    @Override
    public long getCursor(){
        return dependentSequence.get();
    }

    @Override
    public boolean isAlerted(){
        return alerted;
    }

    @Override
    public void alert(){
        alerted = true;
        waitStrategy.signalAllWhenBlocking();
    }

    @Override
    public void clearAlert(){
        alerted = false;
    }

    @Override
    public void checkAlert() throws AlertException{
        if (alerted)
        {
            throw AlertException.INSTANCE;
        }
    }
}
```
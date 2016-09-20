### GC日志参数
-verbose:gc
-XX:+PrintGC  输出GC日志
-XX:+PrintGCDetails  输出GC的详细日志
-XX:+PrintGCTimeStamps  输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps  输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC  在进行GC的前后打印出堆的信息
-XX:+PrintGCApplicationStoppedTime  //输出GC造成应用暂停的时间
-XX:+UseGCLogFileRotation  启用GC日志文件的自动转储 (Since Java)
-XX:+PrintTenuringDistribution

-XX:+CITime：打印消耗在JIT编译的时间
-XX:ErrorFile=./hs_err_pid.log：保存错误日志或者数据到指定文件中
-XX:+ExtendedDTraceProbes：开启solaris特有的dtrace探针
-XX:HeapDumpPath=./java_pid.hprof：指定导出堆信息时的路径或文件名
-XX:+HeapDumpOnOutOfMemoryError：当首次遭遇内存溢出时导出此时堆中相关信息
-XX:OnError=";"：出现致命ERROR之后运行自定义命令
-XX:OnOutOfMemoryError=";"：当首次遭遇内存溢出时执行自定义命令

-XX:-PrintClassHistogram：遇到Ctrl-Break后打印类实例的柱状信息，与jmap -histo功能相同
-XX:-PrintConcurrentLocks：遇到Ctrl-Break后打印并发锁的相关信息，与jstack -l功能相同
-XX:-PrintCommandLineFlags：打印在命令行中出现过的标记
-XX:-PrintCompilation：当一个方法被编译时打印相关信息

-XX:-TraceClassLoading：跟踪类的加载信息
-XX:-TraceClassLoadingPreorder：跟踪被引用到的所有类的加载信息
-XX:-TraceClassResolution：跟踪常量池
-XX:-TraceClassUnloading：跟踪类的卸载信息
-XX:-TraceLoaderConstraints：跟踪类加载器约束的相关信息

-server -XX:+DoEscapeAnalysis -XX:+EliminateLocks: 逃逸分析，锁消除

-Xloggc:../logs/gc.log  日志文件的输出路径
-XX:NumberOfGClogFiles=1  GC日志文件的循环数目 (Since Java)
-XX:GCLogFileSize=1M  控制GC日志文件的大小 (Since Java)

>-XX:+PrintGC包含-verbose:gc
-XX:+PrintGCDetails //包含-XX:+PrintGC
只要设置-XX:+PrintGCDetails 就会自动带上-verbose:gc和-XX:+PrintGC


### JVM内存分配参数
#### 堆分配
-Xms：初始堆大小，默认为物理内存的1/64（小于1GB）；默认（MinHeapFreeRatio参数可以调整）GC后空余堆内存比例小于40%时，则放大堆内存的预估最大值，但不超过固定最大值。

-Xmx：最大堆大小，默认(MaxHeapFreeRatio参数可以调整)GC后空余堆内存比例大于70%时，JVM会减少堆直到-Xms的最小限制

-Xmn：新生代的内存空间大小，注意：此处的大小是（eden+ 2 survivor space)。与jmap -heap中显示的New gen是不同的。整个堆大小=新生代大小+老生代大小。在保证堆大小不变的情况下，增大新生代后，将会减小老生代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。

-XX:NewSize/-XX:MaxNewSize

-XX:NewRation：老年代（不包括永久代）和新生代（Eden+2*survivor）的比值，4则表示新生代：老年代=1:4

-XX:SurvivorRatio：新生代中Eden区域与Survivor区域的容量比值，默认值为8。两个Survivor区与一个Eden区的比值为2:8，一个Survivor区占整个年轻代的1/10。

-XX:YoungGenerationSizeIncrement=（Y）：YoungGen分配新内存时的增长比例，默认是20%
-XX:TenuredGenerationSizeIncrement=（Y）：TenuredGen空间分配新内存时的增长比例，默认是20%
-XX:AdaptiveSizeDecrementScaleFactor=（Y）：空间缩小比例，如果空间增长比例是X，那么缩小比例是X/D

-Xss：每个线程的堆栈大小。JDK5以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。应根据应用的线程所需内存大小进行适当调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。一般小的应用，如果栈不是很深，应该是128k够用的，大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:"-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了。

#### 永久代分配
-XX:PermSize：设置永久代(perm gen)初始值。默认值为物理内存的1/64。
-XX:MaxPermSize：设置持久代最大值。物理内存的1/4。
>jdk1.8中取消了永久代，取而代之的是metaspace，元空间存储在native memory即本地内存中

#### GC参数配置
-XX:-DisableExplicitGC：禁止调用System.gc()，但JVM的gc仍然有效。
-Xnoclassgc：禁用垃圾回收
-XX:+ScavengeBeforeFullGC：新生代GC优先于Full GC执行。
-XX:+CollectGen0First：FullGC时是否先YGC，默认false
-XX:TargetSurvivorRatio=90：允许90%的Survivor空间被占用（默认为50%）。提高对于Survivor的使用率—超过就会尝试垃圾回收。
-XX:PretenureSizeThreshold：对象超过多大是直接在旧生代分配默认为0，单位字节，新生代采用Parallel Scavenge GC时无效，另一种直接在旧生代分配的情况是大的数组对象，且数组中无外部引用对象。

-XX:MaxTenuringThreshold：垃圾最大年龄，如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活 时间，增加在年轻代即被回收的概率该参数只有在串行GC时才有效。

-XX:LargePageSizeInBytes：内存页的大小不可设置过大，会影响Perm的大小（128M）
-XX:TLABWasteTargetPercent：TLAB占eden区的百分比，默认1%

-XX:+AggressiveOpts：加快编译
-XX:+MaxFDLimit：最大化文件描述符的数量限制。
-XX:+UseBiasedLocking：锁机制的性能改善，偏向锁。默认启用，激烈竞争的时候会增加系统负担
-XX:+UseBiasedLockingStartupDelay=0：jvm启动时启用偏向锁。

-XX:+UseFastAccessorMethods：启用原始类型的getter方法优化
-XX:+UseThreadPriorities：启用本地线程优先级API，即使 java.lang.Thread.setPriority() 生效，反之无效。
-XX:SoftRefLRUPolicyMSPerMB=0：“软引用”的对象在最后一次被访问后能存活0毫秒（默认为1秒）。
-XX:+StringCache：默认启用。启用字符串缓存。

### 串行收集器（Serial/Serial Old）
>-XX:+UseSerialGC
新生代、老年代、持久代串行回收，新生代复制算法，老年代标记压缩

串行回收器是稳定，也是效率最高的垃圾收集器，由于串行单线程收集，所以会产生较长时间停顿。

串行回收期间应用程序线程全部暂停，只有一个GC线程执行回收完成后重启应用程序线程


### 并行收集器
#### ParNew(Serial收集器多线程版本)
>默认新生代并行、老年代和持久代串行，新生代复制算法，老年代标记压缩

-XX:+UseParNewGC：设置年轻代为并行收集，可与CMS收集同时使用，JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值

#### Parallel Scavenge/Old
>默认新生代并行、老年代串行，新生代复制算法，老年代标记压缩

-XX:+UseParallelGC：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
-XX:+UseParallelOldGC：年老代垃圾收集方式为并行收集（Parallel Compacting)，这个是JAVA 6出现的参数选项

-XX:+UseAdaptiveSizePolicy：自动选择年轻代区大小和相应的Survivor区比例，设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

-XX:MaxGCPauseMillis：每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。

-XX:GCTimeRatio：设置垃圾回收时间占程序运行时间的百分比，公式为1/(1+n)，默认99

-XX:ParallelGCThreads：并行收集器的线程数，此值最好配置与处理器数目相等。同样适用于CMS


### CMS收集器（Concurrent Mark Sweep， 并发标记清除）
>- 新生代并行GC、老年代并发GC
- 用于老年代的垃圾收集器，与应用程序一起执行，停顿时间减少，降低吞吐量

-XX:+UseConcMarkSweepGC：使用CMS内存收集。

1. 运行过程：
- 初始标记：标记根直接关联的对象速度快，会产生停顿
- 并发标记：主要标记过程，和应用线程一起执行，标记全部对象
- 重新标记：修正标记结果，因为之前与应用线程一起执行，产生停顿
- 并发清除：基于标记结果，清理垃圾对象，与应用线程一起执行
- 并发重置

2. 特点
- 尽可能降低停顿
- 会影响系统整体吞吐量和性能（并发分走CPU性能）
- 清理不彻底（并发清理时继续产生垃圾）
- 与用户线程一起运行，不能在空间快满时清理,预留空间不够产生concurrent mode failure，失败以后开启串行收集器
- 标记清除算法会产生大量的内存碎片

#### CMS参数配置
-XX:+AggressiveHeap：试图是使用大量的物理内存，长时间大内存使用的优化，能检查计算资源（内存，处理器数量）。至少需要256MB内存，大量的CPU／内存（在1.4.1在4CPU的机器上已经显示有提升）

-XX:+CMSParallelRemarkEnabled：降低标记停顿
-XX:+UseCMSInitiatingOccupancyOnly：使用手动定义初始化定义开始CMS收集，禁止hostspot自行触发CMS GC
-XX:ParallelCMSThreads：设置CMS线程数量，一般情况下CPU数量

-XX:CMSFullGCsBeforeCompaction：多少次后进行内存整理，由于并发收集器不对内存空间进行压缩，整理，所以运行一段时间以后会产生"碎片"，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩，整理。
-XX+UseCMSCompactAtFullCollection：在FULL GC的时候，对年老代的压缩，CMS是不会移动内存的，因此，这个非常容易产生碎片，导致内存不够用，因此，内存的压缩这个时候就会被启用。增加这个参数是个好习惯。可能会影响性能,但是可以消除碎片

-XX:CMSInitiatingOccupancyFraction=70：使用cms作为垃圾回收，使用70％后开始CMS收集，为了保证不出现promotion failed错误,该值的设置需要满足CMSInitiatingOccupancyFraction计算公式
-XX:CMSInitiatingPermOccupancyFraction：设置Perm Gen使用到达多少比率时触发

-XX:+CMSIncrementalMode：设置为增量模式     用于单CPU情况
-XX:+CMSClassUnloadingEnabled：允许对类元数据进行回收

### 垃圾收集器组合方式
| | 新生代GC方式 | 老年代和持久代GC方式 |
| --- | --- | --- |
-XX:+UseSerialGC | Serial串行GC | Serial-Old串行GC |
-XX:+UseParNewGC | ParNew并行GC | Serial Old串行GC |
-XX:+UseParallelGC | Parallel Scavenge并行回收GC | Serial-Old并行GC |
-XX:+UseParallelOldGC | Parallel Scavenge并行回收GC | Parallel Old并行GC |
-XX:+UseConcMarkSweepGC | ParNew 并行GC | CMS并发GC，当出现“Concurrent Mode Failure”时，采用Serial Old串行GC |
-XX:+UseConcMarkSweepGC -XX:+UseParNewGC | Serial串行GC | 同上 |

### G1收集器
G1（Garbage-First）收集器是现今收集器技术的最新成果之一，之前一直处于实验阶段，直到jdk7u4之后，才正式作为商用的收集器。

与前几个收集器相比，G1收集器有以下特点：
- 并行与并发
- 分代收集（仍然保留了分代的概念）
- 空间整合（整体上属于“标记-整理”算法，不会导致空间碎片）
- 可预测的停顿（比CMS更先进的地方在于能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒）
此外，G1收集器将Java堆划分为多个大小相等的Region（独立区域），新生代与老年代都是一部分Region的集合，G1的收集范围则是这一个个Region

G1的工作过程如下：
- 初始标记（Initial Marking）
- 并发标记（Concurrent Marking）
- 最终标记（Final Marking）
- 筛选回收（Live Data Counting and Evacuation）
初始标记阶段仅仅只是标记一下GC Roots能够直接关联的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段的用户程序并发运行的时候，能在正确可用的Region中创建对象，这个阶段需要暂停线程。并发标记阶段从GC Roots进行可达性分析，找出存活的对象，这个阶段与用户线程并发执行的。最终标记阶段则是修正在并发标记阶段因为用户程序的并发执行而导致标记产生变动的那一部分记录，这部分记录被保存在Remembered Set Logs中，最终标记阶段再把Logs中的记录合并到Remembered Set中，这个阶段是并行执行的，仍然需要暂停用户线程。最后在筛选阶段首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间制定回收计划。整个执行过成功如下：
![](http://7xkjk9.com1.z0.glb.clouddn.com/jvm-13.jpg)

### GC算法
- 引用计数法：为对象设置引用计数器，计数为零的时候可回收（引用有加法减法、影响性能；循环引用难处理）（java未使用）

- 标记清除法：标记阶段（通过根节点开始标记可达对象和垃圾对象）和清除阶段（清除垃圾对象）

- 标记压缩法：标记阶段同上，但在清理对象时将所有存活对象压缩到内存一端，然后清理边界外的空间

- 复制算法：内存空间分为两块，垃圾回收时复制存活对象到空间内存（大对象及经历GC次数多的对象进入老年代，年轻对象进入复制空间），然后清理当前内存空间，交换两块内存的角色。复制算法不适用于存活对象多的空间，如老年代。另外复制算法主要的问题是空间浪费。

>分代思想
根据对象存活周期进行分类，短命对象为新生代，长命对象归为老年代
- 少量对象存活，适合复制算法
- 大量对象存活，适合标记清理或这标记压缩

### 可触及性
- 可触及对象：从根节点可以触及到该对象
- 可复活对象：对象引用被释放，就是可复活对象（在finalize中可能复活该对象）
- 不可触及对象：在finalize后可能进入不可触及状态，不可能复活，可以回收

### 根
- 栈中引用的对象
- 方法区中静态成员或者常量引用的对象（全局对象）
- JNI方法栈中引用对象

### Stop-The-World
Java中一种全局暂停的现象,全局停顿，所有Java代码停止，native代码可以执行，但不能和JVM交互。

1. Stop-The-World多半由于GC引起
- Dump线程
- 死锁检查
- 堆Dump

2. GC时为什么会有全局停顿？
>类比在聚会时打扫房间，聚会时很乱，又有新的垃圾产生，房间永远打扫不干净，只有让大家停止活动了，才能将房间打扫干净。

3. 危害
- 长时间服务停止，没有响应
- 遇到HA系统，可能引起主备切换，严重危害生产环境。




















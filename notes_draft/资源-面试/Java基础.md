#### String类为什么是final的
String不可变性，String类用final修饰，但可以通过反射方式改变String中的值。String不变性主要是为了线程安全，String常量池的使用可以节省内存空间，提高效率。

在jdk6当调用substring方法的时候，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset的值是不同的。如果你有一个很长很长的字符串，但是当你使用substring进行切割的时候你只需要很短的一段。但是你却引用了整个字符串（因为这个非常长的字符数组一直在被引用，所以无法被回收，就可能导致内存泄露）。在jdk7中，substring方法会在堆内存中创建一个新的数组。

>jdk1.6常量池放在方法区，jdk1.7常量池放在堆内存，jdk1.8放在元空间里面，和堆相独立。所以导致string的intern方法因为以上变化在不同版本会有不同表现。


#### HashMap的源码，实现原理，底层结构
HashMap基于哈希表的Map接口的实现。此实现提供所有可选的映射操作，并允许使用null值和null键。（除了不同步和允许使用null之外，HashMap类与Hashtable大致相同。）null值hashcode为0，所以以null为键的键值对存储在table[0]。

HashMap在1.6中使用链表(Entry<K,V>)加数组(Entry数组)的方式实现，HashMap中主要是通过key的hashCode来计算hash值的，只要hashCode相同，计算出来的hash值就一样。如果发生hash冲突通过链表解决。HashMap中的负载因子loadFactor默认为0.75，resize过程中一般保证为大于initialCapacity的最小的2的n次幂（一般容量初始为16，之后每次乘2）。
>JDK 1.6 当数量大于容量 * 负载因子即会扩充容量。
JDK 1.7 初次扩充为：当数量大于容量时扩充；第二次及以后为：当数量大于容量 * 负载因子时扩充。
JDK 1.8 初次扩充为：与负载因子无关；第二次及以后为：与负载因子有关。

JDK1.8中使用位桶+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树。
![](http://images2015.cnblogs.com/blog/616953/201603/616953-20160304192851940-1880633940.png)


#### 说说你知道的几个Java集合类：list、set、queue、map实现类
![](http://pic002.cnblogs.com/images/2012/80896/2012053020261738.gif)


#### 描述一下ArrayList和LinkedList各自实现和区别
1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2. 对于随机访问get和set，ArrayList优于LinkedList，因为LinkedList要移动指针。
3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。
4. 无论ArrayList还是LinkedList，遍历建议使用foreach，尤其是数据量较大时LinkedList避免使用get遍历。


#### Java中的队列都有哪些，有什么区别
##### 普通队列
LinkedList：双向链表实现的LinkedList既是List，也是Queue。它是唯一一个允许放入null的Queue。

ArrayDeque：以循环数组实现的双向Queue。大小是2的倍数，默认是16。

PriorityQueue：用二叉堆实现的优先级队列。按元素实现的Comparable接口或传入Comparator的比较结果来出队，数值越小，优先级越高，越先出队。

##### 线程安全的队列
ConcurrentLinkedQueue/ConcurrentLinkedDeque：无界的并发优化的Queue，基于链表，实现了依赖于CAS的无锁算法。

PriorityBlockingQueue：无界的并发优化的PriorityQueue，也是基于二叉堆。使用一把公共的读写锁。虽然实现了BlockingQueue接口，其实没有任何阻塞队列的特征，空间不够时会自动扩容。

DelayQueue：内部包含一个PriorityQueue，同样是无界的。元素需实现Delayed接口，每次调用时需返回当前离触发时间还有多久，小于0表示该触发了。pull()时会用peek()查看队头的元素，检查是否到达触发时间。ScheduledThreadPoolExecutor用了类似的结构。

##### 线程安全的阻塞队列
ArrayBlockingQueue
LinkedBlockingQueueLinkedBlockingDeque


#### 反射中，Class.forName和classloader的区别
class.forName和classLoader都可用来对类进行加载。前者除了将类的class文件加载到jvm中之外，还会对类进行解释，执行类中的static块。而classLoader只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象。


#### Java7、Java8的新特性
[java5、java6、java7、java8的新特性][1]
[1]:[http://blog.csdn.net/qq_30641447/article/details/49853067]


#### Java数组和链表两种结构的操作效率，在哪些情况下(从开头开始，从结尾开始，从中间开始)，哪些操作(插入，查找，删除)的效率高
- 数组静态分配内存，链表动态分配内存；
- 数组在内存中连续，链表不连续；
- ~~数组元素在栈区，链表元素在堆区；~~ 数组也存储在堆区
- 数组利用下标定位，时间复杂度为O(1)，链表定位元素时间复杂度O(n)；
- 数组插入或删除元素的时间复杂度O(n)，链表的时间复杂度O(1)。


#### Java内存泄露的问题调查定位：jmap，jstack的使用



#### String、StringBuilder、StringBuffer区别
String不变性，StringBuffer和StringBuilder可变，但是StringBuffer线程安全，StringBuilder线程不安全。


#### hashtable和hashmap的区别
HashTable线程安全，HashMap允许null键。


#### 异常的结构，运行时异常和非运行时异常，各举个例子
运行时异常都是RuntimeException类及其子类异常，如NullPointerException、IndexOutOfBoundsException等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

非运行时异常是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。


#### String类的常用方法
substring,split,indexOf,charAt,start/endWith,toLower/UpperCase,replace,matches,trim


#### Java的引用类型有哪几种
强引用、软引用、弱引用和虚引用。
- JVM会持有一般对象直到他们不再是可触及的状态。换句话说，当没有任何有效引用指向他们的时候会被垃圾回收，无效引用不会被计算在内。
- 软引用指向的对象会在不存在任何指向他们的引用并且内存空间不足情况下被垃圾收集。大多数情况下被用来实现内存敏感的缓存。没有GC的时间限制，会在OOM发生之前清理完毕。
- 弱引用指向的对象会在没有任何引用指向他们的时候立即被垃圾收集。如果一个对象只有弱引用的话，那么这个对象是不可触及的。这些对象会在任何时候被垃圾收集并且会在下一个GC周期里被丢弃。
- 虚引用指向的是已经执行finalize方法，但是还没有回收内存的对象。


#### 抽象类和接口的区别
- 抽象类的成员可以具有访问级别,而接口的成员全部public级别
- 抽象类可以包含字段,而接口不可以,
- 抽象类可以继承接口,而接口不能继承抽象类搜索
- 抽象类的成员可以具有具体实现,而接口不行
- 抽象的子类可以选择性实现其基类的抽象方法,而接口的子类必须实现
- java8里边接口可以通过default关键字定义接口实现

####. java的基础类型和字节大小。
- Int: 4 字节 Short: 2字节 Long: 8字节
- Byte: 1字节 Character: 2字节
- Float: 4字节 Double: 8字节
- 引用：4个字节，Object：8个字节。空对象至少12个字节。

#### Hashtable,HashMap,ConcurrentHashMap底层实现原理与线程安全问题
HashMap线程不安全，HashTable通过对主要的方法如put，get加入关键字synchronized进行修饰保证其线程安全。

ConcurrentHashMap在jdk1.7中使用segment进行分段，每个segment包含了一个Entry的table，segment使用ReentrantLock加锁保证其线程安全
![](http://images2015.cnblogs.com/blog/764863/201606/764863-20160620202714522-1795796503.png)
而在JDK1.8中取消了segment，直接采用transient volatile Node<K,V>[] table 保存数据，对应的数据操作则使用CAS + 同步的方式进行加锁。


#### 如果不让你用Java Jdk提供的工具，你自己实现一个Map，你怎么做。
数据的添加、修改、删除、查询基本操作，什么样的Map（键怎么存？Hash、红黑树等）。


#### Hash冲突怎么办？哪些解决散列冲突的方法？
线性探测、线性补偿探测、伪随机探测、再散列等。


#### HashMap冲突很厉害，最差性能，你会怎么解决
JDK1.8中在冲突节点长度超过8时将链表转为红黑树存储提高性能


#### hashCode() 与 equals() 生成算法、方法怎么重写
HashMap先根据hashcode进行分桶，在判断equals方式是否相等。
>- 如果两个对象相等，那么他们一定有相同的哈希值（hash code）。
- 如果两个对象的哈希值相等，那么这两个对象有可能相等也有可能不相等。（需要再通过equals来判断）

#### Java中的length和length()
- 数组是一个容器对象(http://blog.csdn.net/zhangjg_blog/article/details/16116613)，其中包含固定数量的同一类型的值。一旦数组被创建，他的长度就是固定的了。数组的长度可以作为final实例变量的长度。因此，长度可以被视为一个数组的属性。

- String背后的数据结构是一个char数组,所以没有必要来定义一个不必要的属性（因为该属性在char数值中已经提供了）。和C不同的是，Java中char的数组并不等于字符串，虽然String的内部机制是char数组实现的。


#### Java中的Switch对整型、字符型、字符串型的具体实现细节
- switch对int的判断是直接比较整数的值。
- 对char类型进行比较的时候，实际上比较的是ascii码，编译器会把char型变量转换成对应的int型变量。
- 字符串的switch是通过equals()和hashCode()方法来实现的

>其实swich只支持一种数据类型，那就是整型，其他数据类型都是转换成整型之后在使用switch的。


#### Java的各种打包方式（JAR/WAR/EAR/CAR）
##### JAR (Java Archive file)
>包含内容：class、properties文件，是文件封装的最小单元；包含Java类的普通库、资源（resources）、辅助文件（auxiliary files）等
部署文件 ： application-client.xml
容器： 应用服务器（application servers）
级别：小


##### WAR (Web Archive file)
>包含内容：Servlet、JSP页面、JSP标记库、JAR库文件、HTML/XML文档和其他公用资源文件，如图片、音频文件等
部署文件 ： web.xml
容器： 小型服务程序容器（servlet containers）
级别：中

##### EAR（Enterprise Archive file）
>包含内容：除了包含JAR、WAR以外，还包括EJB组件
部署文件 ： application.xml
容器： EJB容器（EJB containers）
级别： 大

##### car包(webx特有的打包方式)
>传统的web工程就是将工程打包成一个war包部署到web服务器上就可以运行web服务。
Webx工程是以car包为单位，一个工程可以打包为一个car包，多个car包可以打包成一个war包部署到web服务器上。
这样做的好处不言而喻就是可以将一个大工程分解为多个小工程独立去开发部署。


#### Java中Comparable和Comparator
- 同一个类的两个对象之间要想比较，对应的类就要实现Comparable接口，并实现compareTo()方法

- 在一些情况下，你不希望修改一个原有的类，但是你还想让他可以比较，Comparator接口可以实现这样的功能。通过使用Comparator接口，你可以针对其中特定的属性/字段来进行比较。比如，当我们要比较两个人的时候，我可能通过年龄比较、也可能通过身高比较。这种情况使用Comparable就无法实现（因为要实现Comparable接口，其中的compareTo方法只能有一个，无法实现多种比较）。

>Java中的Collections和Arrays中都包含排序的sort方法，该方法可以接收一个Comparator的实例（比较器）来进行排序。


#### 组合和集成的优缺点比较
|组合关系 | 继承关系 |
|优点：不破坏封装，整体类与局部类间松耦合，彼此相对独立|缺点：破坏封装，子类与父类之间紧密耦合，子类依赖父类的实现，子类缺乏独立性|
|优点：具有较好的可扩展性  | 缺点：支持扩展，但是往往以增加系统结构的复杂度为代价 |
|优点：支持动态组合。在运行时，整体对象可以选择不同类型的局部对象 |   缺点：不支持动态继承。在运行时，子类无法选择不同的父类 |
|优点：整体类可以对局部类进行包装，封装局部类的接口，提供新的接口  |  缺点：子类不能改变父类的接口 |
|缺点：整体类不能自动获得和局部类同样的接口  |  优点：子类能自动继承父类的接口 |
|缺点：创建整体类的对象时，需要创建所有局部类的对象  | 优点：创建子类的对象时，无须创建父类的对象 |

#### 类型擦除
>类型擦除指的是通过类型参数合并，将泛型类型实例关联到同一份字节码上。编译器只为泛型类型生成一份字节码，并将其实例关联到这份字节码上。

1. 虚拟机中没有泛型，只有普通类和普通方法,所有泛型类的类型参数在编译时都会被擦除,泛型类并没有自己独有的Class类对象。比如并不存在List<String>.class或是List<Integer>.class，而只有List.class。
2. 创建泛型对象时请指明类型，让编译器尽早的做参数检查（Effective Java，第23条：请不要在新代码中使用原生态类型）
3. 不要忽略编译器的警告信息，那意味着潜在的ClassCastException等着你。
4. 静态变量是被泛型类的所有实例所共享的。对于声明为MyClass<T>的类，访问其中的静态变量的方法仍然是MyClass.myStaticVar。不管是通过new MyClass<String>还是new MyClass<Integer>创建的对象，都是共享一个静态变量。
5.泛型的类型参数不能用在Java异常处理的catch语句中。因为异常处理是由JVM在运行时刻来进行的。由于类型信息被擦除，JVM是无法区分两个异常类型MyException<String>和MyException<Integer>的。对于JVM来说，它们都是MyException类型的。也就无法执行与异常对应的catch语句。


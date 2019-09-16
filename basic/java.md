# String在常量池中
相同的String在常量池中只有一个，但是不同情况下，String的存储是不同的。

<img src="/Users/linzhang/Documents/Java/knowledge/basic/image/string.png" alt="avatar" style="zoom:25%;" />

# Java的序列化是什么？有哪些你知道的序列化方式？
Java的序列化是把内存中的数据输出为可存储的格式。可以将数据序列化为二进制，xml，json等等格式。被transient修饰的不可被序列化

# New 一个HashMap，向其中添加Long，Boolean数据，主要是手机号是否可用的数据。添加一条，添加两条，一直到添加1000w条。在这个过程中，HashMap的数据结构时怎么变化的？1000w个数据添加完之后，HashMap占用了多少内存？
HashMap在最开始的时候是数组+链表的结构，新的hash会被存储到hash表中，hash碰撞发生了，会放在链表中。碰撞增多以后，就会转变为数组+红黑树的结构。数组的大小跟设置的hash算法相关。
Long是64位，Boolean是32位。那么一条记录就是(64+32)/8=12个byte. 那么1000万条无重复的记录就是12000万个byte，总共114M。

# 1000w甚至1亿个手机号过滤重复，可以使用哪些方式/哪些数据类型？各有什么特点？
1亿个手机号就是1个G。可以使用布隆过滤器来过滤重复数据。优点是快速，缺点是不够精确。结果是没有，那么肯定没有；结果是有，不一定有。
也可以利用redis集群，普通的key value datatype即可。Redis会对key做一致性hash，手机号这样的数据会均匀地分布在redis集群中，读取很高效。

# ConcurrentHashMap是如何做到高并发线程安全？
ConcurrentHashMap是读写锁。而且写锁是分段的所以性能稍好。

# Java中Synchronized关键字的内存语义是什么？
当线程释放锁时，把线程本地内存中的数据刷到主内存；
当线程获得锁时，线程本地内存无效，导致线程必须从主内存中获取数据。

# Java中Volatile关键字的内存语义是什么？
当一个线程在执行数据更改时，首先从主内存中拿到数据，刷到线程的本地内存中，操作完毕后再刷回主内存，在多线程下会导致数据不一致。Volatile修饰的变量，当被某一个线程修改后，会通知其它线程该变量在线程本地内存中无效，迫使其它线程使用该变量前，重新读取主内存。
要注意的是，volatile只保证可见性，但是不是原子性的。所以多线程修改volatile修饰的变量，最终的结果并不一致。
Volatile的用处：比如在AtomicInt的IncreaseAndGet方法里，先获取volatile修饰的值，然后进行CAS，如果CAS不成功就再获取，直到CAS成功为止。

# 什么是Java中原子性操作？
就是在操作系统层面无法分割的操作，是原子的。比如get，set，CAS等

# 什么是Java中的CAS操作,AtomicLong实现原理？
Compare and set。操作系统的原子操作，如果值比较结果相等，那么就设置新值，否则退出。
用volatile修饰long变量，首先获取值，然后进行CAS操作；如果CAS成功，退出；如果不成功，重新获取long值，再CAS，不成功就循环往复。

# 什么是可重入锁、乐观锁、悲观锁、公平锁、非公平锁、独占锁、共享锁？
可重入锁指的是当前线程对锁修饰的代码块是可以重入的，synchronized和reentrancelock都是可重入锁。
乐观锁是，对比设定的值，如果不同，就停止；如果相同，就继续。
悲观锁是，当某一个线程获得锁，其它线程必须等待锁释放，才能进行。
公平锁是，等待最多时间的线程，最先获得锁。
非公平锁是，等待的线程随机获得锁。
独占锁 比如readwritelock中的write
共享锁 比如readwritelock中的read
偏向锁是指，当该线程曾经执行过，那么优先让该线程获得锁。

# 讲讲独占锁 ReentrantLock 原理？谈谈读写锁 ReentrantReadWriteLock 原理？
ReentrantLock跟synchronized类似，都是重量级的排他锁。不同的是ReentrantLock提供了更多的功能，比如trylock 没有锁时不block，trylock timeout, await, signal等。
ReentrantLock获取锁时失效线程的工作内存，并重新加载主内存到工作内存；释放锁时将线程工作内存的数据刷到主内存。
ReentrantReadWriteLock中，读锁是共享锁，写锁是独占锁。读锁被全部释放后，写锁独占；写锁被释放后，读锁才能获得。

# ThreadLocal 作为变量的线程隔离方式，其内部是如何做的？
内部是个map的结构, key是线程的identity

# CyclicBarrier内部的实现与 CountDownLatch 有何不同？
CyclicBarrier是阻塞一定数量的线程，到达一定数量后这些线程才继续执行。内部是reentrencelock，线程在调用await之后阻塞，只有一个线程进入自旋，当count==0时，释放锁，那么所有的线程依次获得锁，继续执行。
CountDownLatch是一定数量的线程执行完毕后，触发事件。内部是CAS, 没次线程执行完毕减去1，当为0时出发事件。
Semaphore是有一个双向链表的结构，node的prev和next被设置为volatile。当获取锁的时候，CAS node的next；当释放锁的时候，CAS node的next为空，当前的prev为空；如果不成功，则自旋重试。

# 有3个线程。线程A和线程B并行执行，线程C需要A和B执行完成后才能执行。可以怎么实现？
线程A和B用线程池执行；执行完毕后加CyclicBarrier.wait，CyclicBarrier的值设为2；并调用线程B的代码传入作为CyclicBarrier完成后的action。
也可以用reactive，concat或者merge两个action，then执行线程C

# 现在有两个线程同时操作一个整数I，做自增操作，如何实现I的线程安全性？
AtomicInteger， java8提供了atomicAddr

# Java中的多个线程如何传递数据？
线程首先从主内存加载数据到工作内存，对变量的操作都在线程的工作内存中进行；当线程执行完毕，工作内存中的数据会刷回主内存。所以在不考虑锁的情形下，多线程的数据会出现不一致的情形。
传递数据是指传递参数吗？可以在线程启动时传递，也可以让线程读取变量

# LinkedBlockingQueue 原理
LinkedBlockingQueue先设定链表的长度，当长度等于设定值时，就await后续的线程；当元素被dequeue，那么就signal线程继续执行。

# LinkedBlockingQueue  vs ArrayBlockingQueue
前者可以设定最大值为int.max。后者是有限的

# CountDownLatch 与线程的 Join 方法区别是什么？
Join和CountDownLatch都可以等待几个线程执行完毕后继续；但是Join要直接操作线程，CountDownLatch可以调用在线程池的方法，这里thread是不可见的。
而且CountDownLatch不一定非得在多线程里CountDown

# 什么是Java指令重排序？
对于不相干的指令，不保证在同一线程内的是顺序执行的

# Spring是如何实现AOP的？
1. what：定义一个filter/Interceptor/aspect来做“切面”: filter过滤器：能够拿到http的request和response;interceptor拦截器：能够拿到bean名称，方法名称等; aspect切面：能够拿到方法的参数
2. where：定义一个jointPoint来决定“往哪里切入”
3. When：定义一个方法注释上@before, @After, @Around来决定“什么时候切入”

# Spring有哪些设计模式？

# Spring中是怎么解决循环依赖的？比如A依赖于B，B又依赖于A。
A依赖于B，那么先初始化A，然后再初始化B。这时对于B的依赖A，那么使用A的一个未初始化完毕的引用，当B初始化完毕，A也初始化完毕。
对Java 接口代理模式的实现原理的理解？如何使用 Java 反射实现动态代理？ Java 接口代理模式的指定增强？

# 谈谈对Cglib 类增强动态代理的实现？和JDK 动态代理有什么区别？
代理模式是为了在不改变原代码的情况下，写一个新的类，实现同样的接口，在类的方法内部，调用原始类的方法，并对方法进行环绕增强。
这里有三种方法：
	1. 自己实现代理类
	用JDK的动态代理，Proxy.newProxyInstance
	用Cglib包实现MethodInterceptor
JDK动态代理和Cglib动态代理的区别在于，前者必须有一个接口，后者不需要


# BeanFactory 和 FactoryBean 有什么区别？
BeanFactory可以获取是Ioc的bean
FactoryBean是创建的bean

# BeanFactory 和 ApplicationContext 又有什么不同？
BeanFactory是ApplicationContext的一个子集

# Float、Decimal 存储金额的区别？Datetime、Timestamp 存储时间的区别？Char、Varchar、Varbinary 存储字符的区别？
Float在二进制转换过程中会丢失精度，Decimal作了优化，适用于金融类计算。
Datetime没有指定时区，timestamp是从1970年的标准时间到现在的秒数，所以自带时区属性。
Char的长度不变，varchar是可变长度。
Varbinary保存了可变长度的二进制数据

# 深拷贝和浅拷贝
深拷贝是把值copy了。浅拷贝把所有的field的reference copy了

# 描述一下 JVM 加载 Class 文件的原理机制?
先由bootstrap class loader加载核心类库，由ext class loader加载引用的包，app class loader加载编写的类，还可以自定义class loader，比如加密class文件等。
当寻找某个类时，是自下向上的，从自定义class loader查找缓存，如果没有，则向上找app class loader，一直到bootstrap class loader的缓存。如果bootstrap的缓存中还没有找到，那么就在目录中找，未找到就app找目录，一直到自定义class loader找目录。

# Java 堆的结构是什么样子的？

# 完全二叉树

# 在 Java 中，对象什么时候可以被垃圾回收？
当新生代满了，触发minor GC；
老年代满了，触发major/full GC；
Metadata代满了，触发full GC；
没有多余的空间分配内存时，触发GC
调用runtime.gc, 不一定能立即触发

# 简述Minor GC 和 Major GC？垃圾回收机制？
Monor GC发生在，当无法在eden space分配内存，比如新生代已经满了，或者无法找到一块足够大的闲置内存来分配，这时就触发minor GC。
Minor GC会stop the world，没有引用的对象被回收，还有引用的对象被复制到survival space，或者从survival 1 复制到survival 2. Minor GC只支持serial GC，parnew和G1。其中serial GC是用于客户端程序。

Major GC发生在，当无法在old space分配内存时。
Major GC的类型有CMS，parallel，serial old和G1。其中parallel更适合大量的后台计算，比较不在乎GC的时间；CMS是为了更低的延迟，但是CMS不压缩内存，只会在full gc几次之后压缩；G1是为了在满足延迟条件的情况下，更好的吞吐量。
所以CMS会导致老年代碎片化严重，需要更大的内存；G1不但可以压缩碎片，还可以对大对象回收。
Major GC会stop the world。

# ConcurrentLinkedQueue 内部是如何使用 CAS 非阻塞算法来保证多线程下入队出队操作的线程安全？
当前node的引用是volatile的，如果引用改变，那么就自旋；如果没有变，那么就新增

# 并发组件CopyOnWriteArrayList 是如何通过写时拷贝实现并发安全的 List？
ArrayList并不是线程安全的，CopyOnWriteArrayList在写时，先加上锁，然后copy出一个+1 size的数组

# 同步与异步？阻塞与非阻塞？
同步是指必须等待执行结果返回以后，再执行下一步操作；
异步是指发出指令后无需等待执行结果，就执行下一步操作；结果可以通过回调来处理；本质上是“同步”的操作被另外一个线程并行处理了，该是阻塞还是阻塞。
阻塞是指当前线程等待执行结果完成后，继续执行
非阻塞是指当前线程发出I/O指令后，该线程又去执行下一个操作；等I/O的结果返回，才会有线程去处理。
线程阻塞的问题在于，程序必须调度大量线程来处理I/O请求；而且当大多数线程都在等待时，计算机资源如CPU和memory的利用率不高。
所以当I/O频繁地操作，最好用非阻塞的模型来解决问题。

# ThreadLocalRandom 是如何利用 ThreadLocal 的原理来解决 Random 的局限性？
Random需要多线程来竞争seed，所以导致性能下降
而ThreadLocalRandom 在每个线程里维护一个seed，所以就不存在竞争的问题。ThreadLocalRandom.current也能避免产生相同随机数

# Netty线程模型
## 单线程模型
一个线程(reactor thread)处理所有的I/O操作，以及处理操作
1. Receive TCP connection

2. Initial TCP connection

3. Read request or response message
  Send request or response message

<img src="/Users/linzhang/Documents/Java/knowledge/basic/image/reactor-single.png" alt="avatar" style="zoom:100%;" />

## 多线程模型
1. 一个线程（acceptor thread）处理所有的I/O操作，然后转发至线程池
	• Receive TCP connection
	• Initial TCP connection
2. 一个线程池(reactor thread pool) 处理request和response
	• Read request or response message
	• Send request or response message
<img src="/Users/linzhang/Documents/Java/knowledge/basic/image/reactor-multiple.png" alt="avatar" style="zoom:100%;" />

## Master-slave多线程模型
1. 一个线程池(main reactor thread pool) 来处理TCP请求
2. 一个线程池(sub reactor thread pool)来处理I/O操作
<img src="/Users/linzhang/Documents/Java/knowledge/basic/image/reactor-master-slave.png" alt="avatar" style="zoom:100%;" />

## Netty
1. bossGroup来处理tcp connections
2. workerGroup来处理I/O操作
3. netty的thread mode：
	1. 单线程mode
	<img src="/Users/linzhang/Documents/Java/knowledge/basic/image/netty-single.png" alt="avatar" style="zoom:100%;" />
	2. 多线程mode
	<img src="/Users/linzhang/Documents/Java/knowledge/basic/image/netty-multiple.png" alt="avatar" style="zoom:100%;" />
	3. Master-slave mode：
   <img src="/Users/linzhang/Documents/Java/knowledge/basic/image/netty-master-slave.png" alt="avatar" style="zoom:100%;" />

## 什么是零拷贝
这里说一下kafka的零拷贝：
	1. 读取步骤
	文件系统----->操作系统内核缓冲----->应用程序的用户缓冲----->写入内核空间并且放入socket缓冲------>网卡的接口缓冲----->通过网络发送
	2. 零拷贝
	应用程序的页面缓存直接复制到网卡中（待确认）

# 集合类
## HashMap
类似dictionary, 而且是范型。是Hashtable和Properties的替代
Hashtable到处标记有Syncrinize，所以效率很低
Hashmap不是线程安全的，ConcurrentHashmap是线程安全的，而且读用了volatile，写用了分段锁，所以效率高
Hashmap是数组和链表的合集，也就是说，hashmap是有初始容量和load factor的，初始容量默认为16，load factor为0.75。 如果超出了load factor和初始容量，那么就会生成一个double当前size的新hashtable，并且rehash所有数据然后放入新的Hashmap。
Hashmap遇到哈希碰撞，会放入链表中。只有hash值和key相同，才能取得value。如果链表过大（大于8个），会转成红黑树的结构。
HashMap是乱序的

## LinkedHashMap
LinkedHashMap按照插入顺序排序

## TreeMap
红黑树. 可以指定键值排列规则. 非线程安全

## Hashset
只有键值，value为空的TreeMap。非线程安全

## HashMap vs LinkedHashMap vs TreeMap
1. HashMap用作一般的HashMap，非线程安全的情况下，数据量事先定好，不会暴增
2. LinkedHashMap在需要在插入顺序的情况下，以及有大量的iteration的情况下
3. TreeMap是在需要自定义顺序的情况下

## Vector和ArrayList和CopyOnWriteArrayList
1. 都是数组，而且是初始化时无需定义size的，而且都可以自动增长和缩小
2. Vector是线程安全的，是legacy code
3. 因为ArrayList不是线程安全的，所以会导致数组遍历时，数组会被另外的线程修改，那么就会抛出异常。这个过程叫做fail fast.
4. 在需要线程安全的场景下，应该使用CopyOnWriteArrayList


## 小总结
1. 大量随机访问，用ArrayList; 经常性插入和删除元素，用LinkedList
2. HashMap快速访问，TreeMap保持排序，LinkedMap以插入顺序
3. linkedList提供栈和队列的行为
4. Set不接受重复元素，HashSet快速访问，Treeset保持排序，LinkedSet保持插入顺序
5. 不应该使用过时的vector，hashtable和stack

# 线程池

## ExecuteService

### newFixedThreadPool

特性：

1. 创建一个固定线程数的线程池, 新的task会放入一个无限大的linkedBlockingQueue。
2. thread会一直存在，就算是任务完成了，thread也会存在, 必须显式的shutdown。

优点：适合long run task，间隔时间run的task

### newCachedThreadPool

特性：

1. 可以创建一个无限扩大的线程池, 新的task会导致一个新的线程，或者之前的线程被重新使用。
2. 内部使用的是SynchronousQueue，只有在task被移除，新的task才能进入。
3. thread会在完成task之后，一定时间内关闭thread并且移除出pool。

优点：适合非常多short lived task （如果不是I/O bound, 可以考虑newWorkStealingPool）

## ScheduledThreadPoolExecutor

特性：

1. 可以创建一个延迟执行的线程池。线程池内的初始线程会一直存在，但是会扩展到int.max大
2. 内部使用delayedworkingqueue。delay最少的task能够被优先执行。

## newWorkStealingPool

特性：

1. 默认创建core count个线程，每个线程都有自己的queue
2. 优先从线程自己的queue中获取task，如果自己queue为空，才从别的queue “steal” task。
3. 使用ForkJoinPool

优点：性能好

缺点：不适合I/O bound的操作，因为等很久的话，线程被阻塞，影响性能。

## Folk/Join

特性：

1. 讲tasks划分为subtasks，然后对结果汇总
2. 使用的是ForkJoinPool

优点：对结果汇总更加容易。executors可不好做


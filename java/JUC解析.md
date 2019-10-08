### 1. JUC 示意图

从整体来看，concurrent包的实现示意图如下：

![](../photo/concurrent.png)

### 2. JUC 代码结构
concurrent包的组织结构如下：

#### 2.1 java.util.concurrent

Utility classes commonly useful in concurrent programming.

在并发编程中常用的实用程序类。

##### 2.1.1 接口
+ BlockingDeque<E>

+ BlockingQueue<E>

+ Callable<V>

+ CompletionService<V>

+ CompletionStage<T>

+ ConcurrentMap<K, V>

+ ConcurrentNavigableMap<K,V>

+ Delayed

+ Executor

+ ExecutorService

+ Future<V>

+ RejectedExecutionHandler

+ RunnableFuture<V>

+ RunnableScheduledFuture<V>

+ ScheduledExecutorService

+ ScheduledFuture<V>

+ ThreadFactory

+ TransferQueue<E>

##### 2.1.2 抽象类
+ AbstractExecutorService

+ CountedCompleter<T>

##### 2.1.3 异常类
+ BrokenBarrierException

+ CancellationException

+ CompletionException

+ ExecutionException

+ RejectedExecutionException

+ TimeoutException

##### 2.1.4 Queue
+ ArrayBlockingQueue<E>

 ArrayBlockingQueue是数组实现的线程安全的有界的阻塞队列。

+ ConcurrentLinkedQueue<E>

ConcurrentLinkedQueue是单向链表实现的无界队列，该队列按 FIFO（先进先出）排序元素。

+ DelayQueue<E extends Delayed>

+ LinkedBlockingQueue<E>

LinkedBlockingQueue是单向链表实现的(指定大小)阻塞队列，该队列按 FIFO（先进先出）排序元素。

+ LinkedTransferQueue<E>

+ PriorityBlockingQueue<E>

+ SynchronousQueue<E>

##### 2.1.5 Deque
+ ConcurrentLinkedDeque<E>

ConcurrentLinkedDeque是双向链表实现的无界队列，该队列同时支持FIFO和FILO两种操作方式。

+ LinkedBlockingDeque<E>

LinkedBlockingDeque是双向链表实现的(指定大小)双向并发阻塞队列，该阻塞队列同时支持FIFO和FILO两种操作方式。

##### 2.1.6 Map
+ ConcurrentHashMap<K,V>

ConcurrentHashMap是线程安全的哈希表(相当于线程安全的HashMap)；它继承于AbstractMap类，并且实现ConcurrentMap接口。ConcurrentHashMap是通过“锁分段”来实现的，它支持并发。

+ ConcurrentSkipListMap<K,V>

ConcurrentSkipListMap是线程安全的有序的哈希表(相当于线程安全的TreeMap); 它继承于AbstractMap类，并且实现ConcurrentNavigableMap接口。ConcurrentSkipListMap是通过“跳表”来实现的，它支持并发。

##### 2.1.7 Set
+ ConcurrentSkipListSet<E>

 ConcurrentSkipListSet是线程安全的有序的集合(相当于线程安全的TreeSet)；它继承于AbstractSet，并实现了NavigableSet接口。ConcurrentSkipListSet是通过ConcurrentSkipListMap实现的，它也支持并发。

+ CopyOnWriteArraySet<E>

 CopyOnWriteArraySet相当于线程安全的HashSet，它继承于AbstractSet类。CopyOnWriteArraySet内部包含一个CopyOnWriteArrayList对象，它是通过CopyOnWriteArrayList实现的。
##### 2.1.8 List
+ CopyOnWriteArrayList<E>

CopyOnWriteArrayList相当于线程安全的ArrayList，它实现了List接口。CopyOnWriteArrayList是支持高并发的。

##### 2.1.9 同步工具类

+ CountDownLatch

CountDownLatch 一种非常简单、但很常用的同步辅助类。其作用是在完成一组正在其他线程中执行的操作之前,允许一个或多个线程一直阻塞。

+ CyclicBarrier

CyclicBarrier 一种可重置的多路同步点，在某些并发编程场景很有用。它允许一组线程互相等待，直到到达某个公共的屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier在释放等待线程后可以重用，所以称它为循环的barrier。

+ Exchanger<V>

Exchanger允许两个线程在某个汇合点交换对象，在某些管道设计时比较有用。Exchanger提供了一个同步点，在这个同步点，一对线程可以交换数据。每个线程通过exchange()方法的入口提供数据给他的伙伴线程，并接收他的伙伴线程提供的数据并返回。当两个线程通过Exchanger交换了对象，这个交换对于两个线程来说都是安全的。Exchanger可以认为是 SynchronousQueue 的双向形式，在运用到遗传算法和管道设计的应用中比较有用。

+ Phaser

Phaser一种可重用的同步屏障，功能上类似于CyclicBarrier和CountDownLatch，但使用上更为灵活。非常适用于在多线程环境下同步协调分阶段计算任务（Fork/Join框架中的子任务之间需同步时，优先使用Phaser）

+ Semaphore

Semaphore 信号量是一类经典的同步工具。信号量通常用来限制线程可以同时访问的（物理或逻辑）资源数量。

##### 2.1.10 线程池

+ ScheduledThreadPoolExecutor

+ ThreadPoolExecutor

##### 2.1.11 工具类
+ ThreadLocalRandom

+ Executors

+ TimeUnit

##### 2.1.12 ForkJoin
+ ForkJoinPool

+ ForkJoinTask<V>

+ ForkJoinWorkerThread

+ RecursiveAction

+ RecursiveTask<V>

##### 2.1.13 其他
+ ExecutorCompletionService<V>

+ FutureTask<V>

JDK8新增类型
+ CompletableFuture<T>

#### 2.2 java.util.concurrent.atomic
A small toolkit of classes that support lock-free thread-safe programming on single variables.

一个小的类工具包，支持对单个变量进行无锁线程安全编程。

可以分为以下几类：

##### 2.2.1 基本类
+ AtomicBoolean

+ AtomicInteger

+ AtomicLong

总结一下操作基本类型的原子类，它们的内部都维护者一个对应的基本类型的成员变量value，这个变量是被volatile关键字修饰的，这可以保证每次一个线程要使用它都会拿到最新的值。然后在进行一些原子操作的时候，需要依赖Unsafe类，这个类里面就有一些CAS函数，一些原子操作就是通过自旋方式，不断地使用CAS函数进行尝试直到达到自己的目的。

##### 2.2.2 引用类型

+ AtomicMarkableReference<V>

+ AtomicReference<V>

+ AtomicStampedReference<V>

##### 2.2.3 数组类型
+ AtomicIntegerArray

+ AtomicLongArray

+ AtomicReferenceArray<E>

##### 2.2.4 属性原子修改器（Updater）
+ AtomicIntegerFieldUpdater<T>

+ AtomicLongFieldUpdater<T>

+ AtomicReferenceFieldUpdater<T,V>

##### 2.2.5 JDK8新增类型
+ DoubleAccumulator

+ DoubleAdder

+ LongAccumulator

+ LongAdder

##### 2.2.6 Striped64
？？

#### 2.3 java.util.concurrent.locks
Interfaces and classes providing a framework for locking and waiting for conditions that is distinct from built-in synchronization and monitors.

与内置的同步和监视器不同，这些类和接口支持加锁与等待条件等操作。

### 3. JUC 细节概述
以下按照自底向上，分模块介绍各个主题。

#### 3.1 UNSAFE
Unsafe 类仅供 JDK 内部调用，这个类可以用来在任意内存地址位置处读写数据，另外，还支持一些CAS原子操作。

如果需要使用它，需要通过其他方式。具体见下文。

##### 3.1.1 JDK中使用 UNSAFE

JDK8中获取 Unsafe 方式。
```java
// Unsafe mechanics
private static final sun.misc.Unsafe UNSAFE;
private static final long valueOffset;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> ak = Cell.class;
        valueOffset = UNSAFE.objectFieldOffset
            (ak.getDeclaredField("value"));
    } catch (Exception e) {
        throw new Error(e);
    }
}

```

作为对比，看下JDK11中使用方式。
```java
 private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
```

##### 3.1.2 如何使用 UNSAFE 类（JDK8之前适用，之后JDK的实现方式变了，可用性存疑）
```java
// UNSAFE mechanics

private static final sun.misc.Unsafe UNSAFE;
private static final long nextOffset;

static {
    try {
        java.lang.reflect.Field theUnsafe = sun.misc.Unsafe.class.getDeclaredField("theUnsafe");
        theUnsafe.setAccessible(true);
        UNSAFE = (sun.misc.Unsafe) theUnsafe.get(null);
        // UNSAFE = sun.misc.Unsafe.getUnsafe();

        nextOffset = UNSAFE.objectFieldOffset(Node.class.getDeclaredField("next"));
    }
    catch (Exception e) {
        throw new Error(e);
    }
}
```

#### 3.2 CAS
Unsafe 类提供很多 native 的调用方法，其中 CAS 方法是 JUC 包的基石。

##### 3.2.1 CAS 原理
CAS 是英文单词 CompareAndSwap 的缩写，中文意思是：比较并替换。CAS需要有3个操作数：内存地址V，旧的预期值A，即将要更新的目标值B。

CAS指令执行时，当且仅当内存地址V的值与预期值A相等时，将内存地址V的值修改为B，否则就什么都不做。

整个比较并替换的操作是一个原子操作。

示例： AtomicInteger 的 compareAndSet
```java
public final boolean compareAndSet(int expect, int update) {   
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
 }
```

其中 unsafe.compareAndSwapInt 是一个 native 方法。

#### 3.2.2 CAS 底层实现
在任何处理器平台下，都会有一些原子性操作，供操作系统使用，这里只讲x86下面的。

在单处理器情况下，每条指令的执行都是原子性的，但在多处理器情况下，只有那些单独的读操作或写操作才是原子性的。为了弥补这一缺点，x86提供了附加的lock前缀，使带lock前缀的读修改写指令也能原子性执行。带lock前缀的指令在操作时会锁住总线，使自身的执行即使在多处理器间也是原子性执行的。

xchg指令不带lock前缀也是原子性执行，也就是说xchg执行时默认会锁内存总线。

原子性操作是线程间同步的基础，linux专门定义了一种只进行原子操作的类型atomic_t，并提供相关的原子读写调用API。

这里只看 xchg 指令在 x86下的实现：

atomic_cmpxchg是由cmpxchg指令完成的。它把旧值同atomic_t类型的值相比较，如果相同，就把新值存入atomic_t类型的值中，返回atomic_t类型变量中原有的值。

```java
static inline int atomic_cmpxchg(atomic_t *v, int old, int new)
{
	return cmpxchg(&v->counter, old, new);
}

#define cmpxchg(ptr, o, n)						\
	((__typeof__(*(ptr)))__cmpxchg((ptr), (unsigned long)(o),	\
				       (unsigned long)(n),		\
				       sizeof(*(ptr))))

static inline unsigned long __cmpxchg(volatile void *ptr, unsigned long old,
				      unsigned long new, int size)
{
	unsigned long prev;
	switch (size) {
	case 1:
		asm volatile(LOCK_PREFIX "cmpxchgb %b1,%2"
			     : "=a"(prev)
			     : "q"(new), "m"(*__xg(ptr)), "0"(old)
			     : "memory");
		return prev;
	case 2:
		asm volatile(LOCK_PREFIX "cmpxchgw %w1,%2"
			     : "=a"(prev)
			     : "r"(new), "m"(*__xg(ptr)), "0"(old)
			     : "memory");
		return prev;
	case 4:
		asm volatile(LOCK_PREFIX "cmpxchgl %k1,%2"
			     : "=a"(prev)
			     : "r"(new), "m"(*__xg(ptr)), "0"(old)
			     : "memory");
		return prev;
	case 8:
		asm volatile(LOCK_PREFIX "cmpxchgq %1,%2"
			     : "=a"(prev)
			     : "r"(new), "m"(*__xg(ptr)), "0"(old)
			     : "memory");
		return prev;
	}
	return old;
}

```

##### 3.2.3 CAS 的缺点

CAS虽然很高效的解决了原子操作问题，但是CAS仍然存在三大问题。

+ 使用时基于乐观策略，往往需要重试（自旋），时间长，开销很大

如果竞争激烈，虽然 CAS 保证肯定有一个线程可以操作成功，但是这也意味着其他线程所做的（耗时）操作都是“无用功”。好的 CAS 使用方式肯定是只用在关键地方！

+ 只能保证一个字长内存的原子操作

这是硬件指令的限制。意味着，如果需要原子地改变多个对象，只能将它们封装之后（它们的引用肯定是一个字长）才能使用CAS。这时候，使用其他同步操作（锁或同步块等）或许更优。

+ ABA问题

如果内存地址 V 初次读取的值是 A，并且在准备赋值的时候检查到它的值仍然为 A（如果在这段期间它的值曾经被改成了 B，后来又被改回为 A），那 CAS 操作就会误认为它从来没有被改变过。这个漏洞称为CAS操作的"ABA"问题。Java并发包为了解决这个问题，提供了一个带有标记的原子引用类 AtomicStampedReference，它可以通过控制变量值的版本来保证CAS的正确性。因此，在使用CAS前要考虑清楚"ABA"问题是否会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。

##### 3.2.4 CAS 的使用场景
（1）JDK中的CAS，主要是JUC并发包。

+ java.util.concurrent.atomic 包

+ 跳表 java.util.concurrent.ConcurrentSkipListMap

+ 无锁队列 java.util.concurrent.ConcurrentLinkedQueue

等等

（2）JVM中的CAS，主要是堆中对象的分配。

在单线程的情况下，一般有两种分配策略：

+ 指针碰撞

这种一般适用于内存是绝对规整的（内存是否规整取决于内存回收策略），分配空间的工作只是将指针像空闲内存一侧移动对象大小的距离即可。

+ 空闲列表

这种适用于内存非规整的情况，这种情况下JVM会维护一个内存列表，记录那些内存区域是空闲的（包括可用大小）。给对象分配空间的时候去空闲列表里查询到合适的区域然后进行分配即可。

但是JVM不可能一直在单线程状态下运行，那样效率太差了。由于在给一个对象分配内存的时候不是原子性的操作，至少需要以下几步：查找空闲列表，分配内存，修改空闲列表等等，这些操作都不是线程安全的。解决并发时的安全问题也有两种策略：

+ CAS：实际上虚拟机采用CAS配合上失败重试的方式保证更新操作的原子性。

+ TLAB：如果使用CAS其实对性能还是会有影响的，所以JVM又提出了一种更高级的优化策略：每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲区（TLAB），线程内部需要分配内存时直接在TLAB上分配就行，避免了线程冲突。只有当缓冲区的内存用光需要重新分配内存的时候才会进行CAS操作分配更大的内存空间。虚拟机是否使用TLAB，可以通过-XX:+/-UseTLAB参数来进行配置（jdk5及以后的版本默认是启用TLAB的）。

（3）高性能内存队列 disruptor 中的 CAS，主要是 Ring Buffer 的节点分配。

LMAX是一种新型零售金融交易平台，它能够以很低的延迟（latency）产生大量交易（吞吐量）。这个系统是建立在JVM平台上，核心是一个业务逻辑处理器，它能够在一个线程里每秒处理6百万订单. 业务逻辑处理器完全是运行在内存中（in-memory），使用事件源驱动方式（event sourcing）。业务逻辑处理器的核心是Disruptors，这是一个并发组件，能够在无锁的情况下实现网络的Queue并发操作。

#### 3.3 volatile
在将 java.util.concurrent.atomic 之前，还需先认识一下 volatile 关键字。

`JSR-133`对`JDK5`之前的旧内存模型的修补主要有两个：

+ （1）增强`volatile`的内存语义。旧内存模型允许`volatile`变量与普通变量重排序。`JSR-133`严格限制`volatile`变量与普通变量的重排序，使`volatile`的写-读和锁的释放-获取具有相同的内存语义。

+ （2）增强`final`的内存语义。在旧内存模型中，多次读取同一个`final`变量的值可能会不相同。为此，`JSR-133`为`final`增加了两个重排序规则。现在，`final`具有了初始化安全性。

##### 3.3.1 `volatile`变量自身具有下列特性：
+ 可见性。对一个`volatile`变量的读，总是能看到（任意线程）对这个`volatile`变量最后的写入。

+ 原子性：对任意单个`volatile`变量的读/写具有原子性，但类似于`volatile++`这种复合操作不具有原子性。

`JSR-133`开始，`volatile`变量的写-读可以实现线程之间的通信。

从内存语义的角度来说，`volatile`与锁有相同的效果：`volatile`写和锁的释放有相同的内存语义；`volatile`读与锁的获取有相同的内存语义。

下面对`volatile`写和`volatile`读的内存语义做个总结：

+ 线程A写一个`volatile`变量，实质上是线程A向接下来将要读这个`volatile`变量的某个线程发出了（其对共享变量所在修改的）消息。

+ 线程B读一个`volatile`变量，实质上是线程B接收了之前某个线程发出的（在写这个`volatile`变量之前对共享变量所做修改的）消息。

+ 线程A写一个`volatile`变量，随后线程B读这个`volatile`变量，这个过程实质上是线程A通过主内存向线程B发送消息。

##### 3.3.2 下面是`JMM`针对编译器制定的`volatile`重排序规则表：

+ 当第二个操作是`volatile`写时，不管第一个操作是什么，都不能重排序。这个规则确保`volatile`写之前的操作不会被编译器重排序到`volatile`写之后。

+ 当第一个操作是`volatile`读时，不管第二个操作是什么，都不能重排序。这个规则确保`volatile`读之后的操作不会被编译器重排序到`volatile`读之前。

+ 当第一个操作是`volatile`写，第二个操作是`volatile`读时，不能重排序。

`JMM`采取保守策略。下面是基于保守策略的`JMM`内存屏障插入策略：

+ 在每个`volatile`写操作的前面插入一个`StoreStore`屏障。

+ 在每个`volatile`写操作的后面插入一个`StoreLoad`屏障。

+ 在每个`volatile`读操作的后面插入一个`LoadLoad`屏障。

+ 在每个`volatile`读操作的后面插入一个`LoadStore`屏障。

#### 3.4 synchronized
##### 3.4.1 synchronized 的使用场景
（1）修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是synchronized后面括号()括起来的对象；

（2）修饰一个非静态方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象；

（3）修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象；

synchronized(Sync.class)实现了全局锁的效果。static synchronized方法，static方法可以直接类名加方法名调用，方法中无法使用this，所以它锁的不是this，而是类的Class对象，所以，static synchronized方法也相当于全局锁，相当于锁住了代码段。

A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是类，该类所有的对象是同一把锁。

B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。

C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

##### 3.4.2 JVM 对 synchronized 的优化
临界区（例如synchronized修饰的方法或者代码块），在同一时刻 JVM 只允许一个线程进入执行。实际上，JVM 设置了一把锁，抢到了这把锁就可以执行，否则只能阻塞，等待其他线程释放锁。

JVM 煞费心机地设置了偏向锁和轻量级锁，就是为了避免阻塞，避免操作系统的介入（因为操作系统在内核中维护Mutex（互斥量）需要进行线程阻塞，切换，系统开销很大）。

JVM 对 synchronized 的优化主要是应对如下情况：

偏向锁：通常只有一个线程在临界区执行

轻量级锁：可以有多个线程交替进入临界区，在竞争不激烈的时候，稍微自旋等待一下就能获得锁。

重量级锁：出现了激烈的竞争，只好让线程阻塞了。

（1）偏向锁

偏向锁，在没有别的线程竞争的时候，一直偏向，可以让其一直执行下去。

线程进入临界区时，JVM 会查看锁对象的所谓“对象头”，其中有个叫做 Mark Word 的东西，里边有几个标识位，还有其他数据。

JVM 使用 CAS 操作把线程 ID 记录到了这个 Mark Word 当中，修改了标识位为 01，这样该线程就拥有这把锁了，可以进入临界区。这个过程不用操作系统介入。

（2）轻量级锁

当出现线程竞争时，比如其他线程也要进入进入临界区时，JVM 会将这个偏向锁升级一下，变成一个轻量级的锁。

JVM 把锁对象恢复成无锁状态，在参与竞争的每个线程的栈帧中各自分配了一个空间，叫做 Lock Record， 把锁对象的 Mark Word 在每个 Lock Record 各自复制了一份，叫做 Displaced Mark Word。然后把当前线程的 Lock Record 的地址使用 CAS 放到了 Mark Word 当中，并且把锁标志位改为00，这其实就意味着当前线程已经获得了这个轻量级的锁了，可以继续进入临界区执行。

其他线程没有获得锁，但还是不阻塞，JVM 让其自旋几次，等待一会儿。

等到当前线程退出临界区，释放锁的时候，需要把这个 Displaced markd word 使用CAS复制回去。接下来其他线程就可以加锁了。

线程之间交替着进入临界区，相安无事，很少出现真正的竞争。

即使是出现了竞争，想获得锁的线程只要自旋几次，等待一会儿，锁就可能释放了。

很明显，如果没有竞争或者轻度的竞争，轻量级锁仅仅使用CAS操作和 Lock record 就避免了重量级互斥锁的开销，对JVM来说，确实是个好主意。

（3）重量级锁

竞争线程自旋次数太多了，太浪费CPU了，升级为重量级锁。

这个重量级锁需要操作系统的帮忙，依赖操作系统底层的 Mutex Lock。

JVM 创建了一个 monitor 对象， 把这个对象的地址更新到了Mark word当中。

持有锁的线程继续执行，其他竞争线程则阻塞。


### 参考资料
#### 1. Module java.base
https://docs.oracle.com/en/java/javase/11/docs/api/java.base/module-summary.html

#### 2. java高并发：CAS无锁原理及广泛应用
https://blog.csdn.net/liubenlong007/article/details/53761730

#### 3. LMAX-Exchange/disruptor
https://github.com/LMAX-Exchange/disruptor

#### 4. linux内核部件分析（二）——原子性操作atomic_t
https://blog.csdn.net/qb_2008/article/details/6840808

#### 5. 详解JUC之原子类使用及实现
https://blog.csdn.net/timheath/article/details/71441008

#### 6. 一个想休息的线程：JVM到底是怎么处理锁的？怎么不让我阻塞呢？
[ 码农翻身](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665514802&idx=1&sn=b27962488ddd23cda5f67c3f1db12b56&chksm=80d67f71b7a1f6673c4222b128603db7a9b85029f6f5443de2cb1ecba535f3f802df2aedf076&mpshare=1&scene=1&srcid=110830eTDViSgl2X39rj7TFe#rd)

####TODO 7. Java多线程系列目录(共43篇)
http://www.cnblogs.com/skywang12345/p/java_threads_category.html

#### 8. JUC五种常见同步工具类总结
https://blog.csdn.net/luoyoub/article/details/80635652

####TODO 9. 锁机制学习笔记
https://www.cnblogs.com/cm4j/p/juc_lock.html

####TODO 10. JUC源码分析—AQS
https://yiweifen.com/v-1-259342.html

####TODO 11. Java并发编程：Callable、Future和FutureTask
https://www.cnblogs.com/dolphin0520/p/3949310.html

####TODO 12. JUC.Condition学习笔记[附详细源码解析]
https://www.cnblogs.com/cm4j/p/juc_condition.html

####TODO 13. JUC源码解析：目录（基于JDK 8）
https://www.jianshu.com/p/af9c0f404a93

####TODO 14. Java多线程系列之“JUC集合“详解
https://blog.csdn.net/ZYC88888/article/details/80018239

####TODO 15. Java并发包中的同步队列SynchronousQueue实现原理
http://ifeve.com/java-synchronousqueue/

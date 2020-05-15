### 1. NoClassDefFoundError

NoClassDefFoundError 错误的发生，是因为 Java 虚拟机在编译时能找到合适的类，而在运行时不能找到合适的类导致的错误。与 ClassNotFoundException 的不同在于，这个错误发生只在运行时需要加载对应的类不成功，而不是编译时发生。

NoClassDefFoundError 发生在 JVM 在动态运行时，根据你提供的类名，在 classpath 中找到对应的类进行加载，但当它找不到这个类时，就发生了 java.lang.NoClassDefFoundError 的错误，而 ClassNotFoundException 是在编译的时候在 classpath 中找不到对应的类而发生的错误。

### 2. finally

不要在 finally 中使用 return 语句。

finally 总是执行，除非程序或者线程被中断。

### 3. 静态内部类

内部类如果不是 static，它本身对外面那个类有引用关系，这一点其实从构造阶段就能看出来，你可以写段代码试试；有强引用就是 strong reachable 状态

### 4. 引用

不同的引用类型，主要体现的是对象不同的可达性（reachable）状态和对垃圾收集的影响

### 5. LRU 与 LinkedHashMap

LinkedHashMap 在构造函数中可以指定链接方式：一种是按照插入的顺序组织成双链表；一种是根据访问的顺序组织成双链表，维持链表头 head 是最长时间没有访问的结点。很显然，实现 LRU 用到的应该是后一种方式

### 6. 并发编程“串行的故事”

起源是一个硬件的核心矛盾：CPU 与内存、I/O 的速度差异，出现了 CPU 缓存，系统软件（操作系统、编译器）在解决这个核心矛盾的同时，引入了可见性、原子性和有序性问题；

简言之就是，导致可见性的原因是 CPU 缓存，导致原子性问题的原因是线程切换，导致有序性的原因是编译优化。

### 7. Integer 的值缓存

值默认缓存 是-128 到 127 之间。

这种缓存机制并不是只有 Integer 才有，同样存在于其他的一些包装类，比如：

- Boolean，缓存了 true/false 对应实例，确切说，只会返回两个常量实例 Boolean.TRUE/FALSE。
- Short，同样是缓存了-128 到 127 之间的数值。
- Byte，数值有限，所以全部都被缓存。
- Character，缓存范围'\u0000' 到 '\u007F'

### 8. Java 提供的默认排序算法

需要区分是 Arrays.sort()还是 Collections.sort() （底层是调用 Arrays.sort()）；什么数据类型；多大的数据集（太小的数据集，复杂排序是没必要的，Java 会直接进行二分插入排序）等。

对于原始数据类型，目前使用的是所谓双轴快速排序（Dual-Pivot QuickSort），是一种改进的快速排序算法，早期版本是相对传统的快速排序。

而对于对象数据类型，目前则是使用 TimSort，思想上也是一种归并和二分插入排序（binarySort）结合的优化排序算法。TimSort 并不是 Java 的独创，简单说它的思路是查找数据集中已经排好序的分区（这里叫 run），然后合并这些分区来达到排序的目的。

另外，Java 8 引入了并行排序算法（直接使用 parallelSort 方法），这是为了充分利用现代多核处理器的计算能力，底层实现基于 fork-join 框架，当处理的数据集比较小的时候，差距不明显，甚至还表现差一点；但是，当数据集增长到数万或百万以上时，提高就非常大了，具体还是取决于处理器和系统环境。

### 9. 有序 Map

虽然 LinkedHashMap 和 TreeMap 都可以保证某种顺序，但二者还是非常不同的。

LinkedHashMap 通常提供的是遍历顺序符合插入顺序，它的实现是通过为条目（键值对）维护一个双向链表。注意，通过特定构造函数，我们可以创建反映访问顺序的实例，所谓的 put、get、compute 等，都算作“访问”。

对于 TreeMap，它的整体顺序是由键的顺序关系决定的，通过 Comparator 或 Comparable（自然顺序）来决定。

### 10. Java IO 方式

首先，传统的 java.io 包，它基于流模型实现，提供了我们最熟知的一些 IO 功能，比如 File 抽象、输入输出流等。交互方式是同步、阻塞的方式，也就是说，在读取输入流或者写入输出流时，在读、写动作完成之前，线程会一直阻塞在那里，它们之间的调用是可靠的线性顺序。

第二，在 Java 1.4 中引入了 NIO 框架（java.nio 包），提供了 Channel、Selector、Buffer 等新的抽象，可以构建多路复用的、同步非阻塞 IO 程序，同时提供了更接近操作系统底层的高性能数据操作方式。

第三，在 Java 7 中，NIO 有了进一步的改进，也就是 NIO 2，引入了异步非阻塞 IO 方式，也有很多人叫它 AIO（Asynchronous IO）。异步 IO 操作基于事件和回调机制，可以简单 理解为，应用操作直接返回，而不会阻塞在那里，当后台处理完成，操作系统会通知相应线程进行后续工作。

Selector 是基于底层操作系统机制，不同模式、不同版本都存在区别：Linux 上依赖于 epoll。Windows 上 NIO2（AIO）模式则是依赖于 iocp。

### 11. 构建器模式（Builder）

通常会被实现成 fluent 风格的 API，也有人叫它方法链。

使用构建器模式，可以比较优雅地解决构建复杂对象的麻烦，这里的“复杂”是指类似需要输入的参数组合较多，如果用构造函数，我们往往需要为每一种可能的输入参数组合实现相应的构造函数，一系列复杂的构造函数会让代码阅读性和可维护性变得很差。

### 12. 是为什么并发容器里面没有 ConcurrentTreeMap 呢？

这是因为 TreeMap 要实现高效的线程安全是非常困难的，它的实现基于复杂的红黑树。为保证访问效率，当我们插入或删除节点时，会移动节点进行平衡操作，这导致在并发场景中 难以进行合理粒度的同步。

而 SkipList 结构则要相对简单很多，通过层次结构提高访问速度，虽然不够紧凑，空间使用有一定提高（O(nlogn)），但是在增删元素时线程安全的开销 要好很多。

### 13. 线程安全队列实现

从行为特征来看，绝大部分 Queue 都是实现了 BlockingQueue 接口。在常规队列操作基础上，Blocking 意味着其提供了特定的等待性操作，获取时（take）等待元素进队，或者插 入时（put）等待队列出现空位。

另一个 BlockingQueue 经常被考察的点，就是是否有界（Bounded、Unbounded），这一点也往往会影响我们在应用开发中的选择，我这里简单总结一下。

ArrayBlockingQueue 是最典型的的有界队列，其内部以 final 的数组保存数据，数组的大小就决定了队列的边界，所以我们在创建 ArrayBlockingQueue 时，都要指定容量，如 public ArrayBlockingQueue(int capacity, boolean fair)

LinkedBlockingQueue，容易被误解为无边界，但其实其行为和内部代码都是基于有界的逻辑实现的，只不过如果我们没有在创建队列时就指定容量，那么其容量限制就自动被 设置为 Integer.MAX_VALUE，成为了无界队列。

SynchronousQueue，这是一个非常奇葩的队列实现，每个删除操作都要等待插入操作，反之每个插入操作也都要等待删除动作。那么这个队列的容量是多少呢？是 1 吗？其实不 是的，其内部容量是 0。

PriorityBlockingQueue 是无边界的优先队列，虽然严格意义上来讲，其大小总归是要受系统资源影响。

DelayedQueue 和 LinkedTransferQueue 同样是无边界的队列。对于无边界的队列，有一个自然的结果，就是 put 操作永远也不会发生其他 BlockingQueue 的那种等待情况。

类似 ConcurrentLinkedQueue 等，则是基于 CAS 的无锁技术，不需要在每个操作时使用锁，所以扩展性表现要更加优异。相对比较另类的 SynchronousQueue，在 Java 6 中，其实现发生了非常大的变化，利用 CAS 替换掉了原本基于锁的逻辑，同步开销比较小。它是 Executors.newCachedThreadPool()的默认队列

### 14. Executors

目前提供了 5 种不同的线程池创建配置：
newCachedThreadPool()，它是一种用来处理大量短时间工作任务的线程池，具有几个鲜明特点：它会试图缓存线程并重用，当无缓存线程可用时，就会创建新的工作线程；如 果线程闲置的时间超过 60 秒，则被终止并移出缓存；长时间闲置时，这种线程池，不会消耗什么资源。其内部使用 SynchronousQueue 作为工作队列。

newFixedThreadPool(int nThreads)，重用指定数目（nThreads）的线程，其背后使用的是无界的工作队列，任何时候最多有 nThreads 个工作线程是活动的。这意味着，如 果任务数量超过了活动队列数目，将在工作队列中等待空闲线程出现；如果有工作线程退出，将会有新的工作线程被创建，以补足指定的数目 nThreads。

newSingleThreadExecutor()，它的特点在于工作线程数目被限制为 1，操作一个无界的工作队列，所以它保证了所有任务的都是被顺序执行，最多会有一个任务处于活动状 态，并且不允许使用者改动线程池实例，因此可以避免其改变线程数目。

newSingleThreadScheduledExecutor()和 newScheduledThreadPool(int corePoolSize)，创建的是个 ScheduledExecutorService，可以进行定时或周期性的工作调度， 区别在于单一工作线程还是多个工作线程。

newWorkStealingPool(int parallelism)，这是一个经常被人忽略的线程池，Java 8 才加入这个创建方法，其内部会构建 ForkJoinPool，利用 Work-Stealing 算法，并行地处 理任务，不保证处理顺序。

### 15. 字节码操纵技术

除了动态代理，还可以应用在什么地方？

各种 Mock 框架
ORM 框架
IOC 容器
部分 Profiler 工具，或者运行时诊断工具等
生成形式化代码的工具

甚至可以认为，字节码操纵技术是工具和基础框架必不可少的部分，大大减少了开发者的负担。

### 16. Spring Bean 的生命周期

典型回答 Spring Bean 生命周期比较复杂，可以分为创建和销毁两个过程。

首先，创建 Bean 会经过一系列的步骤，主要包括：

实例化 Bean 对象。

设置 Bean 属性。

如果我们通过各种 Aware 接口声明了依赖关系，则会注入 Bean 对容器基础设施层面的依赖。具体包括 BeanNameAware、BeanFactoryAware 和 ApplicationContextAware， 分别会注入 Bean ID、Bean Factory 或者 ApplicationContext。

调用 BeanPostProcessor 的前置初始化方法 postProcessBeforeInitialization。

如果实现了 InitializingBean 接口，则会调用 afterPropertiesSet 方法。

调用 Bean 自身定义的 init 方法。

调用 BeanPostProcessor 的后置初始化方法 postProcessAfterInitialization。

创建过程完毕。

第二，Spring Bean 的销毁过程会依次调用 DisposableBean 的 destroy 方法和 Bean 自身定制的 destroy 方法。

Spring Bean 有五个作用域，其中最基础的有下面两种：

Singleton，这是 Spring 的默认作用域，也就是为每个 IOC 容器创建唯一的一个 Bean 实例。

Prototype，针对每个 getBean 请求，容器都会单独创建一个 Bean 实例。

从 Bean 的特点来看，Prototype 适合有状态的 Bean，而 Singleton 则更适合无状态的情况。另外，使用 Prototype 作用域需要经过仔细思考，毕竟频繁创建和销毁 Bean 是有明显开 销的。

如果是 Web 容器，则支持另外三种作用域：

Request，为每个 HTTP 请求创建单独的 Bean 实例。

Session，很显然 Bean 实例的作用域是 Session 范围。

GlobalSession，用于 Portlet 容器，因为每个 Portlet 有单独的 Session，GlobalSession 提供一个全局性的 HTTP Session。

### 17. Spring AOP

引入了其他几个关键概念：
Aspect，通常叫作方面，它是跨不同 Java 类层面的横切性逻辑。在实现形式上，既可以是 XML 文件中配置的普通类，也可以在类代码中用“@Aspect”注解去声明。在运行时，Spring 框架会创建类似 Advisor 来指代它，其内部会包括切入的时机（Pointcut）和切入的动作（Advice）。

Join Point，它是 Aspect 可以切入的特定点，在 Spring 里面只有方法可以作为 Join Point。Join Point 仅仅是可利用的机会。

Pointcut，它负责具体定义 Aspect 被应用在哪些 Join Point，可以通过指定具体的类名和方法名来实现，或者也可以使用正则表达式来定义条件。

Pointcut 是解决了切面编程中的 Where 问题，让程序可以知道哪些机会点可以应用某个切面动作。

Advice，它定义了切面中能够采取的动作。而 Advice 则是明确了切面编程中的 What，也就是做什么；同时通过指定 Before、After 或者 Around，定义了 When，也就是什么时候做。

### 18. Netty

按照官方定义，它是一个异步的、基于事件 Client/Server 的网络框架，目标是提供一种简单、快速构建网络应用的方式，同时保证高吞吐量、低延时、高可靠性。

Netty 的设计强调了 “Separation Of Concerns”，通过精巧设计的事件机制，将业务逻辑和无关技术逻辑进行隔离，并通过各种方便的抽象，一定程度上填补了了基础平台和业务开 发之间的鸿沟，更有利于在应用开发中普及业界的最佳实践。

另外，Netty > java.nio + java. net！

除了核心的事件机制等，Netty 还额外提供了很多功能，例如：

从网络协议的角度，Netty 除了支持传输层的 UDP、TCP、SCTP 协议，也支持 HTTP(s)、WebSocket 等多种应用层协议，它并不是单一协议的 API。

在应用中，需要将数据从 Java 对象转换成为各种应用协议的数据格式，或者进行反向的转换，Netty 为此提供了一系列扩展的编解码框架，与应用开发场景无缝衔接，并且性能良好。

它扩展了 Java NIO Bufer，提供了自己的 ByteBuf 实现，并且深度支持 Direct Bufer 等技术，甚至 hack 了 Java 内部对 Direct Bufer 的分配和销毁等。同时，Netty 也提供了更 加完善的 Scatter/Gather 机制实现。

可以看到，Netty 的能力范围大大超过了 Java 核心类库中的 NIO 等 API，可以说它是一个从应用视角出发的产物。

更加优雅的 Reactor 模式实现、灵活的线程模型、利用 EventLoop 等创新性的机制，可以非常高效地管理成百上千的 Channel。

充分利用了 Java 的 Zero-Copy 机制，并且从多种角度，“斤斤计较”般的降低内存分配和回收的开销。例如，使用池化的 Direct Bufer 等技术，在提高 IO 性能的同时，减少了对象 的创建和销毁；利用反射等技术直接操纵 SelectionKey，使用数组而不是 Java 容器等。

使用更多本地代码。例如，直接利用 JNI 调用 Open SSL 等方式，获得比 Java 内建 SSL 引擎更好的性能。

在通信协议、序列化等其他角度的优化。

总的来说，Netty 并没有 Java 核心类库那些强烈的通用性、跨平台等各种负担，针对性能等特定目标以及 Linux 等特定环境，采取了一些极致的优化手段。

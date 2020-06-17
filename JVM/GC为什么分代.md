# Java 的 GC 为什么要分代？

作者：RednaxelaFX
链接：https://www.zhihu.com/question/53613423/answer/135743258
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### GC ROOT 是啥？

所谓“GC roots”，或者说 tracing GC 的“根集合”，就是一组必须活跃的引用。

例如说，这些引用可能包括：

（1）所有 Java 线程当前活跃的栈帧里指向 GC 堆里的对象的引用；换句话说，当前所有正在被调用的方法的引用类型的参数/局部变量/临时值。

（2）VM 的一些静态数据结构里指向 GC 堆里的对象的引用，例如说 HotSpot VM 里的 Universe 里有很多这样的引用。

（3）JNI handles，包括 global handles 和 local handles

- （看情况）所有当前被加载的 Java 类
- （看情况）Java 类的引用类型静态变量
- （看情况）Java 类的运行时常量池里的引用类型常量（String 或 Class 类型）
- （看情况）String 常量池（StringTable）里的引用

注意，是一组必须活跃的引用，不是对象。

### Tracing GC 的根本思路？

就是：给定一个集合的引用作为根出发，通过引用关系遍历对象图，能被遍历到的（可到达的）对象就被判定为存活，其余对象（也就是没有被遍历到的）就自然被判定为死亡。

注意再注意：tracing GC 的本质是通过找出所有活对象来把其余空间认定为“无用”，而不是找出所有死掉的对象并回收它们占用的空间。

GC roots 这组引用是 tracing GC 的起点。要实现语义正确的 tracing GC，就必须要能完整枚举出所有的 GC roots，否则就可能会漏扫描应该存活的对象，导致 GC 错误回收了这些被漏扫的活对象。

### 分代式 GC 对 GC roots 的定义有什么影响呢？

答案是：分代式 GC 是一种部分收集（partial collection）的做法。

在执行部分收集时，从 GC 堆的非收集部分指向收集部分的引用，也必须作为 GC roots 的一部分。

具体到分两代的分代式 GC 来说，如果第 0 代叫做 young gen，第 1 代叫做 old gen，那么如果有 minor GC / young GC 只收集 young gen 里的垃圾，则 young gen 属于“收集部分”，而 old gen 属于“非收集部分”，那么从 old gen 指向 young gen 的引用就必须作为 minor GC / young GC 的 GC roots 的一部分。

继续具体到 HotSpot VM 里的分两代式 GC 来说，除了 old gen 到 young gen 的引用之外，有些带有弱引用语义的结构，例如说记录所有当前被加载的类的 SystemDictionary、记录字符串常量引用的 StringTable 等，在 young GC 时必须要作为 strong GC roots，而在收集整堆的 full GC 时则不会被看作 strong GC roots。

换句话说，young GC 比 full GC 的 GC roots 还要更大一些。如果不能理解这个道理，那整个讨论也就无从谈起了。

### 那么分代有什么好处？

对传统的、基本的 GC 实现来说，由于它们在 GC 的整个工作过程中都要“stop-the-world”，如果能想办法缩短 GC 一次工作的时间长度就是件重要的事情。如果说收集整个 GC 堆耗时太长，那不如只收集其中的一部分？

于是就有好几种不同的划分（partition）GC 堆的方式来实现部分收集，而分代式 GC 就是这其中的一个思路。

这个思路所基于的基本假设大家都很熟悉了：weak generational hypothesis——大部分对象的生命期很短（die young），而没有 die young 的对象则很可能会存活很长时间（live long）。

这是对过往的很多应用行为分析之后得出的一个假设。基于这个假设，如果让新创建的对象都在 young gen 里创建，然后频繁收集 young gen，则大部分垃圾都能在 young GC 中被收集掉。由于 young gen 的大小配置通常只占整个 GC 堆的较小部分，而且较高的对象死亡率（或者说较低的对象存活率）让它非常适合使用 copying 算法来收集，这样就不但能降低单次 GC 的时间长度，还可以提高 GC 的工作效率。

但是！有些比较先进的 GC 算法是增量式（incremental）的，或者部分并发（mostly-concurrent），或者干脆完全并发（fully-concurrent）的。

对于这些 GC 来说，解决 stop-the-world 时间太长的问题并不是选择分代的主要原因。

就 Azul 的 Pauless 到 C4 的发展历程来看，选择实现分代的最大好处是，GC 能够应付的应用内存分配速率（allocation rate）可以得到巨大的提升。

并发 GC 根本上要跟应用玩追赶游戏：应用一边在分配，GC 一边在收集，如果 GC 收集的速度能跟得上应用分配的速度，那就一切都很完美；一旦 GC 开始跟不上了，垃圾就会渐渐堆积起来，最终到可用空间彻底耗尽的时候，应用的分配请求就只能暂时等一等了，等 GC 追赶上来。

所以，对于一个并发 GC 来说，能够尽快回收出越多空间，就能够应付越高的应用内存分配速率，从而更好地保持 GC 以完美的并发模式工作。虽然并不是所有应用中的对象生命周期都完美吻合 weak generational hypothesis 的假设，但这个假设在很大范围内还是适用的，因而也可以帮助并发 GC 改善性能。就先写这么多…

### Major GC 和 Full GC 的区别是什么？

作者：RednaxelaFX
链接：https://www.zhihu.com/question/41922036/answer/93079526
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

针对 HotSpot VM 的实现，它里面的 GC 其实准确分类只有两大种：

（1）Partial GC：并不收集整个 GC 堆的模式

- Young GC：只收集 young gen 的 GC
- Old GC：只收集 old gen 的 GC。只有 CMS 的 concurrent collection 是这个模式
- Mixed GC：收集整个 young gen 以及部分 old gen 的 GC。只有 G1 有这个模式

（2）Full GC：收集整个堆，包括 young gen、old gen、perm gen（如果存在的话）等所有部分的模式。

Major GC 通常是跟 full GC 是等价的，收集整个 GC 堆。但因为 HotSpot VM 发展了这么多年，外界对各种名词的解读已经完全混乱了，当有人说“major GC”的时候一定要问清楚他想要指的是上面的 full GC 还是 old GC。

最简单的分代式 GC 策略，按 HotSpot VM 的 serial GC 的实现来看，触发条件是：

- young GC：当 young gen 中的 eden 区分配满的时候触发。注意 young GC 中有部分存活对象会晋升到 old gen，所以 young GC 后 old gen 的占用量通常会有所升高。
- full GC：当准备要触发一次 young GC 时，如果发现统计数据说之前 young GC 的平均晋升大小比目前 old gen 剩余的空间大，则不会触发 young GC 而是转为触发 full GC（因为 HotSpot VM 的 GC 里，除了 CMS 的 concurrent collection 之外，其它能收集 old gen 的 GC 都会同时收集整个 GC 堆，包括 young gen，所以不需要事先触发一次单独的 young GC）；或者，如果有 perm gen 的话，要在 perm gen 分配空间但已经没有足够空间时，也要触发一次 full GC；或者 System.gc()、heap dump 带 GC，默认也是触发 full GC。

HotSpot VM 里其它非并发 GC 的触发条件复杂一些，不过大致的原理与上面说的其实一样。

当然也总有例外。Parallel Scavenge（-XX:+UseParallelGC）框架下，默认是在要触发 full GC 前先执行一次 young GC，并且两次 GC 之间能让应用程序稍微运行一小下，以期降低 full GC 的暂停时间（因为 young GC 会尽量清理了 young gen 的死对象，减少了 full GC 的工作量）。控制这个行为的 VM 参数是-XX:+ScavengeBeforeFullGC。这是 HotSpot VM 里的奇葩嗯。

并发 GC 的触发条件就不太一样。以 CMS GC 为例，它主要是定时去检查 old gen 的使用量，当使用量超过了触发比例就会启动一次 CMS GC，对 old gen 做并发收集。

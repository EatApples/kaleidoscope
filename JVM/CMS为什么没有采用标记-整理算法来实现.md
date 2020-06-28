### CMS 为什么没有采用标记-整理算法来实现？

https://hllvm-group.iteye.com/group/topic/38223#post-248757

分代式 GC 里，年老代常用 mark-sweep；或者是 mark-sweep/mark-compact 的混合方式，一般情况下用 mark-sweep，统计估算碎片量达到一定程度时用 mark-compact。这是因为传统上大家认为年老代的对象可能会长时间存活且存活率高，或者是比较大，这样拷贝起来不划算，还不如采用就地收集的方式。

Mark-sweep、mark-compact、copying 这三种基本算法里，只有 mark-sweep 是不移动对象（也就是不用拷贝）的，所以选用 mark-sweep。

简要对比三种基本算法：
| | mark-sweep | mark-compact | copying |
|------------|--------------------|------------------|-----------------------------------------|
| 速度 | 中等 | 最慢 | 最快 |
| 空间开销 | 少（但会堆积碎片） | 少（不堆积碎片） | 通常需要活对象的 2 倍大小（不堆积碎片） |
| 移动对象？ | 否 | 是 | 是 |

在分代式假设中，年轻代中的对象在 minor GC 时的存活率应该很低，这样用 copying 算法就是最合算的，因为其时间开销与活对象的大小成正比，如果没多少活对象，它就非常快；而且 young gen 本身应该比较小，就算需要 2 倍空间也只会浪费不太多的空间。

而年老代被 GC 时对象存活率可能会很高，而且假定可用剩余空间不太多，这样 copying 算法就不太合适，于是更可能选用另两种算法，特别是不用移动对象的 mark-sweep 算法。

不过 HotSpot VM 中除了 CMS 之外的其它收集器都是会移动对象的，也就是要么是 copying、要么是 mark-compact 的变种。

把 GC 之外的代码（主要是应用程序的逻辑）叫做 mutator，把 GC 的代码叫做 collector。两者之间需要保持同步，这样才可以保证两者所观察到的对象图是一致的。

如果有一个串行、不并发、不分代、不增量式的 collector，那么它在工作的时候总是能观察到整个对象图。因而它跟 mutator 之间的同步方式非常简单：mutator 一侧不用做任何特殊的事情，只要在需要 GC 时同步调用 collector 即可，就跟普通函数调用一样。

如果有一个分代式的，或者增量式的 collector，那它在工作的时候就只会观察到整个对象图的一部分；它观察不到的部分就有可能与 mutator 产生不一致，于是需要 mutator 配合：它与 mutator 之间需要额外的同步。Mutator 在改变对象图中的引用关系时必须执行一些额外代码，让 collector 记录下这些变化。有两种做法，一种是 write barrier，一种是 read barrier。

Write barrier 就是当改写一个引用时：

```
a.x = b
```

插入一块额外的代码，变成：

```
write_barrier(a, &(a->x), b);
a->x = b;
```

Read barrier 就是当读取一个引用时：

```
b = a.x
```

插入一块额外的代码，变成：

```
read_barrier(&(a->x));
b = a->x;
```

通常一个程序里对引用的读远比对引用的写要更频繁，所以通常认为 read barrier 的开销远大于 write barrier，所以很少有 GC 使用 read barrier。

如果只用 write barrier，那么“移动对象”这个动作就必须要完全暂停 mutator，让 collector 把对象都移动好，然后把指针都修正好，接下来才可以恢复 mutator 的执行。也就是说 collector“移动对象”这个动作无法与 mutator 并发进行。

如果用到了 read barrier（虽少见但不是不存在，例如 Azul C4 Collector），那移动对象就可以单个单个的进行，而且不需要立即修正所有的指针，所以可以看作整个过程 collector 都与 mutator 是并发的。

CMS 没有使用 read barrier，只用了 write barrier。这样，如果它要选用 mark-compact 为基本算法的话，就只有 mark 阶段可以并发执行（其中 root scanning 阶段仍然需要暂停 mutator，这是 initial marking；后面的 concurrent marking 才可以跟 mutator 并发执行），然后整个 compact 阶段都要暂停 mutator。回想最初提到的：compact 阶段的时间开销与活对象的大小成正比，这对年老代来说就不划算了。

于是选用 mark-sweep 为基本算法就是很合理的选择：mark 与 sweep 阶段都可以与 mutator 并发执行。Sweep 阶段由于不移动对象所以不用修正指针，所以不用暂停 mutator。

那碎片堆积起来了怎么办呢？HotSpot VM 里 CMS 只负责并发收集年老代（而不是整个 GC 堆）。如果并发收集所回收到的空间赶不上分配的需求，就会回退到使用 serial GC 的 mark-compact 算法做 full GC。也就是 mark-sweep 为主，mark-compact 为备份的经典配置。但这种配置方式也埋下了隐患：使用 CMS 时必须非常小心的调优，尽量推迟由碎片化引致的 full GC 的发生。一旦发生 full GC，暂停时间可能又会很长，这样原本为低延迟而选择 CMS 的优势就没了。

所以新的 Garbage-First（G1）GC 就回到了以 copying 为基础的算法上，把整个 GC 堆划分为许多小区域（region），通过每次 GC 只选择收集很少量 region 来控制移动对象带来的暂停时间。这样既能实现低延迟也不会受碎片化的影响。
（注意：G1 虽然有 concurrent global marking，但那是可选的，真正带来暂停时间的工作仍然是基于 copying 算法而不是 mark-compact 的）

### Remembered Set

https://hllvm-group.iteye.com/group/topic/21468#post-272070
https://hllvm-group.iteye.com/group/topic/44381#post-272188

Remembered Set 是一种抽象概念，而 card table 可以是 remembered set 的一种实现方式。

Remembered Set 是在实现部分垃圾收集（partial GC）时用于记录从非收集部分指向收集部分的指针的集合的抽象数据结构。

分代式 GC 是一种部分垃圾收集的实现方式。当分两代时，通常把这两代叫做 young gen 和 old gen；通常能单独收集的只是 young gen。此时 remembered set 记录的就是从 old gen 指向 young gen 的跨代指针。

Regional collector 也是一种部分垃圾收集的实现方式。此时 remembered set 就要记录跨 region 的指针。

不过就像平时讨论 GC 时大家只关心 tracing GC 而通常不把 reference counting 算在里面（严格说 reference counting 也是一种 GC），remembered set 与 card table 也有一些平时讨论时隐含的假设（虽然严格说那些假设并不必要）。

Points-into remembered set

G1 GC 的 heap 与 HotSpot VM 的其它 GC 一样有一个覆盖整个 heap 的 card table。

逻辑上说，G1 GC 的 remembered set（下面简称 RSet）是每个 region 有一份。这个 RSet 记录的是从别的 region 指向该 region 的 card。所以这是一种“points-into”的 remembered set。

用 card table 实现的 remembered set 通常是 points-out 的，也就是说 card table 要记录的是从它覆盖的范围出发指向别的范围的指针。以分代式 GC 的 card table 为例，要记录 old -> young 的跨代指针，被标记的 card 是 old gen 范围内的。

G1 GC 则是在 points-out 的 card table 之上再加了一层结构来构成 points-into RSet：每个 region 会记录下到底哪些别的 region 有指向自己的指针，而这些指针分别在哪些 card 的范围内。这个 RSet 其实是一个 hash table，key 是别的 region 的起始地址，value 是一个集合，里面的元素是 card table 的 index。

举例来说，如果 region A 的 RSet 里有一项的 key 是 region B，value 里有 index 为 1234 的 card，它的意思就是 region B 的一个 card 里有引用指向 region A。所以对 region A 来说，该 RSet 记录的是 points-into 的关系；而 card table 仍然记录了 points-out 的关系。

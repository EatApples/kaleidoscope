### Remembered Set
https://hllvm-group.iteye.com/group/topic/21468#post-272070


Remembered Set是一种抽象概念，而card table可以是remembered set的一种实现方式。

Remembered Set是在实现部分垃圾收集（partial GC）时用于记录从非收集部分指向收集部分的指针的集合的抽象数据结构。

分代式GC是一种部分垃圾收集的实现方式。当分两代时，通常把这两代叫做young gen和old gen；通常能单独收集的只是young gen。此时remembered set记录的就是从old gen指向young gen的跨代指针。

Regional collector也是一种部分垃圾收集的实现方式。此时remembered set就要记录跨region的指针。

不过就像平时讨论GC时大家只关心tracing GC而通常不把reference counting算在里面（严格说reference counting也是一种GC），remembered set与card table也有一些平时讨论时隐含的假设（虽然严格说那些假设并不必要）。
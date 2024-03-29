### CMS 的重新标记做了啥

CMS：incremental update write barrier，也叫做 insertion barrier；
G1：SATB write barrier（Snapshot-At-The-Beginning），也叫做 deletion barrier。

作者：fleuria
链接：https://www.zhihu.com/question/37028283/answer/438344734
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

R 大上面提到的 INC write barrier 和 SATB write barrier 是答案的关键字。

INC write barrier 和 SATB write barrier 都属于 Concurrent Marking Write Barrier，与 Remember Set 用到的 Write Barrier 有区分。

Concurrent Marking Write Barrier 用于维护 concurrent mark 的中间状态。在 Concurrent Mark 开始后，对象图随着用户的线程的改动而不停地变化，要跟踪对象图的变化，就需要通过 Write Barrier 去 hook 每一次引用赋值，比较自然能想到的是，如果一个对象的某字段指向新的对象，则把新对象加入 mark stack 等待遍历：

write_barrier_ref(Object* src, Object\*\* slot, Object* new_ref)
{ \*slot = new_ref;
if( is_marked(src) )
enqueue(new_ref);
}

这样的 write barrier 属于 Increment Update 的方式，简称 INC，表示它会增量同步堆内的对象图。但是 INC barrier 存有一项缺陷：无法发现 Concurrent Mark 期间堆外根集（寄存器、栈）的引用变化。为此在 concurrent mark 之后，需要一个 remark 阶段再 STW 扫一遍根集，这是 CMS GC 的特点，有时 remark 阶段时间并不短。

（1）并发标记阶段会漏掉一下老年代的存活对象，这是因为在并发的过程中，用户应用线程可能会对老年代的对象产生引用上的改变。某一些被改变的标记可能会被遗漏。
（2）并发预清理(Concurrent Preclean)主要目的是减少重标记（Remark）步骤 Stop-the-World 的时间。这一步同样也是并发的，不会停止用户应用线程。在前面的并发标记中，一些引用被改变了。当某一块块（Card）中的对象引用发生改变时，JVM 会标记这个空间为“脏块”（Dirty Card）。在预清理阶段，JVM 根据之前记录的这些“脏对象”重新标记了他们新的可达对象。这一步结束后空间重新进入 clean 状态。另外，一些必要的最终重标记之前的准备步骤也会在这一步做好。
（3）重标记(Remark)这是 CMS 算法中第二个会触发 Stop-the-World 事件的步骤，由于前一步是一个并发的步骤，预清理的速度可能会赶不上用户应用对对象改变的速度，所以需要一个 Stop-the-World 的暂停来完整的标记所有对象结束整个标记阶段。通常 CMS 会在年轻代为空时来运行重标记阶段，以此避免一个接一个的 Stop-the-World 阶段。

G1GC 用的 Snapshot-At-The-Beginning 的 Write Barrier 是另一条思路，并不持续跟踪对象图的变化，而是打下 concurrent mark 那一刻的快照，而将之后所有新分配的对象统统视为活跃不管，做到：

- Anything live at Initial Marking is considered live.
- Anything allocated since Initial Marking is considered live.

SATB Barrier 不需要最后的 remark，代价就是因为新分配的对象统统视为活跃，有更多的 float garbage。最后回收到的垃圾对象，一定是开始 mark 那一刻之前产生的垃圾。这时要跟踪的不是新引用的赋值，反而是旧引用的被解除，以维持快照时刻的对象关系：

write_barrier_slot(Object* src, Object\*\* slot, Object* new_ref) {
old_ref = *slot;
if( !is_marked(old_ref) ){
enqueue(old_ref);
}
*slot = new_ref;
}

但是 G1GC 为什么仍有最终标记阶段？G1GC 中 Write Barrier 产生的标记并不是实时更新的，而会记录在本线程的 update buffer 中（它扮演的角色有点类似 golang 里的 chan？），当写满一个 buffer 后，再把整个 buffer 加入到全局的 update buffer 队列中，供 Refinement Thread 消费来真正地做 Mark。到最终标记阶段，需要做的事情就是把这些 buffer 都给 flush 出来，完成所有标记，这点与 CMS 的 Remark 有很大不同

#### 1. 内存管理设计精要

https://draveness.me/system-design-memory-management/

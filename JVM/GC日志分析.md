JVM 层 GC 调优（下）
https://blog.51cto.com/zero01/2150696

### -XX:+UseParallelGC

```
2018-07-25T21:07:10.876+0800: 0.293:        # GC发生的时间
[GC (Allocation Failure)                    # 触发GC的原因
Desired survivor size 524288 bytes,         # 期望存活对象的大小为524288字节
new threshold 7 (max 15)                    # 存活的对象年龄为7，最大值为15，表示年龄大于15岁则会晋升到老年代
[PSYoungGen:                                # GC发生的区域，可以看到这里是Young区
4088K->496K(4608K)]                         # GC后的Young区内存占用大小从4088K减小到了496K，括号里的4608K是Young区的总大小
4088K->1599K(15872K),                       # GC后堆的内存占用大小从4088K减小到了1599K，括号里的15872K是堆的总大小
0.0045483 secs]                             # 本次GC总耗费的时间，单位为秒
[Times: user=0.01 sys=0.00, real=0.00 secs] # 本次GC所耗费的时间的统计信息，user是用户态耗费的时间，sys是内核态耗费的时间，real是整个过程实际花费的时间。单位为秒
```

### CMS

```
2018-07-26T10:13:12.984+0800: 0.296:        # GC发生的时间
[GC (Allocation Failure)                    # 触发GC的原因，只是GC的话就表示的是Young GC，Full GC会显示为Full GC
2018-07-26T10:13:12.985+0800: 0.296:        # GC发生的时间
[ParNew                                     # 搭配的年轻代收集器为ParNew，因为CMS是老年代的收集器
Desired survivor size 262144 bytes,         # 期望存活对象的大小为262144字节
new threshold 1 (max 6)                     # 存活的对象年龄为1，最大值为6，表示年龄大于6岁则会晋升到老年代
- age   1:                                  # 当前存活对象的年龄
521072 bytes,     521072 total              # 存活对象所占的Young区内存大小，单位为字节
: 4416K->512K(4928K),                       # GC后的Young区内存占用大小从4416K减小到了512K，括号里的4928K是Young区的总大小
0.0082787 secs]                             # 本次Yong GC所耗费的时间，单位为秒
4416K->1598K(15872K),                       # GC后堆的内存占用大小从4416K减小到了1598K，括号里的15872K是堆的总大小
0.0083763 secs]                             # 本次GC总耗费的时间，单位为秒
[Times: user=0.00 sys=0.00, real=0.01 secs] # 本次GC所耗费的时间的统计信息，user是用户态耗费的时间，sys是内核态耗费的时间，real是整个过程实际花费的时间。单位为秒

老年代的GC信息，以下这个片段是CMS GC特有的日志信息格式，也就是完整的一次Full GC过程：

2018-07-26T10:13:14.275+0800: 1.586:          # 标记开始的时间
[GC (CMS Initial Mark) [1 CMS-initial-mark:   # 表示初始化标记
6375K(10944K)]                                # 初始化标记后Old区的内存占用大小为6375K，括号里的10944K是Old区的总大小
7541K(15872K),                                # 初始化标记后堆的内存占用大小为7541K，括号里的15872K是堆的总大小
0.0017697 secs]                               # 本次初始化标记所耗时
[Times: user=0.00 sys=0.00, real=0.00 secs]   # 之前已介绍过了，不再赘述

2018-07-26T10:13:14.277+0800: 1.588: [CMS-concurrent-mark-start]
                                              # 开始并发标记

2018-07-26T10:13:14.288+0800: 1.599: [CMS-concurrent-mark: 0.011/0.011 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
                                              # 并发标记结束，以及本次并发标记所耗时

2018-07-26T10:13:14.288+0800: 1.599: [CMS-concurrent-preclean-start]
                                              # 开始标记预清理

2018-07-26T10:13:14.288+0800: 1.599: [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]                                               # 标记预清理结束，以及本次标记预清理所耗时

2018-07-26T10:13:14.303+0800: 1.614: [GC (CMS Final Remark)
                                    # 收集阶段，这个阶段会标记老年代全部的存活对象，包括那些在并发标记阶段更改的或者新创建的引用对象
[YG occupancy: 1645 K (4928 K)]     # young区当前占用为1645K，括号里的4928k是young区的总大小
2018-07-26T10:13:14.303+0800: 1.614: [Rescan (parallel) , 0.0018194 secs]      # 在应用停止的阶段完成存活对象的标记工作，以及耗时
2018-07-26T10:13:14.304+0800: 1.616: [weak refs processing, 0.0002306 secs]    # 弱引用处理，以及耗时
2018-07-26T10:13:14.305+0800: 1.616: [class unloading, 0.0019443 secs]         # 类卸载，以及耗时
2018-07-26T10:13:14.307+0800: 1.618: [scrub symbol table, 0.0011837 secs]      # 清理符号表，以及耗时
2018-07-26T10:13:14.308+0800: 1.619: [scrub string table, 0.0003768 secs]      # 清理字符串表，以及耗时
[1 CMS-remark: 6375K(10944K)]        # remark后Old区的内存占用大小为6375K，括号里的10944K是Old区的总大小
8020K(15872K),                       # remark后堆的内存占用大小为8020K，括号里的15872K是堆的总大小
0.0058148 secs]                      # 本次remark所耗时
[Times: user=0.00 sys=0.00, real=0.00 secs]                                    # 之前已介绍过了，不再赘述

2018-07-26T10:13:14.316+0800: 1.628: [CMS-concurrent-sweep-start]              # 开始清理标记

2018-07-26T10:13:14.318+0800: 1.630: [CMS-concurrent-sweep: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
                                     # 清理标记结束，以及本次清理标记所耗时

2018-07-26T10:13:14.318+0800: 1.630: [CMS-concurrent-reset-start]             # 开始重置标记

2018-07-26T10:13:14.319+0800: 1.631: [CMS-concurrent-reset: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
                                     # 重置标记结束，以及本次重置标记所耗时

```

### G1

```
2018-07-26T11:07:32.970+0800: 0.222: [GC pause (G1 Evacuation Pause) (young)    # 触发GC的原因及触发的是哪个区的GC
Desired survivor size 524288 bytes,                                             # 期望存活对象的大小为524288字节
new threshold 15 (max 15)                                 # 当前存活的对象年龄为15，最大值为15，表示年龄大于15岁则会晋升到老年代
, 0.0039562 secs]                                         # GC耗时
   [Parallel Time: 3.1 ms,                                # 停止应用的时间
   GC Workers: 1]                                         # GC工作线程数
      [GC Worker Start (ms):  222.0]                      # GC工作线程启动的时间点
      [Ext Root Scanning (ms):  1.5]                      # 根节点扫描，以及耗时
      [Update RS (ms):  0.0]                              # 每一个线程更新Remembered Sets花费的时间
         [Processed Buffers:  0]                          # 每一个线程处理的Update Buffers的数量
      [Scan RS (ms):  0.0]                                # 每一个工作线程扫描Remembered Sets花费的时间
      [Code Root Scanning (ms):  0.1]                     # 根节点扫描耗时
      [Object Copy (ms):  1.5]              # 每一个工作线程把Collection Sets的区域里的活跃对象复制到另一个区域里花费时间
      [Termination (ms):  0.0]              # 每一个工作线程提供中断花费的时间
         [Termination Attempts:  1]         # 每一个线程提供中断的次数
      [GC Worker Other (ms):  0.0]          # 每个工作线程执行其他任务（上述未统计的内容）的耗时
      [GC Worker Total (ms):  3.1]          # 每一个工作线程的总生存时间
      [GC Worker End (ms):  225.1]          # 每一个工作线程的终止时间
   [Code Root Fixup: 0.0 ms]                # 修正根节点耗时
   [Code Root Purge: 0.0 ms]                # 根节点清除耗时
   [Clear CT: 0.0 ms]                       # 清理CT（Card Table）的耗时
   [Other: 0.8 ms]                                       # 其他任务（上述未统计的内容）的耗时
      [Choose CSet: 0.0 ms]                              # 选择分区的耗时
      [Ref Proc: 0.7 ms]                                 # 执行关联（Reference objects）的耗时
      [Ref Enq: 0.0 ms]                                  # 将references放入ReferenceQueues的耗时
      [Redirty Cards: 0.0 ms]                            # 清理垃圾碎片耗时
      [Humongous Register: 0.0 ms]                       # 注册大对象耗时
      [Humongous Reclaim: 0.0 ms]                        # 回收大对象耗时
      [Free CSet: 0.0 ms]                                # 释放CS（collection set）的耗时
   [Eden: 3072.0K(3072.0K)->0.0B(2048.0K)   # Eden区容量为3072.0K，使用了3072.0K，GC后变为0，容量目标大小增加到2048.0K。
   Survivors: 0.0B->1024.0K                 # Survivors区在GC前为0，GC后为1024.0K
   Heap: 3072.0K(16.0M)->1206.5K(16.0M)]    # GC前，堆容量为16M，使用3072.0K。GC后，Heap容量为16M，使用1206.5K。


G1收集器里还有一个全局并发标记（global concurrent marking）概念，触发时其日志格式如下，与CMS的标记过程有些类似


2018-07-26T11:07:34.649+0800: 1.901: [GC pause (G1 Evacuation Pause) (young) (initial-mark)
                                     # 初始化标记，从根上直接可达的对象都被标记，这个阶段背负着一个完整的疏散暂停

... 略 ...

2018-07-26T11:07:34.652+0800: 1.905: [GC concurrent-root-region-scan-start]
                                    # 开始并发根区扫描，扫描从初始标记阶段的存活对象直接可达的根区集合
2018-07-26T11:07:34.653+0800: 1.905: [GC concurrent-root-region-scan-end, 0.0003898 secs]
                                    # 并发根区扫描结束及耗时
2018-07-26T11:07:34.653+0800: 1.905: [GC concurrent-mark-start]
                                    # 开始并发标记存活的对象
2018-07-26T11:07:34.677+0800: 1.930: [GC concurrent-mark-end, 0.0245824 secs]
                                    # 并发标记结束及耗时

# 这个是stop-the-world的重新标记阶段。它完成了从上一个阶段的标记工作(SATB buffers processing)的剩余部分。
2018-07-26T11:07:34.678+0800: 1.930: [GC remark 2018-07-26T11:07:34.678+0800: 1.931: [Finalize Marking, 0.0000376 secs]
                                    # 完成重新标记及耗时
2018-07-26T11:07:34.678+0800: 1.931: [GC ref-proc, 0.0000413 secs]
2018-07-26T11:07:34.678+0800: 1.931: [Unloading, 0.0052688 secs], 0.0054642 secs]
                                    # 卸载及耗时
 [Times: user=0.00 sys=0.00, real=0.01 secs]

2018-07-26T11:07:34.684+0800: 1.936: [GC cleanup 12M->12M(16M), 0.0001071 secs]
                                    # 独占清理空的区域及耗时
 [Times: user=0.00 sys=0.00, real=0.00 secs]
```

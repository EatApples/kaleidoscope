### ParNew 和 PSYoungGen 和 DefNew 是一个东西么？

https://hllvm-group.iteye.com/group/topic/37095#post-242695
https://blogs.oracle.com/jonthecollector/our-collectors

串行收集器：
DefNew：是使用-XX:+UseSerialGC（新生代，老年代都使用串行回收收集器）。

并行收集器：
ParNew：是使用-XX:+UseParNewGC（新生代使用并行收集器，老年代使用串行回收收集器）或者-XX:+UseConcMarkSweepGC(新生代使用并行收集器，老年代使用 CMS)。

PSYoungGen：是使用-XX:+UseParallelOldGC（新生代，老年代都使用并行回收收集器）或者-XX:+UseParallelGC（新生代使用并行回收收集器，老年代使用串行收集器）

garbage-first heap：是使用-XX:+UseG1GC（G1 收集器）

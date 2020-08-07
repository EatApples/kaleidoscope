“concurrent mode failure”问题排查
https://blog.csdn.net/yangguosb/article/details/79857844

### concurrent mode failure 是什么？

CMS 垃圾收集器特有的错误，CMS 的垃圾清理和引用线程是并行进行的，如果在并行清理的过程中老年代的空间不足以容纳应用产生的垃圾，则会抛出“concurrent mode failure”。

### concurrent mode failure 影响

老年代的垃圾收集器从 CMS 退化为 Serial Old，所有应用线程被暂停，停顿时间变长。

### 可能原因及方案

原因 1：CMS 触发太晚
方案：将-XX:CMSInitiatingOccupancyFraction=N 调小；

原因 2：空间碎片太多
方案：开启空间碎片整理，并将空间碎片整理周期设置在合理范围；

-XX:+UseCMSCompactAtFullCollection （空间碎片整理）
-XX:CMSFullGCsBeforeCompaction=n

原因 3：垃圾产生速度超过清理速度

- 晋升阈值过小；
- Survivor 空间过小，导致溢出；
- Eden 区过小，导致晋升速率提高；
- 存在大对象；

https://blogs.oracle.com/poonam/understanding-cms-gc-logs

得知导致 concurrent mode failure 的原因有是： there was not enough space in the CMS generation to promote the worst case surviving young generation objects. We name this failure as “full promotion guarantee failure”

解决的方案有： The concurrent mode failure can either be avoided increasing the tenured generation size or initiating the CMS collection at a lesser heap occupancy by setting CMSInitiatingOccupancyFraction to a lower value and setting UseCMSInitiatingOccupancyOnly to true.

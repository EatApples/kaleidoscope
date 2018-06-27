### JDK定时任务对比
这是你从未体验的全新版本，只要三分钟，你就会和我一样，彻底了解JDK定时任务。

你将获得：

（1）Timer 的实现（定时任务是先设置下一次任务的执行时间，再执行本次任务）

（2）Timer 是单线程，只有要一个任务异常，所有任务挂起

（3）Timer 使用  System.currentTimeMillis()，强依赖操作系统时间

（4）ScheduledThreadPoolExecutor 的实现（定时任务是先执行本次任务，再设置下一次任务的执行时间）

（5）ScheduledThreadPoolExecutor 是基于线程池的（需要设置合理的线程池大小）

（6）ScheduledThreadPoolExecutor 使用nanoTime，和系统时间是完全无关的

### 参考资料
#### 1. Java 定时任务实现原理详解
https://blog.csdn.net/u013332124/article/details/79603943

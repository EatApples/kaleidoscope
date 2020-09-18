### java 中的获取时间方式，有什么不同

https://www.sohu.com/a/120143500_494943

### System.currentTimeMillis();

System.currentTimeMillis()就是返回的当前时间距离 1970/01/01 08:00:00 的毫秒数。

就实现上来说，currentTimeMillis 其实是通过 gettimeofday 来实现的。

### System.nanoTime();

Linux::supports_monotonic_clock 决定了走哪个具体的分支：

（1）如果 linux 支持 CLOCK_MONOTONIC，那返回的值是从操作系统启动的时间到现在的相对纳秒数，

- CLOCK_MONOTONIC：从系统启动这一刻起开始计时，不受系统时间被用户改变的影响；
- CLOCK_REALTIME：系统实时时间，随系统实时时间改变而改变,即从 UTC1970-1-1 0:0:0 开始计时；

最终是调用的 clock_gettime 系统调用。因此 nanoTime 其实算出来的是一个相对的时间，相对于系统启动的时候的时间。

（2）如果不支持 CLOCK_MONOTONIC，则调用 gettimeofday，获得毫秒数，并转换成纳秒数。

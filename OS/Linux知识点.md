### 1. Buffers/Cached

buffer 和 cache 是两个在计算机技术中被用滥的名词，放在不通语境下会有不同的意义。

在内存管理中，我们需要特别澄清一下，

这里的 buffer 指 Linux 内存中的：Buffer cache(缓冲区缓存)。

这里的 cache 指 Linux 内存中的：Page cache(页面缓存)。

翻译成中文可以叫做缓冲区缓存和页面缓存。

在历史上，它们一个（buffer）被用来当成对 io 设备写的缓存，而另一个（cache）被用来当作对 io 设备的读缓存，这里的 io 设备，主要指的是块设备文件和文件系统上的普通文件。

但是现在，它们的意义已经不一样了。在当前的内核中，page cache 顾名思义就是针对内存页的缓存，说白了就是，如果有内存是以 page 进行分配管理的，都可以使用 page cache 作为其缓存来使用。

当然，不是所有的内存都是以页（page）进行管理的，也有很多是针对块（block）进行管理的，这部分内存使用如果要用到 cache 功能，则都集中到 buffer cache 中来使用。（从这个角度出发，是不是 buffer cache 改名叫做 block cache 更好？）然而，也不是所有块（block）都有固定长度，系统上块的长度主要是根据所使用的块设备决定的，而页长度在 X86 上无论是 32 位还是 64 位都是 4k。

Page cache 主要用来作为文件系统上的文件数据的缓存来用，尤其是针对当进程对文件有 read/write 操作的时候。

Buffer cache 的主要功能：在系统对块设备进行读写时，对块进行数据缓存。

### 2. 如何回收 cache？

Linux 内核会在内存将要耗尽的时候，触发内存回收的工作，以便释放出内存给急需内存的进程使用。

一般情况下，这个操作中主要的内存释放都来自于对 buffer／cache 的释放。尤其是 cache 空间，它主要用来做缓存，只是在内存够用的时候加快进程对文件的读写速度，那么在内存压力较大的情况下，当然有必要清空释放 cache，作为 free 空间分给相关进程使用。所以一般情况下，我们认为 buffer/cache 空间可以被释放，这个理解是正确的。

但是这种清缓存的工作也并不是没有成本。清缓存必须保证 cache 中的数据跟对应文件中的数据一致，才能对 cache 进行释放。所以伴随着 cache 清除的行为，一般都是系统 IO 飙高。因为内核要对比 cache 中的数据和对应硬盘文件上的数据是否一致，如果不一致需要写回，之后才能回收。

在系统中除了内存将被耗尽的时候可以清缓存以外，我们还可以使用下面这个文件来人工触发缓存清除的操作：

echo 1 > /proc/sys/vm/drop_caches: 表示清除 pagecache。

echo 2 > /proc/sys/vm/drop_caches: 表示清除回收 slab 分配器中的对象（包括目录项缓存和 inode 缓存）。slab 分配器是内核中管理内存的一种机制，其中很多缓存数据实现都是用的 pagecache。

echo 3 > /proc/sys/vm/drop_caches: 表示清除 pagecache 和 slab 分配器中的缓存对象。

### 3. cache 都能被回收么？

大家普遍认为，buffers 和 cached 所占用的内存空间是可以在内存压力较大的时候被释放当做空闲空间用的。

但真的是这样么？

当 cache 作为文件缓存被释放的时候会引发 IO 变高，这是 cache 加快文件访问速度所要付出的成本。

（1）tmpfs：tmpfs 中存储的文件会占用 cache 空间，除非文件删除否则这个 cache 不会被自动释放。

（2）共享内存：使用 shmget 方式申请的共享内存会占用 cache 空间，除非共享内存被 ipcrm 或者使用 shmctl 去 IPC_RMID，否则相关的 cache 空间都不会被自动释放。

（3）mmap：使用 mmap 方法申请的 MAP_SHARED 标志的内存会占用 cache 空间，除非进程将这段内存 munmap，否则相关的 cache 空间都不会被自动释放。

实际上 shmget、mmap 的共享内存，在内核层都是通过 tmpfs 实现的，tmpfs 实现的存储用的都是 cache。

我们应该明白，内存的使用并不是简单的概念，cache 也并不是真的可以当成空闲空间用的。

### 4. 文件映射（file）和匿名映射（anon）？

匿名映射举例：进程使用 malloc 申请内存，或者使用 mmap(MAP_ANONYMOUS 的方式)申请的内存。

文件映射举例：进程使用 mmap 映射文件系统上的文件，包括普通的文件，也包括临时文件系统（tmpfs）。另外，Sys V 的 IPC 和 POSIX 的 IPC （IPC 是进程间通信机制，在这里主要指共享内存，信号量数组和消息队列）也都是通过文件映射方式体现在用户空间内存中的。

### 5. 谁该被 SWAP？

首先是 Swap 机制。Swap 是交换技术，当内存不够用的时候，我们可以选择性的将一块磁盘、分区或者一个文件当成交换空间，将内存上一些临时用不到的数据放到交换空间上，以释放内存资源给急用的进程。

哪些数据可能会被交换出去呢？

内存管理需要将内存区分为活跃的（Active）和不活跃的（Inactive），再加上一个进程使用的 用户空间内存映射 包括文件映射（file）和匿名映射（anon），所以就包括了 Active（anon）、Inactive（anon）、Active（file）和 Inactive（file）。

文件 cache 是一定不需要 swap 的，因为是 cache，就意味着它本身就是硬盘上的文件（当然你现在应该知道了，它也不仅仅只有文件），那么如果是硬盘上的文件，就不用 swap 交换出去，只要写回脏数据，保持数据一致之后清除就可以了，这就是刚才说过的缓存清楚机制。

还要补充一点，如果内存被 mlock 标记加锁了，则也不会交换，这是对内存加 mlock 锁的唯一作用。

另外再说明一下，HugePages 也是不会交换的。

能交换出去的内存主要包括：

Inactive（anon 匿名映射）这部分内存。需要注意的是，内核也将共享内存作为计数统计进了 Inactive（anon）中去了（是的，共享内存也可以被 Swap）。

### 6. 何时 SWAP？

搞清楚了谁该 swap，那么还要知道什么时候该 swap。这看起来比较简单，内存耗尽而且 cache 也没什么可以回收的时候就应该触发 swap。其实现实情况也没这么简单，实际上系统在内存压力可能不大的情况下也会 swap。

如果没有 swap，内存耗尽的结果一般都是触发 oom killer，会杀掉此时积分比较高的进程。

### 7. 缺页异常(page fault)

进程申请内存可能用到很多种方法，最常见的就是 malloc 和 mmap。但是这对于我们并不重要，因为无论是 malloc 还是 mmap，或是其他的申请内存的方法，都不会真正的让内核去给进程分配一个实际的物理内存空间。真正会触发分配物理内存的行为是 缺页异常(page fault)。

缺页异常就是我们可以在 memory.stat 中看到的 total_pgfault，这种异常一般分两种，一种叫 major fault，另一种叫 minor fault。这两种异常的主要区别是，进程所请求的内存数据是否会引发磁盘 io？如果会引发，就是一个 major fault，如果不引发，那就是 minor fault。就是说如果产生了 major fault，这个数据基本上就意味着已经被交换到了 swap 空间上。

### 资料来源

#### 1. Linux cgroup - memory 子系统讲解

https://billtian.github.io/digoal.blog/2017/01/11/02.html

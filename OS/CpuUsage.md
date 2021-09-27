# CPU 使用率

（1）什么是 CPU 使用率

```log
 1. %CPU -- CPU Usage
 The task's share of the elapsed CPU time since the last screen update, expressed as a percentage of total CPU time.
 In a true SMP environment, if a process is multi-threaded and top is not operating in Threads mode, amounts greater than 100% may be reported. You toggle
 Threads mode with the `H' interactive command.
 Also for multi-processor environments, if Irix mode is Off, top will operate in Solaris mode where a task's cpu usage will be divided by the total number
 of CPUs. You toggle Irix/Solaris modes with the `I' interactive command.
```

以上截取自 man top 中对于 CPU 使用率的定义，总结来说某个进程的 CPU 使用率就是这个进程在一段时间内占用的 CPU 时间占总的 CPU 时间的百分比。

比如某个开启多线程的进程 1s 内占用了 CPU0 0.6s, CPU1 0.9s, 那么它的占用率是 150%。

（2）uptime 显示的平均负载

单位时间内，系统平均活跃进程数（可运行状态，不可中断状态），与 CPU 使用率没有关系。理想的情况是等于 CPU 的个数（CPU 个数通过 cat /proc/cpuinfo 查看）

（3）平均负载与 CPU 使用率

CPU 密集型进程，两者一致。IO 密集型进程，平均负载高，CPU 使用率不一定高

（4）统计 CPU 的使用情况

时钟中断：是一种硬中断，由时间硬件（系统定时器，一种可编程硬件）产生。当 CPU 接收到时钟中断信号后，会在处理完当前指令后调用 时钟中断处理程序 来完成更新系统时间、执行周期性任务等。

可以发现，统计 CPU 使用情况是在 时钟中断处理程序 中完成的。

每个 CPU 的使用情况通过 cpu_usage_stat 结构来记录，我们来看看其定义：

```cpp
struct cpu_usage_stat {
    cputime64_t user;
    cputime64_t nice;
    cputime64_t system;
    cputime64_t softirq;
    cputime64_t irq;
    cputime64_t idle;
    cputime64_t iowait;
    cputime64_t steal;
    cputime64_t guest;
};
```

从 cpu_usage_stat 结构的定义可以看出，其每个字段与 top 命令的 CPU 使用率类型一一对应。在内核初始化时，会为每个 CPU 创建一个 cpu_usage_stat 结构，用于统计 CPU 的使用情况。每次执行 时钟中断处理程序 都会调用 account_process_tick 函数进行 CPU 使用情况统计。

### 1. 系统的 CPU 使用率

平常我们使用 top 命令来查看系统的性能情况，在 top 命令中可以看到很多不同类型的 CPU 使用率。

示例：

```s
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

下面，我们来介绍一下这些 CPU 使用率的意义：

```s
us：user time，表示 CPU 执行用户进程的时间，包括 nice 时间。通常都是希望用户空间 CPU 越高越好。
sy：system time，表示 CPU 在内核运行的时间，包括 IRQ 和 softirq。系统 CPU 占用越高，表明系统某部分存在瓶颈。通常这个值越低越好。
ni：nice time，具有优先级的用户进程执行时占用的 CPU 利用率百分比。
id：idle time，表示系统处于空闲期，等待进程运行。
wa：waiting time，表示 CPU 在等待 IO 操作完成所花费的时间。系统不应该花费大量的时间来等待 IO 操作，否则就说明 IO 存在瓶颈。
hi：hard IRQ time，表示系统处理硬中断所花费的时间。
si：soft IRQ time，表示系统处理软中断所花费的时间。
st：steal time，被强制等待（involuntary wait）虚拟 CPU 的时间，此时 Hypervisor 在为另一个虚拟处理器服务。
```

要获取各个 CPU 的使用情况信息，可以通过读取 /proc/stat 文件获取

```s
/proc/stat

kernel/system statistics. Varies with architecture. Common entries include:
cpu 3357 0 4313 1362393
The amount of time, measured in units of USER_HZ (1/100ths of a second on most architectures, use sysconf(_SC_CLK_TCK) to obtain the right value), that the system spent in various states:

```

注意：时间的计量单位为 USER_HZ，一般为 10ms，具体地值通过 sysconf(\_SC_CLK_TCK) 获取

示例：

| CPU  | user     | nice | system   | idle       | iowait | irq | softirq | steal | guest | guest_nice |
| ---- | -------- | ---- | -------- | ---------- | ------ | --- | ------- | ----- | ----- | ---------- |
| cpu  | 24177107 | 5555 | 17717188 | 5582304389 | 668384 | 0   | 1707872 | 44535 | 0     | 0          |
| cpu0 | 6338734  | 1953 | 4723078  | 1394600226 | 166976 | 0   | 491538  | 12345 | 0     | 0          |
| cpu1 | 6121526  | 1516 | 4437878  | 1395397035 | 155085 | 0   | 429598  | 8472  | 0     | 0          |
| cpu2 | 5767712  | 1270 | 4180340  | 1396635128 | 143316 | 0   | 396138  | 7471  | 0     | 0          |
| cpu3 | 5949133  | 814  | 4375891  | 1395671998 | 203006 | 0   | 390597  | 16244 | 0     | 0          |

第一行代表所有 CPU 的总和，而第二行开始表示每个 CPU 核心的使用情况信息。

详细解释如下：

```log
user

(1) Time spent in user mode.

nice

(2) Time spent in user mode with low priority (nice).

system

(3) Time spent in system mode.

idle

(4) Time spent in the idle task. This value should be USER_HZ times the second entry in the /proc/uptime pseudo-file.

iowait (since Linux 2.5.41)

(5) Time waiting for I/O to complete.

irq (since Linux 2.6.0-test4)

(6) Time servicing interrupts.

softirq (since Linux 2.6.0-test4)

(7) Time servicing softirqs.

steal (since Linux 2.6.11)

(8) Stolen time, which is the time spent in other operating systems when running in a virtualized environment

guest (since Linux 2.6.24)

(9) Time spent running a virtual CPU for guest operating systems under the control of the Linux kernel.

guest_nice (since Linux 2.6.33)

(10) Time spent running a niced guest (virtual CPU for guest operating systems under the control of the Linux kernel).
```

所以，top 命令的 CPU 使用率计算公式如下：

```s
CPU时间=User time+Nice time+System time+Hardirq time+Softirq time+Waiting time+Idle time+Steal time
%us =(User time + Nice time)/CPU时间*100%
%sy=(System time + Hardirq time +Softirq time)/CPU时间*100%
%id=(Idle time)/CPU时间*100%
%ni=(Nice time)/CPU时间*100%
%wa=(Waiting time)/CPU时间*100%
%hi=(Hardirq time)/CPU时间*100%
%si=(Softirq time)/CPU时间*100%
%st=(Steal time)/CPU时间*100%
```

### 2. 进程的 CPU 使用率

公式一：某进程 cpu 使用率 = 该进程 cpu 时间 / 总 cpu 时间。

/proc/pid/stat 中可以得出进程自启动以来占用的 cpu 时间。以 bash 进程为例:

```s
79 (bash) S 46 79 79 34816 0 0 0 0 0 0 46 135 387954 4807 20 0 1 0 6114 232049254400 873 18446744073709551615 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
```

第 14 项 utime 和第 15 项 stime 分别表示 bash 自启动起来，执行用户代码态占用的时间和执行内核态代码占用的时间，单位是 clock tick，clock tick 是时间单位。这两项的详细解释如下（摘自 man proc）：

```s
(14) utime %lu
Amount of time that this process has been scheduled in user mode, measured in clock ticks (divide by sysconf(\_SC_CLK_TCK)). This includes
guest time, guest_time (time spent running a virtual CPU, see below), so that applications that are not aware of the guest time field do not
lose that time from their calculations.
(15) stime %lu
Amount of time that this process has been scheduled in kernel mode, measured in clock ticks (divide by sysconf(\_SC_CLK_TCK)).
每个 clock tick 占用多少时间呢？
```

可以通过 sysconf(\_SC_CLK_TCK)获取 1 秒内有多少个 clock tick（通常是 100）。也就是说 1 clock tick 为 1 / 100 秒。

有了上面的基础，

我们可以每隔 period 秒读取/proc/pid/stat，解析其中的 utime 和 stime，将其和(utime+stime)减去上一次采样时这两项的和(lastutime + laststime)，这就是 period 秒内该进程占用 CPU 的时间，单位为 clock tick。

总的 CPU 时间为 period \* sysconf(\_SC_CLK_TCK)，单位也为 clock tick。

所以公式如下：

公式二：某进程 cpu 使用率 = ((utime+stime) - (lastutime + laststime)) / period \* sysconf(\_SC_CLK_TCK)

top 源码示例：

```cpp
static void frame_make (void) {
    ...
   // whoa either first time or thread/task mode change, (re)prime the pump...
   if (Pseudo_row == PROC_XTRA) {
      procs_refresh();
      usleep(LIB_USLEEP);
      putp(Cap_clr_scr);
   } else
      putp(Batch ? "\n\n" : Cap_home);

   sysinfo_refresh(0);
   procs_refresh();
   cpus_refresh();
   ...
}

static void procs_refresh (void) {
   ...
   procps_uptime(&uptime_cur, NULL);
   et = uptime_cur - uptime_sav;
   if (et < 0.01) et = 0.005;
   uptime_sav = uptime_cur;
   // if in Solaris mode, adjust our scaling for all cpus
   Frame_etscale = 100.0f / ((float)Hertz * (float)et * (Rc.mode_irixps ? 1 : Cpu_cnt));
   ...
}

```

对于 procs_refresh 函数，调用了两次，通过 procps_uptime 统计进程的 CPU 使用时间，然后根据之前的公式一计算。

可以使用一下指令追踪 top 的调用，可以发现有 open("/proc/{pid}/stat")调用，且一个 pid 被调用两次：

```s
strace -o strace.top.log -T -tt -e trace=all top -bn 1
```

计算进程 CPU 使用率时，如果是 Irix 模式，则不对 CPU 核数做平均。

### 3. 线程的 CPU 使用率

与进程类似，通过读取 /proc/{pid}/task/{tid}/stat 获取线程 CPU 使用时间，进而算出 CPU 使用率。

```s
/proc/self
When a process accesses this magic symbolic link, it
resolves to the process's own /proc/[pid] directory.

/proc/[pid]/task (since Linux 2.6.0-test6)
This is a directory that contains one subdirectory for each thread in the process. The name of each subdirectory is the numerical thread ID ([tid]) of the thread (see gettid(2)). Within each of these subdirectories, there is a set of files with the same names and contents as under the /proc/[pid] directories. For attributes that are shared by all threads, the contents for each of the files under the task/[tid] subdirectories will be the same as in the corresponding file in the parent /proc/[pid] directory (e.g., in a multithreaded process, all of the task/[tid]/cwd files will have the same value as the /proc/[pid]/cwd file in the parent directory, since all of the threads in a process share a working directory). For attributes that are distinct for each thread, the corresponding files under task/[tid] may have different values (e.g., various fields in each of the task/[tid]/status files may be different for each thread).
```

### 4. MXBean 中的 CPU 使用率

#### 4.1 VirtualVM 是如何展示线程 CPU 使用率的

这里以 VirtualVM 的源码举例。用来说明 VirtualVM 的线程 CPU 使用率是通过 ThreadMXBean.getThreadCpuTime 获得，然后通过公式：
线程 CPU 使用时间/CPU 总时间 求得的。

（1）CPUSamplerSupport#doRefreshImpl(threadCPUTimer, threadCPUView);

```java
if (threadsCPU != null) {
    threadCPUTimer = new javax.swing.Timer(refreshRate, new ActionListener() {
        public void actionPerformed(ActionEvent e) {
            threadCPURefresher.refresh();
        }
    });
    threadCPURefresher.setRefreshRate(refreshRate);
}

public synchronized final void refresh() {
    if (checkRefresh()) {
        long currentTime = System.currentTimeMillis();
        if (currentTime - lastRefresh >= REFRESH_THRESHOLD) {
            lastRefresh = currentTime;
            doRefresh();
        }
    }
}
```

CPU 采样时，会定时调用 doRefresh 函数

（2）ThreadsCPUView#refresh

```java
if (currentThreadsInfo != null) {
    threadCPUInfoPerSec = currentThreadsInfo.getCPUTimePerSecond(info);
}
currentThreadsInfo = info;
```

首先记录上一次的时间 threadCPUInfoPerSec 。

（3）ThreadsCPU#getThreadsCPUInfo

```java
public ThreadsCPUInfo getThreadsCPUInfo() throws MBeanException, ReflectionException, IOException, InstanceNotFoundException {
    long[] ids = threadBean.getAllThreadIds();
    ThreadInfo[] tids = threadBean.getThreadInfo(ids);
    long[] tinfo;

    if (useBulkOperation) {
        Object[] args = new Object[] {ids};
        String[] sigs = new String[] {"[J"};  // NOI18N

        try {
            tinfo = (long[])connection.invoke(THREAD_NAME, "getThreadCpuTime", args, sigs);  // NOI18N
        } catch (javax.management.ReflectionException ex) {
            LOGGER.log(Level.INFO, "getThreadCpuTime failed", ex);
            useBulkOperation = false;
            return getThreadsCPUInfo();
        }
    } else {
        tinfo = new long[ids.length];

        for (int i = 0; i < ids.length; i++) {
            tinfo[i] = threadBean.getThreadCpuTime(ids[i]);
        }
    }
    long time = System.currentTimeMillis();

    return new ThreadsCPUInfo(time,tids,tinfo);
}
```

ThreadsCPUInfo 中的当前时间，也是调用 System.currentTimeMillis 获得的。
线程的 CPU 时间，是通过 ThreadMXBean.getThreadCpuTime 获得的。

（4）ThreadsCPUInfo

```java
List<Long> getCPUTimePerSecond(ThreadsCPUInfo newInfo) {
    assert newInfo.timestamp >= timestamp;
    List<Long> diff = getThreadCPUTimeDiff(newInfo);
    double secs = (newInfo.timestamp - timestamp) / 1000.0;
    List<Long> diffPerSec = new ArrayList(diff.size());

    for (Long d : diff) {
        diffPerSec.add(new Long((long)(d/secs)));
    }
    return diffPerSec;
}
```

计算线程 CPU 使用率时，CPU 总时间等于当前 ThreadsCPUInfo 的时间减去上一次的时间，线程 CPU 时间通过 getThreadCPUTimeDiff 函数计算得出。

（5）ThreadsCPUInfo#getThreadCPUTimeDiff

```java
List<Long> getThreadCPUTimeDiff(ThreadsCPUInfo info) {
    List<Long> cpuTimeDiff = new ArrayList(threads.size());
    List<ThreadInfo> newThreads = info.getThreads();
    List<Long> newCPUTime = info.getThreadCPUTime();

    totalDiffCPUTime = 0;
    for (int i=0; i<newThreads.size(); i++) {
        ThreadInfo ti = newThreads.get(i);
        Long oldAlloc = cputimeMap.get(ti.getThreadId());
        long diff;

        if (oldAlloc == null) {
            oldAlloc = Long.valueOf(0);
        }
        diff = newCPUTime.get(i)-oldAlloc;
        cpuTimeDiff.add(diff);
        totalDiffCPUTime += diff;
    }
    return cpuTimeDiff;
}
```

线程 CPU 时间其实也是求差。

（6）ThreadsCPUInfo 对象中的线程的 CPU 时间是如何获得的（getThreadCPUTime）？

```java
ThreadsCPUInfo(long time, ThreadInfo[] tinfo, long[] cpuinfo) {
    cputimeMap = new HashMap(threads.size()*4/3);
    totalCPUTime = 0;
    for (int i = 0; i <tinfo.length; i++) {
        ThreadInfo ti = tinfo[i];
        if (ti != null) {
            threads.add(ti);
            cputime.add(cpuinfo[i]);
            cputimeMap.put(ti.getThreadId(),cpuinfo[i]);
            totalCPUTime+=cpuinfo[i];
        }
    }
    timestamp = time;
}
```

原来是通过 cputimeMap 缓存的线程线程的 CPU 时间。

#### 4.2 OperatingSystemMXBean 如何获取系统与进程 CPU 使用率

这里下载了 OPENJDK 的源码，地址为：https://github.com/openjdk/jdk/tree/jdk8-b120

（1）OperatingSystemMXBean
OperatingSystemMXBean.getSystemCpuLoad
OperatingSystemMXBean.getProcessCpuTime

主要关注这 2 个函数。因为在 sentinel 的 SystemSlot 计算中用到了。

（2）sun.management.OperatingSystemImpl

```java
public native long getProcessCpuTime();
public native double getSystemCpuLoad();

```

实现与具体的操作系统有关。

（3）OperatingSystemImpl.c

这里的实现是：src/solaris/native/sun/management/OperatingSystemImpl.c

```cpp
JNIEXPORT jlong JNICALL
Java_sun_management_OperatingSystemImpl_getProcessCpuTime
  (JNIEnv *env, jobject mbean)
{
#ifdef __APPLE__
    struct rusage usage;
    if (getrusage(RUSAGE_SELF, &usage) != 0) {
        throw_internal_error(env, "getrusage failed");
        return -1;
    }
    jlong microsecs =
        usage.ru_utime.tv_sec * 1000 * 1000 + usage.ru_utime.tv_usec +
        usage.ru_stime.tv_sec * 1000 * 1000 + usage.ru_stime.tv_usec;
    return microsecs * 1000;
#else
    jlong clk_tck, ns_per_clock_tick;
    jlong cpu_time_ns;
    struct tms time;

    /*
     * BSDNOTE: FreeBSD implements _SC_CLK_TCK since FreeBSD 5, so
     *          add a magic to handle it
     */
#if defined(__solaris__) || defined(_SC_CLK_TCK)
    clk_tck = (jlong) sysconf(_SC_CLK_TCK);
#elif defined(__linux__) || defined(_ALLBSD_SOURCE)
    clk_tck = 100;
#endif
    if (clk_tck == -1) {
        throw_internal_error(env,
                             "sysconf failed - not able to get clock tick");
        return -1;
    }

    times(&time);
    ns_per_clock_tick = (jlong) 1000 * 1000 * 1000 / (jlong) clk_tck;
    cpu_time_ns = ((jlong)time.tms_utime + (jlong) time.tms_stime) *
                      ns_per_clock_tick;
    return cpu_time_ns;
#endif
}
```

直接调用系统函数 times 获得进程的 CPU 使用时间（单位：clk_tck），最后的实际时间还需要转换为 ns

（4）LinuxOperatingSystem.c
这里的实现是：/src/solaris/native/sun/management/LinuxOperatingSystem.c

```cpp
JNIEXPORT jdouble JNICALL
Java_sun_management_OperatingSystemImpl_getSystemCpuLoad
(JNIEnv *env, jobject dummy)
{
    if(perfInit() == 0) {
        return get_cpu_load(-1);
    } else {
        return -1.0;
    }
}

double get_cpu_load(int which) {
    double u, s;
    u = get_cpuload_internal(which, &s, CPU_LOAD_GLOBAL);
    if (u < 0) {
        return -1.0;
    }
    // Cap total systemload to 1.0
    return MIN((u + s), 1.0);
}

static double get_cpuload_internal(int which, double *pkernelLoad, CpuLoadTarget target) {
    uint64_t udiff, kdiff, tdiff;
    ticks *pticks, tmp;
    double user_load = -1.0;
    int failed = 0;

    *pkernelLoad = 0.0;

    pthread_mutex_lock(&lock);

    if(perfInit() == 0) {

        if (target == CPU_LOAD_VM_ONLY) {
            pticks = &counters.jvmTicks;
        } else if (which == -1) {
            pticks = &counters.cpuTicks;
        } else {
            pticks = &counters.cpus[which];
        }

        tmp = *pticks;

        if (target == CPU_LOAD_VM_ONLY) {
            if (get_jvmticks(pticks) != 0) {
                failed = 1;
            }
        } else if (get_totalticks(which, pticks) < 0) {
            failed = 1;
        }

        if(!failed) {
            // seems like we sometimes end up with less kernel ticks when
            // reading /proc/self/stat a second time, timing issue between cpus?
            if (pticks->usedKernel < tmp.usedKernel) {
                kdiff = 0;
            } else {
                kdiff = pticks->usedKernel - tmp.usedKernel;
            }
            tdiff = pticks->total - tmp.total;
            udiff = pticks->used - tmp.used;

            if (tdiff == 0) {
                user_load = 0;
            } else {
                if (tdiff < (udiff + kdiff)) {
                    tdiff = udiff + kdiff;
                }
                *pkernelLoad = (kdiff / (double)tdiff);
                // BUG9044876, normalize return values to sane values
                *pkernelLoad = MAX(*pkernelLoad, 0.0);
                *pkernelLoad = MIN(*pkernelLoad, 1.0);

                user_load = (udiff / (double)tdiff);
                user_load = MAX(user_load, 0.0);
                user_load = MIN(user_load, 1.0);
            }
        }
    }
    pthread_mutex_unlock(&lock);
    return user_load;
}

static int get_jvmticks(ticks *pticks) {
    uint64_t userTicks;
    uint64_t systemTicks;

    if (read_ticks("/proc/self/stat", &userTicks, &systemTicks) < 0) {
        return -1;
    }

    // get the total
    if (get_totalticks(-1, pticks) < 0) {
        return -1;
    }

    pticks->used       = userTicks;
    pticks->usedKernel = systemTicks;

    return 0;
}

static int get_totalticks(int which, ticks *pticks) {
    FILE         *fh;
    uint64_t        userTicks, niceTicks, systemTicks, idleTicks;
    int             n;

    if((fh = fopen("/proc/stat", "r")) == NULL) {
        return -1;
    }

    n = fscanf(fh, "cpu " DEC_64 " " DEC_64 " " DEC_64 " " DEC_64,
           &userTicks, &niceTicks, &systemTicks, &idleTicks);

    // Move to next line
    next_line(fh);

    //find the line for requested cpu faster to just iterate linefeeds?
    if (which != -1) {
        int i;
        for (i = 0; i < which; i++) {
            if (fscanf(fh, "cpu%*d " DEC_64 " " DEC_64 " " DEC_64 " " DEC_64, &userTicks, &niceTicks, &systemTicks, &idleTicks) != 4) {
                fclose(fh);
                return -2;
            }
            next_line(fh);
        }
        n = fscanf(fh, "cpu%*d " DEC_64 " " DEC_64 " " DEC_64 " " DEC_64 "\n",
           &userTicks, &niceTicks, &systemTicks, &idleTicks);
    }

    fclose(fh);
    if (n != 4) {
        return -2;
    }

    pticks->used       = userTicks + niceTicks;
    pticks->usedKernel = systemTicks;
    pticks->total      = userTicks + niceTicks + systemTicks + idleTicks;

    return 0;
}
```

发现最终系统的 CPU 时间来自 /proc/stat，进程的 CPU 时间来自 /proc/self/stat，可知系统的 CPU 使用率，与之前的分析一致。

#### 4.3 ThreadMXBean 如何获得线程 CPU

（1）ThreadMXBean
ThreadMXBean.getThreadCpuTime

我们只关注线程的 CPU 使用时间，之后通过公式再计算使用率。

（2）sun.management.ThreadImpl

```java
public long getThreadCpuTime(long id) {
    long[] ids = new long[1];
    ids[0] = id;
    final long[] times = getThreadCpuTime(ids);
    return times[0];
}

public long[] getThreadCpuTime(long[] ids) {
    boolean verified = verifyThreadCpuTime(ids);

    int length = ids.length;
    long[] times = new long[length];
    java.util.Arrays.fill(times, -1);

    if (verified) {
        if (length == 1) {
            long id = ids[0];
            if (id == Thread.currentThread().getId()) {
                id = 0;
            }
            times[0] = getThreadTotalCpuTime0(id);
        } else {
            getThreadTotalCpuTime1(ids, times);
        }
    }
    return times;
}

private static native long getThreadTotalCpuTime0(long id);
private static native void getThreadTotalCpuTime1(long[] ids, long[] result);
```

与具体的实现有关

（3）ThreadImpl.c
这里的实现是：src/share/native/sun/management/ThreadImpl.c

```cpp
JNIEXPORT jlong JNICALL
Java_sun_management_ThreadImpl_getThreadTotalCpuTime0
  (JNIEnv *env, jclass cls, jlong tid)
{
    return jmm_interface->GetThreadCpuTimeWithKind(env, tid, JNI_TRUE /* user+sys */);
}

JNIEXPORT void JNICALL
Java_sun_management_ThreadImpl_getThreadTotalCpuTime1
  (JNIEnv *env, jclass cls, jlongArray ids, jlongArray timeArray)
{
    jmm_interface->GetThreadCpuTimesWithKind(env, ids, timeArray,
                                             JNI_TRUE /* user+sys */);
}

```

（4）management.cpp

来自：jdk-jdk8-b120/hotspot/src/share/vm/services/management.cpp

JVM 以 management 动态链接库的形式，向 JDK 提供一套监控和管理虚拟机的 jmm 接口。

```cpp
// Returns the CPU time consumed by a given thread (in nanoseconds).
// If thread_id == 0, CPU time for the current thread is returned.
// If user_sys_cpu_time = true, user level and system CPU time of
// a given thread is returned; otherwise, only user level CPU time
// is returned.
JVM_ENTRY(jlong, jmm_GetThreadCpuTimeWithKind(JNIEnv *env, jlong thread_id, jboolean user_sys_cpu_time))
  if (!os::is_thread_cpu_time_supported()) {
    return -1;
  }

  if (thread_id < 0) {
    THROW_MSG_(vmSymbols::java_lang_IllegalArgumentException(),
               "Invalid thread ID", -1);
  }

  JavaThread* java_thread = NULL;
  if (thread_id == 0) {
    // current thread
    return os::current_thread_cpu_time(user_sys_cpu_time != 0);
  } else {
    MutexLockerEx ml(Threads_lock);
    java_thread = find_java_thread_from_id(thread_id);
    if (java_thread != NULL) {
      return os::thread_cpu_time((Thread*) java_thread, user_sys_cpu_time != 0);
    }
  }
  return -1;
JVM_END

// Gets an array containing the CPU times consumed by a set of threads
// (in nanoseconds).  Each element of the array is the CPU time for the
// thread ID specified in the corresponding entry in the given array
// of thread IDs; or -1 if the thread does not exist or has terminated.
// If user_sys_cpu_time = true, the sum of user level and system CPU time
// for the given thread is returned; otherwise, only user level CPU time
// is returned.
JVM_ENTRY(void, jmm_GetThreadCpuTimesWithKind(JNIEnv *env, jlongArray ids,
                                              jlongArray timeArray,
                                              jboolean user_sys_cpu_time))
  // Check if threads is null
  if (ids == NULL || timeArray == NULL) {
    THROW(vmSymbols::java_lang_NullPointerException());
  }

  ResourceMark rm(THREAD);
  typeArrayOop ta = typeArrayOop(JNIHandles::resolve_non_null(ids));
  typeArrayHandle ids_ah(THREAD, ta);

  typeArrayOop tia = typeArrayOop(JNIHandles::resolve_non_null(timeArray));
  typeArrayHandle timeArray_h(THREAD, tia);

  // validate the thread id array
  validate_thread_id_array(ids_ah, CHECK);

  // timeArray must be of the same length as the given array of thread IDs
  int num_threads = ids_ah->length();
  if (num_threads != timeArray_h->length()) {
    THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(),
              "The length of the given long array does not match the length of "
              "the given array of thread IDs");
  }

  MutexLockerEx ml(Threads_lock);
  for (int i = 0; i < num_threads; i++) {
    JavaThread* java_thread = find_java_thread_from_id(ids_ah->long_at(i));
    if (java_thread != NULL) {
      timeArray_h->long_at_put(i, os::thread_cpu_time((Thread*)java_thread,
                                                      user_sys_cpu_time != 0));
    }
  }
JVM_END
```

线程的 CPU 时间依赖于具体操作系统的实现。

（5）os_linux.cpp

来自：jdk-jdk8-b120/hotspot/src/os/linux/vm/os_linux.cpp

```cpp
jlong os::thread_cpu_time(Thread *thread, bool user_sys_cpu_time) {
  if (user_sys_cpu_time && os::Linux::supports_fast_thread_cpu_time()) {
    return os::Linux::fast_thread_cpu_time(thread_cpu_clockid(thread));
  } else {
    return slow_thread_cpu_time(thread, user_sys_cpu_time);
  }
}
// This is the fastest way to get thread cpu time on Linux.
// Returns cpu time (user+sys) for any thread, not only for current.
// POSIX compliant clocks are implemented in the kernels 2.6.16+.
// It might work on 2.6.10+ with a special kernel/glibc patch.
// For reference, please, see IEEE Std 1003.1-2004:
//   http://www.unix.org/single_unix_specification

jlong os::Linux::fast_thread_cpu_time(clockid_t clockid) {
  struct timespec tp;
  int rc = os::Linux::clock_gettime(clockid, &tp);
  assert(rc == 0, "clock_gettime is expected to return 0 code");

  return (tp.tv_sec * NANOSECS_PER_SEC) + tp.tv_nsec;
}


static jlong slow_thread_cpu_time(Thread *thread, bool user_sys_cpu_time) {
  static bool proc_task_unchecked = true;
  static const char *proc_stat_path = "/proc/%d/stat";
  pid_t  tid = thread->osthread()->thread_id();
  char *s;
  char stat[2048];
  int statlen;
  char proc_name[64];
  int count;
  long sys_time, user_time;
  char cdummy;
  int idummy;
  long ldummy;
  FILE *fp;

  // The /proc/<tid>/stat aggregates per-process usage on
  // new Linux kernels 2.6+ where NPTL is supported.
  // The /proc/self/task/<tid>/stat still has the per-thread usage.
  // See bug 6328462.
  // There possibly can be cases where there is no directory
  // /proc/self/task, so we check its availability.
  if (proc_task_unchecked && os::Linux::is_NPTL()) {
    // This is executed only once
    proc_task_unchecked = false;
    fp = fopen("/proc/self/task", "r");
    if (fp != NULL) {
      proc_stat_path = "/proc/self/task/%d/stat";
      fclose(fp);
    }
  }

  sprintf(proc_name, proc_stat_path, tid);
  fp = fopen(proc_name, "r");
  if ( fp == NULL ) return -1;
  statlen = fread(stat, 1, 2047, fp);
  stat[statlen] = '\0';
  fclose(fp);

  // Skip pid and the command string. Note that we could be dealing with
  // weird command names, e.g. user could decide to rename java launcher
  // to "java 1.4.2 :)", then the stat file would look like
  //                1234 (java 1.4.2 :)) R ... ...
  // We don't really need to know the command string, just find the last
  // occurrence of ")" and then start parsing from there. See bug 4726580.
  s = strrchr(stat, ')');
  if (s == NULL ) return -1;

  // Skip blank chars
  do s++; while (isspace(*s));

  count = sscanf(s,"%c %d %d %d %d %d %lu %lu %lu %lu %lu %lu %lu",
                 &cdummy, &idummy, &idummy, &idummy, &idummy, &idummy,
                 &ldummy, &ldummy, &ldummy, &ldummy, &ldummy,
                 &user_time, &sys_time);
  if ( count != 13 ) return -1;
  if (user_sys_cpu_time) {
    return ((jlong)sys_time + (jlong)user_time) * (1000000000 / clock_tics_per_sec);
  } else {
    return (jlong)user_time * (1000000000 / clock_tics_per_sec);
  }
}

```

发现线程的 CPU 时间，最终还是读取 /proc/self/task/%d/stat 的数据，和预期的一致！

### 5. 资料来源

#### 1. /proc/stat 解析

http://gityuan.com/2017/08/12/proc_stat/

#### 2. Accurate calculation of CPU usage given in percentage in Linux?

https://stackoverflow.com/questions/23367857/accurate-calculation-of-cpu-usage-given-in-percentage-in-linux

#### 3. 利用 JMX 统计远程 JAVA 进程的 CPU 和 Memory---JVM managerment API

https://www.cnblogs.com/zhengah/p/4962352.html

#### 4. visualvm 源码

https://github.com/oracle/visualvm

#### 5. top 是如何实现的?

https://www.cnblogs.com/carlsplace/p/12953525.html

#### 6. 聊聊 top 命令中的 CPU 使用率

https://mp.weixin.qq.com/s/qkjGYoheHvs-lX9avrYg_g

#### 7. proc(5) - Linux man page

https://linux.die.net/man/5/proc

#### Linux 下 CPU 的利用率

https://mp.weixin.qq.com/s/M4B_LhpmWJ4j8i9ApoWfPA

#### 8. top.c

https://gitlab.com/procps-ng/procps/-/blob/newlib/top/top.c

#### 9. openJDK

http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/

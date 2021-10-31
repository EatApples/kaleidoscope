# 一，前置知识

### 1，安全点

Java 虚拟机中的 Stop-the-world 是通过安全点（safepoint）机制来实现的。当 Java 虚拟机收到 Stop-the-world 请求，它便会等待所有的线程都到达安全点，才允许请求 Stop-the-world 的线程进行独占的工作。

当然，安全点的初始目的并不是让其他线程停下，而是找到一个稳定的执行状态。在这个执行状态下，Java 虚拟机的堆栈不会发生变化。这么一来，垃圾回收器便能够“安全”地执行可达性分析。

A JVM will therefore need means of bringing threads to safepoints (and keeping them there) so that all sorts of runtime magic can happen. Here's a partial list of activities which JVMs run only once all mutator threads are at a safepoint and cannot leave it until released (at a global safepoint), these are sometime called safepoint operations:

- Some GC phases (the Stop The World kind)
- JVMTI stack sampling methods (not always a global safepoint operation for Zing))
- Class redefinition
- Heap dumping
- Monitor deflation (not a global safepoint operation for Zing)
- Lock unbiasing
- Method deoptimization (not always)
- And many more!

Note that the distinction between requesting a global safepoint and a thread safepoint exists only for some JVM implementations (e.g. Zing, the Azul Systems JVM.). Importantly it does not exist on OpenJDK/Oracle JVMs. This means that Zing can bring a single thread to a safepoint.

To summarize:

- Safepoints are a common JVM implementation detail
- They are used to put mutator threads on hold while the JVM 'fixes stuff up'
- On OpenJDK/Oracle every safepoint operation requires a global safepoint
- All current JVMs have some requirement for global safepoints

### 2，STW

基于可达性分析的算法，需要确保引用在某个瞬间不再变化，看起来像冻结一样，即 STW（Stop The World）。

程序执行时，并非在所有地方都能停下来开始 GC，只有到达安全点时才能暂停。安全点是特定的位置，以“是否具有让程序长时间执行的特征”为标准选定，例如方法调用，循环跳转等。

GC 发生时，让所有线程跑到最近的安全点再停顿下来（也可进入安全区域：在安全区域中，引用关系不再发生变化）。

如何停顿？有 2 种方案 ：抢占式中断和主动式中断。

抢占式中断在 GC 发生时，首先把所有线程全部中断，然后让它们跑到安全点。

主动式中断不直接对线程进行操作，只是设置一个 GC 标志，让各个线程主动去轮询，为真时线程就主动挂起。

### 3，GetStackTrace 的调用

（1）单独来说，JVMTI 的 GetStackTrace()函数并不需要在 Caller 的安全点执行;

（2）但当调用 GetStackTrace()获取其他线程的调用栈时，必须等待，直到目标线程进入安全点；

（3）而且，GetStackTrace()仅能通过单独的线程同步定时调用，不能在 UNIX 信号处理器的 Handler 中被异步调用。

综合来说，GetStackTrace()存在与 JMX 一样的 SafePoint Bias。更多安全点相关的知识可以参考[《Safepoints: Meaning, Side Effects and Overheads》](https://psy-lob-saw.blogspot.com/2015/12/safepoints.html)。

### 4，AsyncGetCallTrace（ASGCT）

假如我们拥有一个函数可以

（1）获取当前线程的调用栈且不受安全点干扰

（2）支持在 UNIX 信号处理器中被异步调用

那么我们只需注册一个 UNIX 信号处理器，在 Handler 中调用该函数获取当前线程的调用栈即可。

由于 UNIX 信号会被发送给进程的随机一线程进行处理，因此最终信号会均匀分布在所有线程上，也就均匀获取了所有线程的调用栈样本。

OracleJDK/OpenJDK 内部提供了这么一个函数——AsyncGetCallTrace

它的原型如下：

```c
// 栈帧
typedef struct {
 jint lineno;
 jmethodID method_id;
} AGCT_CallFrame;
​
// 调用栈
typedef struct {
    JNIEnv *env;
    jint num_frames;
    AGCT_CallFrame *frames;
} AGCT_CallTrace;
​
// 根据ucontext将调用栈填充进trace指针
void AsyncGetCallTrace(AGCT_CallTrace *trace, jint depth, void *ucontext);
```

通过原型可以看到，该函数的使用方式非常简洁，直接通过 ucontext 就能获取到完整的 Java 调用栈。

### 5，ASGCT 任意时刻采样

顾名思义，AsyncGetCallTrace 是“async”的，不受安全点影响，这样的话采样就可能发生在任何时间，包括 Native 代码执行期间、GC 期间等，在这时我们是无法获取 Java 调用栈的，AGCT_CallTrace 的 num_frames 字段正常情况下标识了获取到的调用栈深度，但在如前所述的异常情况下它就表示为负数，最常见的-2 代表此刻正在 GC。

### 6. 获取 ASGCT

由于 AsyncGetCallTrace 非标准 JVMTI 函数，因此我们无法在 jvmti.h 中找到该函数声明，且由于其目标文件也早已链接进 JVM 二进制文件中，所以无法通过简单的声明来获取该函数的地址，这需要通过一些 Trick 方式来解决。

简单说，Agent 最终是作为动态链接库加载到目标 JVM 进程的地址空间中，因此可以在 Agent_OnLoad 内通过 glibc 提供的 dlsym()函数拿到当前地址空间（即目标 JVM 进程地址空间）名为“AsyncGetCallTrace”的符号地址。这样就拿到了该函数的指针，按照上述原型进行类型转换后，就可以正常调用了。

### 7. ASGCT 进行 CPU 采样

通过 AsyncGetCallTrace 实现 CPU Profiler 的大致流程：

（1）编写 Agent_OnLoad()，在入口拿到 jvmtiEnv 和 AsyncGetCallTrace 指针，获取 AsyncGetCallTrace 方式如下:

```c
typedef void (*AsyncGetCallTrace)(AGCT_CallTrace *traces, jint depth, void *ucontext);
// ...
AsyncGetCallTrace agct_ptr = (AsyncGetCallTrace)dlsym(RTLD_DEFAULT, "AsyncGetCallTrace");
if (agct_ptr == NULL) {
    void *libjvm = dlopen("libjvm.so", RTLD_NOW);
    if (!libjvm) {
        // 处理dlerror()...
    }
    agct_ptr = (AsyncGetCallTrace)dlsym(libjvm, "AsyncGetCallTrace");
}
```

（2）在 OnLoad 阶段，我们还需要做一件事，即注册 OnClassLoad 和 OnClassPrepare 这两个 Hook，原因是 jmethodID 是延迟分配的，使用 AGCT 获取 Traces 依赖预先分配好的数据。我们在 OnClassPrepare 的 CallBack 中尝试获取该 Class 的所有 Methods，这样就使 JVMTI 提前分配了所有方法的 jmethodID，如下所示：

```c
void JNICALL OnClassLoad(jvmtiEnv *jvmti, JNIEnv* jni, jthread thread, jclass klass) {}
​
void JNICALL OnClassPrepare(jvmtiEnv *jvmti, JNIEnv *jni, jthread thread, jclass klass) {
    jint method_count;
    jmethodID *methods;
    jvmti->GetClassMethods(klass, &method_count, &methods);
    delete [] methods;
}
​
// ...
​
jvmtiEventCallbacks callbacks = {0};
callbacks.ClassLoad = OnClassLoad;
callbacks.ClassPrepare = OnClassPrepare;
jvmti->SetEventCallbacks(&callbacks, sizeof(callbacks));
jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_CLASS_LOAD, NULL);
jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_CLASS_PREPARE, NULL);
```

（3）利用 SIGPROF 信号来进行定时采样：

```c
// 这里信号handler传进来的的ucontext即AsyncGetCallTrace需要的ucontext
void signal_handler(int signo, siginfo_t *siginfo, void *ucontext) {
    // 使用AsyncCallTrace进行采样，注意处理num_frames为负的异常情况
}
​
// ...
​
// 注册SIGPROF信号的handler
struct sigaction sa;
sigemptyset(&sa.sa_mask);
sa.sa_sigaction = signal_handler;
sa.sa_flags = SA_RESTART | SA_SIGINFO;
sigaction(SIGPROF, &sa, NULL);
​
// 定时产生SIGPROF信号
// interval是nanoseconds表示的采样间隔，AsyncGetCallTrace相对于同步采样来说可以适当高频一些
long sec = interval / 1000000000;
long usec = (interval % 1000000000) / 1000;
struct itimerval tv = {{sec, usec}, {sec, usec}};
setitimer(ITIMER_PROF, &tv, NULL);
```

（4）在 Buffer 中保存每一次的采样结果，最终生成必要的统计数据即可。

按如上步骤即可实现基于 AsyncGetCallTrace 的 CPU Profiler，这是社区中目前性能开销最低、相对效率最高的 CPU Profiler 实现方式，在 Linux 环境下结合 perf_events 还能做到同时采样 Java 栈与 Native 栈，也就能同时分析 Native 代码中存在的性能热点。该方式的典型开源实现有 Async-Profiler 和 Honest-Profiler，Async-Profiler 实现质量较高。

# 二，ASGCT

ASGCT 要不要求安全点？

答：不要求，任意时刻。

ASGCT 会不会引起 GC？

答：ASGCT 是异步的调用，采集的数据存储在内存中（C 语言的 free 与 malloc），大概率不会触发 GC（后续通过性能实验验证）。

ASGCT 会不会引起 STW？

答：不会。任意时刻调用，没有安全点。

### 1. 使用 JVMTI/JVMPI、SIGPROF 和 AsyncGetCallTrace 进行分析

2007 年 5 月 14 日，星期一
Profiling with JVMTI/JVMPI, SIGPROF and AsyncGetCallTrace
http://jeremymanson.blogspot.com/2007/05/profiling-with-jvmtijvmpi-sigprof-and.html

文章提到使用 SIGPROF 信号作为定时器，定时触发 ASGCT 进行采集。

SIGPROF is a POSIX signal. The way it is set up under Linux is that you can set a timer that goes off at intervals, and, whenever the timer expires, the SIGPROF signal is sent to the application. The idea is that you can then collect profiling information at fixed intervals.

The signal is caught and handled by a random running thread. That thread will be doing some task related to running the Java application -- whether that is executing user code, running garbage collection, or doing internal VM maintenance.

It turns out that the kind folks who wrote the Java virtual machine were fully aware of this, and provided an undocumented interface for this type of profiling, used by their Forte Analyzer (which now operates under the Sun Studio umbrella, I believe). Now that they've open-sourced the JVM, this is public knowledge. For those of you who like to see the source code for such things, it can be found in [hotspot/src/share/prims/forte.cpp](http://hg.openjdk.java.net/jdk8/jdk8/hotspot/file/tip/src/share/vm/prims/forte.cpp).

In principle, AsyncGetCallTrace is fairly easy to use. This is less true in practice.

### 2. 关于 SIGPROF、JVMTI 和堆栈跟踪的更多想法

2007 年 6 月 4 日，星期一
More thoughts on SIGPROF, JVMTI and stack traces
http://jeremymanson.blogspot.com/2007/06/more-thoughts-on-sigprof-jvmti-and.html

文章开始聚焦 ASGCT。

If you send a SIGPROF to your JVM and use AsyncGetCallTrace, you find out exactly what your JVM is doing at precisely the moment you sent the signal.

The difference here is fundamentally that all of those other methods tell you what the JVM could be doing, and this one tells you what it is doing. It will even tell you if it is performing garbage collection. This sort of information can be invaluable when you want to know what is soaking up your CPU cycles.

### 3. 为什么许多分析器存在严重问题（更多关于信号分析）

2010 年 8 月 2 日，星期一
Why Many Profilers have Serious Problems (More on Profiling with Signals)
http://jeremymanson.blogspot.com/2010/07/why-many-profilers-have-serious.html

这篇文章增加了关于 JIT 编译器对采样的影响，并描述了为什么使用 AsyncGetCallTrace 和信号进行分析是比典型的当前最好的方法。

All of this JIT talk basically mirrors the Heisenberg Uncertainty Principle, but for profiling. You can either have exact information, or you can have your code behave as it is supposed to behave, but not both. You need to minimize the effects, so you want a profiler that doesn't interfere with or depend on JIT decisions at all. AsyncGetCallTrace fits this bill - you call it, and it returns a result. You don't call it directly from your code. It doesn't wait for a safe point. It doesn't change your code. The JIT doesn't care.

### 4. 轻量级异步采样分析器

2013 年 7 月 9 日，星期二
Lightweight Asynchronous Sampling Profiler
http://jeremymanson.blogspot.com/2013/07/lightweight-asynchronous-sampling.html

据我所知， AsyncGetCallTrace 几乎是唯一一个非平凡的异步安全 JNI/JVMTI 调用。

The answer to this lies in what I meant when I called this function asynchronous. I was a little (deliberately) vague on what "asynchronous" means. When I say that the timer that goes off to grab a stack trace interrupts a running thread asynchronously, I mean that it can interrupt the thread at any point. The thread can be in the middle of GC, or the middle of adding a frame to the stack, or in the middle of acquiring a lock. The reader can imagine that writing something asynchronous-safe is somewhat more tricky than writing something thread-safe: not only do you have to worry about what other threads are doing, you also have to worry about what the current thread is doing.

### 5. 为什么（大多数）采样 Java Profiler 非常糟糕

2016 年 2 月 24 日，星期三
Why (Most) Sampling Java Profilers Are Fucking Terrible
https://psy-lob-saw.blogspot.com/2016/02/why-most-sampling-java-profilers-are.html

文章分析了采样时安全点偏差的问题，意味着之前的采样工具都不准确。

### 6. AsyncGetCallTrace Profiler 的优缺点

2016 年 6 月 23 日，星期四
The Pros and Cons of AsyncGetCallTrace Profilers
http://psy-lob-saw.blogspot.com/2016/06/the-pros-and-cons-of-agct.html

这里面 提到了前面 5 篇内容。

文章提到了 ASGCT 的受限情形：

AsyncGetCallTrace is of limited use when:

Large numbers of samples fail: This can mean the application is spending it's time in GC/Deopts/Runtime code. Watch for failures. I think currently honest-profiler offers better visibility on this, but I'm sure the good folks of JMC can take a hint.

Performance issue is hard to glean from the Java code. E.g. see the issue discussed in a previous post using JMH perfasm (false sharing on the class id in an object header making the conditional inlining of an interface call very expensive).

Due to instruction skid/compilation/available debug information the wrong Java line is blamed. This is potentially very confusing in the presence of inlining and code motion.

# 三，async-profiler

https://github.com/jvm-profiling-tools/async-profiler/wiki

async-profiler 是基于 JVMTI(JVM tool interface) 开发的 Agent，自然而然有两种使用方式：

（1）跟随 Java 进程启动，自动载入共享库

```s
java -agentpath:/path/to/libasyncProfiler.so=start,svg,file=profile.svg ...
```

（2）程序运行时通过 attach api 动态载入

### 1. CPU profiling

In this mode profiler collects stack trace samples that include Java methods, native calls, JVM code and kernel functions.

The general approach is receiving call stacks generated by **perf_events** and matching them up with call stacks generated by **AsyncGetCallTrace**, in order to produce an accurate profile of both Java and native code.

Additionally, async-profiler provides a workaround to recover stack traces in some corner cases ([AsyncGetCallTrace fails to traverse valid Java stacks](https://bugs.openjdk.java.net/browse/JDK-8178287)) where AsyncGetCallTrace fails.

### 2. Restrictions/Limitations

There is no bullet-proof guarantee that the perf_events overflow signal is delivered to the Java thread in a way that guarantees no other code has run, which means that in some rare cases, the captured Java stack might not match the captured native (user+kernel) stack.

You will not see the non-Java frames preceding the Java frames on the stack. For example, if start_thread called JavaMain and then your Java code started running, you will not see the first two frames in the resulting stack. On the other hand, you will see non-Java frames (user and kernel) invoked by your Java code.

Too short profiling interval may cause continuous interruption of heavy system calls like clone(), so that it will never complete; see #97([clone() syscall infinitely restarts because of SIGPROF signals](https://github.com/jvm-profiling-tools/async-profiler/issues/97)). The workaround is simply to increase the interval.

When agent is not loaded at JVM startup (by using -agentpath option) it is highly recommended to use **-XX:+UnlockDiagnosticVMOptions** **-XX:+DebugNonSafepoints** JVM flags. Without those flags the profiler will still work correctly but results might be less accurate. For example, without -XX:+DebugNonSafepoints there is a high chance that simple inlined methods will not appear in the profile. When the agent is attached at runtime, CompiledMethodLoad JVMTI event enables debug info, but only for methods compiled after attaching.

### 3. time-to-safepoint profiling

--begin function, --end function - automatically start/stop profiling when the specified native function is executed.

--ttsp - time-to-safepoint profiling. An alias for
--begin SafepointSynchronize::begin --end RuntimeService::record_safepoint_synchronized

# 资料来源

### 1. JVM CPU Profiler 技术原理及源码深度解析（美团的一篇综述性的文章）

https://tech.meituan.com/2019/10/10/jvm-cpu-profiler.html

### 2. 除了 GC，还有哪些方式会造成 JVM 暂停（作者是 G1 与 C4 的开发者）

With GC Solved, What Else Makes a JVM Pause?
https://www.youtube.com/watch?v=Y39kllzX1P8&ab_channel=OracleDevelopers

### 3. 使用异步分析器提高性能（来自 Async-profiler 作者的演示）

Improving Performance with Async-profiler
https://www.youtube.com/playlist?list=PLNCLTEx3B8h4Yo_WvKWdLvI9mj1XpTKBr

### 4. Async-profiler 演讲的资源链接（演示了 Async-profiler 可以应对的各种情况）

https://github.com/apangin/java-profiling-presentation


### 5. async-profiler 基本原理
https://www.jianshu.com/p/3aedb19d3109
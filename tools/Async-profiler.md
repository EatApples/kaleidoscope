## 一，JVM Agent 简介

JVM Agent 是一个按一定规则编写的特殊程序库，可以在启动阶段通过命令行参数传递给 JVM，作为一个伴生库与目标 JVM 运行在同一个进程中。在 Agent 中可以通过固定的接口获取 JVM 进程内的相关信息。Agent 既可以是用 C/C++/Rust 编写的 JVMTI Agent，也可以是用 Java 编写的 Java Agent。

执行 Java 命令，我们可以看到 Agent 相关的命令行参数：

```s
    -agentlib:<库名>[=<选项>]
                  加载本机代理库 <库名>, 例如 -agentlib:jdwp
                  另请参阅 -agentlib:jdwp=help
    -agentpath:<路径名>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jar 路径>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
```

### 1. JVMTI Agent

JVMTI（JVM Tool Interface）是 JVM 提供的一套标准的 C/C++编程接口，是实现 Debugger、Profiler、Monitor、Thread Analyser 等工具的统一基础，在主流 Java 虚拟机中都有实现。

当我们要基于 JVMTI 实现一个 Agent 时，需要实现如下入口函数：

```java
// $JAVA_HOME/include/jvmti.h

JNIEXPORT jint JNICALL Agent_OnLoad(JavaVM *vm, char *options, void *reserved);
```

使用 C/C++实现该函数，并将代码编译为动态连接库（Linux 上是.so），通过-agentpath 参数将库的完整路径传递给 Java 进程，JVM 就会在启动阶段的合适时机执行该函数。在函数内部，我们可以通过 JavaVM 指针参数拿到 JNI 和 JVMTI 的函数指针表，这样我们就拥有了与 JVM 进行各种复杂交互的能力。

更多 JVMTI 相关的细节可以参考官方文档（https://docs.oracle.com/en/java/javase/12/docs/specs/jvmti.html）。

### 2. Java Agent

在很多场景下，我们没有必要必须使用 C/C++来开发 JVMTI Agent，因为成本高且不易维护。JVM 自身基于 JVMTI 封装了一套 Java 的 Instrument API 接口，允许使用 Java 语言开发 Java Agent（只是一个 jar 包），大大降低了 Agent 的开发成本。社区开源的产品如 Greys、Arthas、JVM-Sandbox、JVM-Profiler 等都是纯 Java 编写的，也是以 Java Agent 形式来运行。

在 Java Agent 中，我们需要在 jar 包的 MANIFEST.MF 中将 Premain-Class 指定为一个入口类，并在该入口类中实现如下方法：

```java
public static void premain(String args, Instrumentation ins) {
    // implement
}
```

这样打包出来的 jar 就是一个 Java Agent，可以通过-javaagent 参数将 jar 传递给 Java 进程伴随启动，JVM 同样会在启动阶段的合适时机执行该方法。

在该方法内部，参数 Instrumentation 接口提供了 Retransform Classes 的能力，我们利用该接口就可以对宿主进程的 Class 进行修改，实现方法耗时统计、故障注入、Trace 等功能。Instrumentation 接口提供的能力较为单一，仅与 Class 字节码操作相关，但由于我们现在已经处于宿主进程环境内，就可以利用 JMX 直接获取宿主进程的内存、线程、锁等信息。无论是 Instrument API 还是 JMX，它们内部仍是统一基于 JVMTI 来实现。

更多 Instrument API 相关的细节可以参考官方文档（https://docs.oracle.com/en/java/javase/12/docs/api/java.instrument/java/lang/instrument/package-summary.html）。

## 二，CPU Profiler 原理解析

### 1. Sampling vs Instrumentation

使用过 JProfiler 的同学应该都知道，JProfiler 的 CPU Profiling 功能提供了两种方式选项: Sampling 和 Instrumentation，它们也是实现 CPU Profiler 的两种手段。

Sampling 方式顾名思义，基于对 StackTrace 的“采样”进行实现，核心原理如下：

- 引入 Profiler 依赖，或直接利用 Agent 技术注入目标 JVM 进程并启动 Profiler。

- 启动一个采样定时器，以固定的采样频率每隔一段时间（毫秒级）对所有线程的调用栈进行 Dump。

- 汇总并统计每次调用栈的 Dump 结果，在一定时间内采到足够的样本后，导出统计结果，内容是每个方法被采样到的次数及方法的调用关系。

Instrumentation 则是利用 Instrument API，对所有必要的 Class 进行字节码增强，在进入每个方法前进行埋点，方法执行结束后统计本次方法执行耗时，最终进行汇总。二者都能得到想要的结果，那么它们有什么区别呢？或者说，孰优孰劣？

Instrumentation 方式对几乎所有方法添加了额外的 AOP 逻辑，这会导致对线上服务造成巨额的性能影响，但其优势是：绝对精准的方法调用次数、调用时间统计。

Sampling 方式基于无侵入的额外线程对所有线程的调用栈快照进行固定频率抽样，相对前者来说它的性能开销很低。但由于它基于“采样”的模式，以及 JVM 固有的只能在安全点（Safe Point）进行采样的“缺陷”，会导致统计结果存在一定的偏差。譬如说：某些方法执行时间极短，但执行频率很高，真实占用了大量的 CPU Time，但 Sampling Profiler 的采样周期不能无限调小，这会导致性能开销骤增，所以会导致大量的样本调用栈中并不存在刚才提到的”高频小方法“，进而导致最终结果无法反映真实的 CPU 热点。更多 Sampling 相关的问题可以参考《Why (Most) Sampling Java Profilers Are Fucking Terrible》。

具体到“孰优孰劣”的问题层面，这两种实现技术并没有非常明显的高下之判，只有在分场景讨论下才有意义。Sampling 由于低开销的特性，更适合用在 CPU 密集型的应用中，以及不可接受大量性能开销的线上服务中。而 Instrumentation 则更适合用在 I/O 密集的应用中、对性能开销不敏感以及确实需要精确统计的场景中。社区的 Profiler 更多的是基于 Sampling 来实现。

### 2. 基于 Java Agent + JMX 实现

一个最简单的 Sampling CPU Profiler 可以用 Java Agent + JMX 方式来实现。以 Java Agent 为入口，进入目标 JVM 进程后开启一个 ScheduledExecutorService，定时利用 JMX 的 threadMXBean.dumpAllThreads()来导出所有线程的 StackTrace，最终汇总并导出即可。

Uber 的 JVM-Profiler 实现原理也是如此，关键部分代码如下：

```java
// com/uber/profiling/profilers/StacktraceCollectorProfiler.java

/*
 * StacktraceCollectorProfiler等同于文中所述CpuProfiler，仅命名偏好不同而已
 * jvm-profiler的CpuProfiler指代的是CpuLoad指标的Profiler
 */

// 实现了Profiler接口，外部由统一的ScheduledExecutorService对所有Profiler定时执行
@Override
public void profile() {
    ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
    // ...
    for (ThreadInfo threadInfo : threadInfos) {
        String threadName = threadInfo.getThreadName();
        // ...
        StackTraceElement[] stackTraceElements = threadInfo.getStackTrace();
        // ...
        for (int i = stackTraceElements.length - 1; i >= 0; i--) {
            StackTraceElement stackTraceElement = stackTraceElements[i];
            // ...
        }
        // ...
    }
}
```

Uber 提供的定时器默认 Interval 是 100ms，对于 CPU Profiler 来说，这略显粗糙。但由于 dumpAllThreads()的执行开销不容小觑，Interval 不宜设置的过小，所以该方法的 CPU Profiling 结果会存在不小的误差。

JVM-Profiler 的优点在于支持多种指标的 Profiling（StackTrace、CPUBusy、Memory、I/O、Method），且支持将 Profiling 结果通过 Kafka 上报回中心 Server 进行分析，也即支持集群诊断。

### 3. 基于 JVMTI + GetStackTrace 实现

使用 Java 实现 Profiler 相对较简单，但也存在一些问题，譬如说 Java Agent 代码与业务代码共享 AppClassLoader，被 JVM 直接加载的 agent.jar 如果引入了第三方依赖，可能会对业务 Class 造成污染。截止发稿时，JVM-Profiler 都存在这个问题，它引入了 Kafka-Client、http-Client、Jackson 等组件，如果与业务代码中的组件版本发生冲突，可能会引发未知错误。Greys/Arthas/JVM-Sandbox 的解决方式是分离入口与核心代码，使用定制的 ClassLoader 加载核心代码，避免影响业务代码。

在更底层的 C/C++层面，我们可以直接对接 JVMTI 接口，使用原生 C API 对 JVM 进行操作，功能更丰富更强大，但开发效率偏低。基于上节同样的原理开发 CPU Profiler，使用 JVMTI 需要进行如下这些步骤：

（1）编写 Agent_OnLoad()，在入口通过 JNI 的 JavaVM\*指针的 GetEnv()函数拿到 JVMTI 的 jvmtiEnv 指针：

```c
// agent.c

JNIEXPORT jint JNICALL Agent_OnLoad(JavaVM *vm, char *options, void *reserved) {
    jvmtiEnv *jvmti;
    (*vm)->GetEnv((void **)&jvmti, JVMTI_VERSION_1_0);
    // ...
    return JNI_OK;
}
```

（2）开启一个线程定时循环，定时使用 jvmtiEnv 指针配合调用如下几个 JVMTI 函数：

```c
// 获取所有线程的jthread
jvmtiError GetAllThreads(jvmtiEnv *env, jint *threads_count_ptr, jthread **threads_ptr);

// 根据jthread获取该线程信息（name、daemon、priority...）
jvmtiError GetThreadInfo(jvmtiEnv *env, jthread thread, jvmtiThreadInfo* info_ptr);

// 根据jthread获取该线程调用栈
jvmtiError GetStackTrace(jvmtiEnv *env,
                         jthread thread,
                         jint start_depth,
                         jint max_frame_count,
                         jvmtiFrameInfo *frame_buffer,
                         jint *count_ptr);
```

主逻辑大致是：首先调用 GetAllThreads()获取所有线程的“句柄”jthread，然后遍历根据 jthread 调用 GetThreadInfo()获取线程信息，按线程名过滤掉不需要的线程后，继续遍历根据 jthread 调用 GetStackTrace()获取线程的调用栈。

（3）在 Buffer 中保存每一次的采样结果，最终生成必要的统计数据即可。

按如上步骤即可实现基于 JVMTI 的 CPU Profiler。但需要说明的是，即便是基于原生 JVMTI 接口使用 GetStackTrace()的方式获取调用栈，也存在与 JMX 相同的问题——只能在安全点（Safe Point）进行采样。

### 4. SafePoint Bias 问题

基于 Sampling 的 CPU Profiler 通过采集程序在不同时间点的调用栈样本来近似地推算出热点方法，因此，从理论上来讲 Sampling CPU Profiler 必须遵循以下两个原则：

（1）样本必须足够多。

（2）程序中所有正在运行的代码点都必须以相同的概率被 Profiler 采样。

如果只能在安全点采样，就违背了第二条原则。因为我们只能采集到位于安全点时刻的调用栈快照，意味着某些代码可能永远没有机会被采样，即使它真实耗费了大量的 CPU 执行时间，这种现象被称为“SafePoint Bias”。

上文我们提到，基于 JMX 与基于 JVMTI 的 Profiler 实现都存在 SafePoint Bias，但一个值得了解的细节是：单独来说，JVMTI 的 GetStackTrace()函数并不需要在 Caller 的安全点执行，但当调用 GetStackTrace()获取其他线程的调用栈时，必须等待，直到目标线程进入安全点；而且，GetStackTrace()仅能通过单独的线程同步定时调用，不能在 UNIX 信号处理器的 Handler 中被异步调用。综合来说，GetStackTrace()存在与 JMX 一样的 SafePoint Bias。更多安全点相关的知识可以参考[《Safepoints: Meaning, Side Effects and Overheads》](https://psy-lob-saw.blogspot.com/2015/12/safepoints.html)。

那么，如何避免 SafePoint Bias？社区提供了一种 Hack 思路——AsyncGetCallTrace。

### 5. 基于 JVMTI + AsyncGetCallTrace 实现

如上节所述，假如我们拥有一个函数可以获取当前线程的调用栈且不受安全点干扰，另外它还支持在 UNIX 信号处理器中被异步调用，那么我们只需注册一个 UNIX 信号处理器，在 Handler 中调用该函数获取当前线程的调用栈即可。由于 UNIX 信号会被发送给进程的随机一线程进行处理，因此最终信号会均匀分布在所有线程上，也就均匀获取了所有线程的调用栈样本。

OracleJDK/OpenJDK 内部提供了这么一个函数——AsyncGetCallTrace，它的原型如下：

文件为：/hotspot/src/share/vm/prims/forte.cpp

```cpp
// 栈帧
typedef struct {
 jint lineno;
 jmethodID method_id;
} AGCT_CallFrame;

// 调用栈
typedef struct {
    JNIEnv *env;
    jint num_frames;
    AGCT_CallFrame *frames;
} AGCT_CallTrace;

// 根据ucontext将调用栈填充进trace指针
void AsyncGetCallTrace(AGCT_CallTrace *trace, jint depth, void *ucontext);
```

通过原型可以看到，该函数的使用方式非常简洁，直接通过 ucontext 就能获取到完整的 Java 调用栈。

顾名思义，AsyncGetCallTrace 是“async”的，不受安全点影响，这样的话采样就可能发生在任何时间，包括 Native 代码执行期间、GC 期间等，在这时我们是无法获取 Java 调用栈的，AGCT_CallTrace 的 num_frames 字段正常情况下标识了获取到的调用栈深度，但在如前所述的异常情况下它就表示为负数，最常见的-2 代表此刻正在 GC。

由于 AsyncGetCallTrace 非标准 JVMTI 函数，因此我们无法在 jvmti.h 中找到该函数声明，且由于其目标文件也早已链接进 JVM 二进制文件中，所以无法通过简单的声明来获取该函数的地址，这需要通过一些 Trick 方式来解决。简单说，Agent 最终是作为动态链接库加载到目标 JVM 进程的地址空间中，因此可以在 Agent_OnLoad 内通过 glibc 提供的 dlsym()函数拿到当前地址空间（即目标 JVM 进程地址空间）名为“AsyncGetCallTrace”的符号地址。这样就拿到了该函数的指针，按照上述原型进行类型转换后，就可以正常调用了。

通过 AsyncGetCallTrace 实现 CPU Profiler 的大致流程：

（1）编写 Agent_OnLoad()，在入口拿到 jvmtiEnv 和 AsyncGetCallTrace 指针，获取 AsyncGetCallTrace
（2）在 OnLoad 阶段，我们还需要做一件事，即注册 OnClassLoad 和 OnClassPrepare 这两个 Hook，原因是 jmethodID 是延迟分配的，使用 AGCT 获取 Traces 依赖预先分配好的数据。我们在 OnClassPrepare 的 CallBack 中尝试获取该 Class 的所有 Methods，这样就使 JVMTI 提前分配了所有方法的 jmethodID
（3）利用 SIGPROF 信号来进行定时采样
（4）在 Buffer 中保存每一次的采样结果，最终生成必要的统计数据即可

按如上步骤即可实现基于 AsyncGetCallTrace 的 CPU Profiler，这是社区中目前性能开销最低、相对效率最高的 CPU Profiler 实现方式，在 Linux 环境下结合 perf_events 还能做到同时采样 Java 栈与 Native 栈，也就能同时分析 Native 代码中存在的性能热点。

该方式的典型开源实现有 Async-Profiler 和 Honest-Profiler，Async-Profiler 实现质量较高，感兴趣的话建议大家阅读参考源码。有趣的是，IntelliJ IDEA 内置的 Java Profiler，其实就是 Async-Profiler 的包装。更多关于 AsyncGetCallTrace 的内容，大家可以参考[《The Pros and Cons of AsyncGetCallTrace Profilers》](http://psy-lob-saw.blogspot.com/2016/06/the-pros-and-cons-of-agct.html)

## 三，Async-Profiler

### 1. 介绍

Async-profiler 是一个对系统性能影响很少的 Java 采样分析器，它的实现是基于 HotSpot 特有的 API，通过这些特有的 API 收集堆栈跟踪和跟踪内存分配，因而其可以和 OpenJDK、Oracle JDK 和其他基于 HotSpot JVM 的 Java 应用在运行时协同工作。

Github 项目链接地址：https://github.com/jvm-profiling-tools/async-profiler

Async-profiler 可以跟踪以下类型的事件：

- CPU 周期；
- 硬件和软件性能计数器，如缓存未命中、分支未命中、页面错误、上下文切换等；
- Java 堆中的分配；
- 满足的锁定尝试，包括 Java 对象监视器和可重入锁；

支持的平台

- Linux / x64 / x86 / ARM / AArch64
- macOS / x64

### 2. 基本用法

async-profiler 通过 profiler.sh 脚本进行启动，并向需要分析的应用程序传递命令，典型的工作流：

（1）启动 Java 应用程序
（2）附加代理并开始分析
（3）运行性能场景
（4）停止分析

```s
# 输出为类似传统jprofiler，jmc格式的树状结构的html
./profiler.sh -d 60 -f /tmp/flamegraph.html 13625
# 火焰图倒置，如果要看哪个最终方法占用的CPU最多，聚合它被其他各个方法调用的总和
./profiler.sh --reverse -d 60 -f /tmp/flamegraph-reverse.svg 13625
# 每条线程一幅火焰图，如果只关心某些特定线程里的消耗
./profiler.sh -t -d 60 -f /tmp/flamegraph-reverse.svg 13625
# 耗时跟踪。默认按CPU消耗分析，如果方法在等待远程调用，数据库返回的阶段，不会统计在内。而耗时跟踪会更适合于找慢调用的情况。
# 官方文档推荐分开每条线程统计，而且将采样间隔从默认10ms降到5ms
./profiler.sh -e wall -t -i 5ms -d 60 -f /tmp/flamegraph-reverse.svg 13625
# 纯java方法的cpu消耗跟踪。如果内核参数不符合又无法改变，那cpu模式可以降级为itimer模式，不读取perf_event，仅采集java方法，不采集内核调用
./profiler.sh -e itimer -d 60 -f /tmp/flamegraph-reverse.svg 13625
# 内存分配跟踪
./profiler.sh -e alloc -d 60 -f /tmp/flamegraph-reverse.svg 13625
# 锁跟踪
# 最新版的master snapshot版支持JDK7/8（目前生产版本） ， 旧版crash in JDK7 !!!!
./profiler.sh -e lock -d 60 -f /tmp/flamegraph-reverse.svg 13625
# ContextSwitch 及更多跟踪
./profiler.sh -e context-switches -d 60 -f /tmp/flamegraph-reverse.svg 13625
# 其他可选监控，执行
./profiler.sh list 13625

```

### 3. Async-profiler 和 火焰图分析

Async-profiler 可以观测运行程序，每一段代码所占用的 cpu 的时间和比例，从而可以分析并找到项目中占用 cpu 时间最长的代码片段，优化热点代码，达到优化内存的效果。

#### 3.1 CPU 性能分析

在此模式下，profiler 收集堆栈跟踪示例，其中包括 Java 方法、native 调用、JVM 代码和内核函数。

为了能够准确的生成 Java 和 native 代码的确切性能报告，常用的方法是接收 perf_events 生成的调用堆栈，并将它们与 AsyncGetCallTrace 生成的调用堆栈进行匹配。此外 Async-profiler 还提供了一种可以在 AsyncGetCallTrace 失败的某些情况下，恢复堆栈跟踪的解决方法。

#### 3.2 查看火焰图

async-profiler 提供开箱即用的 Flame 图形支持，指定参数 -o svg 以将分析结果转储为可在所有主流浏览器中查看的交互式 svg 图像。另外，如果目标文件名以.SVG 结尾，则会自动选择 SVG 输出格式。

如下命令：

```s
./profiler.sh -d 30 -f /tmp/flamegraph.svg PID
```

纵向从下至上，代表方法间的调用关系，横条的宽度代表所占 CPU 时间的多少，绿色代表 java 方法，浅绿色代表被内联的 java 方法，黄色代表 jvm 的 c++方法，红色代表 jvm 其他 native 方法，褐色代表 Linux 内核调用。

#### 3.3 分析选项参数

```s
Usage: ./profiler.sh [action] [options] <pid>
Actions:
  start             start profiling and return immediately
  resume            resume profiling without resetting collected data
  stop              stop profiling
  check             check if the specified profiling event is available
  status            print profiling status
  list              list profiling events supported by the target JVM
  collect           collect profile for the specified period of time
                    and then stop (default action)
Options:
  -e event          profiling event: cpu|alloc|lock|cache-misses etc.
  -d duration       run profiling for <duration> seconds
  -f filename       dump output to <filename>
  -i interval       sampling interval in nanoseconds
  -j jstackdepth    maximum Java stack depth
  -b bufsize        frame buffer size
  -t                profile different threads separately
  -s                simple class names instead of FQN
  -g                print method signatures
  -a                annotate Java method names
  -o fmt            output format: summary|traces|flat|collapsed|svg|tree|jfr
  -I include        output only stack traces containing the specified pattern
  -X exclude        exclude stack traces with the specified pattern
  -v, --version     display version string

  --title string    SVG title
  --width px        SVG width
  --height px       SVG frame height
  --minwidth px     skip frames smaller than px
  --reverse         generate stack-reversed FlameGraph / Call tree

  --all-kernel      only include kernel-mode events
  --all-user        only include user-mode events
  --cstack mode     how to traverse C stack: fp|lbr|no

<pid> is a numeric process ID of the target JVM
      or 'jps' keyword to find running JVM automatically
      or the application's name as it would appear in the jps tool

Example: ./profiler.sh -d 30 -f profile.svg 3456
         ./profiler.sh start -i 999000 jps
         ./profiler.sh stop -o summary,flat jps
         ./profiler.sh -d 5 -e alloc MyAppName
```

下面是 profiler.sh 脚本接受的命令行选项的完整列表：

| 参数        | 解释                                | 示例                                           |
| ----------- | ----------------------------------- | ---------------------------------------------- |
| -e event    | 采集的事件                          | cpu/alloc/lock/cache-misses                    |
| -d N        | 分析持续时间，以秒为单位            | ./profiler.sh -d 30 PID                        |
| -f FILENAME | 要将配置文件信息转储到的文件名      | ./profiler.sh -d 30 -f /tmp/flamegraph.svg PID |
| -i N        | 设置分析间隔(ms，us，s)             | 默认分析间隔为 10ms，最小单位为 1us            |
| -j N        | 最大 java 栈深度                    | 最大 2048                                      |
| -b N        | 帧缓冲大小                          | 若不足，需要增加                               |
| -t          | 间隔采集线程                        | 输出线程的 tid                                 |
| -s          | 使用简单的类名                      | 默认包含包名                                   |
| -g          | 打印方法签名                        | 默认不输出方法参数                             |
| -a          | 通过添加\_[j]后缀来注释 Java 方法名 | 默认不区分                                     |
| -o fmt      | 输出格式                            | summary/traces/flat/collapsed/svg/tree/jfr     |
| -I pattern  | 只输出匹配的栈帧                    | -                                              |
| -X pattern  | 剔除匹配的栈帧                      | -                                              |

#### 3.4 Async-profiler 问题的回答

（0）输出的格式之间的关系

| 格式          | 说明                                                             |
| ------------- | ---------------------------------------------------------------- |
| summary       | 转储基本配置统计信息，无意义                                     |
| traces[=N]    | 线程调用栈，按栈帧区分（最多 N 个样本）；                        |
| flat[=N]      | 统计信息，按方法统计数排序（调用最多的前面 N 个方法）            |
| collapsed[=C] | 火焰图原始数据，其中每一行是一个分号分隔的帧列表，后跟一个计数器 |
| svg[=C]       | 生成 svg 格式的火焰图，浏览器打开                                |
| tree[=C]      | 调用栈按线程聚合，以 HTML 格式生成调用树，浏览器打开             |
| jfr           | 二进制                                                           |

C 是计数器类型：
（1）samples - 计数器是给定跟踪的若干样本（默认）；
（2）total - 计数器是收集的度量的总值，例如总分配大小。

（1）默认值是啥？

不启用 -t -s
默认 -e cpu
默认 -i 10ms
默认 -o summary,traces=200,flat=200

（2） -t 选项是否必要
是的，需要输出线程 tid！

（3）能否统计方法的调用时间
不能。

（4）展示的结果是否按时间排序
不是。

（5）采样统计的是时间还是次数
次数。时间=采样间隔 X 采样次数。

## 四，名词解释

### 1. perf_events

perf_events 是一个面向事件的可观察性工具，可以帮助您解决高级性能和故障排除功能。

perf_event 提供两种类型的 trace 数据：count 和 sample。count 只是记录了 event 的发生次数，sample 记录了大量信息(比如：IP、ADDR、TID、TIME、CPU、BT)。

### 2. BPF

BPF 的字面上意思 Berkeley Packet Filter 意味着它是从包过滤而来。本质上它是一种内核代码注入的技术。

BPF 的好处在哪里？ 是因为它提供了一种在不修改内核代码的情况下，可以灵活修改内核处理策略的方法。

这在包过滤和系统 tracing 这种需要频繁修改规则的场合非常有用。因为如果只在用户态修改策略的话那么所有数据需要复制一份给用户态开销较大；如果在内核态修改策略的话需要修改内核代码重新编译内核，而且容易引人安全问题。BPF 这种内核代码注入技术的生存空间就是它可以在这两者间取得一个平衡。

### 3. ePBF

eBPF(extended Berkeley Packet Filter) 是一种可以在 Linux 内核中运行用户编写的程序，而不需要修改内核代码或加载内核模块的技术。

简单说，eBPF 让 Linux 内核变得可编程化了。

eBPF 程序是一个事件驱动模型。Linux 内核提供了各种 hook point，比如 system calls, function entry/exit, kernel tracepoints, network events 等。

eBPF 程序通过实现想要关注的 hook point 的 callback，并把这些 callback 注册到相应的 hook point 来完成“内核编程”。

### 4. kprobes

Linux kprobes 调试技术是内核开发者们专门为了便于跟踪内核函数执行状态所设计的一种轻量级内核调试技术。

利用 kprobes 技术，内核开发人员可以在内核的绝大多数指定函数中动态的插入探测点来收集所需的调试状态信息而基本不影响内核原有的执行流程。

kprobes 技术目前提供了 3 种探测手段：kprobe、jprobe 和 kretprobe，其中 jprobe 和 kretprobe 是基于 kprobe 实现的，他们分别应用于不同的探测场景中。

### 5. uprobes

uprobes   是 Linux 提供用户态的动态探针，合并于 2012 年 7 月发布的 Linux 3.5 内核中。

uprobes 和 kprobes 十分相似，只不过用在用户态而已。uprobes 可以检测到用户态函数入口和出口的位置。

uprobes 工作原理和 kprobes 差不多，它会在目标位置插入一个断点，这样当程序执行流执行到这个地方会去执行我们设置的 uprobe handle。当我们不再不需探测和收集信息时候，可以移除断点恢复原状。

### 6. PMU

PMU(Performance Monitoring Unit) 是一种非常重要的数据采集方法，因为它大部分是硬件的，所以可以做到一些软件做不到的事情，获取到一些底层硬件的信息。

## 资料来源

#### 1. Async-profiler 介绍

https://blog.csdn.net/fenglibing/article/details/103534117

#### 2. async-profiler 源码

https://github.com/jvm-profiling-tools/async-profiler

#### 3. JVM CPU Profiler 技术原理及源码深度解析

https://mp.weixin.qq.com/s/RKqmy8dw7B7WtQc6Xy2CLA

#### 4. 火焰图源码

https://github.com/brendangregg/FlameGraph
https://www.brendangregg.com/flamegraphs.html

#### 5. 使用 JVMTI/JVMPI、SIGPROF 和 AsyncGetCallTrace 进行分析

http://jeremymanson.blogspot.com/2007/05/profiling-with-jvmtijvmpi-sigprof-and.html

#### 6. async-profiler

http://www.brewin073.top/index/module/blog/article/detail/125

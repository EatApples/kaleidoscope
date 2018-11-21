### 1. JVM加载jar包的顺序

#### 1.1 问题：
NoSuchMethodError
#### 1.2 解决方案：
```
（1）使用-XX:+TraceClassPaths或者在服务器上执行jinfo时，都能得到classpath包含的jar包
（2）这些jar的顺序不同的机器总是不一样的，如果有多个同名的类只会加载其中第一个
（3）问题就是jar的加载顺序问题，而这个顺序实际上是由文件系统决定的，linux内部是用inode来指示文件的
（4）一般情况下，修改了文件名，再改回来，或者从新上传一个，这个编号依然还是这个，需要改名改变文件次序
```

### 2. Bug：StampedLock的中断问题导致CPU爆满

个人感觉NotReally

StampedLock作为JAVA8中出现的新型锁，很可能在大多数场景都可以替代ReentrantReadWriteLock。它对于读/写都提供了四个接口(换成write为写锁)：

```
readLock()
tryReadLock()
tryReadLock(long time, TimeUnit unit)
readLockInterruptibly()
```

这几个方法对应的语义为：
```
获取读锁（阻塞，不响应中断）
获取读锁（立即）
限时获取读锁（响应中断）
获取读锁（阻塞，响应中断）
```

#### 2.1 测试程序
```java
import java.util.concurrent.locks.LockSupport;
import java.util.concurrent.locks.StampedLock;

public class Test {

    public static void main(String[] args) throws InterruptedException {

        final StampedLock lock = new StampedLock();
        new Thread() {

            public void run() {

                long readLong = lock.writeLock();
                LockSupport.parkNanos(6*1000*1000*1000L);
                lock.unlockWrite(readLong);
            }
        }.start();
        Thread.sleep(100);
        for (int i = 0; i < 3; ++i)
            new Thread(new OccupiedCPUReadThread(lock)).start();

    }

    private static class OccupiedCPUReadThread implements Runnable {

        private StampedLock lock;

        public OccupiedCPUReadThread(StampedLock lock) {
            this.lock = lock;
        }

        public void run() {

            Thread.currentThread().interrupt();
            long lockr = lock.readLock();
            System.out.println(Thread.currentThread().getName() + " get read lock");
            lock.unlockRead(lockr);
        }
    }

}

```

#### 2.2 原因：
实现等待的那部分逻辑在一个循环里，里面有一个 LockSupport.pack 来实现等待，满足条件后才跳出循环，结束等待，但如果线程处于中断状态，LockSupport.pack不会开始等待或继续等待，而且也不会清除线程的中断状态，所以造成了在循环里无限调用 LockSupport.pack（pack总是立即返回）的情形，所以cpu就满负荷了

#### 2.3 JDK有话说
API告诉你了就是不响应中断，你中断了飙CPU怪我咯？

### 3. BootStrap class 扩展方案
Java 命令行提供了如何扩展 bootstrap 级别 class 的简单方法：

+ （1）-Xbootclasspath：完全取代基本核心的 Java class 搜索路径。不常用，否则要重新写所有 Java 核心 class

+ （2）-Xbootclasspath/a：后缀在核心 class 搜索路径后面。常用。

+ （3）-Xbootclasspath/p：前缀在核心 class 搜索路径前面。不常用，避免引起无意义的冲突。

### 4. ExtClassLoader 加载器加载路径
-Djava.ext.dirs 会覆盖 Java 本身的 ext 设置。java.ext.dirs 指定的目录由ExtClassLoader加载器加载。

解决方案也很简单，只需在改路径后面补上ext 的路径即可！
比如：-Djava.ext.dirs=./plugin:$JAVA_HOME/jre/lib/ext

### 5. GC调试参数
#### 5.1 配置堆区
```
-Xms
-Xmx
-XX:newSize
-XX:MaxnewSize
-Xmn
```

+ -Xms：表示java虚拟机堆区内存初始内存分配的大小，通常为操作系统可用内存的1/64大小即可，但仍需按照实际情况进行分配。

+ -Xmx：表示java虚拟机堆区内存可被分配的最大上限，通常为操作系统可用内存的1/4大小。但是开发过程中，通常会将 -Xms 与 -Xmx两个参数的配置相同的值，其目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源。

+ -XX:newSize：表示新生代初始内存的大小，应该小于 -Xms的值；

+ -XX:MaxnewSize：表示新生代可被分配的内存的最大上限；当然这个值应该小于 -Xmx的值；

+ -Xmn：至于这个参数则是对 -XX:newSize、-XX:MaxnewSize两个参数的同时配置，也就是说如果通过-Xmn来配置新生代的 内存大小，那么-XX:newSize = -XX:MaxnewSize = -Xmn，虽然会很方便，但需要注意的是这个参数是在JDK1.4版本以后才使用的。

#### 5.2 配置非堆区
```
-XX:PermSize
-XX:MaxPermSize
```
+ -XX:PermSize：表示非堆区初始内存分配大小，其缩写为permanent size（持久化内存）

+ -XX:MaxPermSize：表示对非堆区分配的内存的最大上限。

这里面非常要注意的一点是：在配置之前一定要慎重的考虑一下自身软件所需要的非堆区内存大小，因为此处内存是不会被java垃圾回收机制进行处理的地方。并且更加要注意的是 最大堆内存与最大非堆内存的和绝对不能够超出操作系统的可用内存。

+ -Xss: 设置每个线程可使用的内存大小。

在相同物理内存下，减小这个值能生成更多的线程。当然操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

+ -XX: MaxTenuringThreshold

设置转入老生代的存活次数。如果是0，则直接跳过新生代进入老生代

#### 5.3 常用内存参数配置
```
-Xms256m -Xmx256m -Xmn64m -Xss256k -XX:PermSize=128m -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError
```

### 6. 常用参数使用
#### 6.1 java/javaw

+ java.exe：运行java程序

+ javac.exe：编译的，生成.class文件

+ javaw.exe：跟java命令相对的，运行java命令时，会出现并保持一个console窗口，程序中的信息可以通过System.out在console内输出，而运行javaw，开始时会出现console，当主程序调用之后，console就会消失；javaw 大多用来运行GUI程序

java.exe 用于启动window console  控制台程序

javaw.exe 用于启动 GUI程序

javaws.exe 用于web程序。


#### 6.2 jps
jps 也是一样，它的作用是显示当前系统的java进程情况，及其id号。我们可以通过它来查看我们到底启动了几个java进程（因为每一个java程序都会独占一个java虚拟机实例），和他们的进程号（为下面几个程序做准备），并可通过opt来查看这些进程的详细启动参数。

```
-q 只显示pid，不显示class名称,jar文件名和传递给main 方法的参数
-m 输出传递给main 方法的参数，在嵌入式jvm上可能是null
-l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名
-v 输出传递给JVM的参数
```

注：jps命令有个地方很不好，似乎只能显示当前用户的java进程，要显示其他用户的还是只能用unix/linux的ps命令。

#### 6.3 jstack(查看线程)、jmap(查看内存)和jstat(性能分析)命令

jstack能得到运行java程序的java stack和native stack的信息。可以轻松得知当前线程的运行情况
```
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)
    -m  to print both java and native frames (mixed mode)
    -l  long listing. Prints additional information about locks
    -h or -help to print this help message
```
在thread dump中，要留意下面几种状态

+ 死锁，Deadlock（重点关注）

+ 等待资源，Waiting on condition（重点关注）

+ 等待获取监视器，Waiting on monitor entry（重点关注）

+ 阻塞，Blocked（重点关注）

+ 执行中，Runnable

+ 暂停，Suspended

+ 对象等待中，Object.wait() 或 TIMED_WAITING

+ 停止，Parked

jmap 得到运行java程序的内存分配的详细情况。例如实例个数，大小
```
Usage:
    jmap [option] <pid>
        (to connect to running process)
    jmap [option] <executable <core>
        (to connect to a core file)
    jmap [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

如果运行在64位JVM上，可能需要指定-J-d64命令选项参数。

jmap -permstat pid
打印进程的类加载器和类加载器加载的持久代对象信息，输出：类加载器名称、对象是否存活（不可靠）、对象地址、父类加载器、已加载的类大小等信息
jmap -heap pid
查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况
jmap -histo[:live] pid
查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象

jmap -dump:format=b,file=dumpFileName pid
注意如果Dump文件太大，可能需要加上-J-Xmx512m这种参数指定最大堆内存，即
jhat -J-Xmx512m -port 9998 /tmp/dump.dat
然后就可以在浏览器中输入主机地址:9998查看了

where <option> is one of:
    <none>               to print same info as Solaris pmap
    -heap                to print java heap summary
    -histo[:live]        to print histogram of java object heap; if the "live"
                         suboption is specified, only count live objects
    -permstat            to print permanent generation statistics
    -finalizerinfo       to print information on objects awaiting finalization
    -dump:<dump-options> to dump java heap in hprof binary format
                         dump-options:
                           live         dump only live objects; if not specified,
                                        all objects in the heap are dumped.
                           format=b     binary format
                           file=<file>  dump heap to <file>
                         Example: jmap -dump:live,format=b,file=heap.bin <pid>
    -F                   force. Use with -dump:<dump-options> <pid> or -histo
                         to force a heap dump or histogram when <pid> does not
                         respond. The "live" suboption is not supported
                         in this mode.
    -h | -help           to print this help message
    -J<flag>             to pass <flag> directly to the runtime system
```

jstat 这是一个比较实用的一个命令，可以观察到classloader，compiler，gc相关信息。可以时时监控资源和性能
```
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

Definitions:
  <option>      An option reported by the -options option
  <vmid>        Virtual Machine Identifier. A vmid takes the following form:
                     <lvmid>[@<hostname>[:<port>]]
                Where <lvmid> is the local vm identifier for the target
                Java virtual machine, typically a process id; <hostname> is
                the name of the host running the target Java virtual machine;
                and <port> is the port number for the rmiregistry on the
                target host. See the jvmstat documentation for a more complete
                description of the Virtual Machine Identifier.
  <lines>       Number of samples between header lines.
  <interval>    Sampling interval. The following forms are allowed:
                    <n>["ms"|"s"]
                Where <n> is an integer and the suffix specifies the units as
                milliseconds("ms") or seconds("s"). The default units are "ms".
  <count>       Number of samples to take before terminating.
  -J<flag>      Pass <flag> directly to the runtime system.

-class
-compiler
-gc
-gccapacity
-gccause
-gcnew
-gcnewcapacity
-gcold
-gcoldcapacity
-gcpermcapacity
-gcutil
-printcompilation
```

#### 6.4  jconsole

Jconsole是一个JMX兼容的监视工具。它使用Java虚拟机的JMX机制来提供运行在Java平台的应用程序的性能与资源耗费信息。

#### 6.5 javac

```
 -g                         生成所有调试信息
 -g:none                    不生成任何调试信息
 -g:{lines,vars,source}     只生成某些调试信息
 -nowarn                    不生成任何警告
 -verbose                   输出有关编译器正在执行的操作的消息
 -deprecation               输出使用已过时的 API 的源位置
 -classpath <路径>            指定查找用户类文件和注释处理程序的位置
 -cp <路径>                   指定查找用户类文件和注释处理程序的位置
 -sourcepath <路径>           指定查找输入源文件的位置
 -bootclasspath <路径>        覆盖引导类文件的位置
 -extdirs <目录>              覆盖所安装扩展的位置
 -endorseddirs <目录>         覆盖签名的标准路径的位置
 -proc:{none,only}          控制是否执行注释处理和/或编译。
 -processor <class1>[,<class2>,<class3>...] 要运行的注释处理程序的名称; 绕过默认的搜索进程
 -processorpath <路径>        指定查找注释处理程序的位置
 -d <目录>                    指定放置生成的类文件的位置
 -s <目录>                    指定放置生成的源文件的位置
 -implicit:{none,class}     指定是否为隐式引用文件生成类文件
 -encoding <编码>             指定源文件使用的字符编码
 -source <发行版>              提供与指定发行版的源兼容性
 -target <发行版>              生成特定 VM 版本的类文件
 -version                   版本信息
 -help                      输出标准选项的提要
 -A关键字[=值]                  传递给注释处理程序的选项
 -X                         输出非标准选项的提要
 -J<标记>                     直接将 <标记> 传递给运行时系统
 -Werror                    出现警告时终止编译
 @<文件名>                     从文件读取选项和文件名

```

#### 6.6 javap

javap 是JDK自带的反汇编器，可以查看java编译器为我们生成的字节码。通过它，我们可以对照源代码和字节码，从而了解很多编译器内部的工作
```
用法: javap <options> <classes>
其中, 可能的选项包括:
  -help  --help  -?        输出此用法消息
  -version                 版本信息
  -v  -verbose             输出附加信息
  -l                       输出行号和本地变量表
  -public                  仅显示公共类和成员
  -protected               显示受保护的/公共类和成员
  -package                 显示程序包/受保护的/公共类
                           和成员 (默认)
  -p  -private             显示所有类和成员
  -c                       对代码进行反汇编
  -s                       输出内部类型签名
  -sysinfo                 显示正在处理的类的
                           系统信息 (路径, 大小, 日期, MD5 散列)
  -constants               显示静态最终常量
  -classpath <path>        指定查找用户类文件的位置
  -bootclasspath <path>    覆盖引导类文件的位置
```

#### 6.7 javadoc
```
javadoc - 关键词列表
　　@author 作者名
　　@version 版本标识
　　@parameter 参数及其意义
　　@since 最早使用该方法/类/接口的JDK版本
　　@return 返回值
　　@throws 异常类及抛出条件
　　@deprecated 引起不推荐使用的警告
　　@see reference
　　@override 重写
```

#### 6.8 jar
```
用法: jar {ctxui}[vfm0Me] [jar-file] [manifest-file] [entry-point] [-C dir] files ...

选项包括:
    -c  创建新的归档文件
    -t  列出归档目录
    -x  从档案中提取指定的 (或所有) 文件
    -u  更新现有的归档文件
    -v  在标准输出中生成详细输出
    -f  指定归档文件名
    -m  包含指定清单文件中的清单信息
    -e  为捆绑到可执行 jar 文件的独立应用程序
        指定应用程序入口点
    -0  仅存储; 不使用情况任何 ZIP 压缩
    -M  不创建条目的清单文件
    -i  为指定的 jar 文件生成索引信息
    -C  更改为指定的目录并包含其中的文件
如果有任何目录文件, 则对其进行递归处理。
清单文件名, 归档文件名和入口点名称的指定顺序
与 'm', 'f' 和 'e' 标记的指定顺序相同。

示例 1: 将两个类文件归档到一个名为 classes.jar 的归档文件中:
       jar cvf classes.jar Foo.class Bar.class
示例 2: 使用现有的清单文件 'mymanifest' 并
           将 foo/ 目录中的所有文件归档到 'classes.jar' 中:
       jar cvfm classes.jar mymanifest -C foo/ .
```

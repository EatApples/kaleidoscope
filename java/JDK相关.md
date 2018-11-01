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
``
-Xms256m -Xmx256m -Xmn64m -Xss256k -XX:PermSize=128m -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError
```

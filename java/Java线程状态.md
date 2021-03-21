图解线程状态，看完浑身通透

```
来源：低并发编程
作者：闪客sun
```

### 线程状态的实质

首先你得明白，当我们说一个线程的状态时，说的是什么？
没错，就是一个变量的值而已。

哪个变量？
Thread 类中的一个变量，叫 private volatile int threadStatus = 0;

```java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED;

}
```

### pthread_create

Thread 对象，在调用了 start() 方法后，才显现生机。

这个方法一调用，最终会调用到一个讨厌的 native 方法里：private native void start0();

跟进 jvm 源码之后（hotspot/src/os/linux/vm/os_linux.cpp），调用到了大名鼎鼎的 unix 创建线程的方法，pthread_create。

此时，在操作系统内核中，才有了一个真正的线程，被创建出来。

解释：

1. 在 Java 调用 start() 后，操作系统中才真正出现了一个线程，并且立刻运行。

2. Java 中的线程，和操作系统内核中的线程，是一对一的关系。

3. 调用 start 后，线程状态变为 RUNNABLE，这是由 native 方法里的某部分代码造成的。

### Thread.join()

打开 Thread.join() 的源码，你会发现它非常简单。

```java
public synchronized void join() {
    while (isAlive()) {
        wait();
    }
}
```

也就是说，他的本质仍然是执行了 wait() 方法，而锁对象就是 Thread t 对象本身。

那从 RUNNABLE 到 WAITING，就和执行了 wait() 方法完全一样了。

那从 WAITING 回到 RUNNABLE 是怎么实现的呢？

主线程调用了 wait ，需要另一个线程 notify 才行，难道需要这个子线程 t 在结束之前，调用一下 t.notifyAll() 么？

答案是否定的，那就只有一种可能，线程 t 结束后，由 jvm 自动调用 t.notifyAll()，不用我们程序显示写出（hotspot/src/share/vm/runtime/thread.cpp）。

```c
void JavaThread::exit(...) {
    ...
    ensure_join(this);
    ...
}
static void ensure_join(JavaThread * thread) {
    ...
    lock.notify_all(thread);
    ...
}
```

所以，其实 join 就是 wait，线程结束就是 notifyAll。

### park/unpark

一个线程调用如下方法：LockSupport.park()，该线程状态会从 RUNNABLE 变成 WAITING；另一个线程调用：LockSupport.unpark(Thread 刚刚的线程)，刚刚的线程会从 WAITING 回到 RUNNABLE。

从线程状态流转来看，与 wait 和 notify 相同；从实现机制上看，他们甚至更为简单。

1. park 和 unpark 无需事先获取锁，或者说跟锁压根无关。

2. 没有什么等待队列一说，unpark 会精准唤醒某一个确定的线程。

3. park 和 unpark 没有顺序要求，可以先调用 unpark

线程有一个计数器，初始值为 0。调用 park 就是如果这个值为 0，就将线程挂起，状态改为 WAITING。如果这个值为 1，则将这个值改为 0，其余的什么都不做。

调用 unpark 就是，将这个值改为 1。

### Thread.sleep(long)

wait 需要先获取锁，再释放锁，然后等待被 notify。

join 就是 wait 的封装。

park 需要等待 unpark 来唤醒，或者提前被 unpark 发放了唤醒许可。

那有没有一个方法，仅仅让线程挂起，只能通过等待超时时间到了再被唤醒呢。

这个方法就是 Thread.sleep(long)

#### 1. 图解线程状态，看完浑身通透

https://mp.weixin.qq.com/s/aG_uBLQevyZEFTOLK-kH9g

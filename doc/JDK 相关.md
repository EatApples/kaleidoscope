# JVM加载jar包的顺序

## 问题：NoSuchMethodError
## 解决方案：
```
（1）使用-XX:+TraceClassPaths或者在服务器上执行jinfo时，都能得到classpath包含的jar包
（2）这些jar的顺序不同的机器总是不一样的，如果有多个同名的类只会加载其中第一个
（3）问题就是jar的加载顺序问题，而这个顺序实际上是由文件系统决定的，linux内部是用inode来指示文件的
（4）一般情况下，修改了文件名，再改回来，或者从新上传一个，这个编号依然还是这个，需要改名改变文件次序
```

# Bug:StampedLock的中断问题导致CPU爆满(个人感觉NotReally)

StampedLock作为JAVA8中出现的新型锁，很可能在大多数场景都可以替代ReentrantReadWriteLock。它对于读/写都提供了四个接口(换成write为写锁)：

```
readLock()
tryReadLock()
tryReadLock(long time, TimeUnit unit)
readLockInterruptibly()
```

这几个方法对应的语义为：
获取读锁（阻塞，不响应中断）
获取读锁（立即）
限时获取读锁（响应中断）
获取读锁（阻塞，响应中断）

## 测试程序
```
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

## 原因：
实现等待的那部分逻辑在一个循环里，里面有一个 LockSupport.pack 来实现等待，满足条件后才跳出循环，结束等待，但如果线程处于中断状态，LockSupport.pack不会开始等待或继续等待，而且也不会清除线程的中断状态，所以造成了在循环里无限调用 LockSupport.pack（pack总是立即返回）的情形，所以cpu就满负荷了

## JDK有话说
API告诉你了就是不响应中断，你中断了飙CPU怪我咯？

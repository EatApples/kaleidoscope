### AQS 的独占与共享

JUC 中 ReentrantLock 与 CyclicBarrier 为独占锁，CountDownLatch 与 Semaphore 为共享锁，
ReentrantReadWriteLock 中 writeLock 为独占锁，ReadLock 为共享锁。

独占锁与共享锁的区别：

独占功能
当锁被头节点获取后，只有头节点获取锁，其余节点的线程继续沉睡，等待锁被释放后，才会唤醒下一个节点的线程。


共享功能
只要头节点获取锁成功，就在唤醒自身节点对应的线程的同时，继续唤醒 AQS 队列中的下一个节点的线程，每个节点在唤醒自身的同时还会唤醒下一个节点对应的线程，以实现共享状态的“向后传播”，从而实现共享功能

https://blog.csdn.net/qqinternet/article/details/104686888

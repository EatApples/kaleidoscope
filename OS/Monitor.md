### 1. 起源
时代在进步，并发编程逐渐成为热门，面向对象编程也逐渐普及。自然而然，人们就想把“同步”纳入更加结构化的编程环境中。

其中一种尝试就是 monitor（监视器、管程等翻译无法准确表述其含义，这里就用原文指代）。monitor 最早由 Per Brinch Hansen [BH73] 提出，随后又被 Tony Hoare [H74] 重新定义。monitor 背后的思想非常简单（锁+条件变量）。

考虑到下列使用 C++ 语言写的 monitor 伪代码：
```
monitor class account {
private:
    int balance = 0;

public:
    void deposit(int amount) {
        balance = balance + amount;
    }

    void withdraw(int amount) {
        balance = balance - amount;
    }
};
```
注意：这是“伪”代码，因为 C++ 并不支持 monitor，当然也不存在 monitor 关键字。与之相反，java 支持 monitor，即所谓的 synchronized 关键字。

在这个例子中，deposit 与 withdraw 方法（即所谓的临界区）：如果它们被多线程并发调用，可能产生竞态条件，结果将不再正确。

然而，如果这是一个 monitor 类，则不会有问题。因为 monitor 保证了有且仅有一个线程在同一时刻获得 monitor 对象，进入临界区进行操作。在本例中，多线程可以互斥地调用 deposit 与 withdraw 方法。

monitor 如何保证？答案很简单，使用锁。当一个线程想获得 monitor 时，首先会去抢占 monitor
锁。如果成功，则该线程能进入临界区执行操作。否则，该线程会阻塞，直到获得 monitor 的线程执行完为止。

下面使用 C++ 模拟 monitor 类，与之前的伪代码等效:
```
class account {
private:
    int balance = 0;
    pthread_mutex_t monitor;

public:
    void deposit(int amount) {
      pthread_mutex_lock(&monitor);
      balance = balance + amount;
      pthread_mutex_unlock(&monitor);
    }

    void withdraw(int amount) {
      pthread_mutex_lock(&monitor);
      balance = balance - amount;
      pthread_mutex_unlock(&monitor);
    }
};
```
注意到，monitor 自动化的操作只是简单地获取/释放锁操作。

### 2. 为啥要用 monitor
你可能疑惑：为啥要发明 monitor，直接使用锁不就完事了？

因为那时候，面向对象编程逐渐流行，一个自然而然的想法就是，如何优雅地将并发编程的关键点与面向对象技术相结合。

### 3. 除了锁还有啥？
回到正题，正如之前对 semaphore（信号量）的讨论，仅仅只有锁远远不够。举例来说，为了解决生产者-消费者问题，之前使用 semaphore 时，线程等待直到条件改变（例如生产者等待直到缓冲区变空），或当条件改变时唤醒线程（例如消费者消费缓冲区后有唤醒操作）。

monitor 通过所谓的条件变量来实现这种功能。你可能发现基于 monitor 的与之前基于 semaphore 的有相似之处，然而，不同之处还是很明显：条件变量必须与一个明确的状态变量绑定。在本例中，fullEntries 的状态决定了生产者/消费者是否需要等待。正相反，信号量有内置的计数器来实现同样的效果。所以，为了达到相同的目的，条件变量必须与某些状态值绑定。
```
monitor class BoundedBuffer {
private:
    int buffer[MAX];
    int fill, use;
    int fullEntries = 0;
    cond_t empty;
    cond_t full;

public:
    void produce(int element) {
      if (fullEntries == MAX)   // line P0
            wait(&empty);       // line P1
      buffer[fill] = element;   // line P2
      fill = (fill + 1) % MAX;  // line P3
      fullEntries++;            // line P4
      signal(&full);            // line P5
    }

    int consume() {
      if (fullEntries == 0)     // line C0
          wait(&full);          // line C1
      int tmp = buffer[use];    // line C2
      use = (use + 1) % MAX;    // line C3
      fullEntries--;            // line C4
      signal(&empty);           // line C5
      return tmp;               // line C6
    }
}
```

这段代码的重点在于如何使用条件变量 empty 与 full，及其之上的唤醒与等待操作。等待：阻塞在条件变量上的当前线程；唤醒：恢复在条件变量上阻塞的某个线程。

理解这些操作的语义，也就理解了它们是如何运行的。在所谓的 Hoare 语义下，唤醒操作立马唤醒一个等待线程，然后其开始执行。对于 monitor 锁，原先由唤醒线程持有，之后立马转换到被唤醒的线程，直到被唤醒的线程阻塞或结束。注意，可能有多个等待线程，唤醒操作只能唤醒其中一个，其他的仍将继续等待。

举例来说，假设有2个线程，一个是生产者，一个是消费者。消费者先执行，调用 consume() 方法，发现 fullEntries
= 0 (C0)，缓冲区没有数据，于是调用 wait(&full)
(C1) 等待缓冲区满。随后生产者执行，发现其无需等待(P0)，则在缓冲区放入元素(P2)，fill 加一(P3)，fullEntries 加一(P4)，然后执行唤醒操作(P5)。在 Hoare 语义下，该生产者在执行唤醒操作之后，将不再继续执行。控制权转移到之前等待的消费者，其从等待中恢复(C1)，消费元素(C2)，以及之后的操作。只有当消费者返回之后，生产者才会获得控制权继续执行，直至结束。

### 4. 理论与实践
Tony Hoare，一个理论学家，解决了生产者-消费者问题，定义了等待与唤醒的语义，还发明了快排[H61]。然而，对于等待与唤醒的语义，不得不说，实际上很难实现。老话说得好：理论上，理论与实际并无区别，但实际上，还是有区别的（As the old saying goes, in theory, there is no difference between theory and practice, but in practice, there is）。

不久之后，Butler Lampson 和 David Redell 发明了 Mesa 语言，计划将 monitor 作为同步原语[LR80]。作为知名的系统研究人员，很快他们就发现 Hoare 语义，很容易理解，但很难在系统中实现。为了实现一个可用的 monitor，Lampson 与 Redell 决定修改唤醒的语义。现在，唤醒仅仅作为一种 hint[L83]：唤醒只是将一个等待线程从阻塞状态变为执行状态，但该线程不会立即执行。而且，唤醒线程将继续持有控制权，直到其离开 monitor。

### 5. 又是竞争
在 Mesa 语义下，我们再检查一下上面的代码。假设消费者1进入 monitor，发现缓冲区为空，则进入等待(C1)。此时，生产者填充缓冲区，并唤醒一个线程，将其置为就绪状态。当生产者运行一段时间后，最终将让出CPU。

有没发现问题？假设另一个消费者2调用 consume() 方法，发现缓冲区非空，则开始消费，将 fullEntries 置为0，最后直接返回。还没发现问题吗？好吧，我们来看。此时，消费者1被唤醒，从等待(C1)处恢复，然而，缓冲区已经改变了，消费者1不可能继续消费了。根本原因是，在等待与唤醒期间，条件已经改变了！

幸运的是，从 Hoare 语义切换到 Mesa 语义，只要稍加改变就能正常工作：在唤醒时，被唤醒的线程应该检查当前的状态是否符合预期！因为唤醒只是 hint，在等待与唤醒之间，状态可能已经改变。

在我们的例子中，只需要修改2处，即 P0 与 C0:
```
public:
  void produce(int element) {
    while (fullEntries == MAX)  // line P0 (CHANGED IF->WHILE)
      wait(&empty);             // line P1
    buffer[fill] = element;     // line P2
    fill = (fill + 1) % MAX;    // line P3
    fullEntries++;              // line P4
    signal(&full);              // line P5
  }

  int consume() {
    while (fullEntries == 0)    // line C0 (CHANGED IF->WHILE)
      wait(&full);              // line C1
    int tmp = buffer[use];      // line C2
    use = (use + 1) % MAX;      // line C3
    fullEntries--;              // line C4
    signal(&empty);             // line C5
    return tmp;                 // line C6
  }
```

目前系统使用条件变量的等待与唤醒，都是基于 Mesa 语义。你只需记住：在被唤醒后再次检查条件就行。简单来说，就是使用 while 替代 if 来检查条件是否改变。注意：使用 while 总是正确的，即使你所运行的系统是基于 Hoare 语义的。

### 6. 深入理解
为了深入理解为啥 Mesa 语义易于实现，我们先看下 Mesa 的
monitor 是如何实现的。在 Lampson 与 Redell 的工作中[LR80]，他们引入了3种不同的队列：就绪队列，monitor 锁队列，还有状态变量队列。一个线程，同一时刻，只能处于其中一种队列。注意，程序中可能有多个 monitor 类和状态变量，对于每一个 monitor，都有这3种队列。

### 7. monitor 的改进
Lampson 与 Redell 在 Mesa 的论文中指出，monitor 需要一种全新的唤醒机制。以下面的内存分配程序为例：
```
monitor class allocator {
  int available; // how much memory is available?
  cond_t c;

  void *allocate(int size) {
    while (size > available)
    wait(&c);
    available -= size;
    // and then do whatever the allocator should do
    // and return a chunk of memory
  }

  void free(void *pointer, int size) {
    // free up some memory
    available += size;
    signal(&c);
  }
};
```

本例中，我们略去了详细的细节，仅仅聚焦于条件变量的等待与唤醒。我们将会看到，之前所说的等待与唤醒还有不足之处，你发现了吗？

假设有2个线程调用 allocate 方法，第一个调用 allocate(20)，第二个调用 allocate(10)。由于没有多余的内存，这2个线程将进入等待而阻塞。一段时间后，某个线程调用了 free(p, 15)，释放了15 比特的内存，并发出了唤醒信号。不幸的是，它唤醒的是等待 20 比特的线程。该线程检查条件，发现只有 15 比特内存可用，于是又进入等待。实际上，如果等待 10 比特的线程被唤醒的话，它能继续工作。

对于这个（唤醒丢失）问题，Lampson 与 Redell 提供了一个简单的解决方案：与其唤醒一个，不如唤醒一群（广播）。这样，所有的等待线程将被唤醒，不存在唤醒丢失的问题。

在 Mesa 语义下，广播操作总是正确的，然而，可能会带来性能问题。注意到，广播将唤醒所有等待的线程，但是只有一个线程能继续工作，其他的线程被唤醒后又会进入等待。这个问题，又被称作“惊群效应”，由于大量的上下文切换而开销巨大。除非必要，否则不要轻易使用广播。

### 8. 使用 monitor 实现 semaphore
你可能发现了 monitor 与 semaphore 有很多相似之处。对的，它们之间可以相互实现。这里我们给出使用 monitor 实现 semaphore 的示例：
```
monitor class Semaphore {
  int s; // value of the semaphore
  Semaphore(int value) {
    s = value;
  }

  void wait() {
    while (s <= 0)
      wait();
    s--;
  }

  void post() {
    s++;
    signal();
  }
};
```

如果将 semaphore 当做锁来使用，只需将 semaphore 的值设置为 1，将 wait() 与 post() 置于临界区前后即可。

### 9. 现实中的 monitor
如之前所说，C++ 没有 monitor，但可以使用锁和条件变量来模拟。如下所示，生产者-消费者问题的 C++ 解决方案：
```
class BoundedBuffer {
private:
  int buffer[MAX];
  int fill, use;
  int fullEntries;
  pthread_mutex_t monitor; // monitor lock
  pthread_cond_t empty;
  pthread_cond_t full;
public:
  BoundedBuffer() {
    use = fill = fullEntries = 0;
  }

  void produce(int element) {
    pthread_mutex_lock(&monitor);
    while (fullEntries == MAX)
      pthread_cond_wait(&empty, &monitor);
    buffer[fill] = element;
    fill = (fill + 1) % MAX;
    fullEntries++;
    pthread_cond_signal(&full);
    pthread_mutex_unlock(&monitor);
  }

  int consume() {
    pthread_mutex_lock(&monitor);
    while (fullEntries == 0)
      pthread_cond_wait(&full, &monitor);
    int tmp = buffer[use];
    use = (use + 1) % MAX;
    fullEntries--;
    pthread_cond_signal(&empty);
    pthread_mutex_unlock(&monitor);
    return tmp;
  }
}
```

有趣的是，java 语言的设计者决定支持 monitor，他们认为可以优雅地将同步原语集成到语言中。在 java 中，你可以使用 synchronized 关键字修饰方法或代码块，这些被修饰的对象可被认为是 monitor [S12a,S12b]。

在 java 的早期版本，每个 synchronized 类都有一个条件变量与之绑定。使用时，调用等待或唤醒即可。说来也奇怪，在早期版本，没有增加2个或更多条件变量的方法。你可能注意到了生产者-消费者问题的解决方案，需要2个条件变量的场景了：一个用于缓冲区空的唤醒，一个用于缓冲区满的唤醒。

为了看清只提供一个条件变量的不足，让我们只使用一个条件变量来解决生产者-消费者问题。假设2个消费者先执行，都在等待缓冲区满而阻塞。这时，生产者开启运行，往缓冲区填充了一个元素，随机唤醒了一个消费者。该生产者继续执行，尝试往缓冲区再填充元素，然而缓冲区已满，于是等待缓冲区空而阻塞。这时，有一个生产者等待缓冲区空，一个消费者等待缓冲区满，一个消费者已被唤醒将要执行。

被唤醒的消费者继续执行，消费缓冲区中的元素，随后唤醒了等待条件变量的某个线程。因为只有一个条件变量，该消费者可能唤醒另一个等待缓冲区满的消费者，而不是等待缓冲区空的生产者。这样，只使用一个条件变量来解决生产者-消费者问题便行不通（因为缓冲区中的元素早已被消费，该消费者会继续等待，出现唤醒丢失）。

怎样解决呢？使用广播便可以了。在 java 中，可以使用 notifyAll 来唤醒所有的等待线程。在本例中，广播唤醒了一个生产者与一个消费者。这个消费者发现 fullEntries 为 0，便又进入等待，而那个生产者将正常执行。正如之前的讨论，唤醒所有的等待线程将造成惊群效应。

正是因为使用一个条件变量不便，随后 java 引入了显式的 Condition 类，来有效地解决类似的并发问题。

### 10. 总结
我们介绍了 monitor，其最初由 Brinch Hansen 提出，随后又由 Hoare 发展。在 monitor 内部，线程持有 monitor 锁，阻止其他线程进入，表现出互斥语义。

我们随后介绍了条件变量，其允许线程执行等待与唤醒操作，就像 semaphore 一样。唤醒操作只是一种 hint，表明某些事情改变了，需要由被唤醒的线程去检查条件是否合法，可否继续执行。

为了解决唤醒丢失问题，引入了广播机制，但同时也带来新的问题：惊群效应。如非必要，勿用广播。

最后，由于 C++ 并不支持 monitor，我们使用 pthread 锁和条件变量模拟了 monitor。我们还看到 java 如何使用 synchronized 关键字表现出 monitor 语义，以及在当前环境下只提供一个条件变量的一些不足之处。

### 参考文献
[BH73] “Operating System Principles”
Per Brinch Hansen, Prentice-Hall, 1973
Available: http://portal.acm.org/citation.cfm?id=540365
One of the first books on operating systems; certainly ahead of its time. Introduced monitors as a concurrency primitive.

[H74] “Monitors: An Operating System Structuring Concept”
C.A.R. Hoare
CACM, Volume 17:10, pages 549–557, October 1974
An early reference to monitors; however, Brinch Hansen probably was the true inventor.

[H61] “Quicksort: Algorithm 64”
C.A.R. Hoare
CACM, Volume 4:7, July 1961
The famous quicksort algorithm.

[LR80] “Experience with Processes and Monitors in Mesa”
B.W. Lampson and D.R. Redell
CACM, Volume 23:2, pages 105–117, February 1980
An early and important paper highlighting the differences between theory and practice.

[L83] “Hints for Computer Systems Design”
Butler Lampson
ACM Operating Systems Review, 15:5, October 1983
Lampson, a famous systems researcher, loved using hints in the design of computer systems. A hint is something that is often correct but can be wrong; in this use, a signal() is telling a waiting thread that it changed the condition that the waiter was waiting on, but not to trust that the condition will be in the desired state when the waiting thread wakes up. In this paper about hints for designing systems, one of Lampson’s general hints is that you should use hints. It is not as confusing as it sounds.

[S12a] “Synchronized Methods”
Sun documentation
http://java.sun.com/docs/books/tutorial/essential/concurrency/syncmeth.html

[S12b] “Condition Interface”
Sun documentation
http://java.sun.com/j2se/1.5.0/docs/api/java/util/concurrent/locks/Condition.html

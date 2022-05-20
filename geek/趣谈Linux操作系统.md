```
趣谈 Linux 操作系统
像故事一样的操作系统入门课
刘超 前网易杭州研究院云计算技术部首席架构师
```

### 1. 这个进程如何关闭呢？

我们假设启动的程序包含某个关键字，那就可以使用下面的命令。

```s
ps -ef |grep 关键字 |awk '{print $2}'|xargs kill -9
```

### 2. fork

当父进程调用 fork 创建进程的时候，子进程将各个子系统为父进程创建的数据结构也全部拷贝了一份，甚至连程序代码也是拷贝过来的。

对于 fork 系统调用的返回值，如果当前进程是子进程，就返回 0；如果当前进程是父进程，就返回子进程的进程号。

有个系统调用 waitpid，父进程可以调用它，将子进程的进程号作为参数传给它，这样父进程就知道子进程运行完了没有，成功与否。

### 3. x86

x86 作为一个开放的营商环境，有两种模式，一种模式是实模式，只能寻址 1M，每个段最多 64K。

另一种是保护模式，对于 32 位系统，能够寻址 4G

### 4. 0 号进程

在操作系统里面，先要有个创始进程，有一行指令 set_task_stack_end_magic(&init_task)。这里面有一个参数 init_task，它的定义是 struct task_struct init_task = INIT_TASK(init_task)。它是系统创建的第一个进程，我们称为 0 号进程。这是唯一一个没有通过 fork 或者 kernel_thread 产生的进程，是进程列表的第一个。

rest_init 的第一大工作是，用 kernel_thread(kernel_init, NULL, CLONE_FS) 创建第二个进程，这个是 1 号进程。rest_init 的第一个大事情才完成。我们仅仅形成了用户态所有进程的祖先。

那内核态的进程有没有一个人统一管起来呢？有的，rest_init 第二大事情就是第三个进程，就是 2 号进程。

PID 1 的进程就是我们的 init 进程 systemd，PID 2 的进程是内核线程 kthreadd，这两个我们在内核启动的时候都见过。其中用户态的不带中括号，内核态的带中括号。

### 5. ELF

在 Linux 下面，二进制的程序也要有严格的格式，这个格式我们称为 ELF（Executeable and Linkable Format，可执行与可链接格式）。这个格式可以根据编译的结果不同，分为不同的格式。

在编译的时候，先做预处理工作，例如将头文件嵌入到正文中，将定义的宏展开，然后就是真正的编译过程，最终编译成为.o 文件，这就是 ELF 的第一种类型，可重定位文件（Relocatable File）。

形成的二进制文件叫可执行文件，是 ELF 的第二种格式。

动态链接库，就是 ELF 的第三种类型，共享对象文件（Shared Object）。

### 6. 在 Linux 中，有两种睡眠状态

一种是 TASK_INTERRUPTIBLE，可中断的睡眠状态。

另一种睡眠是 TASK_UNINTERRUPTIBLE，不可中断的睡眠状态。这是一种深度睡眠状态，不可被信号唤醒，只能死等 I/O 操作完成。

因此，这其实是一个比较危险的事情，除非程序员极其有把握，不然还是不要设置成 TASK_UNINTERRUPTIBLE。于是，我们就有了一种新的进程睡眠状态，TASK_KILLABLE，可以终止的新睡眠状态。

### 7. 进程亲缘关系

整个进程其实就是一棵进程树。而拥有同一父进程的所有进程都具有兄弟关系。

```c
struct task_struct __rcu *real_parent; /* real parent process */
struct task_struct __rcu *parent; /* recipient of SIGCHLD, wait4() reports */
struct list_head children;      /* list of my children */
struct list_head sibling;       /* linkage in my parent's children list */
```

parent 指向其父进程。当它终止时，必须向它的父进程发送信号。

children 表示链表的头部。链表中的所有元素都是它的子进程。

sibling 用于把当前进程插入到兄弟链表中。

### 8. 进程权限中 setuid 的原理

通过 chmod u+s program, 给程序设置 set-user-id 标识位, 运行时程序将进程 euid/fsuid 改为程序文件所有者 id

在 Linux 里面，一个进程可以随时通过 setuid 设置用户 ID，所以，游戏程序的用户 B 的 ID 还会保存在一个地方，这就是 suid 和 sgid，也就是 saved uid 和 save gid。

这样就可以很方便地使用 setuid，通过设置 uid 或者 suid 来改变权限。

### 9. 实时调度策略, 高优先级可抢占低优先级进程

- FIFO: 相同优先级进程先来先得
- RR: 轮流调度策略, 采用时间片轮流调度相同优先级进程
- Deadline: 在调度时, 选择 deadline 最近的进程

### 10. 普通调度策略

- normal: 普通进程
- batch: 后台进程, 可以降低优先级
- idle: 空闲时才运行

### 11. 调度类: task_struct 中 \* sched_class 指向封装了调度策略执行逻辑的类(有 5 种)

- stop: 优先级最高. 将中断其他所有进程, 且不能被打断
- dl: 实现 deadline 调度策略
- rt: RR 或 FIFO, 具体策略由 task_struct->policy 指定
- fair: 普通进程调度
- idle: 空闲进程调度

### 12. 普通进程的 fair 完全公平调度算法 CFS(Linux 实现)

- 记录进程运行时间( vruntime 虚拟运行时间)
- 优先调度 vruntime 小的进程
- 按照比例累计 vruntime, 使之考虑进优先级关系

看来 CFS 需要一个数据结构来对 vruntime 进行排序，找出最小的那个。

能够平衡查询和更新速度的是树，在这里使用的是红黑树。

在 task_struct 中有这样的成员变量：

struct sched_entity se;
struct sched_rt_entity rt;
struct sched_dl_entity dl;

这里有实时调度实体 sched_rt_entity，Deadline 调度实体 sched_dl_entity，以及完全公平算法调度实体 sched_entity。

CPU 也是这样的，每个 CPU 都有自己的 struct rq 结构，其用于描述在此 CPU 上所运行的所有进程，其包括一个实时进程队列 rt_rq 和一个 CFS 运行队列 cfs_rq，在调度时，调度器首先会先去实时进程队列找是否有实时进程需要运行，如果没有才会去 CFS 运行队列找是否有进程需要运行。

### 13. 主动调度

主动调度的过程，也即一个运行中的进程主动调用 `__schedule` 让出 CPU。

进程的调度都最终会调用到 `__schedule` 函数。为了方便你记住，我姑且给它起个名字，就叫“进程调度第一定律”。

在 `__schedule` 里面会做两件事情，第一是选取下一个进程，第二是进行上下文切换。

而上下文切换又分用户态进程空间的切换和内核态的切换。

上下文切换主要干两件事情，一是切换进程空间，也即虚拟内存；二是切换寄存器和 CPU 上下文。

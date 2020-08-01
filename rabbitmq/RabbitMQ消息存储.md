### 1. 消息持久化

RabbitMQ 的消息持久化实际包括两部分：队列索引(`rabbit_queue_index`)和消息存储(`rabbit_msg_store`)。

rabbit_queue_index 负责维护队列中落盘消息的信息，包括消息的存储地点、是否已经被交付给消费者、是否已被消费者 ack 等，每个队列都有一个与之对应的 rabbit_queue_index。

rabbit_msg_store 以键值对的形式存储消息，每个节点有且只有一个，所有队列共享。从技术层面讲 rabbit_msg_store 又可以分为 `msg_store_persistent` 和 `msg_store_transient`，其中 msg_store_persistent 负责持久化消息的存储，不会丢失，而 msg_store_transient 负责非持久化消息的存储，重启后消息会丢失。

### 2. 过期消息删除

消息的删除只是从 ETS 表删除执行消息的相关信息，同时更新对应的存储文件的相关信息，并不立即对文件中的消息进程删除，后续会有专门的垃圾回收进程负责合并待回收消息文件。

### 3. 堆积消息处理

节点消息堆积较多时，这些堆积的消息很快就会进入很深的队列中去，这样会增加处理每个消息的平均开销，整个系统的处理能力就会降低。因为要花更多的时间和资源处理堆积的消息，后流入的消息又被挤压到很深的队列中了，系统负载越来越恶化。

因此 RabbitMQ 使用时一定要注意磁盘占用监控和流控监控，这些在控制台上都可以看到，一般来说如果消息堆积过多建议增加消费者或者增强每个消费者的消费能力(比如调高 prefetch_count 消费者一次收到的消息可以提高单个消费者消费能力)。

### 4. disc 和 ram

RabbitMQ 对于 queue 中的 message 的保存方式有两种方式：`disc` 和 `ram`。

ram 和 disc 两种方式的效率比大概是 3:1。

如果使用 ram 方式，RabbitMQ 能够承载的访问量则取决于可用的内存数了。

### 5. RabbitMQ 流控

RabbitMQ 可以对内存和磁盘使用量设置阈值，当达到阈值后，生产者将被阻塞（block），直到对应项恢复正常。除了这两个阈值，RabbitMQ 在正常情况下还用流控（Flow Control）机制来确保稳定性。

Erlang 进程之间并不共享内存（binaries 类型除外），而是通过消息传递来通信，每个进程都有自己的进程邮箱。Erlang 默认没有对进程邮箱大小设限制，所以当有大量消息持续发往某个进程时，会导致该进程邮箱过大，最终内存溢出并崩溃。

在 RabbitMQ 中，如果生产者持续高速发送，而消费者消费速度较低时，如果没有流控，很快就会使内部进程邮箱大小达到内存阈值，阻塞生产者（得益于 block 机制，并不会崩溃）。然后 RabbitMQ 会进行 page 操作，将内存中的数据持久化到磁盘中。

为了解决该问题，RabbitMQ 使用了一种基于信用证的流控机制。消息处理进程有一个信用组 {InitialCredit，MoreCreditAfter}，默认值为{200, 50}。消息发送者进程 A 向接收者进程 B 发消息，每发一条消息，Credit 数量减 1，直到为 0，A 被 block 住；对于接收者 B，每接收 MoreCreditAfter 条消息，会向 A 发送一条消息，给予 A MoreCreditAfter 个 Credit，当 A 的 Credit>0 时，A 可以继续向 B 发送消息

当消息发送的速率超过了 RabbitMQ 的处理能力时，RabbitMQ 会自动减慢这个连接的速率，让 client 端以为网络带宽变小了，发送消息的速率会受限，从而达到流控的目的。

### 6. amqqueue 进程与 Paging

消息的存储和队列功能是在 amqqueue 进程中实现。为了高效处理入队和出队的消息、避免不必要的磁盘 IO，amqqueue 进程为消息设计了 4 种状态和 5 个内部队列。4 种状态包括：

- alpha，消息的内容和索引都在内存中；
- beta，消息的内容在磁盘，索引在内存；
- gamma，消息的内容在磁盘，索引在磁盘和内存中都有；
- delta，消息的内容和索引都在磁盘。

对于持久化消息，RabbitMQ 先将消息的内容和索引保存在磁盘中，然后才处于上面的某种状态（即只可能处于 alpha、gamma、delta 三种状态之一）。

5 个内部队列包括：q1、q2、delta、q3、q4。

- q1 和 q4 队列中只有 alpha 状态的消息；
- q2 和 q3 包含 beta 和 gamma 状态的消息；
- delta 队列是消息按序存盘后的一种逻辑队列，只有 delta 状态的消息。所以 delta 队列并不在内存中，其他 4 个队列则是由 erlang queue 模块实现。

消息从 q1 入队，q4 出队，在内部队列中传递的过程一般是经 q1 顺序到 q4。实际执行并非必然如此：开始时所有队列都为空，消息直接进入 q4（没有消息堆积时）；内存紧张时将 q4 队尾部分消息转入 q3，进而再由 q3 转入 delta，此时新来的消息将存入 q1（有消息堆积时）。

Paging 就是在内存紧张时触发的，paging 将大量 alpha 状态的消息转换为 beta 和 gamma；如果内存依然紧张，继续将 beta 和 gamma 状态转换为 delta 状态。Paging 是一个持续过程，涉及到大量消息的多种状态转换，所以 Paging 的开销较大，严重影响系统性能。

### 7. 测试场景

exchange 和队列都是持久化的，消息也是持久化的、固定为 1K，并且无消费者。在达到内存 paging 阈值后，生产速率降低，并持续较长时间。内存使用情况表明，在内存中的消息数目只有 18M 内容，其他消息已经 page 到磁盘中，然而进程内存仍占用 2G。Erlang 内存使用表明，Queues 占用了 2G，Binaries 占用了 2.1G。

该情况说明在消息从内存 page 到磁盘后（即从 q2、q3 队列转到 delta 后），系统中产生了大量的垃圾（garbage），而 Erlang VM 没有进行及时的垃圾回收（GC）。这导致 RabbitMQ 错误的计算了内存使用量，并持续调用 paging 流程，直到 Erlang VM 隐式垃圾回收。

### 参考资料

#### 1. 【RabbitMQ 学习记录】- 消息队列存储机制源码分析

https://blog.csdn.net/wangyiyungw/article/details/80610699

#### 2. RabbitMQ 进程结构分析与性能调优

https://www.cnblogs.com/purpleraintear/p/6033136.html

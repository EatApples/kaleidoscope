### Zab 与 Paxos

### 联系

（1）两者构建的系统都有一个 Leader 角色，Leader 进程负责协调多个 Follower 进程的运行
（2）Leader 进程都会等待超过半数的 Follower 进程做出正确的反馈后，才会将一个提案进行提交
（3）在 Zab 协议中每个 Proposal 中都包含一个 epoch 值，用来代表当前的 Leader 周期；在 Paxos 算法中，同样存在这样一个标识（Ballot）

### 区别

（1）两者的初衷或者说设计目标不一样
Paxos 算法用于构建一个分布式的一致性状态机系统；

Zab 算法用于构建一个高可用的分布式数据主备系统。

（2）流程上有区别
Paxos 算法，选举出一个新的 Leader 进程需要进行两个阶段。

第一个阶段是读阶段，这个阶段中，这个新的主进程会通过和所有其他进程进行通信的方式收集上一个主进程提出的提案，并将它们提交。

第二个阶段是写阶段，这个阶段中，当前主进程 Leader 开始提出它自己的提案。

Zab 算法存在三个阶段：发现阶段、同步阶段、广播阶段，其中发现阶段等同于 Paxos 的读阶段，广播阶段等同于 Paxos 的写阶段。

同步阶段是 Zab 算法新添加的，在同步阶段，新的 Leader 会确保存在过半的 Follower 已经提交了之前 Leader 周期中的所有事务 Proposal。

同步阶段的引入，能够有效地保证 Leader 在新的周期提出事物 Proposal 之前，所有的进程都已经完成了对之前所有事务的提交。

### Zab 与 Raft

### 相同点

（1）都使用 timeout 来重新选择 leader
（2）采用 quorum 来确定整个系统的一致性（也就是对某一个值的认可），这个 quorum 一般实现是集群中半数以上的服务器，zookeeper 里还提供了带权重的 quorum 实现
（3）都由 leader 来发起写操作
（4）都采用心跳检测存活性
（5）zookeeper 的 zab 实现里选主要求选出来的主拥有 quorum 里最新的历史，而 raft 的 follower 的选主投票根据 term 的大小+日志完成度来选择投票给谁，这点上来看是比较类似的
（6）日志和状态机

Zab 和 Raft 都是同时存在 log（还有快照技术）和状态机（内存树）的存储结构。

日志是以 log 和快照的形式持久化到磁盘，保存的是数据写的完整过程，为重启加载历史数据提供了便利，避免了服务器宕机造成的数据丢失。

状态机（内存树）把数据加载到内存中，避免了查询操作时磁盘读取，读取的是数据的最终值，从而提升读取的性能。

### 不同点

（1）zab 用的是 epoch 和 count 的组合来唯一表示一个值, 而 raft 用的是 term 和 index

（2）raft 协议数据只有单向地从 leader 到 follower（成为 leader 的条件之一就是拥有最新的 log），而 zab 的 zookeeper 实现中 ，一个 prospective leader 需要将自己的 log 更新为 quorum 里面最新的 log，然后才好在 synchronization 阶段将 quorum 里的其他机器的 log 都同步到一致

（3）请求的处理方式不同

ZK 集群中的 client 和任意一个 Node 建立 TCP 的长连接，完成所有的交互动作，而 Raft 第一次随机的获取到一个节点，然后找到 Leader 后，后续直接和 leader 交互（存疑）。

ZK 中的读请求，直接由连接的 Node 处理，不需要和 leader 汇报，也就是 Consul 中的 stale 模式。这种模式可能导致读取到的数据是过时的，但是可以保证一定是半数节点之前确认过的数据。

为了避免 Follower 的数据过时，ZK 有 sync()方法，可以保证读取到最新的数据。可以调用 sync()之后，再查询，确保所有的数据一致后再返回结果。

角色 ZK 引入了 Observer 的角色来提升性能，既可以大幅提升读取的性能，也可以不影响写的速度和选举的速度，同时一定程度上增加了容错的能力。

（4）选主投票的区别

ZK 集群之间投票消息是单向、网状的，类似于广播，比如 A 广播 A 投票给自己，广播出去，然后 B 接收到 A 的这个消息之后，会 PK A 的数据，如果 B 更适合当 leader（数据更新或者 myid 更大），B 会归档 A 的这个投票，但是不会更新自己的数据，也不会广播任何消息。除非发现 A 的数据比 B 当前存储的数据更适合当 leader，就更新自己的数据，且广播自己的最新的投票消息。

而 Raft 集群之间的所有消息都是双向的，发起一个 RPC，会有个回复结果。比如 A 向 B 发起投票，B 要么反馈投票成功，要么反馈投票不成功。

ZK 集群中，一个节点在一个 epoch 下是可以发起多次投票的，当节点发现有比之前更新的数据更适合的 leader 时，就会广播自己的最新投票消息，结合 recvset 这个 Set 的结构，来更新某个结点最新的投票结果。而 Raft 的 follower 在一个 term 里只能投票一次。

ZK 集群中，因为引入了 myid 的概念，系统倾向让数据最新和 myid 最大的节点当 leader。所以即使有半数节点都投票给同一个 Node 当 leader，这个 Node 也不一定能成为 leader，需要等待 200ms，看是不是有更适合的 leader 产生。但是在 Raft 系统中，只要某个 candidate 发现自己投票过半了，就一定能成为 leader。

ZK 集群中，每一轮的选举一定会选出一个 leader，因为每个节点允许多次投票，而且会广播自己的最新投票结果。而 Raft 系统可能涉及选票瓜分，需要重新发起一轮选举才能选出 leader，是通过选举超时时间的随机来降低选票瓜分的概率。所以 ZK 的选举理论上一般需要花费更多的时间，但是只需要一轮。Raft 每一轮选举的时间是大致固定的，但是不一定一轮就能选出 leader。

ZK 集群中，成为公认的 leader 条件更苛刻，raft 模式下，只要新 leader 发一个命令为空的 Log 出来，大家就会认同这个节点为 leader，但是在 ZK 集群中，追随 leader 的 2 种条件都很苛刻：

要么 recvset 中半数节点的选举 following 投票给 A，才会认可 A 为自己的 leader
要么 outofelection 中半数节点都认可 A 为 leader，自己才会认可 A 为自己的 leader

（5）事务操作的区别
Raft 系统中的事务消息是通过双向的 RPC 来完成的，而 Zab 中，则是单向的，一来一回两个消息来完成的。明显 Zab 的性能更好，不需要浪费 leader 一个线程去等待 follower 完成业务操作。

Zab 中 leader 发送一个 Proposal 消息给 follower，发送完成。当 follower 完成 proposal 过程时，再发送一个消息 ACK 到 leader，发送完成。leader 统计 ACK 数量过半时，触发广播 commit。

（6）操作流程当中，Zab 的即时性做的更好
Raft 的集群模式下：Leader 创建日志，广播日志，半数节点复制成功后，自己 commit 日志，运用到状态机中，反馈客户端，并且在下一个心跳包中，通知小弟们 commit。

Zab 的集群模式下：leader 创建 Proposal，广播之后，半数节点复制成功后，广播 commit。同时自己也 commit，commit 完之后再运用到内存树，反馈客户端。

### 资料来源

#### 1. Zookeeper 的 Zab 协议与 Paxos 协议区别

http://blog.itpub.net/31509949/viewspace-2218255/

#### 2. raft 协议和 zab 协议有啥区别？

https://www.zhihu.com/question/28242561/answer/40075530

### 3. Raft Vs Zab

https://www.jianshu.com/p/24307e7ca9da

### 4. Raft 对比 Zab 协议

https://yq.aliyun.com/articles/62901?spm=5176.100239.blogcont62555.13.XJ8eOB

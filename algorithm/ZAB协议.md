### 1. 什么是 Zab 协议？

Zab 协议 的全称是 Zookeeper Atomic Broadcast（Zookeeper 原子广播）。

Zookeeper 是通过 Zab 协议来保证分布式事务的最终一致性。

Zab 协议是为分布式协调服务 Zookeeper 专门设计的一种支持崩溃恢复的原子广播协议 ，是 Zookeeper 保证数据一致性的核心算法。

Zab 借鉴了 Paxos 算法，但又不像 Paxos 那样，是一种通用的分布式一致性算法。它是特别为 Zookeeper 设计的支持崩溃恢复的原子广播协议。

在 Zookeeper 中主要依赖 Zab 协议来实现数据一致性，基于该协议，Zookeeper 实现了一种主备模型（即 Leader 和 Follower 模型）的系统架构来保证集群中各个副本之间数据的一致性。

这里的主备系统架构模型，就是指只有一台客户端（Leader）负责处理外部的写事务请求，然后 Leader 客户端将数据同步到其他 Follower 节点。

Zookeeper 客户端会随机的链接到 zookeeper 集群中的一个节点，如果是读请求，就直接从当前节点中读取数据；如果是写请求，那么节点就会向 Leader 提交事务，Leader 接收到事务提交，会广播该事务，只要超过半数节点写入成功，该事务就会被提交。

Zab 协议的特性：

（1）Zab 协议需要确保那些已经在 Leader 服务器上提交（Commit）的事务最终被所有的服务器提交。

（2）Zab 协议需要确保丢弃那些只在 Leader 上被提出而没有被提交的事务。

### 2. Zab 协议状态

Looking：系统刚启动时 或者 Leader 崩溃后正处于选举状态

Following：Follower 节点所处的状态，Follower 与 Leader 处于数据同步状态

Leading：Leader 所处状态，当前集群中有一个 Leader 为主进程

在 ZooKeeper 的整个生命周期中每个节点都会在 Looking、Following、Leading 状态间不断转换。

（1）ZooKeeper 启动时所有节点初始状态为 Looking，这时集群会尝试选举出一个 Leader 节点，选举出的 Leader 节点切换为 Leading 状态；

（2）当节点发现集群中已经选举出 Leader 则该节点会切换到 Following 状态，然后和 Leader 节点保持同步；

（3）当 Follower 节点与 Leader 失去联系时 Follower 节点则会切换到 Looking 状态，开始新一轮选举；

代码实现中，多了一种状态：Observing 状态

这是 Zookeeper 引入 Observer 之后加入的，Observer 不参与选举，是只读节点，跟 Zab 协议没有关系。

### 3. Zab 协议内容

Zab 协议由 4 个阶段组成,先来看下总体的层次.

Phase 0 – Leader 选举（选出 Leader）
Phase 1 – 发现（发现全局最新的事务）
Phase 2 – 同步（各个 Follower 同步最新的事务）
Phase 3 – 广播（正常工作的流程）

Zab 协议包括两种基本的模式：崩溃恢复和消息广播。

整个 Zookeeper 就是在这两个模式之间切换。

简而言之，当 Leader 服务可以正常使用，就进入消息广播模式，当 Leader 不可用时，则进入崩溃恢复模式。

### 4. 消息广播

ZAB 协议的消息广播过程使用的是一个原子广播协议，类似一个二阶段提交过程（2PC）。对于客户端发送的写请求，全部由 Leader 接收，Leader 将请求封装成一个事务 Proposal，将其发送给所有 Follower，然后，根据所有 Follower 的反馈，如果超过半数成功响应，则执行 commit 操作（先提交自己，再发送 commit 给所有 Follower）。

消息广播具体步骤

（1）客户端发起一个写操作请求。

（2）Leader 服务器将客户端的请求转化为事务 Proposal 提案，同时为每个 Proposal 分配一个全局的 ID，即 ZXID。

（3）Leader 服务器为每个 Follower 服务器分配一个单独的队列，然后将需要广播的 Proposal 依次放到队列中去，并且根据 FIFO 策略进行消息发送。

（4）Follower 接收到 Proposal 后，会首先将其以事务日志的方式写入本地磁盘中，写入成功后向 Leader 反馈一个 Ack 响应消息。

（5）Leader 接收到超过半数以上 Follower 的 Ack 响应消息后，即认为消息发送成功，可以发送 commit 消息。

（6）Leader 向所有 Follower 广播 commit 消息，同时自身也会完成事务提交。Follower 接收到 commit 消息后，会将上一条事务提交。

zookeeper 采用 Zab 协议的核心，就是只要有一台服务器提交了 Proposal，就要确保所有的服务器最终都能正确提交 Proposal。这也是 CAP/BASE 实现最终一致性的一个体现。

Leader 服务器与每一个 Follower 服务器之间都维护了一个单独的 FIFO 消息队列进行收发消息，使用队列消息可以做到异步解耦。 Leader 和 Follower 之间只需要往队列中发消息即可。如果使用同步的方式会引起阻塞，性能要下降很多。

### 5. 崩溃恢复

一旦 Leader 服务器出现崩溃或者由于网络原因导致 Leader 服务器失去了与过半 Follower 的联系，那么就会进入崩溃恢复模式。

在 Zab 协议中，为了保证程序的正确运行，整个恢复过程结束后需要选举出一个新的 Leader 服务器。因此 Zab 协议需要一个高效且可靠的 Leader 选举算法，从而确保能够快速选举出新的 Leader。

Leader 选举算法不仅仅需要让 Leader 自己知道自己已经被选举为 Leader ，同时还需要让集群中的所有其他机器也能够快速感知到选举产生的新 Leader 服务器。

崩溃恢复主要包括两部分：Leader 选举和数据恢复。

刚刚我们说消息广播过程中，Leader 崩溃怎么办？还能保证数据一致吗？如果 Leader 先本地提交了，然后 commit 请求没有发送出去，怎么办？
实际上，当 Leader 崩溃，即进入我们开头所说的崩溃恢复模式（崩溃即：Leader 失去与过半 Follower 的联系）。下面来详细讲述。

假设 1：Leader 在复制数据给所有 Follower 之后崩溃，怎么办？

假设 2：Leader 在收到 Ack 并提交了自己，同时发送了部分 commit 出去之后崩溃怎么办？

针对这些问题，ZAB 定义了 2 个原则：

（1）ZAB 协议确保那些已经在 Leader 提交的事务最终会被所有服务器提交。

（2）ZAB 协议确保丢弃那些只在 Leader 提出/复制，但没有提交的事务。

所以，ZAB 设计了下面这样一个选举算法：能够确保提交已经被 Leader 提交的事务，同时丢弃已经被跳过的事务。

针对这个要求，如果让 Leader 选举算法能够保证新选举出来的 Leader 服务器拥有集群中所有机器编号（即 ZXID 最大）的事务，那么就能够保证这个新选举出来的 Leader 一定具有所有已经提交的提案。

而且这么做有一个好处是：可以省去 Leader 服务器检查事务的提交和丢弃工作的这一步操作。

### 6. Fast Leader Election（快速选举）

前面提到的 FLE 会选举拥有最新 Proposal history （lastZxid 最大）的节点作为 Leader，这样就省去了发现最新提议的步骤。这是基于拥有最新提议的节点也拥有最新的提交记录。

成为 Leader 的条件：

（1）选 epoch 最大的

（2）若 epoch 相等，选 ZXID 最大的

（3）若 epoch 和 ZXID 相等，选择 server_id 最大的（zoo.cfg 中的 myid）

在 Looking 状态中选举出 Leader 节点，Leader 的 LastZXID 总是最新的（只有 lastZXID 的节点才有资格成为 Leader，这种情况下选举出来的 Leader 总有最新的事务日志）。在选举的过程中会对每个 Follower 节点的 ZXID 进行对比只有 highestZXID 的 Follower 才可能当选 Leader

（1）每个 Follower 都向其他节点发送选自身为 Leader 的 Vote 投票请求，等待回复；

（2）Follower 接受到的 Vote 如果比自身的大（ZXID 更加新）时则投票，并更新自身的 Vote（向其他节点广播新的 Vote 投票请求），否则拒绝投票；

（3）每个 Follower 中维护着一个投票记录表，当某个节点收到过半的投票时，结束投票并把该 Follower 选为 Leader，投票结束；

这样，我们刚刚假设的两个问题便能够解决。假设 1 最终会丢弃调用没有提交的数据，假设 2 最终会同步所有服务器的数据。这个时候，就引出了一个问题，如何同步？

### 7. Recovery Phase（恢复阶段）

Follower 节点向准 Leader 推送 FollowerInfo，该信息包含了上一周期的 epoch，接受准 Leader 的 NEWLEADER 指令。

这一阶段 Follower 发送他们的 lastZxid 给 Leader，Leader 根据 lastZxid 决定如何同步数据。

同步策略：

（1）SNAP：如果 Follower 数据太老，Leader 将发送快照 SNAP 指令给 Follower 同步数据；

（2）DIFF：Leader 发送从 Follolwer.lastZXID 到 Leader.lastZXID 议案的 DIFF 指令给 Follower 同步数据；

（3）TRUNC：当 Follower.lastZXID 比 Leader.lastZXID 大时，Leader 发送从 Leader.lastZXID 到 Follower.lastZXID 的 TRUNC 指令让 Follower 丢弃该段数据；（当老 Leader 在 Commit 前挂掉，但是已提交到本地）

Follower 将所有事务都同步完成后，Leader 会把该节点添加到可用 Follower 列表中；

Follower 接收 Leader 的 NEWLEADER 指令，如果该指令中 epoch 比当前 Follower 的 epoch 小那么 Follower 转到 Election 阶段。

### 8. ZXID 如何生成

当崩溃恢复之后，需要在正式工作之前（接收客户端请求），Leader 服务器首先确认事务是否都已经被过半的 Follower 提交了，即是否完成了数据同步。目的是为了保持数据一致。

当所有的 Follower 服务器都成功同步之后，Leader 会将这些服务器加入到可用服务器列表中。

实际上，Leader 服务器处理或丢弃事务都是依赖着 ZXID 的，那么这个 ZXID 如何生成呢？

答：在 ZAB 协议的事务编号 ZXID 设计中，ZXID 是一个 64 位的数字。ZXID 的低 32 位是一个递增的计数器，表示该事务的序号；高 32 位是 Leader 的任期编号（epoch）。

每个新选举出的 Leader 节点，会取出本地日志中最大事务 Proposal 的 ZXID，然后解析出对应的 epoch，把该值加 1 作为该新 Leader 节点的 epoch，然后将低 32 位重新从 0 开始计。低 32 位可以看作是一个简单的递增的计数器。针对客户端的每一个事务请求，Leader 都会产生一个新的事务 Proposal 并对该计数器进行 + 1 操作。

高 32 位代表了每代 Leader 的唯一性，低 32 代表了每代 Leader 中事务的唯一性。同时，也能让 Follower 通过高 32 位识别不同的 Leader，能够有效避免不同的 Leader 错误的使用了相同的 ZXID 编号提出了不一样的 Proposal 的异常情况，简化了数据恢复流程。

基于这样的策略：当 Follower 链接上 Leader 之后，Leader 服务器会根据自己服务器上最后被提交的 ZXID 和 Follower 上的 ZXID 进行比对，比对结果要么回滚，要么和 Leader 同步。

### 9. Zookeeper 处于强一致与弱一致之间，单调一致？

顺序一致性（Sequential Consistency）

### 资料来源

#### 1. Zab vs. Paxos

https://cwiki.apache.org/confluence/display/ZOOKEEPER/Zab+vs.+Paxos

#### 2. Zab1.0

https://cwiki.apache.org/confluence/display/ZOOKEEPER/Zab1.0

#### 3. ZooKeeper Programmer's Guide

http://zookeeper.apache.org/doc/current/zookeeperProgrammers.html#ch_zkGuarantees

#### 4. ZooKeeper’s atomic broadcast protocol: Theory and practice

http://www.tcs.hut.fi/Studies/T-79.5001/reports/2012-deSouzaMedeiros.pdf

#### 5. Zab: High-performance broadcast for primary-backup systems

http://www.cs.cornell.edu/courses/cs6452/2012sp/papers/zab-ieee.pdf

#### 6. 面试官问：ZooKeeper 一致性协议 ZAB 原理

https://www.jianshu.com/p/a3e8a4634484

#### 7. Zookeeper——一致性协议:Zab 协议

https://www.jianshu.com/p/2bceacd60b8a

#### 8. 分布式事务与一致性算法 Paxos & raft & zab

https://blog.csdn.net/followMyInclinations/java/article/details/52870418

#### 9. 深入理解 Zookeeper 系列(1):ZAB 协议

https://www.jianshu.com/p/de1063814b17

#### 10. Zab 系列 3 选举

https://www.jianshu.com/p/0d2390c242f6

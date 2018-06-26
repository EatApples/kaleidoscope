##Paxos

### 1. 分布式系统的特点
* 分布性
* 对等性
* 并发性
* 缺乏全局时钟
* 故障总会发生

### 2. 问题和假设
分布式系统中的节点通信存在两种模型：共享内存（Shared memory）和消息传递（Messages passing）。

基于消息传递通信模型的分布式系统，不可避免的会发生以下错误：进程可能会慢、被杀死或者重启，消息可能会延迟、丢失、重复。在基础Paxos场景中，先不考虑可能出现消息篡改即拜占庭错误的情况。

Paxos算法解决的问题是在一个可能发生上述异常的分布式系统中如何就某个值达成一致，保证不论发生以上任何异常，都不会破坏决议的一致性。

一个典型的场景是，在一个分布式数据库系统中，如果各节点的初始状态一致，每个节点都执行相同的操作序列，那么他们最后能得到一个一致的状态。为保证每个节点执行相同的命令序列，需要在每一条指令上执行一个「一致性算法」以保证每个节点看到的指令一致。一个通用的一致性算法可以应用在许多场景中，是分布式计算中的重要问题。因此从20世纪80年代起对于一致性算法的研究就没有停止过。

为描述Paxos算法，Lamport虚拟了一个叫做Paxos的希腊城邦，这个岛按照议会民主制的政治模式制订法律，但是没有人愿意将自己的全部时间和精力放在这种事情上。所以无论是议员，议长或者传递纸条的服务员都不能承诺别人需要时一定会出现，也无法承诺批准决议或者传递消息的时间。但是这里假设没有拜占庭将军问题（Byzantine failure，即虽然有可能一个消息被传递了两次，但是绝对不会出现错误的消息）；只要等待足够的时间，消息就会被传到。另外，Paxos岛上的议员是不会反对其他议员提出的决议的。

对应于分布式系统，议员对应于各个节点，制定的法律对应于系统的状态。各个节点需要进入一个一致的状态，例如在独立Cache的对称多处理器系统中，各个处理器读内存的某个字节时，必须读到同样的一个值，否则系统就违背了一致性的要求。一致性要求对应于法律条文只能有一个版本。议员和服务员的不确定性对应于节点和消息传递通道的不可靠性。

### 3. Paxos算法片段

The safety requirements for consensus are

一致性需要满足的安全性：
+ Only a value that has been proposed may be chosen

只有提交的值才能成为决定值

+ Only a single value is chosen, and

只能有一个决定值

+ A process never learns that a value has been chosen unless it actually has been

在一个值成为决定值之前是不可见的

three roles in the consensus algorithm be performed by three classes of agents: proposers, acceptors, and learners. In an implementation,
a single process may act as more than one agent

三种角色：发起人；决策人；记录人

Assume that agents can communicate with one another by sending messages. We use the customary asynchronous, non-Byzantine model, in which:

假设没有拜占庭将军问题，即消息不被篡改

Agents operate at arbitrary speed, may fail by stopping, and may restart. Since all agents may fail after a value is chosen and then restart, a solution is impossible unless some information can be re membered by an agent that has failed and restarted.

不可假设速度

Messages can take arbitrarily long to be delivered, can be duplicated, and can be lost, but they are not corrupted.

消息不被篡改

The easiest way to choose a value is to have a single acceptor agent. A proposer sends a proposal to the acceptor, who chooses the first proposed value that it receives

假设只有一个决策人，表决第一个收到的提案，决议成为最终决定。但他挂了怎么办？所以需要决策人是多个。

The value is chosen when a large enough set of acceptors have accepted it.

为了保证一致，提案需要超过半数的决策人表决

In the absence of failure or message loss, we want a value to be chosen even if only one value is proposed by a single proposer. This suggests the
requirement:
	P1. An acceptor must accept the first proposal that it receives

即使只有一个提案也能形成最终决定，需要：决策人表决其第一次接收的提案

带来问题：没有超过半数的决议

P1 and the requirement that a value is chosen only when it is accepted by a majority of acceptors imply that an acceptor must be allowed to accept
more than one proposal

决策人可以表决多个提案

We keep track of the different proposals that an acceptor may accept by assigning a (natural) number to each proposal, so a
proposal consists of a proposal number and a value.

为了区分提案，提案有一个全序的序号

We can allow multiple proposals to be chosen, but we must guarantee that all chosen proposals have the same value

幂等：可以表决多个提案，但提案必须内容一致

P2. If a proposal with value v is chosen, then every higher-numbered proposal that is chosen has value v.

表述为：如果某个提案被表决了，则其他新的表决的提案具有相同的内容

推广：
P2a. If a proposal with value v is chosen, then every higher-numbered proposal accepted by any acceptor has value v

The famous result of Fischer, Lynch, and Patterson [1] implies that a reliable algorithm for electing a proposer must use either randomness or real time—for example, by using timeouts.


### 4. Basic Paxos（来自维基百科）
This protocol is the most basic of the Paxos family. Each instance of the Basic Paxos protocol decides on a single output value. The protocol proceeds over several rounds. A successful round has two phases. A Proposer should not initiate Paxos if it cannot communicate with at least a Quorum of Acceptors:

#### 4.1 Phase 1a: Prepare
A Proposer (the leader) creates a proposal identified with a number N.

This number must be greater than any previous proposal number used by this Proposer.

Then, it sends a Prepare message containing this proposal to a Quorum of Acceptors.

The Proposer decides who is in the Quorum.

#### 4.2 Phase 1b: Promise
If the proposal's number N is higher than any previous proposal number received from any Proposer by the Acceptor, then the Acceptor must return a promise to ignore all future proposals having a number less than N.

If the Acceptor accepted a proposal at some point in the past, it must include the previous proposal number and previous value in its response to the Proposer.

Otherwise, the Acceptor can ignore the received proposal.

It does not have to answer in this case for Paxos to work.

However, for the sake of optimization, sending a denial (Nack) response would tell the Proposer that it can stop its attempt to create consensus with proposal N.

#### 4.3 Phase 2a: Accept Request
If a Proposer receives enough promises from a Quorum of Acceptors, it needs to set a value to its proposal.

If any Acceptors had previously accepted any proposal, then they'll have sent their values to the Proposer, who now must set the value of its proposal to the value associated with the highest proposal number reported by the Acceptors.

If none of the Acceptors had accepted a proposal up to this point, then the Proposer may choose any value for its proposal.

The Proposer sends an Accept Request message to a Quorum of Acceptors with the chosen value for its proposal.

#### 4.4 Phase 2b: Accepted
If an Acceptor receives an Accept Request message for a proposal N, it must accept it if and only if it has not already promised to any prepare proposals having an identifier greater than N.

In this case, it should register the corresponding value v and send an Accepted message to the Proposer and every Learner.

Else, it can ignore the Accept Request.

Note that an Acceptor can accept multiple proposals. These proposals may even have different values in the presence of certain failures.

However, the Paxos protocol will guarantee that the Acceptors will ultimately agree on a single value.

Rounds fail when multiple Proposers send conflicting Prepare messages, or when the Proposer does not receive a Quorum of responses (Promise or Accepted). In these cases, another round must be started with a higher proposal number.

Notice that when Acceptors accept a request, they also acknowledge the leadership of the Proposer. Hence, Paxos can be used to select a leader in a cluster of nodes.

#### 我的理解
只需3分钟，你就会和我一样，彻底理解 Paxos 算法。

#### 1. 基本概念
+ 三种角色：发起人；决策人；记录人
+ 两个阶段：验证阶段；提交阶段
+ 序列号：全局有序编号
+ 消息：序列号+值
+ 信息：序列号或消息
+ 要求：信息不被篡改；信息在两个阶段中超过半数决策人通过才有效

#### 2. 算法描述
#### 2.1 验证阶段

**发起人**：
```java
发送序列号至决策人
```

**决策人**
```java
IF 第一次收到序列号
THEN 记录序列号，回复通过;
ELSE IF 序列号比记录的序列号大
THEN 更新记录的序列号，回复通过;
ELSE 回复拒绝;

IF 回复通过且有记录消息
THEN 额外回复记录的消息;
```

#### 2.2 提交阶段：
**发起人**
```java
IF 验证阶段决策人回复的通过不超过半数
THEN 重新开始验证阶段;

IF 未收到决策人回复的消息
THEN 提交验证阶段的序列号+任意值至决策人;
ELSE 提交验证阶段的序列号+从回复的消息中选取序列号号最大的那个值至决策人;
```
**决策人**
```java
IF 消息的序列号比记录的序列号大或相同
THEN 更新记录的消息，回复通过;
ELSE 回复拒绝;
```

**记录人**
```
只要某个提交的消息得到超过半数的决策人的通过回复，则这个消息的值在现在，以及未来都是这一主题的最终决定值;
也就是说以后可以通过各种消息，但只要是这个主题，消息的值都一样;
这个值可以由发起人通知记录人，也可以是决策人回复消息通过的情况下通知记录人。
```

### 5. 2PC（两阶段提交协议）
确切地说：两阶段提交主要保证了分布式事务的原子性：即所有结点要么全做要么全不做

二阶段提交(Two-phaseCommit)是指，在计算机网络以及数据库领域内，为了使基于分布式系统架构下的所有节点在进行事务提交时保持一致性而设计的一种算法(Algorithm)。

通常，二阶段提交也被称为是一种协议(Protocol))。在分布式系统中，每个节点虽然可以知晓自己的操作时成功或者失败，却无法知道其他节点的操作的成功或失败。当一个事务跨越多个节点时，为了保持事务的ACID特性，需要引入一个作为协调者的组件来统一掌控所有节点(称作参与者)的操作结果并最终指示这些节点是否要把操作结果进行真正的提交(比如将更新后的数据写入磁盘等等)。因此，二阶段提交的算法思路可以概括为：参与者将操作成败通知协调者，再由协调者根据所有参与者的反馈情报决定各参与者是否要提交操作还是中止操作。

所谓的两个阶段是指：第一阶段：准备阶段(投票阶段)和第二阶段：提交阶段（执行阶段）。

#### 5.1 准备阶段

事务协调者(事务管理器)给每个参与者(资源管理器)发送Prepare消息，每个参与者要么直接返回失败(如权限验证失败)，要么在本地执行事务，写本地的redo和undo日志，但不提交，到达一种“万事俱备，只欠东风”的状态。

可以进一步将准备阶段分为以下三个步骤：

1）协调者节点向所有参与者节点询问是否可以执行提交操作(vote)，并开始等待各参与者节点的响应。

2）参与者节点执行询问发起为止的所有事务操作，并将Undo信息和Redo信息写入日志。（注意：若成功这里其实每个参与者已经执行了事务操作）

3）各参与者节点响应协调者节点发起的询问。如果参与者节点的事务操作实际执行成功，则它返回一个”同意”消息；如果参与者节点的事务操作实际执行失败，则它返回一个”中止”消息。

#### 5.2 提交阶段

如果协调者收到了参与者的失败消息或者超时，直接给每个参与者发送回滚(Rollback)消息；否则，发送提交(Commit)消息；参与者根据协调者的指令执行提交或者回滚操作，释放所有事务处理过程中使用的锁资源。(注意:必须在最后阶段释放锁资源)

接下来分两种情况分别讨论提交阶段的过程。

当协调者节点从所有参与者节点获得的相应消息都为”同意”时:

success

1）协调者节点向所有参与者节点发出”正式提交(commit)”的请求。

2）参与者节点正式完成操作，并释放在整个事务期间内占用的资源。

3）参与者节点向协调者节点发送”完成”消息。

4）协调者节点受到所有参与者节点反馈的”完成”消息后，完成事务。

如果任一参与者节点在第一阶段返回的响应消息为”中止”，或者 协调者节点在第一阶段的询问超时之前无法获取所有参与者节点的响应消息时：

fail

1）协调者节点向所有参与者节点发出”回滚操作(rollback)”的请求。

2）参与者节点利用之前写入的Undo信息执行回滚，并释放在整个事务期间内占用的资源。

3）参与者节点向协调者节点发送”回滚完成”消息。

4）协调者节点受到所有参与者节点反馈的”回滚完成”消息后，取消事务。

不管最后结果如何，第二阶段都会结束当前事务。

二阶段提交看起来确实能够提供原子性的操作，但是不幸的事，二阶段提交还是有几个缺点的：

1、同步阻塞问题。执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。

2、单点故障。由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）

3、数据不一致。在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据部一致性的现象。

4、二阶段无法解决的问题：协调者再发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。

由于二阶段提交存在着诸如同步阻塞、单点问题、脑裂等缺陷，所以，研究者们在二阶段提交的基础上做了改进，提出了三阶段提交。

### 6. 3PC（三阶段提交协议）

三阶段提交（Three-phase commit），也叫三阶段提交协议（Three-phase commit protocol），是二阶段提交（2PC）的改进版本。

与两阶段提交不同的是，三阶段提交有两个改动点。

1、引入超时机制。同时在协调者和参与者中都引入超时机制。

2、在第一阶段和第二阶段中插入一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。

也就是说，除了引入超时机制之外，3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有CanCommit、PreCommit、DoCommit三个阶段。

#### 6.1 CanCommit阶段

3PC的CanCommit阶段其实和2PC的准备阶段很像。协调者向参与者发送commit请求，参与者如果可以提交就返回Yes响应，否则返回No响应。

1.事务询问：协调者向参与者发送CanCommit请求。询问是否可以执行事务提交操作。然后开始等待参与者的响应。

2.响应反馈：参与者接到CanCommit请求之后，正常情况下，如果其自身认为可以顺利执行事务，则返回Yes响应，并进入预备状态。否则反馈No

#### 6.2 PreCommit阶段

协调者根据参与者的反应情况来决定是否可以记性事务的PreCommit操作。根据响应情况，有以下两种可能。

假如协调者从所有的参与者获得的反馈都是Yes响应，那么就会执行事务的预执行。

1.发送预提交请求：协调者向参与者发送PreCommit请求，并进入Prepared阶段。

2.事务预提交：参与者接收到PreCommit请求后，会执行事务操作，并将undo和redo信息记录到事务日志中。

3.响应反馈：如果参与者成功的执行了事务操作，则返回ACK响应，同时开始等待最终指令。

假如有任何一个参与者向协调者发送了No响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断。

1.发送中断请求：协调者向所有参与者发送abort请求。

2.中断事务：参与者收到来自协调者的abort请求之后（或超时之后，仍未收到协调者的请求），执行事务的中断。

#### 6.3 doCommit阶段

该阶段进行真正的事务提交，也可以分为以下两种情况。

执行提交

1.发送提交请求：协调接收到参与者发送的ACK响应，那么他将从预提交状态进入到提交状态。并向所有参与者发送doCommit请求。

2.事务提交：参与者接收到doCommit请求之后，执行正式的事务提交。并在完成事务提交之后释放所有事务资源。

3.响应反馈：事务提交完之后，向协调者发送Ack响应。

4.完成事务：协调者接收到所有参与者的ack响应之后，完成事务。

中断事务：协调者没有接收到参与者发送的ACK响应（可能是接受者发送的不是ACK响应，也可能响应超时），那么就会执行中断事务。

1.发送中断请求：协调者向所有参与者发送abort请求

2.事务回滚：参与者接收到abort请求之后，利用其在阶段二记录的undo信息来执行事务的回滚操作，并在完成回滚之后释放所有的事务资源。

3.反馈结果：参与者完成事务回滚之后，向协调者发送ACK消息

4.中断事务：协调者接收到参与者反馈的ACK消息之后，执行事务的中断。

在doCommit阶段，如果参与者无法及时接收到来自协调者的doCommit或者rebort请求时，会在等待超时之后，会继续进行事务的提交。（其实这个应该是基于概率来决定的，当进入第三阶段时，说明参与者在第二阶段已经收到了PreCommit请求，那么协调者产生PreCommit请求的前提条件是他在第二阶段开始之前，收到所有参与者的CanCommit响应都是Yes。（一旦参与者收到了PreCommit，意味他知道大家其实都同意修改了）

所以，一句话概括就是，当进入第三阶段时，由于网络超时等原因，虽然参与者没有收到commit或者abort响应，但是他有理由相信：成功提交的几率很大。 ）

### 7. 2PC与3PC的区别

相对于2PC，3PC主要解决的单点故障问题，并减少阻塞，因为一旦参与者无法及时收到来自协调者的信息之后，他会默认执行commit。而不会一直持有事务资源并处于阻塞状态。但是这种机制也会导致数据一致性问题，因为，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

了解了2PC和3PC之后，我们可以发现，无论是二阶段提交还是三阶段提交都无法彻底解决分布式的一致性问题。

Google Chubby的作者Mike Burrows说过， there is only one consensus protocol, and that’s Paxos” – all other approaches are just broken versions of Paxos. 意即世上只有一种一致性算法，那就是Paxos，所有其他一致性算法都是Paxos算法的不完整版。

### 参考资料
#### 1. 2PC到3PC到Paxos到Raft到ISR
https://segmentfault.com/a/1190000004474543

#### 2. 维基百科中Paxos条目
https://en.wikipedia.org/wiki/Paxos_(computer_science)

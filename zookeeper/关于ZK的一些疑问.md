### 关于 ZK 的一些疑问

（1）ZK 的数据如何同步？

保证数据同步有两种情况：第一种重新选取 leader 之后的数据同步，第二种 leader 处理事务请求后与 follower 的数据同步。

当 leader 收到请求后，将事务请求转化成事务 proposal，由于 leader 为每一个 follower 创建一个队列，并把该事务放入响应队列中，保证事务的顺序性。之后在队列中顺序地向 follower 广播该提案。

follower 接收到提案后，以事务的形式写入本地日志中，并向 leader 发送 ack。

当超过半数的 follower 向 leader 发送恢复，leader 会向其他节点发送 commit 消息，同时 leader 提交该事务。

（2）Watch 是不是只是一种 Hint？

是的，一种 Hint，只是告知 <事件类型，通知状态，节点路径>，不告知具体内容

（3）ZAB 一致性协议如何工作？

ZAB 让整个 Zookeeper 集群在两个模式之间转换，消息广播和崩溃恢复，消息广播可以说是一个简化版本的 2PC，通过崩溃恢复解决了 2PC 的单点问题，通过队列解决了 2PC 的同步阻塞问题。

而支持崩溃恢复后数据准确性的就是数据同步了，数据同步基于事务的 ZXID 的唯一性来保证。通过 + 1 操作可以辨别事务的先后顺序。

（4）ZK 连接状态的迁移过程？CONNECTIONLOSS 与 SESSIONEXPIRED 之间还有其他状态？

应该是 curator 自己的封装

（5）ZK 集群的日志问题如何解决？

zk 自己不会进行日志清理，需要运维人员进行日志清理

从 ZooKeeper 3.4.0 开始，ZooKeeper 提供了自动清理事务日志和快照的功能

ZK 配置参数和日志清理
https://blog.csdn.net/zhouhao88410234/article/details/101293656
http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_logging

（6）ZK 数据如何迁移？

通过集群的扩容和缩容；

拷贝事务日志和快照日志；

（7）Zk Watch 事件的次序性如何保证？即所有客户端看到的次序与服务端看到的次序一致

ZooKeeper 保证了一个顺序：一个客户端在收到 watch 事件之前，一定不会看到它设置过 watch 的值的变动。网络时延和其他因素可能会导致不同的客户端看到 watch 和更新返回值的时间不同。但关键点是，每个客户端所看到的每件事都是有顺序的。

（8）客户端执行 ZK 事件是次序的？

是次序的。本地是一个同步阻塞队列，事件是次序处理的。注意处理的程序不能阻塞，否则会影响后续处理。

（9）ZK 的读能够看见最新的写操作？

ZooKeeper 不能确保任何客户端能够获取（即 Read Request）到一样的数据，除非客户端自己要求：方法是客户端在获取数据之前调用 sync()

资料：zookeeper 面试题 2 比较乱
https://www.cnblogs.com/shan1393/p/9479109.html

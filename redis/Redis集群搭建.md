为了达到redis的高可用，有两种部署方式：主从复制+哨兵机制；集群模式。哨兵机制是redis2.8开始支持。集群模式是redis3.0开始支持。

不管是哨兵模式还是集群模式，Jedis客户端都支持！

### 1. 直接启动
```
nohup ./redis-server --protected-mode no 1>/dev/null 2>&1 &
```
### 2. 指定配置文件启动
```
nohup ./redis-server 配置文件路径/redis.conf 1>/dev/null 2>&1 &
```

配置文件中，注意：
```
bind IP                          # IP白名单，这里应该注释掉
protected-mode no                # 保护模式，这里应该关掉
daemonize yes                    # 后台启动
requirepass YOUR_PASSWORD        # 设置密码
masterauth  YOUR_PASSWORD        # 集群主从同步需要密码，与上面的相同
port 6379                        # 监听端口
cluster-enabled yes              # 集群模式或单点模式
cluster-config-file nodes.conf   # 集群模式下节点信息的存储文件，用户无需编辑
cluster-node-timeout 5000        # 集群节点超时时间
appendonly yes                   # 开启AOF模式
```

如果集群中设置了密码，如果需要使用 redis-trib.rb 的各种命令，需要找到 client.rb（与Redis版本相关的那个，一般在 /usr/local/lib/ruby 目录下），然后修改password！

### 3. 客户端连接
```
./redis-cli -h HOST -p PORT -a  PASSWORD
```

连接操作相关的命令
+ quit：关闭连接（connection）
+ auth：简单密码认证

### 4. 持久化
+ save：将数据同步保存到磁盘
+ bgsave：将数据异步保存到磁盘
+ lastsave：返回上次成功将数据保存到磁盘的Unix时戳
+ shundown：将数据同步保存到磁盘，然后关闭服务


redis 提供两种方式进行持久化，一种是RDB持久化（原理是将Reids在内存中的数据库记录定时dump到磁盘上的RDB持久化），另外一种是AOF持久化（原理是将Reids的操作日志以追加的方式写入文件）。

#### 4.1 RDB
RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘，实际操作过程是fork一个子进程，先将数据集写入临时文件，写入成功后，再替换之前的文件，用二进制压缩存储。

Redis会将数据集的快照dump到dump.rdb文件中。此外，我们也可以通过配置文件来修改Redis服务器dump快照的频率：
```
save 900 1    #在900秒(15分钟)之后，如果至少有1个key发生变化，则dump内存快照。

save 300 10   #在300秒(5分钟)之后，如果至少有10个key发生变化，则dump内存快照。

save 60 10000 #在60秒(1分钟)之后，如果至少有10000个key发生变化，则dump内存快照。
```
优势：
+ （1）整个Redis数据库将只包含一个文件，易于文件备份与恢复。

+ （2）性能最大化。持久化只需fork出子进程，再由子进程完成持久化工作，可以极大的避免服务进程执行IO操作了。

+ （3）相比于AOF机制，如果数据集很大，RDB的启动效率会更高。

劣势：

+ （1）无法保证数据的高可用性（即最大限度的避免数据丢失）。系统一旦在定时持久化之前出现宕机现象，此前没有来得及写入磁盘的数据都将丢失。

+ （2）由于RDB是通过fork子进程来协助完成数据持久化工作的，因此，如果当数据集较大时，可能会导致整个服务器停止服务几百毫秒，甚至是1秒钟。

#### 4.2 AOF
AOF持久化以日志的形式记录服务器所处理的每一个写、删除操作，查询操作不会记录，以文本的方式记录，可以打开文件看到详细的操作记录。

优势：
+ （1）更高的数据安全性（即数据持久性）。在Redis的配置文件中存在三种同步方式，它们分别是：

```
appendfsync always     #每次有数据修改发生时都会写入AOF文件。

appendfsync everysec   #每秒钟同步一次，该策略为AOF的缺省策略。

appendfsync no         #从不同步。高效但是数据不会被持久化。
```

+ （2）采用的是 append 模式，不会破坏日志文件中已经存在的内容。如果只是写入了一半数据就出现了系统崩溃问题，可以通过redis-check-aof工具来解决数据一致性的问题。

+ （3）如果日志过大，Redis可以自动启用rewrite机制。即Redis以append模式不断的将修改数据写入到老的磁盘文件中，同时Redis还会创建一个新的文件用于记录此期间有哪些修改命令被执行。因此在进行rewrite切换时可以更好的保证数据安全性。

+ （4）AOF包含一个格式清晰、易于理解的日志文件用于记录所有的修改操作。事实上，我们也可以通过该文件完成数据的重建。

劣势：

+ （1）对于相同数量的数据集而言，AOF文件通常要大于RDB文件。RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

+ （2）根据同步策略的不同，AOF在运行效率上往往会慢于RDB。总之，每秒同步策略的效率是比较高的，同步禁用策略的效率和RDB一样高效。

#### 4.3 RDB V.S. AOF
二者选择的标准，就是看系统是愿意牺牲一些性能，换取更高的缓存一致性（AOF），还是愿意写操作频繁的时候，不启用备份来换取更高的性能，待手动运行save的时候，再做备份（RDB）。RDB 这个就更有些 eventually consistent 的意思了。

### 5. 恢复
如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令：
```
config get dir
```
### 6. 远程服务控制
+ info：提供服务器的信息和统计
+ monitor：实时转储收到的请求
+ slaveof：改变复制策略设置
+ config：在运行时配置Redis服务器

### 7. Redis Cluster configuration parameters
We are about to create an example cluster deployment. Before we continue, let's introduce the configuration parameters that Redis Cluster introduces in the `redis.conf` file. Some will be obvious, others will be more clear as you continue reading.

+ `cluster-enabled <yes/no>`: If yes enables Redis Cluster support in a specific Redis instance. Otherwise the instance starts as a stand alone instance as usually.

打开才是集群模式，否则是独立应用。
```
814	# cluster-enabled yes
```

+ `cluster-config-file <filename>`: Note that despite the name of this option, this is not an user editable configuration file, but the file where a Redis Cluster node automatically persists the cluster configuration (the state, basically) every time there is a change, in order to be able to re-read it at startup. The file lists things like the other nodes in the cluster, their state, persistent variables, and so forth. Often this file is rewritten and flushed on disk as a result of some message reception.

配置文件用来写入集群信息，用户不可编辑。
```
822	# cluster-config-file nodes-6379.conf
```
+ `cluster-node-timeout <milliseconds>`: The maximum amount of time a Redis Cluster node can be unavailable, without it being considered as failing. If a master node is not reachable for more than the specified amount of time, it will be failed over by its slaves. This parameter controls other important things in Redis Cluster. Notably, every node that can't reach the majority of master nodes for the specified amount of time, will stop accepting queries.

集群节点超时时间。
```
828	# cluster-node-timeout 15000
```
+ `cluster-slave-validity-factor <factor>`: If set to zero, a slave will always try to failover a master, regardless of the amount of time the link between the master and the slave remained disconnected. If the value is positive, a maximum disconnection time is calculated as the node timeout value multiplied by the factor provided with this option, and if the node is a slave, it will not try to start a failover if the master link was disconnected for more than the specified amount of time. For example if the node timeout is set to 5 seconds, and the validity factor is set to 10, a slave disconnected from the master for more than 50 seconds will not try to failover its master. Note that any value different than zero may result in Redis Cluster to be unavailable after a master failure if there is no slave able to failover it. In that case the cluster will return back available only when the original master rejoins the cluster.

失败恢复尝试次数。
```
873	# cluster-slave-validity-factor 10
```

+ `cluster-migration-barrier <count>`: Minimum number of slaves a master will remain connected with, for another slave to migrate to a master which is no longer covered by any slave. See the appropriate section about replica migration in this tutorial for more information.

集群节点迁移阈值，低于阈值则slave迁移。
```
 892	# cluster-migration-barrier 1
```

+ `cluster-require-full-coverage <yes/no>`: If this is set to yes, as it is by default, the cluster stops accepting writes if some percentage of the key space is not covered by any node. If the option is set to no, the cluster will still serve queries even if only requests about a subset of keys can be processed.
集群是否需要全覆盖才提供服务。
```
905	# cluster-require-full-coverage yes
```
### 8. Manual failover
Sometimes it is useful to force a failover without actually causing any problem on a master. For example in order to upgrade the Redis process of one of the master nodes it is a good idea to failover it in order to turn it into a slave with minimal impact on availability.

Manual failovers are supported by Redis Cluster using the `CLUSTER FAILOVER` command, that `must be executed in one of the slaves of the master you want to failover`.

### 9. Adding a new node
Adding a new node is basically the process of adding an empty node and then moving some data into it, in case it is a new `master`, or telling it to setup as a replica of a known node, in case it is a `slave`.

We'll show both, starting with the addition of a new master instance.

In both cases the first step to perform is `adding an empty node`.

通过指定配置文件启动一个新节点，注意 `redis.conf` 与集群中的节点类似。

Now we can use `redis-trib` as usually in order to add the node to the existing cluster.
```
./redis-trib.rb add-node 新节点的<IP:PORT> 集群中节点的<IP:PORT>
```
Now we can connect to the new node to see if it really joined the cluster:
```
> cluster nodes
```
Now it is possible to assign hash slots to this node using the resharding feature of redis-trib. It is basically useless to show this as we already did in a previous section, there is no difference, it is just a resharding having as a target the empty node.

### 10. Adding a new node as a replica

Adding a new Replica can be performed in two ways. The obvious one is to use redis-trib again, but with the `--slave` option, like this:

```
./redis-trib.rb add-node --slave 新节点的<IP:PORT> 集群中节点的<IP:PORT>
```
Note that the command line here is exactly like the one we used to add a new master, so we are not specifying to which master we want to add the replica. In this case what happens is that redis-trib will add the new node as replica of a random master among the masters with less replicas.

However you can specify exactly what master you want to target with your new replica with the following command line:
```
./redis-trib.rb add-node --slave --master-id master的ID 新节点的<IP:PORT> 集群中节点的<IP:PORT>
```
This way we assign the new replica to a specific master.

A more manual way to add a replica to a specific master is to add the new node as an empty master, and then turn it into a replica using the `CLUSTER REPLICATE` command. This also works if the node was added as a slave but you want to move it as a replica of a different master.

For example in order to add a replica for the node 127.0.0.1:7005 that is currently serving hash slots in the range 11423-16383, that has a Node ID 3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e, all I need to do is to connect with the new node (already added as empty master) and send the command:

```
redis 127.0.0.1:7006> cluster replicate 3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e
```

### 11. Removing a node
To remove a slave node just use the del-node command of redis-trib:
```
./redis-trib del-node 集群中节点的<IP:PORT> 需要移除的`<node-id>`
```
The first argument is just a random node in the cluster, the second argument is the ID of the node you want to remove.

You can remove a master node in the same way as well, `however in order to remove a master node it must be empty`. If the master is not empty you need to reshard data away from it to all the other master nodes before.

An alternative to remove a master node is to perform a manual failover of it over one of its slaves and remove the node after it turned into a slave of the new master. Obviously this does not help when you want to reduce the actual number of masters in your cluster, in that case, a resharding is needed.

### 12. Replicas migration
In Redis Cluster it is possible to reconfigure a slave to replicate with a different master at any time just using the following command:
```
CLUSTER REPLICATE <master-node-id>
```

### 13. Resharding the cluster
Resharding basically means to move hash slots from a set of nodes to another set of nodes, and like cluster creation it is accomplished using the redis-trib utility.

To start a resharding just type:
```
./redis-trib.rb reshard <IP:PORT>
```
You only need to specify a single node, redis-trib will find the other nodes automatically.

After the final confirmation you'll see a message for every slot that redis-trib is going to move from a node to another, and a dot will be printed for every actual key moved from one side to the other.

While the resharding is in progress you should be able to see your example program running unaffected. You can stop and restart it multiple times during the resharding if you want.

At the end of the resharding, you can test the health of the cluster with the following command:
```
./redis-trib.rb check <IP:PORT>
```
Reshardings can be performed automatically without the need to manually enter the parameters in an interactive way. This is possible using a command line like the following:
```
./redis-trib.rb reshard --from <node-id> --to <node-id> --slots <number of slots> --yes <host>:<port>
```
This allows to build some automatism if you are likely to reshard often, however currently there is no way for redis-trib to automatically rebalance the cluster checking the distribution of keys across the cluster nodes and intelligently moving slots as needed. This feature will be added in the future.

### 14. Creating the cluster
Note that the minimal cluster that works as expected requires to contain `at least three master nodes`. For your first tests it is strongly suggested to start a six nodes cluster with three masters and three slaves.


Now that we have a number of instances running, we need to create our cluster by writing some meaningful configuration to the nodes.

This is very easy to accomplish as we are helped by the Redis Cluster command line utility called `redis-trib`, a Ruby program executing special commands on instances in order to create new clusters, check or reshard an existing cluster, and so forth.

The redis-trib utility is in the src directory of the Redis source code distribution. You need to install redis gem to be able to run redis-trib.
```
gem install redis
```
To create your cluster simply type:
```
./redis-trib.rb create --replicas 1 <IP:PORT> <IP:PORT> <IP:PORT> （共6个实例）
```

The command used here is create, since we want to create a new cluster. The option --replicas 1 means that we want a slave for every master created. The other arguments are the list of addresses of the instances I want to use to create the new cluster.

Obviously the only setup with our requirements is to create a cluster with 3 masters and 3 slaves.

### 扩展阅读
#### 1. redis持久化方法对比分析
https://www.cnblogs.com/Fairy-02-11/p/6182478.html

#### 2. Redis：默认配置文件redis.conf详解
https://www.cnblogs.com/zxtceq/p/7676911.html

#### 3. Redis 命令参考
http://doc.redisfans.com/index.html

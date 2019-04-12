### 0 前期准备
下载 RabbitMQ, Erlang

RabbitMQ 版本：
https://www.rabbitmq.com/releases/rabbitmq-server

Erlang   版本：
http://www.erlang.org/downloads

### 1 JDK 环境变量设置
```shell
编辑配置文件：

# vi /etc/profile

追加内容：

export JAVA_HOME=/opt/jdk1.8.0_131
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

使配置生效：

# source /etc/profile
```

### 2 Erlang 安装步骤
```shell
1. 解压源码

# tar -xvf otp_src_20.0.tar.gz

2. 进入目录

# cd otp_src_20.0

3. 设置环境

# export ERL_TOP=`pwd`    # Assuming bash/sh

4. 校验依赖

# ./configure --prefix=/opt/erlang/20.0 --without-javac

5. 缺少依赖

# yum list | grep XX    

6. 安装依赖    

#　yum install XX

#　yum install ncurses-devel unixODBC unixODBC-devel

7. 指定编译

# export LANG=C   # Assuming bash/sh

8. 编译

# make

# make install

9. 建立软连接

# ln -s /app/otp_src_20.0/bin/erl /usr/bin/erl
```

### Erlang 安装极简版
```
（1）安装好 yum 源
（2）yum list | grep erlang
（3）yum install erlang的版本
（4）输入 erl 验证
```

### 3 搭建 RabbitMQ 集群
```
这里搭建3台机器的RabbitMQ集群。

0. 命名每个集群机器的hostname

机器1：# hostname HOST1
机器2：# hostname HOST2
机器3：# hostname HOST3

1. 修改所有集群机器的 /etc/hosts
追加以下内容：

IP1  HOST1
IP2  HOST2
IP3  HOST3

2. 保持erlang.cookie一致
先启动一台机器的 RabbitMQ 服务，Erlang 会生成 cookie 文件：.erlang.cookie ；
文件路径一般在 home 路径；
将 .erlang.cookie 复制到其他集群机器的相应位置，并修改权限：

# chmod 600 ~/.erlang.cookie

注意：RabbitMQ 的集群是依赖于 Erlang 的集群来工作的，所以必须先构建起 Erlang 的集群环境。Erlang 的集群中各节点是通过一个 magic cookie 来实现的，这个cookie 就是上文的 .erlang.cookie，文件路径一般在 home 路径。所以必须保证各节点 cookie 保持一致，否则节点之间就无法通信。这里的保持一致指的是 .erlang.cookie 的权限（不一定是600，还有可能是400）和属主属组。例如：

.erlang.cookie
-r--------.  1 root root    20 8月   4 2016 .erlang.cookie

3. 执行以下命令，加入 RabbitMQ 集群

3.0 开启控制台

# ./rabbitmq-plugins enable rabbitmq_management

3.1 启动服务

# nohup ./rabbitmq-server start > rabbitmq.log &

注意：如果增加节点停止节点后再次启动遇到无法启动可以使用detached参数独立运行，这步很关键。

# ./rabbitmq-server -detached

3.2 停止服务应用

# ./rabbitmqctl stop_app

3.3 加入集群

# ./rabbitmqctl join_cluster rabbit@HOST1

3.4 启动服务应用

# ./rabbitmqctl start_app

3.5 查看集群状态

# ./rabbitmqctl cluster_status

4. 添加用户

# ./rabbitmqctl add_user USER PASSWORD

# ./rabbitmqctl set_user_tags USER administrator

# ./rabbitmqctl set_permissions -p / USER ".*" ".*" ".*"

```

### 4 RabbitMQ 启停命令
```
零，开启控制台

# ./rabbitmq-plugins enable rabbitmq_management

一，启动服务

# nohup ./rabbitmq-server start > rabbitmq.log &

二，停止服务应用

# ./rabbitmqctl stop_app

三，启动服务应用

# ./rabbitmqctl start_app

四，停止应用

# ./rabbitmqctl stop
```

### 5 使用 guest 登录
```
Rabbitmq 默认允许 guest 本地访问。为了方便需要允许远程访问。

把 rabbitmq.config.example 复制一份到 /etc/rabbitmq/rabbitmq.config，在 rabbit 段中，按如下格式

[{rabbit, [{loopback_users, []}]}]

（1）找到 rabbitmq.config 文件（find / -name rabbitmq.config ）
（2）打开 rabbitmq.config 文件，去掉这一行注释即可（{loopback_users, []},）
（3）rabbitmq-server start 重启，OK
（4）开启控制台

 # ./rabbitmq-plugins enable rabbitmq_management

```

### 6 rabbitmqctl 命令使用
```
rabbitmqctl list_queues：查看所有队列信息

rabbitmqctl stop_app：关闭应用（关闭当前启动的节点）

rabbitmqctl start_app：启动应用，和上述关闭命令配合使用，达到清空队列的目的

rabbitmqctl reset：从管理数据库中移除所有数据，例如配置过的用户和虚拟宿主, 删除所有持久化的消息（这个命令要在rabbitmqctl stop_app之后使用）

rabbitmqctl force_reset：作用和rabbitmqctl reset一样，区别是无条件重置节点，不管当前管理数据库状态以及集群的配置。如果数据库或者集群配置发生错误才使用这个最后的手段

rabbitmqctl status：节点状态

rabbitmqctl add_user username password：添加用户

rabbitmqctl list_users：列出所有用户

rabbitmqctl list_user_permissions username：列出用户权限

rabbitmqctl change_password username newpassword：修改密码

rabbitmqctl add_vhost vhostpath：创建虚拟主机

rabbitmqctl list_vhosts：列出所有虚拟主机

rabbitmqctl set_permissions -p vhostpath username ".*" ".*" ".*"：设置用户权限

rabbitmqctl list_permissions -p vhostpath：列出虚拟主机上的所有权限

rabbitmqctl clear_permissions -p vhostpath username：清除用户权限

rabbitmqctl -p vhostpath purge_queue ：清除队列里的消息

rabbitmqctl delete_user username：删除用户

rabbitmqctl delete_vhost vhostpath：删除虚拟主机
```

### 7 策略设置
```
Name:         4_mirror_dlx
Pattern:      .*
Apply to:     Exchanges and queues(all)
Definition:
              ha-mode:	               exactly
              ha-params:	             4
              ha-sync-mode:	          automatic
              dead-letter-exchange:	  deadExchange
Priority:     数值越大，优先级越高        
```

设置死信队列的目的是，当消息过期（expired）或被拒绝（rejected）之后，消息会被路由到死信队列，而不是消失。死信队列中的消息，可以看到消息的来源和“死亡”的理由，通过补偿程序，消息还能被恢复。

```
（1）设置 Exchange（死信交换机）
Virtual host:             /
Name:                     deadExchange
Type:                     fanout
Durability:               Durable
Auto delete:              No
Internal:                 No
Alternate exchange:       默认为空就行
Arguments:                默认为空就行
（2）设置 Queue（死信队列）
Virtual host:             /
Name:                     SKYTRAIN_DEAD_LETTER
Durability:               Durable
Node：                    集群中选一台 DISC 型的节点
Auto delete:              No
Message TTL:              默认为空就行
Auto expire:              默认为空就行
Max length:               默认为空就行
Dead letter exchange:     默认为空就行
Dead letter routing key:  默认为空就行
Arguments:                默认为空就行
（3）绑定 Exchanges 和 Queue
From exchange:            deadExchange
Routing key:              #
Arguments:                默认为空就行   
```

### 8 只读用户配置
```
0. 建立用户，给的标签是：
[Management]

1. 赋权部分全部置为空：

Set permission
Virtual Host:/
Configure regexp:
Write regexp:
Read regexp:

2. 效果如下：
Current permissions
Virtual host	Configure regexp	Write regexp	Read regexp
/

该用户就是只读用户了。

```

### 9 剔除 RabbitMQ 集群中无效的节点
```

# ./rabbitmqctl forget_cluster_node NODENAME

对于状态异常的节点，先删除

/var/lib/rabbitmq/mnesia

文件，再做操作（慎用，删除就没有元信息了）

```
### 10 集群官方文档
https://www.rabbitmq.com/clustering.html

### Clustering Guide
#### Overview
A RabbitMQ broker is a logical grouping of one or several Erlang nodes, each running the RabbitMQ application and sharing users, virtual hosts, queues, exchanges, bindings, and runtime parameters. Sometimes we refer to the collection of nodes as a cluster.

#### Port Access
5672, 5671: used by AMQP 0-9-1 and 1.0 clients without and with TLS

15672: HTTP API clients, management UI and rabbitmqadmin (only if the management plugin is enabled)

***5672：客户端连接端口；15672：服务端管理端口***
#### What is Replicated?
All data/state required for the operation of a RabbitMQ broker is replicated across all nodes. An exception to this are message queues, which by default reside on one node, though they are visible and reachable from all nodes. To replicate queues across nodes in a cluster, see the documentation on high availability (note that you will need a working cluster first).

#### Hostname Resolution Requirements
RabbitMQ nodes address each other using domain names, either short or fully-qualified (FQDNs). Therefore hostnames of all cluster members must be resolvable from all cluster nodes, as well as machines on which command line tools such as rabbitmqctl might be used.

Hostname resolution can use any of the standard OS-provided methods:

DNS records

Local host files (e.g. /etc/hosts)

In more restrictive environments, where DNS record or hosts file modification is restricted, impossible or undesired, Erlang VM can be configured to use alternative hostname resolution methods, such as an alternative DNS server, a local file, a non-standard hosts file location, or a mix of methods. Those methods can work in concert with the standard OS hostname resolution methods.
To use FQDNs, see RABBITMQ_USE_LONGNAME in the Configuration guide.

***这里使用本地HOSTS来建立集群。***

#### Cluster Formation
A RabbitMQ cluster can formed in a number of ways:

+ Declaratively by listing cluster nodes in config file
+ Declaratively using DNS-based discovery
+ Declaratively using AWS (EC2) instance discovery (via a plugin)
+ Declaratively using Kubernetes discovery (via a plugin)
+ Declaratively using Consul-based discovery (via a plugin)
+ Declaratively using etcd-based discovery (via a plugin)
+ Manually with rabbitmqctl

Please refer to the Cluster Formation guide for details.

The composition of a cluster can be altered dynamically. All RabbitMQ brokers start out as running on a single node. These nodes can be joined into clusters, and subsequently turned back into individual brokers again.

***集群可动态扩展，RabbitMQ节点可自由的加入/离开集群。***

#### Node Names (Identifiers)
RabbitMQ nodes are identified by node names. A node name consists of two parts, a prefix (usually `rabbit`) and hostname. For example, `rabbit@node1.messaging.svc.local` is a node name with the prefix of `rabbit` and hostname of `node1.messaging.svc.local`.

Node names in a cluster must be unique. If more than one node is running on a given host (this is usually the case in development and QA environments), they must use different prefixes, e.g. `rabbit1@hostname` and `rabbit2@hostname`.

In a cluster, nodes identify and contact each other using node names. This means that the hostname part of every node name must resolve. CLI tools also identify and address nodes using node names.

When a node starts up, it checks whether it has been assigned a node name. This is done via the `RABBITMQ_NODENAME` environment variable. If no value was explicitly configured, the node resolves its hostname and prepends `rabbit` to it to compute its node name.

If a system uses fully qualified domain names (FQDNs) for hostnames, RabbitMQ nodes and CLI tools must be configured to use so called long node names. For server nodes this is done by setting the `RABBITMQ_USE_LONGNAME` environment variable to `true`. For CLI tools, either `RABBITMQ_USE_LONGNAME` must be set or the `--longnames` option must be specified.

***集群中节点的名称需要唯一，一般都叫做 rabbit@hostname，这个名称在集群通信及用户操作中都会用到***

#### Failure Handling
RabbitMQ brokers tolerate the failure of individual nodes. Nodes can be started and stopped at will, as long as they can contact a cluster member node known at the time of shutdown.

RabbitMQ clustering has several modes of dealing with network partitions, primarily consistency oriented. Clustering is meant to be used across LAN. It is not recommended to run clusters that span WAN. The Shovel or Federation plugins are better solutions for connecting brokers across a WAN. Note that Shovel and Federation are not equivalent to clustering.

***RabbitMQ集群具有分区容错性（CAP的P），建议使用有线而不是无线来进行集群通讯。***

#### Disk and RAM Nodes
A node can be a disk node or a RAM node. (Note: disk and disc are used interchangeably). RAM nodes store internal database tables in RAM. This does not include messages, message store indices, queue indices and other node state.

In well over 90% of cases you want all your nodes to be disk nodes; RAM nodes are a special case that can be used to improve the performance clusters with high queue, exchange, or binding churn. RAM nodes do not provide meaningfully higher message rates. When in doubt, use disk nodes only.

Since RAM nodes store internal database tables in RAM only, they must sync them from a peer node on startup. This means that a cluster must contain at least one disk node. It is therefore not possible to manually remove the last remaining disk node in a cluster.

***RAM模式只保存内部数据库表，不存消息，索引等信息。如果有疑虑，直接使用磁盘模式。集群中一定要有一个磁盘节点（一定要有一个巫妖王）。***

#### Clustering Transcript with rabbitmqctl
The following is a transcript of setting up and manipulating a RabbitMQ cluster across three machines - rabbit1, rabbit2, rabbit3.

***这里将搭建一个3节点集群： rabbit1, rabbit2, rabbit3***

We assume that the user is logged into all three machines, that RabbitMQ has been installed on the machines, and that the rabbitmq-server and rabbitmqctl scripts are in the user's PATH.

This transcript can be modified to run on a single host, as explained more details below.

#### How Nodes (and CLI tools) Authenticate to Each Other: the Erlang Cookie
RabbitMQ nodes and CLI tools (e.g. rabbitmqctl) use a cookie to determine whether they are allowed to communicate with each other. For two nodes to be able to communicate they must have the same shared secret called the Erlang cookie. The cookie is just a string of alphanumeric characters up to 255 characters in size. It is usually stored in a local file. The file must be only accessible to the owner (e.g. have UNIX permissions of 600 or similar). Every cluster node must have the same cookie.

***.erlang.cookie 中存储识别字符，最多255个。在linux中需 chmod 600 .erlang.cookie***

If the file does not exist, Erlang VM will automatically create one with a randomly generated value when the RabbitMQ server starts up. Erlang cookie management is best done using automation tools such as Chef, BOSH, Docker or similar.

On UNIX systems, the cookie will be typically located in /var/lib/rabbitmq/.erlang.cookie (used by the server) and $HOME/.erlang.cookie (used by CLI tools). Note that since the value of `$HOME` varies from user to user, it's necessary to place a copy of the cookie file for each user that will be using the CLI tools. This applies to both non-privileged users and `root`.

***.erlang.cookie 一般在 HOME 路径。***

On Windows, the cookie file location varies depending on Erlang version used and whether the HOMEDRIVE or HOMEPATH environment variables are set.

When the cookie is misconfigured (for example, not identical), RabbitMQ will log errors such as **"Connection attempt from disallowed node" and "Could not auto-cluster".** When a CLI tool such as rabbitmqctl fails to authenticate with RabbitMQ, the message usually says

***要是 .erlang.cookie 不符，可能报以下错误：***

```
* epmd reports node 'rabbit' running on port 25672
* TCP connection succeeded but Erlang distribution failed
* suggestion: hostname mismatch?
* suggestion: is the cookie set correctly?
* suggestion: is the Erlang distribution using TLS?
```

An incorrectly placed cookie file or cookie value mismatch are most common scenarios for such failures. When a recent Erlang/OTP version is used, authentication failures contain more information and cookie mismatches can be identified better:
```
* connected to epmd (port 4369) on warp10
* epmd reports node 'rabbit' running on port 25672
* TCP connection succeeded but Erlang distribution failed

* Authentication failed (rejected by the remote node), please check the Erlang cookie
```

***集群中节点靠 .erlang.cookie 来识别自己人。若是口号不对，就报错给你看。***

#### Starting independent nodes
Clusters are set up by re-configuring existing RabbitMQ nodes into a cluster configuration. Hence the first step is to start RabbitMQ on all nodes in the normal way:
```
# on rabbit1
rabbitmq-server -detached
# on rabbit2
rabbitmq-server -detached
# on rabbit3
rabbitmq-server -detached
```
***通过使用 -detached 参数启动***

This creates three independent RabbitMQ brokers, one on each node, as confirmed by the cluster_status command:
```
# on rabbit1
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit1 ...
# => [{nodes,[{disc,[rabbit@rabbit1]}]},{running_nodes,[rabbit@rabbit1]}]
# => ...done.

# on rabbit2
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit2 ...
# => [{nodes,[{disc,[rabbit@rabbit2]}]},{running_nodes,[rabbit@rabbit2]}]
# => ...done.

# on rabbit3
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit3 ...
# => [{nodes,[{disc,[rabbit@rabbit3]}]},{running_nodes,[rabbit@rabbit3]}]
# => ...done.
```
***查看集群状态，发现启动了 3 个单节点***

The node name of a RabbitMQ broker started from the rabbitmq-server shell script is rabbit@shorthostname, where the short node name is lower-case (as in rabbit@rabbit1, above). On Windows, if rabbitmq-server.bat batch file is used, the short node name is upper-case (as in rabbit@RABBIT1). When you type node names, case matters, and these strings must match exactly.

#### Creating the cluster
In order to link up our three nodes in a cluster, we tell two of the nodes, say rabbit@rabbit2 and rabbit@rabbit3, to join the cluster of the third, say rabbit@rabbit1. Prior to that both newly joining members must be reset.

We first join rabbit@rabbit2 in a cluster with rabbit@rabbit1. To do that, on rabbit@rabbit2 we stop the RabbitMQ application and join the rabbit@rabbit1 cluster, then restart the RabbitMQ application. Note that a node must be reset before it can join an existing cluster. Resetting the node removes all resources and data that were previously present on that node. This means that a node cannot be made a member of a cluster and keep its existing data at the same time. When that's desired, using the Blue/Green deployment strategy or backup and restore are the available options.
```
# on rabbit2
rabbitmqctl stop_app
# => Stopping node rabbit@rabbit2 ...done.

rabbitmqctl reset
# => Resetting node rabbit@rabbit2 ...

rabbitmqctl join_cluster rabbit@rabbit1
# => Clustering node rabbit@rabbit2 with [rabbit@rabbit1] ...done.

rabbitmqctl start_app
# => Starting node rabbit@rabbit2 ...done.
```
***节点必须 reset 重置之后才能加入集群，重置意味着节点的个人信息全部清除了（洗白之后才能上岸啊）***

***因为集群中的节点不能保留自己的数据，所以最好使用蓝绿部署或备份恢复策略来留存数据***

We can see that the two nodes are joined in a cluster by running the cluster_status command on either of the nodes:
```
# on rabbit1
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit1 ...
# => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
# =>  {running_nodes,[rabbit@rabbit2,rabbit@rabbit1]}]
# => ...done.

# on rabbit2
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit2 ...
# => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
# =>  {running_nodes,[rabbit@rabbit1,rabbit@rabbit2]}]
# => ...done.
```
***发现 rabbit2 已加入 rabbit1，组成集群***

Now we join rabbit@rabbit3 to the same cluster. The steps are identical to the ones above, except this time we'll cluster to rabbit2 to demonstrate that the node chosen to cluster to does not matter - it is enough to provide one online node and the node will be clustered to the cluster that the specified node belongs to
```
# on rabbit3
rabbitmqctl stop_app
# => Stopping node rabbit@rabbit3 ...done.

# on rabbit3
rabbitmqctl reset
# => Resetting node rabbit@rabbit3 ...

rabbitmqctl join_cluster rabbit@rabbit2
# => Clustering node rabbit@rabbit3 with rabbit@rabbit2 ...done.

rabbitmqctl start_app
# => Starting node rabbit@rabbit3 ...done.
```
***其他节点加入集群，join_cluster 任意一个集群节点均可***

We can see that the three nodes are joined in a cluster by running the cluster_status command on any of the nodes:
```
# on rabbit1
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit1 ...
# => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
# =>  {running_nodes,[rabbit@rabbit3,rabbit@rabbit2,rabbit@rabbit1]}]
# => ...done.

# on rabbit2
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit2 ...
# => [{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
# =>  {running_nodes,[rabbit@rabbit3,rabbit@rabbit1,rabbit@rabbit2]}]
# => ...done.

# on rabbit3
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit3 ...
# => [{nodes,[{disc,[rabbit@rabbit3,rabbit@rabbit2,rabbit@rabbit1]}]},
# =>  {running_nodes,[rabbit@rabbit2,rabbit@rabbit1,rabbit@rabbit3]}]
# => ...done.
```
By following the above steps we can add new nodes to the cluster at any time, while the cluster is running.

***按照以上步骤，节点可以随意地加入集群（注意到集群一直是运行状态）***

#### Restarting cluster nodes
Nodes that have been joined to a cluster can be stopped at any time. It is also ok for them to crash. In both cases the rest of the cluster continues operating, and the nodes automatically "catch up" with (sync from) the other cluster nodes when they start up again.

We shut down the nodes rabbit@rabbit1 and rabbit@rabbit3 and check on the cluster status at each step:

```
rabbit1$ rabbitmqctl stop
Stopping and halting node rabbit@rabbit1 ...done.
rabbit2$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit3,rabbit@rabbit2]}]
...done.
rabbit3$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit3 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit3]}]
...done.
rabbit3$ rabbitmqctl stop
Stopping and halting node rabbit@rabbit3 ...done.
rabbit2$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit2]}]
...done.
```

***发现集群中的节点 stop 之后，检查集群状态（rabbitmqctl cluster_status），会显示***

***集群信息（{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]}）***

***和存活节点（{running_nodes,[rabbit@rabbit3,rabbit@rabbit2]}）***

Now we start the nodes again, checking on the cluster status as we go along:
```
rabbit1$ rabbitmq-server -detached
rabbit1$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit1]}]
...done.
rabbit2$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit1,rabbit@rabbit2]}]
...done.
rabbit3$ rabbitmq-server -detached
rabbit1$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit1,rabbit@rabbit3]}]
...done.
rabbit2$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]
...done.
rabbit3$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit3 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2,rabbit@rabbit3]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit1,rabbit@rabbit3]}]
...done.
```

***重启节点之后，集群中存活节点信息更新。***

There are some important caveats:

When the entire cluster is brought down, the last node to go down must be the first node to be brought online. If this doesn't happen, the nodes will wait 30 seconds for the last disc node to come back online, and fail afterwards.

If the last node to go offline cannot be brought back up, it can be removed from the cluster using the forget_cluster_node command - consult the rabbitmqctl manpage for more information.

***集群中最后停止的节点要最先起来。***

***否则，启动的节点会等30S（等待最后那个磁盘节点启动），最终失败。***

***如果最后停止的节点起不来，可以使用 forget_cluster_node 把它剔除。***

If all cluster nodes stop in a simultaneous and uncontrolled manner (for example with a power cut) you can be left with a situation in which all nodes think that some other node stopped after them. In this case you can use the force_boot command on one node to make it bootable again - consult the rabbitmqctl manpage for more information.

***如果所有的集群节点同时以不受控制的方式停止(例如，断电)，现在的情形可视为：所有节点都认为其他节点是在它们之后停止的。***

***在这种情况下，可以在一个节点上使用 force_boot 命令来重新启动它。***

#### Breaking Up a Cluster
Sometimes it is necessary to remove a node from a cluster. The operator has to do this explicitly using a rabbitmqctl command.

Some peer discovery mechanisms support node health checks and forced removal of nodes not known to the discovery backend. That feature is opt-in (disabled by default).

We first remove rabbit@rabbit3 from the cluster, returning it to independent operation. To do that, on rabbit@rabbit3 we stop the RabbitMQ application, reset the node, and restart the RabbitMQ application.

***这里 rabbit@rabbit3 主动从集群中离开（主动离开）***
```
rabbit3$ rabbitmqctl stop_app
Stopping node rabbit@rabbit3 ...done.
rabbit3$ rabbitmqctl reset
Resetting node rabbit@rabbit3 ...done.
rabbit3$ rabbitmqctl start_app
Starting node rabbit@rabbit3 ...done.
```

Note that it would have been equally valid to list rabbit@rabbit3 as a node.

Running the cluster_status command on the nodes confirms that rabbit@rabbit3 now is no longer part of the cluster and operates independently:

***可以发现 rabbit@rabbit3 现在是一个单点集群。***
```
rabbit1$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
 {running_nodes,[rabbit@rabbit2,rabbit@rabbit1]}]
...done.
rabbit2$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit1,rabbit@rabbit2]}]},
 {running_nodes,[rabbit@rabbit1,rabbit@rabbit2]}]
...done.
rabbit3$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit3 ...
[{nodes,[{disc,[rabbit@rabbit3]}]},{running_nodes,[rabbit@rabbit3]}]
...done.
```

We can also remove nodes remotely. This is useful, for example, when having to deal with an unresponsive node. We can for example remove rabbit@rabbi1 from rabbit@rabbit2.

***这里集群将 rabbit@rabbit1 剔除（被动离开）***
```
rabbit1$ rabbitmqctl stop_app
Stopping node rabbit@rabbit1 ...done.
rabbit2$ rabbitmqctl forget_cluster_node rabbit@rabbit1
Removing node rabbit@rabbit1 from cluster ...
...done.
```

Note that rabbit1 still thinks its clustered with rabbit2, and trying to start it will result in an error. We will need to reset it to be able to start it again.

***被动离开的 rabbit@rabbit1 启动时，发现被集群拒绝（你认为自己是集群的一份子，但集群可不这样认为）***

***reset 之后，可以自己起一个单点集群***


```
rabbit1$ rabbitmqctl start_app
Starting node rabbit@rabbit1 ...
Error: inconsistent_cluster: Node rabbit@rabbit1 thinks it's clustered with node rabbit@rabbit2, but rabbit@rabbit2 disagrees
rabbit1$ rabbitmqctl reset
Resetting node rabbit@rabbit1 ...done.
rabbit1$ rabbitmqctl start_app
Starting node rabbit@mcnulty ...
...done.
```

The cluster_status command now shows all three nodes operating as independent RabbitMQ brokers:

***现在的情形是，集群一分为三：3个单点集群***
```
rabbit1$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit1 ...
[{nodes,[{disc,[rabbit@rabbit1]}]},{running_nodes,[rabbit@rabbit1]}]
...done.
rabbit2$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit2 ...
[{nodes,[{disc,[rabbit@rabbit2]}]},{running_nodes,[rabbit@rabbit2]}]
...done.
rabbit3$ rabbitmqctl cluster_status
Cluster status of node rabbit@rabbit3 ...
[{nodes,[{disc,[rabbit@rabbit3]}]},{running_nodes,[rabbit@rabbit3]}]
...done.
```

Note that rabbit@rabbit2 retains the residual state of the cluster, whereas rabbit@rabbit1 and rabbit@rabbit3 are freshly initialised RabbitMQ brokers. If we want to re-initialise rabbit@rabbit2 we follow the same steps as for the other nodes:
```
rabbit2$ rabbitmqctl stop_app
Stopping node rabbit@rabbit2 ...done.
rabbit2$ rabbitmqctl reset
Resetting node rabbit@rabbit2 ...done.
rabbit2$ rabbitmqctl start_app
          Starting node rabbit@rabbit2 ...done.
```

Besides rabbitmqctl forget_cluster_node and the automatic cleanup of unknown nodes by some peer discovery plugins, there are no scenarios in which a RabbitMQ node will permanently remove its peer node from a cluster.

***这里 rabbit@rabbit2 保留了集群的原始信息（因为 rabbit1 和 rabbit3 都 reset 过，重置清空），可以把  rabbit1 和 rabbit3 当做新节点加入 rabbit@rabbit2 集群（如果不行***，**可以改名再战!!!!**）

#### Hostname Changes
RabbitMQ nodes use hostnames to communicate with each other. Therefore, all node names must be able to resolve names of all cluster peers. This is also true for tools such as rabbitmqctl.

In addition to that, by default RabbitMQ names the database directory using the current hostname of the system. If the hostname changes, a new empty database is created. To avoid data loss it's crucial to set up a fixed and resolvable hostname. Whenever the hostname changes RabbitMQ node must be restarted.

A similar effect can be achieved by using rabbit@localhost as the broker nodename. The impact of this solution is that clustering will not work, because the chosen hostname will not resolve to a routable address from remote hosts. The rabbitmqctl command will similarly fail when invoked from a remote host. A more sophisticated solution that does not suffer from this weakness is to use DNS, e.g. Amazon Route 53 if running on EC2. If you want to use the full hostname for your nodename (RabbitMQ defaults to the short name), and that full hostname is resolveable using DNS, you may want to investigate setting the environment variable RABBITMQ_USE_LONGNAME=true.

***RabbitMQ 使用 hostname 来与集群其他节点通信，一旦节点 hostname 改变，节点需要重启（感觉是集群中的所有节点都需要重启）***

#### Erlang Versions Across the Cluster
All nodes in a cluster must run the same minor version of Erlang: 19.3.4 and 19.3.6 can be mixed but 19.0.1 and 19.3.6 (or 17.5 and 19.3.6) cannot. Compatibility between individual Erlang/OTP patch versions can vary between releases but that's generally rare.

***注意，必须保证集群中 Erlang 的版本一致！***

#### Connecting to Clusters from Clients
A client can connect as normal to any node within a cluster. If that node should fail, and the rest of the cluster survives, then the client should notice the closed connection, and should be able to reconnect to some surviving member of the cluster.

Generally, it's not advisable to bake in node hostnames or IP addresses into client applications: this introduces inflexibility and will require client applications to be edited, recompiled and redeployed should the configuration of the cluster change or the number of nodes in the cluster change.

Instead, we recommend a more abstracted approach: this could be a dynamic DNS service which has a very short TTL configuration, or a plain TCP load balancer, or some sort of mobile IP achieved with pacemaker or similar technologies. In general, this aspect of managing the connection to nodes within a cluster is beyond the scope of RabbitMQ itself, and we recommend the use of other technologies designed specifically to solve these problems.

***不建议直接使用IP连接集群，通常采用DNS等第三方方式***

#### Clusters with RAM nodes
RAM nodes keep their metadata only in memory. As RAM nodes don't have to write to disc as much as disc nodes, they can perform better. However, note that since persistent queue data is always stored on disc, the performance improvements will affect only resource management (e.g. adding/removing queues, exchanges, or vhosts), but not publishing or consuming speed.

RAM nodes are an advanced use case; when setting up your first cluster you should simply not use them. You should have enough disc nodes to handle your redundancy requirements, then if necessary add additional RAM nodes for scale.

A cluster containing only RAM nodes is fragile; if the cluster stops you will not be able to start it again and will lose all data. RabbitMQ will prevent the creation of a RAM-node-only cluster in many situations, but it can't absolutely prevent it.

The examples here show a cluster with one disc and one RAM node for simplicity only; such a cluster is a poor design choice

***总的来说， RAM模式很鸡肋，就把元数据存内存而不是磁盘，并不能提升队列的性能。建议就是最好不要使用！***

#### Creating RAM nodes
We can declare a node as a RAM node when it first joins the cluster. We do this with rabbitmqctl join_cluster as before, but passing the --ram flag:
```
# on rabbit2
rabbitmqctl stop_app
# => Stopping node rabbit@rabbit2 ...done.

rabbitmqctl join_cluster --ram rabbit@rabbit1
# => Clustering node rabbit@rabbit2 with [rabbit@rabbit1] ...done.

rabbitmqctl start_app
# => Starting node rabbit@rabbit2 ...done.
```
RAM nodes are shown as such in the cluster status
```
# on rabbit1
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit1 ...
# => [{nodes,[{disc,[rabbit@rabbit1]},{ram,[rabbit@rabbit2]}]},
# =>  {running_nodes,[rabbit@rabbit2,rabbit@rabbit1]}]
# => ...done.

# on rabbit2
rabbitmqctl cluster_status
# => Cluster status of node rabbit@rabbit2 ...
# => [{nodes,[{disc,[rabbit@rabbit1]},{ram,[rabbit@rabbit2]}]},
# =>  {running_nodes,[rabbit@rabbit1,rabbit@rabbit2]}]
# => ...done.
```
#### Changing node types
We can change the type of a node from ram to disc and vice versa. Say we wanted to reverse the types of rabbit@rabbit2 and rabbit@rabbit1, turning the former from a ram node into a disc node and the latter from a disc node into a ram node. To do that we can use the change_cluster_node_type command. The node must be stopped first.
```
# on rabbit2
rabbitmqctl stop_app
# => Stopping node rabbit@rabbit2 ...done.

rabbitmqctl change_cluster_node_type disc
# => Turning rabbit@rabbit2 into a disc node ...
# => ...done.
# => Starting node rabbit@rabbit2 ...done.

# on rabbit1
rabbitmqctl stop_app
# => Stopping node rabbit@rabbit1 ...done.

rabbitmqctl change_cluster_node_type ram
# => Turning rabbit@rabbit1 into a ram node ...

rabbitmqctl start_app
# => Starting node rabbit@rabbit1 ...done.
```
### 扩展阅读
#### 1. Rabbitmq集群高可用测试
https://www.cnblogs.com/flat_peach/archive/2013/04/07/3004008.html
#### 2. RabbitMQ 集群与高可用配置
https://88250.b3log.org/rabbitmq-clustering-ha

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

### 7 镜像模式配置

```
Policy: all-mirrors
Overview
Virtual Host    /
Pattern .*
Apply to    all
Definition
ha-mode:    all
ha-sync-mode:   automatic
Priority    0
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
见前文
#### Creating the cluster
见前文
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

### 扩展阅读
#### 1. Rabbitmq集群高可用测试
https://www.cnblogs.com/flat_peach/archive/2013/04/07/3004008.html
#### 2. RabbitMQ 集群与高可用配置
https://88250.b3log.org/rabbitmq-clustering-ha

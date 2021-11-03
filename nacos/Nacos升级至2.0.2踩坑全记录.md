# 0. 背景&解决方案

### 1. 初始状态：

客户端版本：1.2.1
服务端版本：1.1.4

我们的目标是将 Nacos 服务端由 A 集群迁移为 B 集群。

为了达到这个目标，迁移过程分为 3 部分：

（1）升级过程：将目前的 A 集群由 Nacos-1.1.4 版本升级为 Nacos-2.0.2 版本

（2）扩容过程：将 A 集群（ Nacos-2.0.2 版本）扩容为 A+B 集群（ Nacos-2.0.2 版本）

（3）缩容过程：将 A+B 集群缩容为 B 集群

### 2. 问题&回答

（1）为什么客户端与服务端版本不一致
答：历史原因

（2）目前状态，Nacos 能正常提供服务吗？
答：恰好可以（具体见排坑指南）

（3）为什么要升级为 2.0.2 版本
答：服务端兼容（具体见排坑指南）

（4）为什么不降级客户端（服务端保持 1.1.4，扩容之后再缩容）
答：
从成本来看，降级客户端需要所有使用方变动，而升级服务端只变动集群；
从安全来看，1.1.4 版本有安全问题，直到 1.4.1 才解决；
发展来看，2.0.2 版本的性能提升，后续肯定会升级，不管是客户端还是服务端。

### 3. 解决方案

#### 3.1 集群中节点启停过程中，实例暂时下线

（1）节点停止时，固有问题

节点停止时，不能对外提供服务。在其他服务端感知这种状态之前（最长一个心跳周期），无论是直接请求，还是其他服务端转发请求，都不能获得数据。

此时，如果客户端拉取数据，获得实例为空（即使实例正常）。

如果使用 Feign 等方式，由于其已经做了容错处理，讲道理不会获得空的实例列表。

解决方案：客户端本地做缓存（当获取的实例为空时，使用上一次可用的实例列表）

注意：如果使用的 AP 模式，理论上会有数据不一致的情形。

（2）节点停止后，心跳补偿

节点停止后，集群中的其他服务端会在可用列表中剔除该节点，后续客户端请求（无论是消费还是注册）都不会映射到该节点。

在集群中，健康服务端列表发生变化时，客户端与服务端的对应关系也发生变化。

服务端会检查之前不有它负责的节点（现在由它负责），发现节点已经过期（集群间同步数据，并不会同步心跳的时间戳），会主动下线『过期』的实例。

在实例心跳再一次上传时，实例才恢复。

这段不一致的时间，有概率出现，但最长为客户端的一个心跳周期。

解决方案：使用心跳补偿，尽可能地缩短实例暂时下线地时间。

（3）节点启动后，tricky 方式（ABA 问题，具体见排坑指南）

其实节点的启停，只要健康服务端列表发生变化，某些实例就可能被服务端认为『过期』而短暂下线。

解决方案：使用心跳补偿

（4）节点启动后，版本问题

如果服务端无法转发客户端的请求，则客户端的请求会失败（无论是消费还是注册）。则映射为新节点的所有客户端都会失败。

解决方案：升级服务端版本，兼容不同的客户端（升级为 2.0.2 版本，正是我们在做的）

#### 3.2 集群升级过程中，高版本同步双写时，实例暂时下线

（1）tricky 方式（ABA 问题）

在我们 2 节点集群的场景中，不会出现。

（2）心跳等内容不会被双写，意味着高版本并不会转发心跳请求

升级过程中，为了节约性能，双写的内容仅是内容发生变更时的状态，心跳等内容不会被双写，因此切换版本时，可能有部分实例的心跳过久而健康检查又刚好开始执行，从而被标记非健康或摘除。

后续心跳处理将会把数据补充回来，最终会一致。

解决方案：使用心跳补偿

### 3.3 集群升级后，关闭同步双写之后，实例暂时下线

当集群中最后一个节点也升级到 2.0.X 版本时，集群会开始进行升级检测。每个节点会对该节点的服务信息和实例信息进行校验，并检测是否还有未完成的双写任务。

当该节点的服务信息和实例信息已经核对成功，并且没有双写任务存在时，该节点会判定自己已经做好升级准备，并修改自己的状态且通知其他 Nacos 节点。

每台节点是否完成升级准备可以从控制台的集群管理中元数据信息中看到"readyToUpgrade": false/true。

当集群中所有节点均判定为准备完毕时。Nacos 集群中的节点会进行升级切换，自动升级到 Nacos2.0 的处理逻辑。

可以从 logs/naming-server.log 日志中观察到 upgrade check result true 及 Upgrade to 2.0.X。

关闭同步双写，会清理低版本的缓存数据。若数据没有在集群中完全同步完成，会有实例数据暂时丢失。

解决方案：使用心跳补偿

#### 3.4 集群扩容，新节点启动，关闭同步双写之后，数据不一致

新节点启动之后，才会出现在集群中其他服务端的可用列表之中（未启动之前，不会提供服务）。

此时，集群已是 2.0.2 版本的集群，可以对客户端的请求做转发（不会出现转发失败的情况）。

由于是新增的节点，内存中不会有历史数据（其不会因为 tricky 方式下线实例）。

新加入的节点并不会出现在客户端的配置中，则消费端不会从该节点获取数据。

解决方案：等待数据同步完成。

或者重启该节点，数据全量拉取，保持一致。风险：集群中节点启停过程中，实例暂时下线（使用心跳补偿可以兜底）。

### 4. 心跳补偿

基本想法是，首先对 Nacos 服务端的服务实例做快照，然后模拟客户端的心跳，将服务实例信息定时向服务端心跳。

目的是在升级、扩容、缩容过程中，保持服务实例不被注销，以免消费端获取不到服务实例。

（1）获取 Nacos 服务端的所有服务名
（2）对于每个服务，获取服务下的所有服务实例信息，保存
（3）将获取的每个服务实例信息转化为对应的心跳信息
（4）启动定时任务，将所有心跳信息通过 HTTP 请求向目标服务端发送
（5）启动多个心跳补偿程序，覆盖 Nacos 集群中的各个节点

# 1. 前期准备

### 1. 配置安全组

（1）所有业务线访问 Nacos 机器的 8848 和 9848 端口；

（2）Nacos 集群节点之间通信，开通 9848，9849，8848，7848 端口；

8848 是主端口，客户端 HTTP 通信时必须；
7848 是 RAFT 端口，服务端之间运行 RAFT 协议必须；

| 端口 | 与主端口的偏移量 | 描述                                                         |
| ---- | ---------------- | ------------------------------------------------------------ |
| 9848 | 1000             | 客户端 gRPC 请求服务端端口，用于客户端向服务端发起连接和请求 |
| 9849 | 1001             | 服务端 gRPC 请求服务端端口，用于服务间同步等                 |

注意：7848 在官方文档中并没有提及，这里别忘了

### 2. 下载源码或者安装包

可以通过源码和发行包两种方式来获取 Nacos。

（1）从 Github 上下载源码方式

```s
git clone https://github.com/alibaba/nacos.git

cd nacos/

mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U

ls -al distribution/target/

// change the $version to your actual path
cd distribution/target/nacos-server-$version/nacos/bin
```

注意：这里下载 [2.0.2 版本](https://github.com/alibaba/nacos/releases/tag/2.0.2)

（2）下载编译后压缩包方式

您可以从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 nacos-server-$version.zip 包。

```s
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
cd nacos/bin
```

### 3. 配置集群配置文件

（1）配置外置数据源（application.properties）

首先初始化 MySQL 数据库：[sql 语句源文件](https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql)

配置文件示例：[application.properties](https://github.com/alibaba/nacos/blob/master/distribution/conf/application.properties)

```yml
#******\*\*\******* Config Module Related Configurations ******\*\*\*******#

### If use MySQL as datasource:

spring.datasource.platform=mysql

### Count of DB:

db.num=1

### Connect URL of DB:

db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=nacos
```

注意：这里配置对应的 MYSQL 的地址。

（2）配置集群文件（cluster.conf）

在 nacos 的解压目录 nacos 的 /conf 目录下，有配置文件 cluster.conf，请每行配置成 ip:port

注意：这里按需配置（见后续升级/扩容/缩容过程）。

（3）配置 Nacos 访问日志路径（application.properties）

```yml
#*************** Access Log Related Configurations ***************#
### If turn on the access log:
server.tomcat.accesslog.enabled=true

### The access log pattern:
server.tomcat.accesslog.pattern=%h %l %u %t "%r" %s %b %D %{User-Agent}i %{Request-Source}i

### The directory of access log:
server.tomcat.basedir=
```

注意：生产环境需要开启访问日志，由于日志很大，需要定时清理（配置 CRONTAB）

### 4. 配置启动脚本参数

修改 startup.sh 的参数，按需配置。

```s
# 开启远程DEBUG端口，生产环境一定不要设置
JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8001"
# JVM参数设置，可按需修改
# 参见 https://opts.console.perfma.com/
JAVA_OPT="${JAVA_OPT} -server -Xms1500m -Xmx1500m -Xmn900m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
# 使用G1 GC
JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:MaxGCPauseMillis=100"
# 开启堆外内存监控（性能会有所下降），jcmd <pid> VM.native_memory scale=MB 查看使用
JAVA_OPT="${JAVA_OPT} -XX:NativeMemoryTracking=summary -XX:MaxDirectMemorySize=256m"
```

### 5. 开启心跳补偿程序

（1）上传心跳补偿程序到 A 集群
（2）分别启动程序（使用启停脚本）
（3）升级/扩容/缩容完成后，关闭心跳补偿程序

# 2. 升级过程

### 1. 关闭 A 集群的 FOLLOWER 节点

（1）查看 1.1.4 集群的 LEADER

```s
# 1.X 查看 LEADER
curl -X GET 'localhost:8848/nacos/v1/ns/raft/leader'
```

（2）关闭其中的 FOLLOWER 节点

```s
# 进入 bin 目录
sh shutdown.sh
```

### 2. 启动 FOLLOWER 节点的 2.0.2 版本

（1）确认 cluster.conf 的配置（应该配置为 A 集群列表）

（2）启动 2.0.2 版本

```s
# 进入 bin 目录
sh startup.sh
```

（3）观察是否启动成功

首先查看 nacos 目录下 logs/start.out 或 logs/nacos.log 观察到 nacos 启动成功的日志，如 Nacos started successfully in cluster mode. use xxx storage 说明程序已启动成功。

之后在观察 logs/naming-server.log 中，可以看到有 upgrade check result false 以及 Check whether close double write 等日志信息。属于正常现象。

### 3. 关闭 1.1.4 集群的 LEADER 节点

同步骤 1

### 4. 启动 LEADER 节点的 2.0.2 版本

同步骤 2

### 5. 关闭 2.0.2 集群的同步双写

（1）查看当前升级状态

```s
curl -X GET 'localhost:8848/nacos/v1/ns/upgrade/ops/metrics'
```

部分节点升级时：

```s
GET /nacos/v1/ns/upgrade/ops/metrics
upgraded                       = false
isAll20XVersion                = false
isDoubleWriteEnabled           = true
```

全部节点升级时：

```s
GET /nacos/v1/ns/upgrade/ops/metrics
upgraded                       = false
isAll20XVersion                = true
isDoubleWriteEnabled           = true
```

关闭同步双写后：

```s
GET /nacos/v1/ns/upgrade/ops/metrics
upgraded                       = true
isAll20XVersion                = true
isDoubleWriteEnabled           = false
```

（2）关闭双写

```s
curl -X PUT 'localhost:8848/nacos/v1/ns/operator/switches?entry=doubleWriteEnabled&value=false'
```

关闭后可以从 logs/naming-server.log 日志中观察到 Disable Double write, stop and clean v1.x cache and features 字样。说明关闭双写。

（3）再次查看当前升级状态

### 6. 重启数据不一致的节点

（1）对于每一个节点，查看服务列表

```s
curl -X GET 'localhost:8848/nacos/v1/ns/catalog/services?hasIpCount=true&pageNo=1&pageSize=1&namespaceId=public'

# 如果有多个 namespace，则需要加上 &namespaceId=命名空间的 ID，默认 namespaceId=public
```

（2）如果有不一致的节点，则进行重启

（3）再次检查服务列表

# 3. 扩容过程

注意：应该先更新配置，再启动节点

### 1. Nacos-2.0.2 集群新增一个节点

（1）分别修改当前集群（不妨用 A 替代） 中每个节点的 cluster.conf 文件，配置为 A+1 节点

（2）确认当前集群列表为 A+1 个 节点

```s
curl -X GET 'localhost:8848/nacos/v1/ns/operator/servers'
```

（3）启动新增节点 的 2.0.2 版本（cluster.conf 文件，配置为 A+1 节点）

```s
# 进入 bin 目录
sh startup.sh
```

（4）在新增节点上关闭双写

```s
curl -X PUT 'localhost:8848/nacos/v1/ns/operator/switches?entry=doubleWriteEnabled&value=false'
```

（5）对于每一个节点，查看服务列表

```s
curl -X GET 'localhost:8848/nacos/v1/ns/catalog/services?hasIpCount=true&pageNo=1&pageSize=1&namespaceId=public'

# 如果有多个 namespace，则需要加上 &namespaceId=命名空间的 ID，默认 namespaceId=public
```

（6）如果有不一致的节点，则进行重启

（7）再次检查服务列表

### 2. 按照 1 中的步骤，依次增加节点，直到扩容完成

# 4. 缩容过程

注意：应该先停止节点，再更新配置

待业务全部迁移之后，就可以进行缩容了。

### 1. Nacos-2.0.2 集群减少一个节点

（1）关闭目标节点（不妨称之为 X）

（2）修改集群中各个节点的 cluster.conf 文件，去除 X

### 2. 按照 1 中的步骤，依次减少节点，直到缩容完成

# 5. 排坑指南

### 1. nacos-2.0.2 集群，没有权限管理页面

需要 ROLE_ADMIN 角色的用户才能看到

（1）权限开关 nacos.core.auth.enabled 默认为 false，开启后客户端-服务端，服务端之间都需要认证（改动很大）
（2）console 前端的权限控制，与角色有关，与这个 nacos.core.auth.enabled 配置无关
（3）管理员角色为 ROLE_ADMIN，可以看到权限管理页面，可以创建用户
（4）貌似权限管理中的资源，是针对配置的，对于注册，没有效果，用户能登录就能操作服务上下线
（5）密码使用 org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder 加密
（6）可以在用户表 users 中添加用户，在角色表 roles 中添加管理员

示例：

```sql
INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);
INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```

### 2. 角色是 ROLE_ADMIN，但是认证时 globalAdmin: false

没有 permissions 表，建出来就好了

```sql
CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);
```

### 3. nacos 连接 mysql 出现：Could not create connection to database server

（1）问题

nacos 连接 mysql 出现

```log
Could not create connection to database server. Attempted reconnect 3 times. Giving up
```

（2）原因

其实主要的问题，就是缺少时区的设置：serverTimezone

```yml
characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true # 错误的写法
characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC # 正确的，且是官网提供的

```

（3）方案

查看官方 application.properties 配置
地址：https://github.com/alibaba/nacos/blob/master/distribution/conf/application.properties

```yml
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
# spring.datasource.platform=mysql

### Count of DB:
# db.num=1

### Connect URL of DB:
# db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
# db.user.0=nacos
# db.password.0=nacos
```

### 4. InstanceUpgradeHelper 报 NoSuchBeanDefinitionException

错误日志：

```log
org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.alibaba.nacos.naming.core.v2.upgrade.doublewrite.execute.InstanceUpgradeHelper' available
```

ISSUE 地址：https://github.com/alibaba/nacos/pull/6115
官方已处理但未合并：https://github.com/alibaba/nacos/pull/6115/commits/222ea6a4fd532502ca203177fb3b65d953e6d275

测试发现，当集群关闭同步双写后，错误日志不再输出。

### 5. Nacos-1.1.4 集群，节点重启之后，出现大量服务注销，随后恢复

#### 5.1 基本事实

（1）客户端的版本为 1.2.1，服务端的版本为 1.1.4
（2）服务端由于 distro 协议，当判断客户端请求的服务不为自己负责时，则会转发到对应的服务端。如果集群节点（活着的）没有变化，同一个客户端请求对应的服务端不变
（3）客户端在心跳续约时，会向服务列表续约，使用轮询的方式。如果上一个失败，则下一个。每次选择的起始服务端是随机的。如果都续约失败，会有报错日志
（4）客户端的相关日志需要显式配置，否则应用不会有客户端的日志
（5）心跳信息通过 PUT 请求上报，1.1.4 的客户端会将心跳数据放置在请求参数（URL）中，而 1.2.1 的客户端将心跳信息放置在请求体（body）中
1.2.1 心跳数据放在 BODY 中，可以通过 HttpServletRequest.getParameterMap 获取
1.1.4 心跳数据放在 URL 中，可以通过 HttpServletRequest.getQueryString 获取

（6）服务端默认进行转发，开关变量 distro 可以通过开关控制，实时生效
（7）服务端在进行转发时，并不会将请求体中的心跳数据转发

#### 5.2 为什么节点重启之前没有问题？

（1）使用 1.2.1 客户端，配置了多个服务端地址。

（2）如果客户端直接向目标服务端续约，则续约成功，没有任何问题。

（3）如果客户端向另一个服务端续约，该服务端会对客户端的续约请求进行转发，而转发并不会带上心跳数据，故转发失败。此时，客户端轮询到目标机器，进而续约续约成功，没有任何问题。

#### 5.3 节点重启时（停止一个可用节点，相当于缩容）的现象解释

（1）当集群中某服务端停止时，该服务端对应的客户端不可续约。在另一个服务端（活着的）感知这种状态之前，其继续进行转发，当然也不能转发成功。当感知下线时，活着的服务端就是对应的服务端，此时续约成功。

（2）在服务端重启（停止到启动的过程）期间，在另一个服务端（活着的）感知这种状态之前，向这台操作的服务端续约的心跳都不会成功。

（3）在服务端启动之后，续约功能正常。

#### 5.4 节点重启后（新增一个可用节点，相当于扩容）的现象解释

这是 distro 协议的固有问题，姑且称之为 ABA 问题。

具体见下文。

### 6. Nacos-1.1.4 集群扩容时，回调和拉取方式，返回的结果为空，后续不能恢复

#### 6.1 distro 协议

Distro 是阿里巴巴的私有协议，开源的 Nacos 就是使用的这个协议，这个协议有以下几个关键点：

（1）distro 协议是为了注册中心而创造出的协议；
（2）客户端与服务端有两个重要的交互，服务注册与心跳发送；
（3）客户端以服务为维度向服务端注册，注册后每隔一段时间向服务端发送一次心跳，心跳需要带上注册服务的全部信息，在客户端看来，服务端节点对等，所以请求的节点是随机的；
（4）客户端请求失败则换一个节点重新发送请求；
（5）服务端节点都存储所有数据，但每个节点只负责其中一部分服务，在接收到客户端的“写”（注册、心跳、下线等）请求后，服务端节点判断请求的服务是否为自己负责，如果是，则处理，否则交由负责的节点处理；
（6）每个服务端节点主动发送健康检查到其他节点，响应的节点被该节点视为健康节点；
（7）服务端在接收到客户端的服务心跳后，如果该服务不存在，则将该心跳请求当作注册请求来处理；
（8）服务端如果长时间未收到客户端心跳，则下线该服务；
（9）负责的节点在接收到服务注册、服务心跳等写请求后将数据写入后即返回，后台异步地将数据同步给其他节点；
（10）节点在收到读请求后直接从本机获取后返回，无论数据是否为最新。

#### 6.2 ABA 问题

（1）比如集群中有 3 个节点 ABC，客户端 X 对应的处理服务端为 A

（2）当服务端 C 停止之后，假设客户端 X 对应的处理服务端变为 B

（3）在 AB 集群稳定之后，再重启服务端 C

（4）当 ABC 集群稳定之后，客户端 X 对应的处理服务端又变为 A

（5）此时 A 会检查 X 的心跳时间，发现距离上一次的时间已经超时，认为 X 已过期，主动下线 X，并同步集群！

（6）直到 X 继续上传心跳，X 才恢复

### 6.3 扩容时的现象解释

（1）扩容时，新加入的节点成为集群的 FOLLOWER 之后，会被其他节点加入可提供服务列表

（2）此时，原先的客户端与对应服务端的关系发生了变化，某些客户端的对应服务端变成了新加入的节点

（3）因为客户端只配置了旧的服务端列表，也就是说，这些客户端继续向这些服务端续约。但是，由于 distro 协议， 它们都会进行转发，而由于不能转发心跳数据，进而续约失败。这些客户端会被下线

（4）此后，新加入其他节点时，现象也是如此。

### 7. Nacos 2.X 的服务端是否与 1.X 的服务端组成集群？

（1）能否支持 Nacos 旧版本客户端？

配置中心兼容支持 Nacos1.0 起的所有版本客户端，服务发现兼容 Nacos1.2 起所有版本客户端。 因此建议使用 Nacos1.2.0 之后版本客户端。 但 nacos1.X 的客户端不具有长连接能力，因此仍然建议使用 Nacos2.0.0 客户端。

（2）兼容性
Nacos2.0 的服务端完全兼容 1.X 客户端。Nacos2.0 客户端由于使用了 gRPC，无法兼容 Nacos1.X 服务端，请勿使用 2.0 以上版本客户端连接 Nacos1.X 服务端。

（3）Nacos 2.0.0 部署及升级文档

https://nacos.io/zh-cn/docs/2.0.0-upgrading.html
由于 Nacos1.X 和 Nacos2.0 的数据结构发生了变化，为了能够完成平滑升降级，需要将数据进行双写，分别生成 Nacos1 和 Nacos2 的数据结构进行存储。

### 8. Nacos 客户端使用注意事项

（1）客户端应该配置多地址，保证高可用
（2）客户端的版本应该和服务端保持一致（或服务端兼容不同的客户端）
（3）即使客户端和服务端版本保持一致，在集群节点变更时，仍然有概率造成某些实例下线（对应的服务端不可用，而其他可用的服务端都进行转发，一圈之后，续约失败）
（4）对于实例为空情况不能容忍的应用，应该使用缓存做好兜底（当获取的实例为空时，使用上一次可用的实例列表）

### 9. Nacos 的临时实例如何注册与注销

Nacos 在 1.0.0 版本 instance 级别增加了一个 ephemeral 字段，该字段表示注册的实例是否是临时实例还是持久化实例。如果是临时实例，则不会在 Nacos 服务端持久化存储，需要通过上报心跳的方式进行包活，如果一段时间内没有上报心跳，则会被 Nacos 服务端摘除。在被摘除后如果又开始上报心跳，则会重新将这个实例注册。持久化实例则会持久化被 Nacos 服务端，此时即使注册实例的客户端进程不在，这个实例也不会从服务端删除，只会将健康状态设为不健康。

同一个服务下可以同时有临时实例和持久化实例，这意味着当这服务的所有实例进程不在时，会有部分实例从服务上摘除，剩下的实例则会保留在服务下。

使用实例的 ephemeral 来判断，ephemeral 为 true 对应的是服务健康检查模式中的 client 模式,为 false 对应的是 server 模式。

持久化实例健康检查失败后会被标记成不健康，而临时实例会直接从列表中被删除。

nacos 持久化节点如何探活？

答：服务端访问客户端的注册 IP 与 PORT。

### 10. Nacos 的一些疑问

（1）Nacos 读取服务器列表的两种方式

方式一：本地读取 cluster.conf
方式二：读取统一配置中心配置文件

如果本地也配置了 cluster.conf 的话，那么会优先读取本地的配置的; 如果本地的读取不到列表，才会去读取远程的服务器列表。

（2）getApacheServerList() 获取服务器列表的方法

优先从本地文件读取服务列表，如果读取到了直接返回；

如果方式一中没有读取到，则判断 useAddressServer=true；如果为 true，则读取远程服务器中的服务器列表，如果读取到了直接返回；

如果方式二 中执行了 maxFailCount=12 次还是没有获取到，则标识 isAddressServerHealth=false，说明远程服务器挂掉了;

如果本地没有数据，并且 useAddressServer=false，那么就会把自己的 IP 加入到服务器列表：也就是说只有一台机器；

这个方法只是获取运维配置的集群服务器列表，并没有去检验每个集群列表的机器是否健康！如果使用方式二，远程配置中心服务器不可访问那么返回的是一个空列表；

（3）如何获取自己的 IP

先看看 JVM 属性配置了 nacos.server.ip=IP 地址没有，如果有就是它；

如果方式一中没有，则看看配置文件 application.properties 中有没有属性 nacos.inetutils.ip-address=IP 地址，如果有就是它；

如果还没有，那判断是否优先使用 hostname。preferHostnameOverIp 的判断逻辑是：

先判断 JVM 属性有没有配置 nacos.preferHostnameOverIp=true/false；

如果 false，再去判断配置文件 application.properties 中有没有属性 nacos.inetutils.prefer-hostname-over-ip=true/false；

如果有的话 就优先获取 hostname：inetAddress.getHostName();

否则的话，就获取所有网卡中第一个非回环地址 findFirstNonLoopbackAddress().getHostAddress()，就是不会找到 127.0.0.1 这样的回环地址，具体代码在类 InetUtils 中。

（4）CheckServerHealthTask 服务器健康检查

系统会每隔 5 秒执行一次服务器健康检查：其实就是给所有的服务器列表发起一个 Http 请求，根据返回值判断是否健康。

解析得到的链接是 http://ip:port/nacos/v1/cs/health 一句话说就是，访问每个服务器列表的 nacos/v1/cs/health 方法，包括自己的。

最终请求的是 HealthController 这个类的 getHealth 方法。

（5）当某一台机器宕机挂掉之后怎么处理的

当服务器挂掉或者宕机，每五秒的健康检查会检查到服务宕机了，会将其剔除。

（6）Nacos 服务消费原理

服务消费者对于服务实例的动态更新主要来源于两个地方，第一个就是本地的定时任务，第二个就是采用服务端的 Push 机制。

开启定时调度，每 10s 去查询一次服务地址。如果本地缓存中存在，则通过 scheduleUpdateIfAbsent 开启定时任务，再从 serviceInfoMap 取出 serviceInfo。

监听服务状态变更事件，然后遍历所有的客户端，通过 udp 协议进行消息的广播通知。那么服务消费者此时应该是建立了一个 udp 服务的监听，否则服务端无法进行数据的推送。

### 11. 消费端 namingLoadCacheAtStart 参数：解决服务端不可用情况

（1）Nacos 集群下线，仍然可以获得实例数据
（2）服务注册停机，获得的实例为空

总结：当服务端不可用时，消费端仍然可以从本地缓存中获取数据

### 12. Nacos-2.0.2 集群回退至 Nacos-1.1.4 集群后，客户端 1.2.1 大量实例无法注册

理论上，心跳信息包含所有元数据，一个心跳之后，会重新注册成功

原因：lightBeatEnabled 参数变为 true

```java
if (!lightBeatEnabled) {
    try {
        body = "beat=" + URLEncoder.encode(JSON.toJSONString(beatInfo), "UTF-8");
    } catch (UnsupportedEncodingException e) {
        throw new NacosException(NacosException.SERVER_ERROR, "encode beatInfo error", e);
    }
}
```

客户端 1.2.1 中 NamingProxy，被 Nacos-2.0.2 告知不要发 body，Nacos-1.1.4 自然就接收不到心跳了

解决方案：启动模拟的 Nacos 服务端，重置 lightBeatEnabled=false

待所有 1.2.1 客户端正常发送 body 之后，就可以恢复 Nacos-1.1.4 集群了。

注意：在重置期间，Nacos 服务是不可用的。

理论上最长不可用时间为 1 个心跳周期（5 秒），等待 1 个心跳周期之后，所有的客户端应该都正常。

PS：其实这个问题，就是高版本客户端，低版本服务端引起的，最好是直接升级服务端至 Nacos-2.0.2，不再回退！

关键代码：

```java
@RestController
@Slf4j
@RequestMapping("/nacos/v1/ns/instance")
public class FakeBeatServer {

    private volatile boolean lightBeatEnabled = false;

    private volatile Set<String> serviceSet = new HashSet<String>();
    //注册
    @PostMapping
    public String register(HttpServletRequest request) throws Exception {

        return "ok";
    }
    //注销
    @DeleteMapping
    public String deregister(HttpServletRequest request) throws Exception {

        return "ok";
    }
    //更新
    @PutMapping
    public String update(HttpServletRequest request) {

        return "ok";
    }
    //心跳，在此通知客户端是否要发body（设置lightBeatEnabled参数）
    @PutMapping("/beat")
    public String beat(HttpServletRequest request) throws Exception {

        Optional beat = Optional.ofNullable(request.getParameter("beat"));
        System.err.println("beat::" + beat.orElse(null));
        log.info("beat：{}", beat);

        HashMap<String, Boolean> switches = new HashMap<>();
        switches.put("lightBeatEnabled", lightBeatEnabled);

        String serviceName = required(request, "serviceName");
        serviceSet.add(serviceName);
        return JSONHelper.toString(switches);
    }
    //----管理 lightBeatEnabled
    @PutMapping("/flag")
    public String flag(HttpServletRequest request) throws Exception {

        String flag = request.getParameter("flag");
        lightBeatEnabled = Boolean.parseBoolean(flag);
        return JSONHelper.toString(lightBeatEnabled);
    }
    //----统计已经通知的服务个数
    @GetMapping("/size")
    public String size() {
        int size = serviceSet.size();
        return JSONHelper.toString(size);
    }
    //----统计已经通知的服务列表
    @GetMapping("/service")
    public String service() {

        return JSONHelper.toString(serviceSet);
    }

    String required(HttpServletRequest req, String key) {
        String value = req.getParameter(key);
        if (StringHelper.isEmpty(value)) {
            return "";
        }

        String encoding = req.getParameter("encoding");
        if (!StringHelper.isEmpty(encoding)) {
            try {
                value = new String(value.getBytes(StandardCharsets.UTF_8), encoding);
            } catch (UnsupportedEncodingException ignore) {
            }
        }

        return value.trim();
    }

}

```

### 13. Nacos 2.0.2 的服务端，在客户端重新注册后，仍然使用缓存的状态（enabled=false）

#### 13.1 原因：Nacos 的元信息被缓存

（1）注册和注销都会有回调，当实例不健康时也会有回调
（2）当实例被判定为不健康时，会被注销，心跳也不能改变这种趋势，但是注册可以
（3）只要缓存数据（enabled，metadata，weight）不过期，就以缓存为准，但注册时可以新增元数据
（4）当实例注销时，不管是主动注销还是服务端判断不健康而注销，心跳相当于注册
（5）健康状态一般由服务端置为 false，注册时可以主动设置为 false，但之后会被置为 true
（6）注销后，缓存不会立即清理
（7）下线后，回调关闭，不会再有任何回调

结论：注册时，元信息可以覆盖，但不能覆盖通过更新接口写入的数据

#### 13.2 缓存的元信息什么时候被清除

（1）缓存信息的过期时间，默认 60 秒

```s
# Nacos 的 application.properties 文件

### The expired time to clean metadata, unit: milliseconds.
# nacos.naming.clean.expired-metadata.expired-time=60000
```

（2）缓存信息的清理时间间隔，默认 5 秒

```s
# Nacos 的 application.properties 文件

### The interval to clean expired metadata, unit: milliseconds.
# nacos.naming.clean.expired-metadata.interval=5000
```

（3）清理的请求，通过 JRaftProtocol 广播至集群，最长等待时间 10 秒

```java
    //com.alibaba.nacos.core.distributed.raft#JRaftProtocol 类
    @Override
    public Response write(WriteRequest request) throws Exception {
        CompletableFuture<Response> future = writeAsync(request);
        // Here you wait for 10 seconds, as long as possible, for the request to complete
        return future.get(10_000L, TimeUnit.MILLISECONDS);
    }
```

（4）实例经过 3 个心跳周期（一个心跳周期为 5 秒）未发送心跳时，会被服务端判定为健康状态为 healthy=false；

（5）healthy=false 的实例经过 3 个心跳周期（一个心跳周期为 5 秒），会被服务端注销

实例注销后，元信息才开始清理。

缓存的元信息的最长清理时间为：15（不健康）+15（注销）+5（开始清理）+60（过期时间）+10（处理时长，不确定）=105 秒

即使客户端主动注销（省去 30 秒），设置 nacos.naming.clean.expired-metadata.expired-time=0，过期时间也在 15 秒左右！

### 14. Nacos 在 2C4G 环境，使用 2G 的堆，机器内存使用率超过 80%

（1）是否 GC 需要优化
（2）是否堆外内存泄露

配置 JVM 参数如下：

```s
# JVM参数设置，可按需修改
# 参见 https://opts.console.perfma.com/
JAVA_OPT="${JAVA_OPT} -server -Xms1500m -Xmx1500m -Xmn900m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
# 使用G1 GC
JAVA_OPT="${JAVA_OPT} -XX:+UseG1GC -XX:MaxGCPauseMillis=100"
# 开启堆外内存监控（性能会有所下降），jcmd <pid> VM.native_memory scale=MB 查看使用
JAVA_OPT="${JAVA_OPT} -XX:NativeMemoryTracking=summary -XX:MaxDirectMemorySize=256m"
```

### 15. 网关-Nacos 信息同步

如何识别应用重启？

Nacos 客户端中增加元信息

```yml
spring.cloud.nacos.discovery.metadata.app_registry_tag=${random.uuid}
```

当应用重启之后，app_registry_tag 元信息会变化！

#### 15.1 目标：

（1）保证实时性：期望能立即同步
使用回调机制；
（2）保证正确性：不能提供服务的实例一定要从网关下线
上线比下线严格；
（3）保证活性：正常提供服务的实例最终在网关上线
使用轮询机制；

#### 15.2 网关下线时机

（1）应用发出的下线请求，下线
（2）应用过期，监听 Nacos 的回调，当健康状态为 false（可能应用正常提供服务），下线
（3）应用注销，监听 Nacos 的回调，获取实例为空（可能应用正常提供服务），下线
（4）轮询，nacos 没有，而网关有，下线

#### 15.3 网关上线时机

（1）上线回调时，对比快照与当前，检查标识有变化且健康状态为 true，上线
（2）轮询时，对比快照与当前，检查标识有变化且健康状态为 true，上线
（3）轮询时，健康状态为 true，但标识无变化，人工处理，或统计持续的次数，达到阈值，再上线
（4）轮询时，网关没有该应用的快照，上线

#### 15.4 快照的维护

（1）初始时，全量应用的快照，并设置监听
（2）应用上线后，更新快照
（3）应用下线时，不做处理（快照的数据只增不减，对于容器应用来说，注意内存的使用）
（4）发现 nacos 有，而网关没有时，写入快照，新增监听

#### 15.5 应对的情况

（1）应用发出下线请求之后，应用可能过期与注销
（2）应用异常宕机，一定会过期与注销，而且长时间无数据
（3）网络抖动，应用可能过期与注销，但标识不变
（4）新起的应用，网关上需要新增监听
（5）应用不带标识（可认为标识为默认值）

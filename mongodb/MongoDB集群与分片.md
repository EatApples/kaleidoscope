### 1. MongoDB 简单操作
#### 1.1 单机启动
```
# Linux 环境
/app/mongo/bin/mongod --dbpath /app/mongo/data/ --logpath /app/mongo/data/log/mongodb.log --logappend --fork &

# Windows 环境
mongod.exe --logpath "E:\DBLOG\mongodb.log" --logappend --dbpath "D:\DB" --port 27017 --serviceName "MGDBService" --serviceDisplayName "MMService" --install --rest --httpinterface

```

#### 1.2 备份数据
```
# {这里面的参数可选}

# 本地备份
./mongodump -v  -d "db名字" {-c "table名字"} --out 文件路径

# 远程备份
./mongodump -v --host=IP地址 --port=端口 {--authenticationDatabase=admin --username=用户名 --password=密码} -d "db名字" {-c "table名字"} --out 文件路径

```


#### 1.3 恢复数据
```
{这里面的参数可选}

# 本地恢复
./mongorestore -v  -d "db名字" --dir 文件路径  --objcheck

# 远程恢复
./mongorestore -v --host=IP地址 --port=端口 {--authenticationDatabase=admin --username=用户名 --password=密码} -d "db名字"  文件路径

```

### 2. MongoDB 高可用
mongoDB中有 master-slave，replica set 与 shard 三种模式。

#### 2.1 master-slave

#### 2.2 replica set
mongoDB 默认情况下只能在 primary 节点上进行读写操作。

对于客户端应用程序来说，对复制集的读写操作是透明的，默认情况它总是在 primary 节点上进行。mongoDB 提供了很多种常见编程语言的驱动程序，驱动程序位于应用程序与 mongod 实例之间，应用程发起与复制集的连接，驱动程序自动选择 primary 节点。当 primary 节点失效，复制集发生故障转移时，复制集将先关闭与所有客户端的 socket 连接，驱动程序将返回一个异常，应用程序收到这个异常，这个时候需要应用程序开发人员去处理这些异常，同时驱动程序会尝试重新与 primary 节点建立连接（这个动作对应用程序来说是透明的）。假如这个时候正在发生一个读操作，在异常处理中你可以重新发起读数据命令，因为读操作不会改变数据库的数据；假如这个时候发生的是写操作，情况就变得微妙起来，如果是非安全模式下的写，就会产生不确定因素，写是否成功不确定，如果是安全模式，驱动程序会通过 getlasterror 命令知道哪些写操作成功了，哪些失败，驱动程序会返回失败的信息给应用程序，针对这个异常信息，应用程序可以决定怎样处置这个写操作，可以重新执行写操作，也可以直接给用户报出这个错误。

#### 2.3 shard
分片集群主要由 mongos 路由进程，复制集组成的片 shards，一组配置服务器 configure 构成，下面对这些模块一一解释。

##### mongos
mongos 路由进程是一个轻量级且非持久性的进程。

轻量级表示它不会保存任何数据库中的数据，它只是将整个分片集群看成一个整体，使分片集群对整个客户端程序来说是透明的，当客户端发起读写操作时，由 mongos 路由进程将该操作路由到具体的片上进行；为了实现对读写请求的路由，mongos 进程必须知道整个分片集群上所有数据库的分片情况即元信息，这些信息是从配置服务器上同步过来的，每次进程启动时都会从 configure 服务器上读元信息，mongos 并非持久化保存这些信息。

##### shards
分片集群中的一个片 shard 实际上就是一个复制集，当然一个片也可以是单个 mongod 实例，只是在分片集群的生产环境中，每个片只是保存整个数据库数据的一部分，如果这部分数据丢失了，那么整个数据库就不完整了，因此应该保证每个片上的数据稳定性和完整性，复制集能够达到这样的要求。因此通过将片配置为复制集的形式，使片 shard 在默认情况下读写都在复制集的 primary 节点上，每个片同时具有自动故障转移、冗余备份的功能，总之复制集所具有的的特性在片上都能得到体现。

##### configure
配置服务器 configure 在整个分片集群中想到重要，上面说到 mongos 会从配置服务器同步元信息，因此配置服务器要能实现这些元信息的持久化。配置服务器上的数据如果丢失，那么整个分片集群就无法使用，因此在生产环境中通常利用三台配置服务器来实现冗余备份，这三台服务器是独立的，并不是复制集架构。

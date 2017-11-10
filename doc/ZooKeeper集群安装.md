# 配置zoo.cfg文件:
```
tickTime=2000
initLimit=10
syncLimit=5
clientPort=2181
dataDir=/export/search/zookeeper-cluster/zookeeper-3.4.6-node1/data
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
```

## tickTime=2000
tickTime这个时间是作为Zookeeper服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳；

## initLimit=10
initLimit这个配置项是用来配置Zookeeper接受客户端（这里所说的客户端不是用户连接Zookeeper服务器的客户端，而是Zookeeper服务器集群中连接到Leader的Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。
当已经超过10个心跳的时间（也就是tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 10*2000=20 秒；

## syncLimit=5
syncLimit这个配置项标识Leader与Follower之间发送消息，请求和应答时间长度，最长不能超过多少个tickTime的时间长度，总的时间长度就是5*2000=10秒；

## dataDir=/export/search/zookeeper-cluster/zookeeper-3.4.6-node1/data
dataDir顾名思义就是Zookeeper保存数据的目录，默认情况下Zookeeper将写数据的日志文件也保存在这个目录里；

## clientPort=2181
clientPort这个端口就是客户端连接Zookeeper服务器的端口，Zookeeper会监听这个端口接受客户端的访问请求；

# server.A=B:C:D
```
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
```

A是一个数字，表示这个是第几号服务器；
B是这个服务器的ip地址；
C第一个端口用来集群成员的信息交换，表示的是这个服务器与集群中的Leader服务器交换信息的端口；
D是在leader挂掉时专门用来进行选举leader所用。


# 创建ServerID标识
除了修改zoo.cfg配置文件，集群模式下还要配置一个文件myid，这个文件在dataDir目录下，这个文件里面就有一个数据就是A的值，在上面配置文件中zoo.cfg中配置的dataDir路径中创建myid文件
```
cat /export/search/zookeeper-cluster/zookeeper-3.4.6-node1/data/myid
1
```
server.X 这个数字就是对应 data/myid中的数字。你在3个server的myid文件中分别写入了1，2，3，那么每个server中的zoo.cfg都配 server.1，server.2，server.3 就OK了

# 启动zookeeper
```
bin/zkServer.sh start  
```

# 检测集群是否启动
```
bin/zkCli.sh -server IP:2181 
bin/zkCli.sh  
```
# 常用命令
```
1. 启动ZK服务:       sh bin/zkServer.sh start
2. 查看ZK服务状态: 	  sh bin/zkServer.sh status
3. 停止ZK服务:       sh bin/zkServer.sh stop
4. 重启ZK服务:       sh bin/zkServer.sh restart
```

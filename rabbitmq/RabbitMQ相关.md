### 1. 问题：
RabbitMQ服务器连不上了

### 1.1 原因：
由于打开文件最大数为默认的1024，可连接的Socket数量只有829。连接服务器的socket达到上限，导致之后试图连接的项目超时。

### 1.2 解决方案：
去rabbitMQ服务器上，调大打开文件最大数，重启 rabbitMQ 服务器生效后，连接正常。
```
参考步骤：
（1）查看文件句柄数：	ulimit -a 				
（2）打开设置：		vi /etc/profile  		
（3）加入这一行：		ulimit -n 65535  		
（4）使设置生效：		source /etc/profile 	
（5）重启RabbitMQ服务：	nohup /app/rabbitmq_server-3.6.6/sbin/rabbitmq-server start > rabbitmq.log &
```

### 2. RabbitMQ 的延迟队列
Time To Live(TTL)

Dead Letter Exchanges（DLX）

### 3. RabbitMQ 事务
RabbitMQ为我们提供了两种方式：

+ 方式一：通过AMQP事务机制实现，这也是从AMQP协议层面提供的解决方案；

+ 方式二：通过将channel设置成confirm模式来实现；

RabbitMQ中与事务机制有关的方法有三个，分别是Channel里面的txSelect()，txCommit()以及txRollback()，txSelect用于将当前Channel设置成是transaction模式，txCommit用于提交事务，txRollback用于回滚事务，在通过txSelect开启事务之后，我们便可以发布消息给broker代理服务器了，如果txCommit提交成功了，则消息一定是到达broker了，如果在txCommit执行之前broker异常奔溃或者由于其他原因抛出异常，这个时候我们便可以捕获异常通过txRollback回滚事务了；

```
1. 普通confirm模式。每发送一条消息后，调用waitForConfirms()方法，等待服务器端confirm。实际上是一种串行confirm了。
2. 批量confirm模式。每次发送一批消息后，调用waitForConfirms()方法，等待服务器端confirm。
3. 异步confirm模式。提供一个回调方法，服务器端confirm了一条(或多条)消息后SDK会回调这个方法。
```

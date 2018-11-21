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
Time To Live（TTL）

Dead Letter Exchanges（DLX）

### 3. RabbitMQ 事务
RabbitMQ为我们提供了两种方式：

（1）方式一：通过AMQP事务机制实现，这也是从AMQP协议层面提供的解决方案；

事务的实现主要是对信道（Channel）的设置，主要的方法有三个：

channel.txSelect() 声明启动事务模式；

channel.txComment() 提交事务；

channel.txRollback() 回滚事务；

（2）方式二：通过将channel设置成confirm模式来实现；

+ 普通confirm模式。每发送一条消息后，调用waitForConfirms()方法（channel.waitForConfirms() 或 channel.waitForConfirmsOrDie()），等待服务器端confirm。实际上是一种串行confirm了。

+ 批量confirm模式。每次发送一批消息后，调用waitForConfirms()（channel.waitForConfirms() 或 channel.waitForConfirmsOrDie()）方法，等待服务器端confirm。

+ 异步confirm模式。提供一个回调方法（channel.addConfirmListener()），服务器端confirm了一条(或多条)消息后SDK会回调这个方法。


### 参考资料
#### 1. RabbitMQ系列（四）RabbitMQ事务和Confirm发送方消息确认——深入解读
https://www.cnblogs.com/vipstone/p/9350075.html

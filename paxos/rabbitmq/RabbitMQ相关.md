### 1 问题：
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

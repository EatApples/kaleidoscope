### 测试了 RabbitMQ 的多地址连接
```java
Connection connection = factory.newConnection(
                new Address[] { new Address("IPA", 5672), new Address("IPB", 5672) });
```

（1）首先需要设置连接可恢复：
> factory.setAutomaticRecoveryEnabled(true);

（2）建立连接时，从 List 表中依次选取地址尝试连接，直到成功建立连接为止（或全部失败抛异常）。

```java
for (Address addr : shuffled) {
            try {
                FrameHandler frameHandler = factory.create(addr);
                RecoveryAwareAMQConnection conn = new RecoveryAwareAMQConnection(params, frameHandler, metricsCollector);
                conn.start();
                metricsCollector.newConnection(conn);
                return conn;
            } catch (IOException e) {
                lastException = e;
            }
        }

        throw (lastException != null) ? lastException : new IOException("failed to connect");
```

（3）客户端一次只与一个服务器建立连接。

（4）若建立连接的服务器 DOWN，连接恢复时仍是依次选取。

### Spring Boot项目配置RabbitMQ集群
来自：https://my.oschina.net/placeholder/blog/1612946
```yml
//具体参看了配置的源码
org.springframework.boot.autoconfigure.amqp.RabbitProperties


//RabbitMQ单机
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: your_username
    password: your_password

//或者  RabbitMQ单机，只使用addresses
spring:
  rabbitmq:
    addresses:ip1:port1
    username: your_username
    password: your_password


//RabbitMQ集群，addresses一定要逗号分隔
spring:
  rabbitmq:
    addresses:ip1:port1,ip2:port2,ip3:port3
    username: your_username
    password: your_password
```

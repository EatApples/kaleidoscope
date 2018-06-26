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

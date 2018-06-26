#### 0. 模式：默认
发送：
```java
channel.basicPublish("", ROUTING_KEY, MESSAGE);
其中 EXCHANGE_NAME为空，ROUTING_KEY值为QUEUE_NAME
简化版的direct
```
接收：
```
EXCHANGE_NAME没有bind
```
#### 1. 模式：direct
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");

channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);
```
接收：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");

channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
```


#### 2. 模式：topic
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "topic");

channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);

direct的加强版，ROUTING_KEY用(.)隔开，(*，仅匹配一个)、(#，匹配零或多个)
```
接收：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "topic");
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
```

#### 3. 模式：fanout
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
channel.basicPublish(EXCHANGE_NAME, "", MESSAGE);
topic的完全版，ROUTING_KEY为空，意味着只要绑定EXCHANGE_NAME，都能路由
```
接收：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
```

#### 4. 模式：诡异的headers
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "headers");
发送消息定义一些header
```
接收：
```java
 channel.exchangeDeclare(EXCHANGE_NAME, "headers");
 接收绑定一些header，忽略ROUTING_KEY，带有这些header的消息都会被接收
 ```

注意：不管是何种模式，对于队列来说，同一个消息经过路由规则，不管能匹配多少规则，消息只会被路由到该队列一次！而那些不能匹配规则的消息将被丢弃！

A message with a routing key will be delivered to the queue only once, even though it matches many bindings.  

If it doesn't match any binding so it will be discarded.

#### 0. 模式：默认
发送：
```java
channel.basicPublish("", ROUTING_KEY, MESSAGE);

其中 EXCHANGE_NAME 为空（默认的交换器），ROUTING_KEY 值为 QUEUE_NAME

简化版的 direct
```

接收：
```
EXCHANGE_NAME 没有 bind，也就是说 channel 不用调用 bind 函数

硬要调用，会报：ACCESS_REFUSED - operation not permitted on the default exchange
```

#### 1. 模式：direct
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");

channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);

发送端指定 ROUTING_KEY 的值，消息具体路由到哪个队列，由路由器根据匹配规则决定。

当路由器发现完全匹配的 ROUTING_KEY 时，消息就会被路由到与其绑定的相应队列，这样的队列可以有多个。

如果路由器没有发现完全匹配的 ROUTING_KEY 时，则该消息被丢弃！
```

接收：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");

channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

topic 的弱化版，ROUTING_KEY 只能用来做完全匹配，而不能模糊匹配。
```

#### 2. 模式：topic
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "topic");

channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MESSAGE);

发送端指定 ROUTING_KEY 的值，消息具体路由到哪个队列，由路由器根据匹配规则决定。

这里的 ROUTING_KEY 可以模糊匹配。

不能匹配的消息将被丢弃。

```

接收：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "topic");

channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

direct 的加强版，ROUTING_KEY 用点号(.)隔开区间，可以模糊匹配(*，仅匹配一个；#，匹配零或多个)
```

#### 3. 模式：fanout
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

channel.basicPublish(EXCHANGE_NAME, "", MESSAGE);

ROUTING_KEY 为空（即使使用了也不生效）
```

接收：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");

topic 的完全版，ROUTING_KEY 为空（即使使用了也不生效），意味着只要绑定 EXCHANGE_NAME，都能路由
```

#### 4. 模式：诡异的headers
发送：
```java
channel.exchangeDeclare(EXCHANGE_NAME, "headers");

发送消息定义一些 header，即一些KV值

ROUTING_KEY 在该模式下不做路由！
```

接收：
```java
 channel.exchangeDeclare(EXCHANGE_NAME, "headers");

 接收绑定一些 header，忽略 ROUTING_KEY，带有这些 header 的消息都会被接收
 ```

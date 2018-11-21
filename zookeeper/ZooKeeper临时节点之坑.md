
#### 1. 不处理zk的连接状态变化事件导致zk客户端断开后与zk服务器集群没有重连。后果：连接丢失后EPHEMERAL节点会删除并且客户端watch丢失。

zk客户端连接重试失败并且达到sessiontimeout时间则会收到Expired状态的连接事件，在此事件中应该由应用程序重试建立zk客户端。

CONNECTIONLOSS，不用操作

SESSIONEXPIRED，需要进行初始化操作

#### 2. 在synconnected事件中创建EPHEMERAL节点没有判断此节点是否已经存在，在已经存在的情况下没有判断是否应该删除重建，后果：EPHEMERAL节点丢失导致可用的服务器不在可用服务器列表中。
服务异常终止后立即重启，旧的临时节点还在，新的临时节点并没有建立。之后，旧的临时节点由于过期而消失，结果就是临时节点“莫名”消失。

在synconnected状态的连接事件中要同时判断sessionId是否变化以及EPHEMERAL节点是否已经存在。对sessionId发生了变化且EPHEMERAL节点已经存在的情况要先删除后重建，这个是使用Curator也避免不了的。

#### 3. 应用程序关闭时不主动关闭zk客户端，后果：导致可用服务器列表包含已经失效的服务器
PHEMERAL节点在sessiontimeout之前都存在。

#### 4. 创建一个zk客户端时，zk客户端连接zk服务器是异步的，如果在连接还没有建立时就调用zk客户端会抛异常。

正确的做法是在synconnected状态的连接事件中进行连接后的处理或者阻塞线程在连接事件中通知取消阻塞。

Curator提供了连接时同步阻塞的功能，可以避免此问题。

#### 5. 在zk的事件中执行长时间的业务
所有的zk事件都在EventThread中按顺序执行，如果一个事件长时间执行会导致其他事件无法及时响应。

#### 6. 使用2.X版本的Curator时，ExponentialBackoffRetry的maxRetries参数设置的再大都会被限制到29：MAX_RETRIES_LIMIT。
重试次数不够导致机房断网测试时zk的客户端可能永久丢失连接。

据说新版本里已经增加了ForeverRetry。

### 资料来源
#### 1. 采用zookeeper的EPHEMERAL节点机制实现服务集群的陷阱
https://yq.aliyun.com/articles/227260?spm=a2c4e.11153940.blogcont601745.12.10a76817PMcTKK

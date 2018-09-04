### ZooKeeper 连接
来自：ZooKeeper 会话超时 http://blog.51cto.com/nileader/938106

#### 1. 会话概述
在ZooKeeper中，客户端和服务端建立连接后，会话随之建立，生成一个全局唯一的会话ID(Session ID)。服务器和客户端之间维持的是一个长连接，在SESSION_TIMEOUT时间内，服务器会确定客户端是否正常连接(客户端会定时向服务器发送heart_beat，服务器重置下次SESSION_TIMEOUT时间)。

因此，在正常情况下，Session一直有效，并且ZK集群所有机器上都保存这个Session信息。

在出现网络或其它问题情况下（例如客户端所连接的那台ZK机器挂了，或是其它原因的网络闪断），客户端与当前连接的那台服务器之间连接断了，这个时候客户端会主动在地址列表（实例化ZK对象的时候传入构造方法的那个参数connectString）中选择新的地址进行连接。

#### 2. 连接断开（CONNECTIONLOSS）
在服务器与客户端之间维持会话的过程中，用户可能会看到两类异常CONNECTIONLOSS(连接断开)和SESSIONEXPIRED(Session过期)。

连接断开(CONNECTIONLOSS)一般发生在网络的闪断或是客户端所连接的服务器挂机的时候，客户端会重新选择一个Server IP尝试连接，这里主要就是从地址列表中获取一个新的Server地址进行连接：

ZK客户端捕获“连接断开”异常 ——> 获取一个新的ZK地址 ——> 尝试连接

在这个流程中，我们可以发现，整个过程不需要开发者额外的程序介入，都是ZK客户端自己会进行的，并且，使用的会话ID都是同一个，所以结论就是：发生CONNECTIONLOSS的情况，应用不需要做什么事情，等待ZK客户端建立新的连接即可。

#### How should I handle the CONNECTION_LOSS error?

Eventually, when connectivity between the client and at least one of the servers is re-established, the session will either again transition to the "connected" state (if reconnected within the session timeout value) or it will transition to the "expired" state (if reconnected after the session timeout).

The ZK client library will handle reconnect for you automatically. In particular we have heuristics built into the client library to handle things like "herd effect", etc... Only create a new session when you are notified of session expiration (mandatory).

#### 3. 会话超时（SESSIONEXPIRED）
SESSIONEXPIRED通常是ZK客户端与服务器的连接断了，试图连接上新的ZK机器，但是这个过程如果耗时过长，超过了SESSION_TIMEOUT 后还没有成功连接上服务器，那么服务器认为这个Session已经结束了（服务器无法确认是因为其它异常原因还是客户端主动结束会话）。

由于在ZK中，很多数据和状态都是和会话绑定的，一旦会话失效，那么ZK就开始清除和这个会话有关的信息，包括这个会话创建的临时节点和注册的所有Watcher。在这之后，由于网络恢复后，客户端可能会重新连接上服务器，但是很不幸，服务器会告诉客户端一个异常：SESSIONEXPIRED（会话过期）。此时客户端的状态变成 CLOSED状态，应用要做的事情就是的看自己应用的复杂程序了，要重新实例zookeeper对象，然后重新操作所有临时数据（包括临时节点和注册Watcher），总之，会话超时在ZK使用过程中是真实存在的。

所以这里也简单总结下，一旦发生会话超时，那么存储在ZK上的所有临时数据与注册的订阅者都会被移除，此时需要重新创建一个ZooKeeper客户端实例，需要自己编码做一些额外的处理。

#### How should I handle SESSION_EXPIRED?
Session expiration is managed by the ZooKeeper cluster itself, not by the client.

When the ZK client establishes a session with the cluster it provides a "timeout" value. This value is used by the cluster to determine when the client's session expires.

Expirations happens when the cluster does not hear from the client within the specified session timeout period (i.e. no heartbeat).

At session expiration the cluster will delete any/all ephemeral nodes owned by that session and immediately notify any/all connected clients of the change (anyone watching those znodes).

At this point the client of the expired session is still disconnected from the cluster, it will not be notified of the session expiration until/unless it is able to re-establish a connection to the cluster. The client will stay in disconnected state until the TCP connection is re-established with the cluster, at which point the watcher of the expired session will receive the "session expired" notification.


#### 4. 会话时间（Session Time）
在实例化一个ZK客户端的时候，需要设置一个会话的超时时间。这里需要注意的一点是，客户端并不是可以随意设置这个会话超时时间，在ZK服务器端对会话超时时间是有限制的，主要是minSessionTimeout和maxSessionTimeout这两个参数设置的。

Session超时时间限制，如果客户端设置的超时时间不在这个范围，那么会被强制设置为最大或最小时间。 默认的Session超时时间是在2 * tickTime ~ 20 * tickTime。所以，如果应用对于这个会话超时时间有特殊的需求的话，一定要和ZK管理员沟通好，确认好服务端是否设置了对会话时间的限制。

#### 问题：ZooKeeper的断线重连会对临时节点、Watcher 有哪些影响？

连接断了之后，ZK不会马上移除临时数据，只有当SESSIONEXPIRED之后，才会把这个会话建立的临时数据移除，包括这个会话创建的临时节点和注册的所有Watcher。因此，用户需要谨慎设置Session_TimeOut。

发生CONNECTIONLOSS之后，只要在session_timeout之内再次连接上（即不发生SESSIONEXPIRED），那么这个连接注册的watches依然在。

对某个节点注册了watch，但是节点被删除了，那么注册在这个节点上的watches都会失效。（显然，节点没了，触发的事件也没了。如果节点再次出现，watch 继续生效？？？）

### 扩展阅读
#### 1. ZooKeeper FAQ
https://wiki.apache.org/hadoop/ZooKeeper/FAQ

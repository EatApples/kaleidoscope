### ZooKeeper Watches
All of the read operations in ZooKeeper - getData(), getChildren(), and exists() - have the option of setting a watch as a side effect. Here is ZooKeeper's definition of a watch: a watch event is one-time trigger, sent to the client that set the watch, which occurs when the data for which the watch was set changes. There are three key points to consider in this definition of a watch:

只有3种读操作才可以设置 watch：（注释来自ZK客户端的API）
#### getData()
触发 getData 函数Watcher 的行为：删除当前节点，当前节点数据更新

sets data on the node, or deletes the node.

#### getChildren()
触发 getChildren 函数Watcher 的行为：删除当前节点，创建或删除其子节点（子节点更新也会触发？---并不会）

deletes the node of the given path or creates/delete a child under the node.

#### exists()
触发 exists 函数Watcher 的行为：创建或删除当前节点，当前节点数据更新

creates/delete the node or sets the data on the node.

#### watch 有3个特点：
#### 1. One-time trigger

One watch event will be sent to the client when the data has changed. For example, if a client does a getData("/znode1", true) and later the data for /znode1 is changed or deleted, the client will get a watch event for /znode1. If /znode1 changes again, no watch event will be sent unless the client has done another read that sets a new watch.

一个watch事件将会在数据发生变更时发送给客户端。例如，如果客户端执行操作getData(“/znode1″, true)，而后/znode1 发生变更或是删除了，客户端都会得到一个/znode1 的watch事件。如果/znode1 再次发生变更，则在客户端没有设置新的watch的情况下，是不会再给这个客户端发送watch事件的。

#### 注册只能确保一次消费

无论是服务端还是客户端，一旦一个 Watcher 被触发，ZooKeeper 都会将其从相应的存储中移除。

因此，开发人员在 Watcher 的使用上要记住的一点是需要反复注册。这样的设计有效地减轻了服务端的压力。

如果注册一个 Watcher 之后一直有效，那么针对那些更新非常频繁的节点，服务端会不断地向客户端发送事件通知，这无论对于网络还是服务端性能的影响都非常大。

#### 2. Sent to the client

This implies that an event is on the way to the client, but may not reach the client before the successful return code to the change operation reaches the client that initiated the change. Watches are sent asynchronously to watchers. ZooKeeper provides an ordering guarantee: a client will never see a change for which it has set a watch until it first sees the watch event. Network delays or other factors may cause different clients to see watches and return codes from updates at different times. The key point is that everything seen by the different clients will have a consistent order.

这就是说，一个事件会发送给客户端，但可能在操作成功的返回值到达发起变动的客户端之前，这个事件还没有送达watch的客户端。Watch是异步发送的。但ZooKeeper保证了一个顺序：一个客户端在收到watch事件之前，一定不会看到它设置过watch的值的变动。网络时延和其他因素可能会导致不同的客户端看到watch和更新返回值的时间不同。但关键点是，每个客户端所看到的每件事都是有顺序的。

#### 客户端串行执行
客户端 Watcher 回调的过程是一个串行同步的过程，这为我们保证了顺序，同时，需要开发人员注意的一点是，千万不要因为一个 Watcher 的处理逻辑影响了整个客户端的 Watcher 回调。

#### 3. The data for which the watch was set

This refers to the different ways a node can change. It helps to think of ZooKeeper as maintaining two lists of watches: data watches and child watches. getData() and exists() set data watches. getChildren() sets child watches. Alternatively, it may help to think of watches being set according to the kind of data returned. getData() and exists() return information about the data of the node, whereas getChildren() returns a list of children. Thus, setData() will trigger data watches for the znode being set (assuming the set is successful). A successful create() will trigger a data watch for the znode being created and a child watch for the parent znode. A successful delete() will trigger both a data watch and a child watch (since there can be no more children) for a znode being deleted as well as a child watch for the parent znode.

这是指节点发生变动的不同方式。你可以认为ZooKeeper维护了两个watch列表：data watch和child watch。getData()和exists()设置data watch，而getChildren()设置child watch。或者，可以认为watch是根据返回值设置的。getData()和exists()返回节点本身的信息，而getChildren()返回子节点的列表。因此，setData()会触发znode上设置的data watch（如果set成功的话）。一个成功的create() 操作会触发被创建的znode上的数据watch，以及其父节点上的child watch。而一个成功的 delete()操作将会同时触发一个znode的data watch和child watch（因为这样就没有子节点了），同时也会触发其父节点的child watch。

Watches are maintained locally at the ZooKeeper server to which the client is connected. This allows watches to be lightweight to set, maintain, and dispatch. When a client connects to a new server, the watch will be triggered for any session events. Watches will not be received while disconnected from a server. When a client reconnects, any previously registered watches will be reregistered and triggered if needed. In general this all occurs transparently. There is one case where a watch may be missed: a watch for the existence of a znode not yet created will be missed if the znode is created and deleted while disconnected.

Watch由client连接上的ZooKeeper服务器在本地维护。这样可以减小设置、维护和分发watch的开销。当一个客户端连接到一个新的服务器上时，watch将会被以任意会话事件触发。当与一个服务器失去连接的时候，是无法接收到watch的。而当client重新连接时，如果需要的话，所有先前注册过的watch，都会被重新注册。通常这是完全透明的。只有在一个特殊情况下，watch可能会丢失：对于一个未创建的znode的exist watch，如果在客户端断开连接期间被创建了，并且随后在客户端连接上之前又删除了，这种情况下，这个watch事件可能会被丢失。

#### 轻量级设计
WatchedEvent 是 ZooKeeper 整个 Watcher 通知机制的最小通知单元，这个数据结构中只包含三部分的内容：通知状态、事件类型和节点路径。也就是说，Watcher 通知非常简单，只会告诉客户端发生了事件，而不会说明事件的具体内容。

例如针对 NodeDataChanged 事件，ZooKeeper 的 Watcher 只会通知客户指定数据节点的数据内容发生了变更，而对于原始数据以及变更后的新数据都无法从这个事件中直接获取到，而是需要客户端主动重新去获取数据，这也是 ZooKeeper 的 Watcher 机制的一个非常重要的特性。

另外，客户端向服务端注册 Watcher 的时候，并不会把客户端真实的 Watcher 对象传递到服务端，仅仅只是在客户端请求中使用 boolean 类型属性进行了标记，同时服务端也仅仅只是保存了当前连接的 ServerCnxn 对象。这样轻量级的 Watcher 机制设计，在网络开销和服务端内存开销上都是非常廉价的。


Watches are maintained locally at the ZooKeeper server to which the client is connected. This allows watches to be lightweight to set, maintain, and dispatch. When a client connects to a new server, the watch will be triggered for any session events. Watches will not be received while disconnected from a server. When a client reconnects, any previously registered watches will be reregistered and triggered if needed. In general this all occurs transparently. There is one case where a watch may be missed: a watch for the existence of a znode not yet created will be missed if the znode is created and deleted while disconnected.


### What ZooKeeper Guarantees about Watches
With regard to watches, ZooKeeper maintains these guarantees:

对于watch，ZooKeeper提供了这些保障：

+ Watches are ordered with respect to other events, other watches, and asynchronous replies. The ZooKeeper client libraries ensures that everything is dispatched in order.

Watch与其他事件、其他watch以及异步回复都是有序的。ZooKeeper客户端库保证所有事件都会按顺序分发。

+ A client will see a watch event for a znode it is watching before seeing the new data that corresponds to that znode.

客户端会保障它在看到相应的znode的新数据之前接收到watch事件。

+ The order of watch events from ZooKeeper corresponds to the order of the updates as seen by the ZooKeeper service.

从ZooKeeper接收到的watch事件顺序一定和ZooKeeper服务所看到的事件顺序是一致的。

### Things to Remember about Watches

关于Watch的一些值得注意的事情

+ Watches are one time triggers; if you get a watch event and you want to get notified of future changes, you must set another watch.

Watch是一次性触发器，如果你得到了一个watch事件，而你希望在以后发生变更时继续得到通知，你应该再设置一个watch。

+ Because watches are one time triggers and there is latency between getting the event and sending a new request to get a watch you cannot reliably see every change that happens to a node in ZooKeeper. Be prepared to handle the case where the znode changes multiple times between getting the event and setting the watch again. (You may not care, but at least realize it may happen.)

因为watch是一次性触发器，而获得事件再发送一个新的设置watch的请求这一过程会有延时，所以你无法确保你看到了所有发生在ZooKeeper上的一个节点上的事件。所以请处理好在这个时间窗口中可能会发生多次znode变更的这种情况。（你可以不处理，但至少请认识到这一点）。

+ A watch object, or function/context pair, will only be triggered once for a given notification. For example, if the same watch object is registered for an exists and a getData call for the same file and that file is then deleted, the watch object would only be invoked once with the deletion notification for the file.

一个watch对象或一个函数/上下文对，为一个事件只会被通知一次。比如，如果同一个watch对象在同一个文件上分别通过exists和getData注册了两次，而这个文件之后被删除了，这时这个watch对象将只会收到一次该文件的deletion通知。

+ When you disconnect from a server (for example, when the server fails), you will not get any watches until the connection is reestablished. For this reason session events are sent to all outstanding watch handlers. Use session events to go into a safe mode: you will not be receiving events while disconnected, so your process should act conservatively in that mode.

当你从一个服务器上断开时（比如服务器出故障了），在再次连接上之前，你将无法获得任何watch。请使用这些会话事件来进入安全模式：在disconnected状态下你将不会收到事件，所以你的程序在此期间应该谨慎行事。


### Curator使用
Curator是Netflix公司一个开源的zookeeper客户端，在原生API接口上进行了包装，解决了很多ZooKeeper客户端非常底层的细节开发。同时内部实现了诸如Session超时重连，Watcher反复注册等功能，实现了Fluent风格的API接口，是使用最广泛的zookeeper客户端之一。

#### Curator提供了三种Watcher(Cache)来监听结点的变化：

Curator是Netflix公司一个开源的zookeeper客户端，在原生API接口上进行了包装，解决了很多ZooKeeper客户端非常底层的细节开发。同时内部实现了诸如Session超时重连，Watcher反复注册等功能，实现了Fluent风格的API接口，是使用最广泛的zookeeper客户端之一。

##### Path Cache
A Path Cache is used to watch a ZNode. Whenever a child is added, updated or removed, the Path Cache will change its state to contain the current set of children, the children's data and the children's state. Path caches in the Curator Framework are provided by the PathChildrenCache class. Changes to the path are passed to registered PathChildrenCacheListener instances.

对指定路径节点的一级子目录监听，不对该节点的操作监听，对其子目录的增删改操作监听。如果指定路径删除后又创建，Watcher失效。

##### Node Cache
A utility that attempts to keep the data from a node locally cached. This class will watch the node, respond to update/create/delete events, pull down the data, etc. You can register a listener that will get notified when changes occur.

对一个节点进行监听，监听事件包括指定路径的增删改操作。如果指定路径删除后又创建，Watcher继续生效。

##### Tree Cache
A utility that attempts to keep all data from all children of a ZK path locally cached. This class will watch the ZK path, respond to update/create/delete events, pull down the data, etc. You can register a listener that will get notified when changes occur.

综合NodeCache和PathChildrenCahce的特性，是对整个目录进行监听，可以设置监听深度。如果指定路径删除后又创建，Watcher继续生效。

#### Curator的一些测试
（1）测试 Curator 的 pathcache，是否多个只触发一次？

一个对象被添加多次，则触发一次。若有多个对象，则触发多次！

（2）测试 pathcache 是否随节点的消失而消失？

nodecache在节点删除后再创建，能恢复。pathcache则永远消失。

TreeCache=NodeCache+PathCache，删除后再创建，可以恢复！！！

（3）测试 CONNECTIONLOSS 与 SESSIONEXPIRED

CONNECTIONLOSS，不用操作

SESSIONEXPIRED，需要进行初始化操作

#### 发现 Zookeeper 的 bug？反正特别扯
addAuthInfo 授权操作，此处会触发当前连接的所有路径的子节点创建事件！！！

### 参考资料
#### 1. ZooKeeper Watches
http://zookeeper.apache.org/doc/r3.1.2/zookeeperProgrammers.html#ch_zkWatches

#### 2. 【Apache ZooKeeper】理解ZooKeeper中的Watches
https://blog.csdn.net/ganglia/article/details/11720737

#### 3. Caches
http://curator.apache.org/curator-recipes/index.html

#### 4. ZooKeeper Watch机制
http://danylolivia.iteye.com/blog/2341343

#### 5. Apache ZooKeeper Watcher 机制源码解释
https://www.ibm.com/developerworks/cn/opensource/os-cn-apache-zookeeper-watcher/

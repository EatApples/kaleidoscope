### Eureka的核心代码

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>Dalston.SR5</version>
    <type>pom</type>
</dependency>
```
从D版本的依赖查到依赖的 netflix 版本是 1.3.6
```
org.springframework.cloud » spring-cloud-netflix-dependencies   1.3.6.RELEASE
```
进而查到依赖的原生 eureka 版本是 1.6.2
```
com.netflix.eureka » eureka-client  1.6.2
com.netflix.eureka » eureka-core    1.6.2
com.netflix.ribbon » ribbon-eureka  2.2.2
```

### 源码重点
Eureka 是 Netflix 开源的服务注册发现组件，分成 Client 和 Server 两部分。
+ Eureka-Server ：通过 REST 协议暴露服务，提供应用服务的注册和发现的功能。

+ Application Provider ：应用服务提供者，内嵌 Eureka-Client ，通过它向 Eureka-Server 注册自身服务。

+ Application Consumer ：应用服务消费者，内嵌 Eureka-Client ，通过它从 Eureka-Server 获取服务列表。

请注意下，Application Provider 和 Application Consumer 强调扮演的角色，实际可以在同一 JVM 进程，即是服务的提供者，又是服务的消费者。

#### 1. eureka-client 项目：

+ com.netflix.appinfo 包：Eureka-Client 的应用配置。此处的应用指的就是上文提到的 Application Provider，Application Consumer
```java
/**
 * 契约过期时间，单位：秒
 */
private static final int LEASE_EXPIRATION_DURATION_SECONDS = 90;
/**
 * 租约续约频率，单位：秒。
 */
private static final int LEASE_RENEWAL_INTERVAL_SECONDS = 30;
//getHeartbeatExecutorThreadPoolSize
static final int DEFAULT_EXECUTOR_THREAD_POOL_SIZE = 5;
//getHeartbeatExecutorExponentialBackOffBound
static final int DEFAULT_EXECUTOR_THREAD_POOL_BACKOFF_BOUND = 10;

//从 Eureka-Server 拉取注册信息频率，单位：秒
public int getRegistryFetchIntervalSeconds() {
    return configInstance.getIntProperty(
            namespace + REGISTRY_REFRESH_INTERVAL_KEY, 30).get();
}

//向 Eureka-Server 同步应用信息变化初始化延迟，单位：秒
public int getInstanceInfoReplicationIntervalSeconds() {
    return configInstance.getIntProperty(
            namespace + REGISTRATION_REPLICATION_INTERVAL_KEY, 30).get();
}


```

+ com.netflix.discovery.DiscoveryClient 类：注册发现客户端实现类。

```java
/**
 * 注册信息的应用实例数
 */
private volatile int registrySize = 0;
/**
 * 最后成功从 Eureka-Server 拉取注册信息时间戳
 */
private volatile long lastSuccessfulRegistryFetchTimestamp = -1;
/**
 * 最后成功向 Eureka-Server 心跳时间戳
 */
private volatile long lastSuccessfulHeartbeatTimestamp = -1;

private boolean fetchRegistry(boolean forceFullRegistryFetch) {
  //从 Eureka-Server 获取注册信息( 根据条件判断，可能是全量，也可能是增量 )
  // 执行 全量获取
  /**
   * Gets the full registry information from the eureka server and stores it locally.
   * When applying the full registry, the following flow is observed:
   *
   * if (update generation have not advanced (due to another thread))
   *   atomically set the registry to the new registry
   * fi
   */
  getAndStoreFullRegistry();

  // 执行 增量获取

  getAndUpdateDelta(applications);
}

private void initScheduledTasks() {
  // 从 Eureka-Server 拉取注册信息执行器
  // 向 Eureka-Server 心跳（续租）执行器
  // 注册 应用实例状态变更监听器
}
```

+ com.netflix.discovery.shared 包：Eureka-Client 和 Eureka-Server 注册发现相关的共享重用的代码。
```java

```

#### 2. eureka-client-jersey2 项目：

Jersey 是 JAX-RS（JSR311）开源参考实现，用于构建 RESTful Web Service。

+ Eureka-Server 使用 Jersey Server 创建 RESTful Server 。

+ Eureka-Client 使用 Jersey Client 请求 Eureka-Server 。

#### 3. eureka-core 项目：
eureka-core 模块为 Eureka-Server 的功能实现：

+ com.netflix.eureka.EurekaBootStrap 类：Eureka-Server 启动类。

+ com.netflix.eureka.cluster 包：Eureka-Server 集群数据复制相关的代码。

+ com.netflix.eureka.lease 包：应用注册后的租约管理( 注册 / 取消 / 续期 / 过期 )。

+ com.netflix.eureka.resousrces 包：资源，基于 Jersey Server 实现，相当于 Spring MVC 的控制层代码。
```java
com.netflix.eureka.resources.ApplicationsResource，处理所有应用的请求操作的 Resource ( Controller )

应用实例注册、下线、过期时，不会很快刷新到 readWriteCacheMap 缓存里。默认配置下，最大延迟在 30 秒。
为什么可以使用缓存？在 CAP 的选择上，Eureka 选择了 AP ，不同于 Zookeeper 选择了 CP 。

```
+ com.netflix.eureka.transport 包：Eureka-Server 对 Eureka-Server 的 RESTful HTTP 客户端，基于 com.netflix.discovery.shared.transport 封装实现。

#### 4. eureka-server 项目：
eureka-server 模块，将 eureka-client + eureka-core + eureka-resources 三者打包成 Eureka-Server 的 war 包
```java
// CircularQueues here for debugging/statistics purposes only
/**
 * 最近注册的调试队列
 * key ：添加时的时间戳
 * value ：字符串 = 应用名(应用实例信息编号)
 */
private final CircularQueue<Pair<Long, String>> recentRegisteredQueue;
/**
 * 最近取消注册的调试队列
 * key ：添加时的时间戳
 * value ：字符串 = 应用名(应用实例信息编号)
 */
private final CircularQueue<Pair<Long, String>> recentCanceledQueue;
```
多节点部署的Eureka Server必然涉及到不同节点之间的注册表信息的一致性，在CAP中，Eureka 注重的满足了AP，对C只满足的弱一致性(最终一致性)，牺牲了强一致性保证了高可用性，但是Eureka Sever中依然有方式保证节点之间的注册表的信息的一致性。

register(注册)、cancel(下线)、renew(更新)、evict(剔除)，这四个方法对应了Eureka Client与Eureka Server的交互行为相对应，是对注册表信息中的服务实例的租约管理方法。

在PeerAwareInstanceRegistryImpl中，对Abstractinstanceregistry中的register()、cancel()、renew()等方法都添加了同步到PeerEurekaNode的操作，使Server集群中的注册表信息保持最终一致性。

需要在意的是Eureka Server在接受到对应的同步复制请求后如何修改自身的注册表信息，以及反馈给发起同步复制请求的Eureka Server：

+ 问题1：同步注册信息的时候，被同步的一方也同样存在相同服务实例的租约，如果被同步一方的 lastDirtyTimestamp 比较小，那么被同步一方的注册表中关于该服务实例的租约将会被覆，如果被同步的一方的 lastDirtyTimestamp 的比较大，那么租约将不会被覆盖(这部分在 AbstractInstanceRegistry.register())。但是这时发起同步的 Eureka Server 中的租约就是dirty的，该如何处理？

通过续租(心跳)同步，当 Eureka Client 与 Eureka Server 发起 renew() 请求的时候，接受 renew() 将持有最新的 lastDirtyTimestamp，通过同步心跳(续租)的方式，将该服务实例的最新 InstanceInfo 同步覆盖到 peer 节点的注册表中，维持 Server 集群注册表信息的一致性。

所以，我们发现整一个Eureka Server的集群是通过续租(心跳)的操作来维持集群的注册表信息的最终一致性，但是由于网络延迟或者波动原因，无法做到强一致性。

+ 问题2：同步续约(心跳)信息的时候，被同步一方的租约不存在或者是 lastDirtyTimestamp 比较小(被同步一方的租约是dirty)，如何处理？

如果是被同步一方 Eureka Server 的该服务实例的租约不存在或者是 lastDirtyTimestamp 比较小，那么它将在设置返回的 response status 为 404 ；发起同步的一方会将这个服务实例的信息通过同步注册的方式再次发送。在 Eureka Client 与 Eureka Server 之间的续租(心跳)就是这样一个流程。

+ 问题3：或者被同步一方的 lastDirtyTimestamp 比较大(发起同步的一方的租约是dirty)，又如何处理？

如果被同步一方 Eureka Server 的该服务实例的租约的 lastDirtyTimestamp 比较大，那么它将在设置返回的 response status 为 409，同时将本地的该服务实例的 InstanceInfo 发到 response 中；发起同步的一方会将根据 409 的状态，抽取出 response 中的 InstanceInfo，将其注册到本地注册表中。


#### 5. eureka-examples 项目：
eureka-examples 模块，提供 Eureka-Client 使用例子。

### 问题列表
#### 1. eureka client 与 eureka server 是如何通信的？
Eureka-Client 启动时，首先执行一次全量获取进行本地缓存注册信息。

Eureka-Client 在初始化过程中，创建获取注册信息线程，固定间隔向 Eureka-Server 发起获取注册信息( fetch )，刷新本地注册信息缓存。

Eureka-Client 本地应用实例与 Eureka-Server 的该应用实例状态不同的原因，因为应用实例的覆盖状态。

Eureka-Client 只会向 Eureka-Server 列表中的一个进行通信，除非该服务失效，才会选择下一个。

#### 2. eureka server 之间是如何通信的？

Eureka-Server 多节点注册信息， P2P 同步。

一个 Eureka-Server 收到 Eureka-Client 注册请求后(replication=false)，Eureka-Server 会自己模拟 Eureka-Client 发送注册请求(replication=true)到其它的 Eureka-Server。

也就是说，Eureka-Server 之间的信息同步是推模式！

#### 3. eureka server 如何保证高可用？
只要 eureka server 之间存在一条**互相可达**的链路，则它们之间能互相通信，注册信息达到最终一致性。

意思是只有两两互联，才能保证高可用。

测试用例：
+ 首先搭建2节点集群，A->A，A->B，B->B，B->A，A/B节点相互注册。
+ 现在启动C，C->A，C->B，则C向A/B注册。
+ 如果只有C->A，而没有C->B，则C只向A同步数据！也就是说即使A/B互联，C同步给A的数据，A并不会同步给B！

解释：这是合理的，考虑以下场景：5节点两两互联，共有4X5=20条通信链。
+ 当其中某个节点有更新信息时，它会同步给其他4个节点，会有4次通信。
+ 如果这4个节点再次传播，则又有4X4=16次通信！当这些信息带有时间戳时，只有时间戳大于本地时才触发更新。
+ 如果可以继续传播，则直到集群中所有信息一致，该传播才会终止。好处就是最终一致，坏处就是带来额外的通信。
+ 如果只传播一次，好处就是只有直接与之关联的节点会更新，通信次数固定，坏处就是其他可达但未直接关联的节点不会更新，集群状态不一致！考虑到 Eureka 保证 AP 而不是 CP，这种方式可以接受。

### 扩展阅读
#### 1. Eureka 源码解析 —— 应用实例注册发现（六）之全量获取
http://www.iocoder.cn/Eureka/instance-registry-fetch-all/

#### 2. 深入了解EurekaClient的注册过程
https://blog.csdn.net/weixin_40615418/article/details/78731080#itme1

#### 3. Eureka REST operations
https://github.com/Netflix/eureka/wiki/Eureka-REST-operations

#### 4. 深度剖析服务发现组件Netflix Eureka
https://blog.csdn.net/jek123456/article/details/74171039

#### 5. Spring Cloud Eureka
https://xujin.org/categories/Spring-Cloud-Eureka/

#### 6. Eureka Server之间的注册表信息同步
http://blueskykong.com/2018/02/09/eureka-instance-registry/

#### 7. Eureka 源码解析
http://www.iocoder.cn/categories/Eureka/?github

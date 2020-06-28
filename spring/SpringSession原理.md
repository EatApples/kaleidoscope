### spring-session 原理

原文地址:spring-session（一）揭秘
https://www.cnblogs.com/lxyit/p/9672097.html
作者：怀瑾握瑜

### 1. 为什么要用 spring-session？

在传统单机 web 应用中，一般使用 tomcat/jetty 等 web 容器，用户的 session 都是由容器管理。浏览器使用 cookie 记录 sessionId，容器根据 sessionId 判断用户是否存在会话 session。这里的限制是，session 存储在 web 容器中，被单台服务器容器管理。

网站逐渐演变，分布式应用和集群是趋势（提高性能），用户的请求可能被负载分发至不同的服务器。传统的 web 容器管理用户 session 的方式行不通了，除非集群或者分布式 web 应用能够共享 session。尽管 tomcat 等支持这样做。但是这样存在以下两点问题：

（1）需要侵入 web 容器，提高问题的复杂
（2）web 容器之间共享 session，集群机器之间势必要交互耦合

基于以上原因，必须提供新的可靠的分布式/集群 session 的解决方案，突破 traditional-session 单机限制（即 web 容器 session 方式，下面简称 traditional-session），spring-session 应运而生。

### 2. spring-session 核心原理

传统模式中，当 request 进入 web 容器，根据 request 获取 session 时。如果 web 容器中存在 session 则返回，如果不存在，web 容器则创建一个 session。然后返回 response 时，将 sessionId 作为 response 的 header 一并返回给客户端或者浏览器。

traditional-session 的局限性在于：单机 session。spring-session 的核心思想：将 session 从 web 容器中剥离，存储在独立的存储服务器中。

spring-session 目前支持多种形式的 session 存储器：Redis、Database、MongoDB 等。session 的管理责任委托给 spring-session 承担。当 request 进入 web 容器，根据 request 获取 session 时，由 spring-session 负责从存储器中获取 session，如果存在则返回，如果不存在则创建并持久化至存储器中。

### 3. spring-session 的实现

JSR340 是 Java Servlet 3.1 的规范提案，其中定义了大量的 api，包括：servlet、Filter、Session 等，是标准的 web 容器（如 tomcat/jetty/weblogic 等）需要遵循的规约。

在日常的应用开发中，开发者也在频繁的使用 servlet-api，比如获取请求的 session：

```
HttpServletRequest request = ...
HttpSession session = request.getSession(false);
```

其中 HttpServletRequest 和 HttpSession 都是 servlet 规范中定义的接口，web 容器实现的标准。那如果引入 spring-session，要如何获取 session？

（1）遵循 servlet 规范，同样方式获取 session，对应用代码无侵入且对于 developers 透明化
（2）全新实现一套 session 规范，定义一套新的 api 和 session 管理机制

两种方案都可以实现，但是显然第一种更友好，且具有兼容性。spring-session 正是第一种方案的实现。

实现第一种方案的关键点在于做到透明和兼容。

适配器模式：接口适配。仍然使用 HttpServletRequest 获取 session，获取到的 session 仍然是 HttpSession 类型
装饰模式：类型包装增强。Session 不能存储在 web 容器内，要外化存储

spring-session 分为以下核心模块：

```
（1）SessionRepositoryFilter：Servlet规范中Filter的实现，用来切换HttpSession至Spring Session，包装HttpServletRequest和HttpServletResponse
（2）HttpServerletRequest/HttpServletResponse/HttpSessionWrapper包装器：包装原有的HttpServletRequest、HttpServletResponse和Spring Session，实现切换Session和透明继承HttpSession的关键之所在
（3）Session：Spring Session模块
（4）SessionRepository：管理Spring Session的模块
（5）HttpSessionStrategy：映射HttpRequest和HttpResponse到Session的策略
```

### 4. 本地 MapSession 的使用

MapSession：基于 java.util.Map 的 ExpiringSession 的实现

RedisSession：基于 MapSession 和 Redis 的 ExpiringSession 实现，提供 Session 的持久化能力

在 RedisSession 中有两个非常重要的成员属性：

（1）cached：实际上是一个 MapSession 实例，用于做本地缓存，每次在 getAttribute 时无需从 Redis 中获取，主要为了 improve 性能
（2）delta：用于跟踪变化数据，做持久化

基于 MapSession 的基本映射实现的 Session，能够追踪发生变化的所有属性，当调用 saveDelta 方法后，变化的属性将被持久化！

loadSession 中，将 Redis 中存储的 Session 信息转换为 MapSession 对象，以便从 Session 中获取属性时能够从内存直接获取提高性能

### 5. RedisSession 的存储

当创建一个 RedisSession，然后存储在 Redis 中时，RedisSession 的存储细节如下：

```
spring:session:sessions:33fdd1b6-b496-4b33-9f7d-df96679d32fe
spring:session:sessions:expires:33fdd1b6-b496-4b33-9f7d-df96679d32fe
spring:session:expirations:1439245080000
```

Redis 会为每个 RedisSession 存储三个 k-v。

第一个 k-v 用来存储 Session 的详细信息，包括 Session 的过期时间间隔、最近的访问时间、attributes 等等。这个 k 的过期时间为 Session 的最大过期时间 + 5 分钟。如果默认的最大过期时间为 30 分钟，则这个 k 的过期时间为 35 分钟

第二个 k-v 用来表示 Session 在 Redis 中的过期（TTL 值），这个 k-v 不存储任何有用数据，只是表示 Session 过期而设置。这个 k 在 Redis 中的过期时间即为 Session 的过期时间间隔

第三个 k-v 存储这个 Session 的 id，是一个 Set 类型的 Redis 数据结构。这个 k 中的最后的 1439245080000 值是一个时间戳，根据这个 Session 过期时刻滚动至下一分钟而计算得出

这里不由好奇，为什么一个 RedisSession 却如此复杂的存储。关于这个可以参考 spring-session 作者本人在 github 上的两篇回答：

Why does Spring Session use spring:session:expirations?
https://github.com/spring-projects/spring-session/issues/92

Clarify Redis expirations and cleanup task
https://github.com/spring-projects/spring-session/commit/d96c8f2ce5968ced90cb43009156a61bd21853d1

简单描述下，为什么 RedisSession 的存储用到了三个 Key，而非一个 Redis 过期 Key。对于 Session 的实现，需要支持 HttpSessionEvent，即 Session 创建、过期、销毁等事件。当应用用监听器设置监听相应事件，Session 发生上述行为时，监听器能够做出相应的处理。Redis 的强大之处在于支持 KeySpace Notification（键空间通知）。即可以监视某个 key 的变化，如删除、更新、过期。当 key 发生上述行为是，以便可以接受到变化的通知做出相应的处理。

Redis 使用以下两种方式删除过期的键：
（1）当一个键被访问时，程序会对这个键进行检查，如果键已经过期，那么该键将被删除。
（2）底层系统会在后台渐进地查找并删除那些过期的键，从而处理那些已经过期、但是不会被访问到的键。
当过期键被以上两个程序的任意一个发现、 并且将键从数据库中删除时， Redis 会产生一个 expired 通知。

Redis 并不保证生存时间（TTL）变为 0 的键会立即被删除： 如果程序没有访问这个过期键， 或者带有生存时间的键非常多的话， 那么在键的生存时间变为 0 ， 直到键真正被删除这中间， 可能会有一段比较显著的时间间隔。因此， Redis 产生 expired 通知的时间为过期键被删除的时候， 而不是键的生存时间变为 0 的时候。

spring-session 为了能够及时的产生 Session 的过期时的过期事件，所以增加了：

```
spring:session:sessions:expires:33fdd1b6-b496-4b33-9f7d-df96679d32fe
spring:session:expirations:1439245080000
```

spring-session 中有个定时任务，每个整分钟都会查询相应的 `spring:session:expirations:整分钟的时间戳中的过期 SessionId`，然后再访问一次这个 SessionId，即 `spring:session:sessions:expires:SessionId`，以便能够让 Redis 及时的产生 key 过期事件——即 Session 过期事件。

### 6. 补充：Session 事件的抽象

众所周知，Servlet 规范中有对 HttpSession 的事件的处理，如：HttpSessionEvent/HttpSessionIdListener/HttpSessionListener，可以查看 javax.servlet。

在 Spring-Session 中也有相应的 Session 事件机制实现，包括 Session 创建/过期/删除事件。

Session Event 最顶层是 ApplicationEvent，即 Spring 上下文事件对象。由此可以看出 Spring-Session 的事件机制是基于 Spring 上下文事件实现。抽象的 AbstractSessionEvent 事件对象提供了获取 Session（这里的是指 Spring Session 的对象）和 SessionId。

基于事件的类型，分类为：

```
Session 创建事件
Session 删除事件
Session 过期事件
```

### 7. 补充：事件的触发机制

这里的事件触发机制只介绍基于 RedisSession 的实现。基于内存 Map 实现的 MapSession 不支持 Session 事件机制。其他的 Session 实现这里也不做关注。

Session Event 事件基于 Spring 的 ApplicationEvent 实现。先简单认识 spring 上下文事件机制：

```
ApplicationEventPublisher 实现用于发布 Spring 上下文事件 ApplicationEvent
ApplicationListener 实现用于监听 Spring 上下文事件 ApplicationEvent
ApplicationEvent 抽象上下文事件
```

那么在 Spring-Session 中必然包含事件发布者 ApplicationEventPublisher 发布 Session 事件和 ApplicationListener 监听 Session 事件。

Spring-Session 中的 RedisSession 的 SessionRepository 是 RedisOperationSessionRepository。所有关于 RedisSession 的管理操作都是由其实现，所以 Session 的产生源是 RedisOperationSessionRepository。在应用启动时由 Spring-Session 自动配置：在 spring-session-data-redis 模块中， RedisHttpSessionConfiguration 创建 RedisOperationSessionRepository Bean 时，将调用 set 方法将 ApplicationEventPublisher 配置。

对于 ApplicationListener 是由应用开发者自行实现，注册成 Bean 即可。当有 Session Event 发布时，即可监听。

以上部分探索了 Session 事件的发布者和监听者，但是核心事件的触发发布则是由 Redis 的键空间通知机制触发，当有 Session 创建/删除/过期时，Redis 键空间会通知 Spring-Session 应用。

Spring-Session 事件流程：

```
（1）Redis 键空间 Session 事件源：Session 创建通道/Session 删除通道/Session 过期通道
（2）Spring-Session 中的 RedisOperationSessionRepository 消息监听器监听 Redis 的事件类型
（3）RedisOperationSessionRepository 负责将其传播至 ApplicationEventPublisher
（4）ApplicationEventPublisher 将其包装成 ApplicationEvent 类型的 Session Event 发布
（5）ApplicationListener 监听 Session Event，处理相应逻辑
```

### 8. 总结

spring-session 提供集群环境下 HttpSession 的透明集成。spring-session 的优势在于开箱即用，具有较强的设计模式。且支持多种持久化方式，其中 RedisSession 较为成熟，与 spring-data-redis 整合，可谓威力无穷。

### 扩展阅读
从Spring Session源码看Session机制的实现细节
https://cloud.tencent.com/developer/article/1110590
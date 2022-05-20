# Apollo - 可靠的配置管理系统

Apollo 是一个可靠的配置管理系统。它可以集中管理不同应用程序和不同集群的配置。适用于微服务配置管理场景。

服务端基于 Spring Boot 和 Spring Cloud 开发，无需额外安装 Tomcat 等应用容器即可简单运行。

Java SDK 不依赖任何框架，可以在所有 Java 运行时环境中运行。它还对 Spring/Spring Boot 环境有很好的支持。

.Net SDK 不依赖任何框架，可以在所有 .Net 运行时环境中运行。

## 1. Apollo 1.9.2

### 1.1 xstream 版本升级到 14.18

### 1.2 使用方式对比 v1.9.1 版本没有变动

升级步骤为：

apollo-configservice
apollo-adminservice
apollo-portal

## 2. Apollo 1.9.1

与 v1.9.0 相比，没啥变化

## 3. Apollo 1.9.0

### 3.1 xstream 版本升级到 1.4.17

### 3.2 公共 namespace 支持多种格式，例如：json, xml, yml, yaml, txt

### 3.3 多 Apollo 管理端（portal）session 共享

### 3.4 支持 spring.config.import 方式配置加载的 namespace

示例：

```yml
# old way
# apollo.bootstrap.enabled=true
# apollo.bootstrap.namespaces=namespace1,namespace2,namespace3
# new way
spring.config.import=apollo://namespace3, apollo://namespace2, apollo://namespace1
```

### 3.5 增加 Apollo 关键字的解释

```log
apollo.accesskey.secret
apollo.autoUpdateInjectedSpringProperties
apollo.meta
apollo.bootstrap.eagerLoad.enabled
apollo.bootstrap.enabled
apollo.bootstrap.namespaces
apollo.cluster
apollo.property.order.enable
app.id
```

具体见：[spring configuration metadata](https://github.com/apolloconfig/apollo/blob/master/apollo-client/src/main/resources/META-INF/additional-spring-configuration-metadata.json)

### 3.6 数据库表结构有变化

（1）[ApolloConfigDB 的变化](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/delta/v180-v190/apolloconfigdb-v180-v190.sql)

（2）[ApolloPortalDB 的变化](https://github.com/apolloconfig/apollo/blob/master/scripts/sql/delta/v180-v190/apolloportaldb-v180-v190.sql)

将各个表中的 DataChange_CreatedBy 与 DataChange_LastModifiedBy 字段，由 32 字符扩展到 64 字符；

Users 表中增加 UserDisplayName 字段；

Users 表中 Password 字段，由 64 字符扩展到 512 字符（用于 bcrypt 加密）；

## 4. Apollo 1.8.2

升级 xstream 版本为 1.4.17

与 v1.8.1 相比，没啥变化

## 5. Apollo 1.8.1

修复启用 ldap 时 apollo portal 无法启动的问题

与 v1.8.0 相比，没啥变化

## 6. Apollo 1.8.0

### 6.1 将 AppId 的长度从 32 个字符扩展到 64 个字符

### 6.2 为配置发布添加 webhook 通知支持

### 6.3 支持自定义 server.properties 位置

### 6.4 添加 nacos 服务发现支持

### 6.5 升级 spring-boot 到 2.4.2 和 spring-cloud 到 2020.0.1

### 6.6 数据库表结构有变化

（1）[ApolloConfigDB 的变化](https://github.com/ctripcorp/apollo/blob/master/scripts/sql/delta/v170-v180/apolloconfigdb-v170-v180.sql)

（2）[ApolloPortalDB 的变化](https://github.com/ctripcorp/apollo/blob/master/scripts/sql/delta/v170-v180/apolloportaldb-v170-v180.sql)

将各个表中的 AppId 属性，由 32 字符扩展到 64 字符；

## 7. Apollo 1.7.2

升级 xstream 版本为 1.4.15

与 v1.7.1 相比，没啥变化

## 8. Apollo 1.7.1

添加对 admin 服务的访问控制支持

与 v1.7.0 相比，没啥变化

## 9. Apollo 1.7.0

与 v1.6.2 相比，没啥变化

### 9.1 支持撤销已修改但未发布的配置

### 9.2 支持回滚到指定版本

### 9.3 支持从 apollo-client 中的非规范化表达式中提取占位符，例如 ${user.address}/user/gateway

### 9.4 支持导出多个配置

## 10. Apollo 1.6.2

限制自定义使用 yaml 类型

与 v1.6.1 相比，没啥变化

## 11. Apollo 1.6.1

修复权限记录会被错误删除的问题

与 v1.6.0 相比，没啥变化

## 12. Apollo 1.6.0

### 12.1 支持按发布状态排序项目

### 12.2 添加访问密钥机制，以便只能为经过身份验证的客户端检索敏感配置

### 12.3 添加有序属性功能，以便用户可以选择保持配置顺序，如 apollo 门户中显示的那样

### 12.4 支持添加自定义环境，无需任何代码更改

### 12.5 数据库表结构有变化

（1）[ApolloConfigDB 的变化](https://github.com/ctripcorp/apollo/blob/master/scripts/sql/delta/v151-v160/apolloconfigdb-v151-v160.sql)

新增了 AccessKey 表

## 13. Apollo 1.5.1

修复 AppId 显示为 Department 问题

与 v1.5.0 相比，没啥变化

## 14. Apollo 1.5.0

与 v1.4.0 相比，没啥变化

### 14.1 增加 prometheus 集成，通过/metrics 与 /prometheus 暴露信息

### 14.2 portal 增加 i18n 支持和英文翻译

### 14.3 config 支持配置服务器端的长轮询超时

## 15. Apollo 1.4.0

与 v1.3.0 相比，没啥变化

### 15.1 支持系统环境变量 APP_ID

### 15.2 支持 apollo-configservice/apollo-adminservice/apollo-portal 与 JDK 9、10 和 11 一起运行

### 15.3 添加 TXT 文件格式支持

### 15.4 portal 支持比较集群间的配置

# 一，特性

## 1. 统一管理不同环境、不同集群的配置

Apollo 提供了一个统一的接口来集中管理不同环境、不同集群、不同命名空间的配置

相同的代码库在不同的集群中部署时可能有不同的配置

有了命名空间的概念，很容易支持多个应用程序共享相同的配置，同时也允许他们自定义配置

用户界面提供多种语言（目前为中文和英文）

支持 4 个维度管理配置（Key-Value）

• application (应用)
• environment (环境)
• cluster (集群)
• namespace (命名空间)

## 2. 配置更改实时生效（热发布）

用户修改配置并在 Apollo 发布后，sdk 会实时（1 秒）接收到最新配置并通知应用

在配置中心中，一个重要的功能就是配置发布后实时推送到客户端。下面我们简要看一下这块是怎么设计实现的。

![release-message-notification-design](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/release-message-notification-design.png)

上图简要描述了配置发布的大致过程：

1，用户在 Portal 操作配置发布

2，Portal 调用 Admin Service 的接口操作发布

3，Admin Service 发布配置后，发送 ReleaseMessage 给各个 Config Service

4，Config Service 收到 ReleaseMessage 后，通知对应的客户端

### 2.1 发送 ReleaseMessage 的实现方式

Admin Service 在配置发布后，需要通知所有的 Config Service 有配置发布，从而 Config Service 可以通知对应的客户端来拉取最新的配置。

从概念上来看，这是一个典型的消息使用场景，Admin Service 作为 producer 发出消息，各个 Config Service 作为 consumer 消费消息。通过一个消息组件（Message Queue）就能很好的实现 Admin Service 和 Config Service 的解耦。

在实现上，考虑到 Apollo 的实际使用场景，以及为了尽可能减少外部依赖，我们没有采用外部的消息中间件，而是通过数据库实现了一个简单的消息队列。

实现方式如下：

（1）Admin Service 在配置发布后会往 ReleaseMessage 表插入一条消息记录，消息内容就是配置发布的 AppId+Cluster+Namespace，参见 DatabaseMessageSender

（2）Config Service 有一个线程会每秒扫描一次 ReleaseMessage 表，看看是否有新的消息记录，参见 ReleaseMessageScanner

（3）Config Service 如果发现有新的消息记录，那么就会通知到所有的消息监听器（ReleaseMessageListener），如 NotificationControllerV2，消息监听器的注册过程参见 ConfigServiceAutoConfiguration

（4）NotificationControllerV2 得到配置发布的 AppId+Cluster+Namespace 后，会通知对应的客户端

示意图如下：![发送ReleaseMessage的实现方式](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/release-message-design.png)

### 2.2 Config Service 通知客户端的实现方式

上一节中简要描述了 NotificationControllerV2 是如何得知有配置发布的，那 NotificationControllerV2 在得知有配置发布后是如何通知到客户端的呢？

实现方式如下：

（1）客户端会发起一个 Http 请求到 Config Service 的 notifications/v2 接口，也就是 NotificationControllerV2，参见 RemoteConfigLongPollService

（2）NotificationControllerV2 不会立即返回结果，而是通过 Spring DeferredResult 把请求挂起

（3）如果在 60 秒内没有该客户端关心的配置发布，那么会返回 Http 状态码 304 给客户端

（4）如果有该客户端关心的配置发布，NotificationControllerV2 会调用 DeferredResult 的 setResult 方法，传入有配置变化的 namespace 信息，同时该请求会立即返回。客户端从返回的结果中获取到配置变化的 namespace 后，会立即请求 Config Service 获取该 namespace 的最新配置。

## 3. 发布版本管理

每个配置发布都有版本号，方便支持配置回滚

## 4. 灰度发布

支持灰度配置发布，例如点击发布后，仅对部分应用实例生效。经过一段时间的观察，如果没有问题，我们可以将配置推送到所有应用实例

## 5. 授权管理、发布审批和运营审计

针对应用和配置管理设计了强大的授权机制，配置的管理分为编辑和发布两个操作，大大减少了人为错误

所有操作都有审计日志，方便跟踪问题

## 6. 客户端配置信息监控

很容易查看哪些实例正在使用配置以及它们使用的版本

## 7. 提供丰富的 SDK

提供 Java 和.Net 原生 sdks，方便应用集成

支持 Spring Placeholder、Annotation 和 Spring Boot ConfigurationProperties，方便应用使用（需要 Spring 3.1.1+）

提供 Http API，方便非 Java 和.Net 应用集成

还提供丰富的第三方 sdk，例如 Golang、Python、NodeJS、PHP、C 等

## 8. 开放平台 API

Apollo 本身提供了统一的配置管理接口，支持多环境、多数据中心的配置管理、权限、流程治理等特性

不过为了通用性，Apollo 不会对配置的修改做太多限制，只要符合基本格式就可以保存

在我们的研究中，我们发现对于一些用户来说，他们的配置可能有比较复杂的格式，比如 xml，json，格式需要验证

还有一些用户比如 DAL，不仅有特定的格式，而且在保存之前还需要验证输入的值，比如检查数据库、用户名和密码是否匹配

对于此类应用，Apollo 允许应用通过开放的 API 修改和发布配置，内置了强大的授权和权限控制机制

## 9. 简单部署

配置中心作为一种基础设施服务，对可用性的要求非常高，这迫使 Apollo 尽可能少地依赖外部依赖

目前唯一的外部依赖是 MySQL，所以部署非常简单。Apollo 只要安装了 Java 和 MySQL 就可以运行

Apollo 还提供了打包脚本，可以一键生成所有需要的安装包，并支持自定义运行时参数

[部署架构](https://www.apolloconfig.com/#/zh/deployment/deployment-architecture)

# 二，模块

## 1. 四个核心模块及其主要功能

### 1.1 ConfigService

提供配置获取接口

提供配置推送接口

服务于 Apollo 客户端

### 1.2 AdminService

提供配置管理接口

提供配置修改发布接口

服务于管理界面 Portal

### 1.3 Client

为应用获取配置，支持实时更新

通过 MetaServer 获取 ConfigService 的服务列表

使用客户端软负载 SLB 方式调用 ConfigService

#### 1.3.1 Apollo 客户端的实现原理：

1、客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。

2、客户端还会定时从 Apollo 配置中心服务端拉取应用的最新配置。

这是一个 fallback 机制，为了防止推送机制失效导致配置不更新。

客户端定时拉取会上报本地版本，所以一般情况下，对于定时拉取的操作，服务端都会返回 304 - Not Modified。

定时频率默认为每 5 分钟拉取一次，客户端也可以通过在运行时指定 System Property: apollo.refreshInterval 来覆盖，单位为分钟。

3、客户端从 Apollo 配置中心服务端获取到应用的最新配置后，会保存在内存中

4、客户端会把从服务端获取到的配置在本地文件系统缓存一份

在遇到服务不可用，或网络不通的时候，依然能从本地恢复配置

5、应用程序从 Apollo 客户端获取最新的配置、订阅配置更新通知

#### 1.3.2 配置更新推送实现

前面提到了 Apollo 客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。

长连接实际上我们是通过 Http Long Polling 实现的，具体而言：

1、客户端发起一个 Http 请求到服务端

2、服务端会保持住这个连接 30 秒

如果在 30 秒内有客户端关心的配置变化，被保持住的客户端请求会立即返回，并告知客户端有配置变化的 namespace 信息，客户端会据此拉取对应 namespace 的最新配置

如果在 30 秒内没有客户端关心的配置变化，那么会返回 Http 状态码 304 给客户端

3、客户端在收到服务端请求后会立即重新发起连接，回到第一步考虑到会有数万客户端向服务端发起长连，在服务端我们使用了 async servlet(Spring DeferredResult) 来服务 Http Long Polling 请求。

### 1.4 Portal

配置管理界面

通过 MetaServer 获取 AdminService 的服务列表

使用客户端软负载 SLB 方式调用 AdminService

## 2. 三个辅助服务发现模块

### 2.1 Eureka

用于服务发现和注册

Config/AdminService 注册实例并定期报心跳

和 ConfigService 住在一起部署

为什么选择 Eureka

为什么我们采用 Eureka 作为服务注册中心，而不是使用传统的 zk、etcd 呢？我大致总结了一下，有以下几方面的原因：

（1）它提供了完整的 Service Registry 和 Service Discovery 实现首先是提供了完整的实现，并且也经受住了 Netflix 自己的生产环境考验，相对使用起来会比较省心

（2）和 Spring Cloud 无缝集成 我们的项目本身就使用了 Spring Cloud 和 Spring Boot，同时 Spring Cloud 还有一套非常完善的开源代码来整合 Eureka，所以使用起来非常方便。

另外，Eureka 还支持在我们应用自身的容器中启动，也就是说我们的应用启动完之后，既充当了 Eureka 的角色，同时也是服务的提供者。这样就极大的提高了服务的可用性。

这一点是我们选择 Eureka 而不是 zk、etcd 等的主要原因，为了提高配置中心的可用性和降低部署复杂度，我们需要尽可能地减少外部依赖。

（3）最后一点是开源，由于代码是开源的，所以非常便于我们了解它的实现原理和排查问题。

### 2.2 MetaServer

Portal 通过域名访问 MetaServer 获取 AdminService 的地址列表

Client 通过域名访问 MetaServer 获取 ConfigService 的地址列表

相当于一个 Eureka Proxy

逻辑角色，和 ConfigService 住在一起部署

### 2.3 NginxLB

和域名系统配合，协助 Portal 访问 MetaServer 获取 AdminService 地址列表

和域名系统配合，协助 Client 访问 MetaServer 获取 ConfigService 地址列表

和域名系统配合，协助用户访问 Portal 进行配置管理

# 三，总体设计

![Apollo架构模块的概览](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/overall-architecture.png)

我们可以从下往上看：

Config Service 提供配置的读取、推送等功能，服务对象是 Apollo 客户端

Admin Service 提供配置的修改、发布等功能，服务对象是 Apollo Portal（管理界面）

Config Service 和 Admin Service 都是多实例、无状态部署，所以需要将自己注册到 Eureka 中并保持心跳
在 Eureka 之上我们架了一层 Meta Server 用于封装 Eureka 的服务发现接口

Client 通过域名访问 Meta Server 获取 Config Service 服务列表（IP+Port），而后直接通过 IP+Port 访问服务，同时在 Client 侧会做 load balance、错误重试

Portal 通过域名访问 Meta Server 获取 Admin Service 服务列表（IP+Port），而后直接通过 IP+Port 访问服务，同时在 Portal 侧会做 load balance、错误重试

为了简化部署，我们实际上会把 Config Service、Eureka 和 Meta Server 三个逻辑角色部署在同一个 JVM 进程中

## 1. 服务端高可用

![服务端高可用](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/overall-architecture.png)

首先最下面是一个 DB，我们的配置是放在 DB 里的，然后在 DB 之上有两个服务：Config Service 和 Admin Service；

Config Service 提供配置的读取、推送等功能，服务对象是 Apollo 客户端；

Admin Service 提供配置的修改、发布等功能，服务对象是 Apollo Portal（管理界面）；

Config Service 和 Admin Service 都是多实例、无状态部署，所以需要将自己注册到 Eureka 中并保持心跳；

在 Eureka 之上我们架了一层 Meta Server 用于封装 Eureka 的服务发现接口，主要是为了让客户端和 Eureka 解耦；

Client 通过域名访问 Meta Server 获取 Config Service 服务列表（IP+Port），而后直接通过 IP+Port 访问服务，同时在 Client 侧会做 load balance、错误重试；

Portal 通过域名访问 Meta Server 获取 Admin Service 服务列表（IP+Port），而后直接通过 IP+Port 访问服务，同时在 Portal 侧会做 load balance、错误重试；

为了简化部署，我们实际上会把 Config Service、Eureka 和 Meta Server 三个逻辑角色部署在同一个 JVM 进程中；

通过上述的设计，可以看到整个服务端是无单点，有效地保证了服务端的可用性。

## 2. 客户端高可用

![客户端高可用](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/client-architecture.png)

客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。（通过 Http Long Polling 实现）；

客户端还会定时从 Apollo 配置中心服务端拉取应用的最新配置；

这是一个 fallback 机制，为了防止推送机制失效导致配置不更新；

客户端定时拉取会上报本地版本，所以一般情况下，对于定时拉取的操作，服务端都会返回 304 - Not Modified。

客户端从 Apollo 配置中心服务端获取到应用的最新配置后，会保存在内存中，所以我们的应用程序来获取配置的时候其实始终是从内存中获取的；

客户端还会把从服务端获取到的配置在本地文件系统缓存一份；

这主要是为了容灾，假设应用程序重启的时候，恰好远端服务全挂了，或者网络有故障，应用程序依然能从本地恢复配置。

通过这种推拉结合的机制，以及内存和本地文件双缓存的方式，有效地保证了客户端的可用性。

## 3. 可用性考虑

| 场景                     | 影响                                  | 降级                                                                                                                                                        | 原因                                                                                       |
| ------------------------ | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 某台 Config Service 下线 | 无影响                                |                                                                                                                                                             | Config Service 无状态，客户端重连其它 Config Service                                       |
| 所有 Config Service 下线 | 客户端无法读取最新配置，Portal 无影响 | 客户端重启时，可以读取本地缓存配置文件。如果是新扩容的机器，可以从其它机器上获取已缓存的配置文件，具体信息可以参考 Java 客户端使用指南 - 1.2.3 本地缓存路径 |                                                                                            |
| 某台 Admin Service 下线  | 无影响                                |                                                                                                                                                             | Admin Service 无状态，Portal 重连其它 Admin Service                                        |
| 所有 Admin Service 下线  | 客户端无影响，Portal 无法更新配置     |                                                                                                                                                             |                                                                                            |
| 某台 Portal 下线         | 无影响                                |                                                                                                                                                             | Portal 域名通过 SLB 绑定多台服务器，重试后指向可用的服务器                                 |
| 全部 Portal 下线         | 客户端无影响，Portal 无法更新配置     |                                                                                                                                                             |
| 某个数据中心下线         | 无影响                                |                                                                                                                                                             | 多数据中心部署，数据完全同步，Meta Server/Portal 域名通过 SLB 自动切换到其它存活的数据中心 |
| 数据库宕机               | 客户端无影响，Portal 无法更新配置     | Config Service 开启配置缓存后，对配置的读取不受数据库宕机影响                                                                                               |                                                                                            |

# 四，异常排查

## 1. eureka 注册失败的信息

如果启动遇到了异常，可以分别查看 service 和 portal 目录下的 log 文件排查问题。

注：在启动 apollo-configservice 的过程中会在日志中输出 eureka 注册失败的信息，如

```java
com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused。
```

需要注意的是，这个是预期的情况，因为 apollo-configservice 需要向 Meta Server（它自己）注册服务，但是因为在启动过程中，自己还没起来，所以会报这个错。后面会进行重试的动作，所以等自己服务起来后就会注册正常了。

如果提示系统出错，请重试或联系系统负责人，请稍后几秒钟重试一下，因为通过 Eureka 注册的服务有一个刷新的延时。

# 五，资料来源

### 1. [开源配置中心 Apollo 的设计与实现](https://www.infoq.cn/article/open-source-configuration-center-apollo)

### 2. [design-and-implementation-of-apollo.pdf](https://github.com/apolloconfig/apollo-community/blob/master/slides/design-and-implementation-of-apollo.pdf)

### 3. [GitHub 9K Star！Apollo 作者手把手教你微服务配置中心之道](https://mp.weixin.qq.com/s/iDmYJre_ULEIxuliu1EbIQ)

### 4. [configuration-center-makes-microservices-smart.pdf](https://github.com/apolloconfig/apollo-community/blob/master/slides/configuration-center-makes-microservices-smart.pdf)

### 5. [微服务架构~携程 Apollo 配置中心架构剖析](https://mp.weixin.qq.com/s/-hUaQPzfsl9Lm3IqQW3VDQ)

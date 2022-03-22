### 1. 命名空间的作用

[Apollo 核心概念之“Namespace”](https://www.apolloconfig.com/#/zh/design/apollo-core-concept-namespace)

很容易支持多个应用程序共享相同的配置，同时也允许他们自定义配置。

### 2. 配置的加载次序

[和 Spring 集成的原理](https://www.apolloconfig.com/#/zh/design/apollo-design?id=_31-%e5%92%8cspring%e9%9b%86%e6%88%90%e7%9a%84%e5%8e%9f%e7%90%86)

Spring 从 3.1 版本开始增加了 ConfigurableEnvironment 和 PropertySource：

（1）ConfigurableEnvironment

- Spring 的 ApplicationContext 会包含一个 Environment（实现 ConfigurableEnvironment 接口）

- ConfigurableEnvironment 自身包含了很多个 PropertySource

（2）PropertySource

- 属性源

- 可以理解为很多个 Key - Value 的属性配置

在运行时的结构形如：

![](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/environment.png)

需要注意的是，PropertySource 之间是有优先级顺序的，如果有一个 Key 在多个 property source 中都存在，那么在前面的 property source 优先。

所以对上图的例子：

```java
env.getProperty(“key1”) -> value1
env.getProperty(“key2”) -> value2
env.getProperty(“key3”) -> value4
```

在理解了上述原理后，Apollo 和 Spring/Spring Boot 集成的手段就呼之欲出了：在应用启动阶段，Apollo 从远端获取配置，然后组装成 PropertySource 并插入到第一个即可，如下图所示：

![](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/environment-remote-source.png)

相关代码可以参考[PropertySourcesProcessor](https://github.com/apolloconfig/apollo/blob/master/apollo-client/src/main/java/com/ctrip/framework/apollo/spring/config/PropertySourcesProcessor.java)

### 3. 配置更改如何热生效

![配置发布后的实时推送设计](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/release-message-notification-design.png)

[发送 ReleaseMessage 的实现方式](https://www.apolloconfig.com/#/zh/design/apollo-design?id=_211-%e5%8f%91%e9%80%81releasemessage%e7%9a%84%e5%ae%9e%e7%8e%b0%e6%96%b9%e5%bc%8f)

sdk 会实时（1 秒）接收到最新配置并通知应用

Admin Service 在配置发布后，需要通知所有的 Config Service 有配置发布，从而 Config Service 可以通知对应的客户端来拉取最新的配置。

从概念上来看，这是一个典型的消息使用场景，Admin Service 作为 producer 发出消息，各个 Config Service 作为 consumer 消费消息。通过一个消息组件（Message Queue）就能很好的实现 Admin Service 和 Config Service 的解耦。

在实现上，考虑到 Apollo 的实际使用场景，以及为了尽可能减少外部依赖，我们没有采用外部的消息中间件，而是通过数据库实现了一个简单的消息队列。

实现方式如下：

![](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/release-message-design.png)

Admin Service 在配置发布后会往 ReleaseMessage 表插入一条消息记录，消息内容就是配置发布的 AppId+Cluster+Namespace

Config Service 有一个线程会每秒扫描一次 ReleaseMessage 表，看看是否有新的消息记录

Config Service 如果发现有新的消息记录，解析得到配置发布的 AppId+Cluster+Namespace 后，通知到对应的客户端

[Config Service 通知客户端的实现方式](https://www.apolloconfig.com/#/zh/design/apollo-design?id=_211-%e5%8f%91%e9%80%81releasemessage%e7%9a%84%e5%ae%9e%e7%8e%b0%e6%96%b9%e5%bc%8f)

实现方式如下：

（1）客户端会发起一个 Http 请求到 Config Service 的 notifications/v2 接口，也就是[NotificationControllerV2](https://github.com/apolloconfig/apollo/blob/master/apollo-configservice/src/main/java/com/ctrip/framework/apollo/configservice/controller/NotificationControllerV2.java)，参见[RemoteConfigLongPollService](https://github.com/apolloconfig/apollo/blob/master/apollo-client/src/main/java/com/ctrip/framework/apollo/internals/RemoteConfigLongPollService.java)

（2）NotificationControllerV2 不会立即返回结果，而是通过[Spring DeferredResult](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/context/request/async/DeferredResult.html)把请求挂起

（3）如果在 60 秒内没有该客户端关心的配置发布，那么会返回 Http 状态码 304 给客户端

（4）如果有该客户端关心的配置发布，NotificationControllerV2 会调用 DeferredResult 的 setResult 方法，传入有配置变化的 namespace 信息，同时该请求会立即返回。客户端从返回的结果中获取到配置变化的 namespace 后，会立即请求 Config Service 获取该 namespace 的最新配置。

### 4. 配置回滚的原理

[回滚已发布配置](https://www.apolloconfig.com/#/zh/usage/apollo-user-guide?id=_16-%e5%9b%9e%e6%bb%9a%e5%b7%b2%e5%8f%91%e5%b8%83%e9%85%8d%e7%bd%ae)

发布已经回滚到之前的版本了，但是已经被编辑的内容还是待发布状态，这个取消不了了。

### 5. 灰度发布原理

[灰度发布使用指南](https://www.apolloconfig.com/#/zh/usage/apollo-user-guide?id=%e4%ba%94%e3%80%81%e7%81%b0%e5%ba%a6%e5%8f%91%e5%b8%83%e4%bd%bf%e7%94%a8%e6%8c%87%e5%8d%97)

通过灰度发布功能，可以实现：

对于一些对程序有比较大影响的配置，可以先在一个或者多个实例生效，观察一段时间没问题后再全量发布配置。

对于一些需要调优的配置参数，可以通过灰度发布功能来实现 A/B 测试。可以在不同的机器上应用不同的配置，不断调整、测评一段时间后找出较优的配置再全量发布配置。

### 6. 审计日志记录哪些信息

[建表语句](https://github.com/apolloconfig/apollo/tree/master/scripts/sql)

审计表建表语句如下：

```sql
CREATE TABLE `Audit` (
  `Id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `EntityName` varchar(50) NOT NULL DEFAULT 'default' COMMENT '表名',
  `EntityId` int(10) unsigned DEFAULT NULL COMMENT '记录ID',
  `OpName` varchar(50) NOT NULL DEFAULT 'default' COMMENT '操作类型',
  `Comment` varchar(500) DEFAULT NULL COMMENT '备注',
  `IsDeleted` bit(1) NOT NULL DEFAULT b'0' COMMENT '1: deleted, 0: normal',
  `DataChange_CreatedBy` varchar(32) NOT NULL DEFAULT 'default' COMMENT '创建人邮箱前缀',
  `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `DataChange_LastModifiedBy` varchar(32) DEFAULT '' COMMENT '最后修改人邮箱前缀',
  `DataChange_LastTime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
  PRIMARY KEY (`Id`),
  KEY `DataChange_LastTime` (`DataChange_LastTime`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='日志审计表';
```

审计信息，记录用户在何时使用何种方式操作了哪个实体。

### 7. 服务端展示客户端使用情况的原理

[建表语句](https://github.com/apolloconfig/apollo/tree/master/scripts/sql)

使用配置的应用实例的建表语句如下：

```sql
CREATE TABLE `Instance` (
  `Id` int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增Id',
  `AppId` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'AppID',
  `ClusterName` varchar(32) NOT NULL DEFAULT 'default' COMMENT 'ClusterName',
  `DataCenter` varchar(64) NOT NULL DEFAULT 'default' COMMENT 'Data Center Name',
  `Ip` varchar(32) NOT NULL DEFAULT '' COMMENT 'instance ip',
  `DataChange_CreatedTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `DataChange_LastTime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
  PRIMARY KEY (`Id`),
  UNIQUE KEY `IX_UNIQUE_KEY` (`AppId`,`ClusterName`,`Ip`,`DataCenter`),
  KEY `IX_IP` (`Ip`),
  KEY `IX_DataChange_LastTime` (`DataChange_LastTime`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='使用配置的应用实例';
```

很容易查看哪些实例正在使用配置以及它们使用的版本

### 8. 支持 Spring Placeholder、Annotation 和 Spring Boot ConfigurationProperties 的异同

[Spring 整合方式](https://www.apolloconfig.com/#/zh/usage/java-sdk-user-guide?id=_32-spring%e6%95%b4%e5%90%88%e6%96%b9%e5%bc%8f)

（1）Spring 应用通常会使用 Placeholder 来注入配置，使用的格式形如${someKey:someDefaultValue}，如${timeout:100}。冒号前面的是 key，冒号后面的是默认值。

建议在实际使用时尽量给出默认值，以免由于 key 没有定义导致运行时错误。

Placeholder 方式：

- 代码中直接使用，如：@Value("${someKeyFromApollo:someDefaultValue}")
- 配置文件中使用替换 placeholder，如：spring.datasource.url: ${someKeyFromApollo:someDefaultValue}
- 直接托管 spring 的配置，如在 apollo 中直接配置 spring.datasource.url=jdbc:mysql://localhost:3306/somedb?characterEncoding=utf8

（2）Apollo 同时还增加了几个新的 Annotation 来简化在 Spring 环境中的使用。

- @ApolloConfig

用来自动注入 Config 对象

- @ApolloConfigChangeListener

用来自动注册 ConfigChangeListener

- @ApolloJsonValue

用来把配置的 json 字符串自动注入为对象

（3）Spring boot 的@ConfigurationProperties 方式

需要注意的是，@ConfigurationProperties 如果需要在 Apollo 配置变化时自动更新注入的值，需要配合使用 EnvironmentChangeEvent 或 RefreshScope。

### 9. 开放平台 API 有哪些，如何接入

[Apollo 开放平台接入指南](https://www.apolloconfig.com/#/zh/usage/apollo-open-api-platform)

Apollo 允许应用通过开放的 API 修改和发布配置，内置了强大的授权和权限控制机制

### 10. 依赖的 Mysql 数据库有哪些表，各有什么作用

[主体 E-R Diagram](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/apollo-erd.png)

- App

App 信息

- AppNamespace

App 下 Namespace 的元信息

- Cluster

集群信息

- Namespace

集群下的 namespace

- Item

Namespace 的配置，每个 Item 是一个 key, value 组合

- Release

Namespace 发布的配置，每个发布包含发布时该 Namespace 的所有配置

- Commit

Namespace 下的配置更改记录

- Audit

审计信息，记录用户在何时使用何种方式操作了哪个实体。

[权限相关 E-R Diagram](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/apollo-erd-role-permission.png)

- User

Apollo portal 用户

- UserRole

用户和角色的关系

- Role

角色

- RolePermission

角色和权限的关系

- Permission

权限

对应到具体的实体资源和操作，如修改 NamespaceA 的配置，发布 NamespaceB 的配置等。

- Consumer

第三方应用

- ConsumerToken

发给第三方应用的 token

- ConsumerRole

第三方应用和角色的关系

- ConsumerAudit

第三方应用访问审计

### 11. Apollo 客户端的实现原理

![](https://cdn.jsdelivr.net/gh/apolloconfig/apollo@master/doc/images/client-architecture.png)

（1）客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。（通过 Http Long Polling 实现）

（2）客户端还会定时从 Apollo 配置中心服务端拉取应用的最新配置。

这是一个 fallback 机制，为了防止推送机制失效导致配置不更新

客户端定时拉取会上报本地版本，所以一般情况下，对于定时拉取的操作，服务端都会返回 304 - Not Modified

定时频率默认为每 5 分钟拉取一次，客户端也可以通过在运行时指定 System Property: apollo.refreshInterval 来覆盖，单位为分钟。

（3）客户端从 Apollo 配置中心服务端获取到应用的最新配置后，会保存在内存中

（4）客户端会把从服务端获取到的配置在本地文件系统缓存一份

在遇到服务不可用，或网络不通的时候，依然能从本地恢复配置

（5）应用程序可以从 Apollo 客户端获取最新的配置、订阅配置更新通知

### 12. springboot 启动如何拉取 apollo 的公共配置项以及应用的关联配置项

[需求场景](https://github.com/ctripcorp/apollo/issues/1680)

假设公共 namespace 有 public1 和 public2，那么设置

```yml
apollo.bootstrap.namespaces=application,public1,public2
```

即可。

启动客户端项目的时候，客户端启动加载的配置是根据 apollo.bootstrap.namespaces 的配置按顺序加载，且以第一个为准（同名的 key 不会被覆盖）。

### 13. 从 Apollo 读取配置

[文档链接](https://www.apolloconfig.com/#/zh/usage/other-language-client-user-guide)

（1）通过带缓存的 Http 接口从 Apollo 读取配置

该接口会从缓存中获取配置，适合频率较高的配置拉取请求，如简单的每 30 秒轮询一次配置。

由于缓存最多会有一秒的延时，所以如果需要配合配置推送通知实现实时更新配置的话，请参考通过不带缓存的 Http 接口从 Apollo 读取配置。

URL: {config_server_url}/configfiles/json/{appId}/{clusterName}/{namespaceName}?ip={clientIp}
Method: GET

参数说明：

| 参数名            | 是否必须 | 参数值                | 备注                               |
| ----------------- | -------- | --------------------- | ---------------------------------- |
| config_server_url | 是       | Apollo 配置服务的地址 | 是 meta 地址，不是管理端地址       |
| appId             | 是       | 应用的 appId          |                                    |
| clusterName       | 是       | 集群名                | 传入 default 即可                  |
| namespaceName     | 是       | Namespace 的名字      | 传入 application 即可              |
| ip                | 否       | 应用部署的机器 ip     | 这个参数是可选的，用来实现灰度发布 |

（2）通过不带缓存的 Http 接口从 Apollo 读取配置

该接口会直接从数据库中获取配置，可以配合配置推送通知实现实时更新配置。

URL: {config_server_url}/configs/{appId}/{clusterName}/{namespaceName}?releaseKey={releaseKey}&ip={clientIp}
Method: GET

参数说明：

| 参数名            | 是否必须 | 参数值                | 备注                                                          |
| ----------------- | -------- | --------------------- | ------------------------------------------------------------- |
| config_server_url | 是       | Apollo 配置服务的地址 | 是 meta 地址，不是管理端地址                                  |
| appId             | 是       | 应用的 appId          |
| clusterName       | 是       | 集群名                | 传入 default 即可                                             |
| namespaceName     | 是       | Namespace 的名字      | 如传入 application 即可                                       |
| releaseKey        | 否       | 上一次的 releaseKey   | 如果版本比下来没有变化，则服务端直接返回 304 以节省流量和运算 |
| ip                | 否       | 应用部署的机器 ip     | 这个参数是可选的，用来实现灰度发布                            |

### 14. AppId 的设置

[AppId](https://www.apolloconfig.com/#/zh/usage/java-sdk-user-guide?id=_121-appid)

AppId 是应用的身份信息，是从服务端获取配置的一个重要信息。

有以下几种方式设置，按照优先级从高到低分别为：

（1）System Property
Apollo 0.7.0+支持通过 System Property 传入 app.id 信息，如

```s
-Dapp.id=YOUR-APP-ID
```

（2）操作系统的 System Environment
Apollo 1.4.0+支持通过操作系统的 System Environment APP_ID 来传入 app.id 信息，如

```s
APP_ID=YOUR-APP-ID
```

（3）Spring Boot application.properties

Apollo 1.0.0+支持通过 Spring Boot 的 application.properties 文件配置，如

```yml
app.id=YOUR-APP-ID
```

该配置方式不适用于多个 war 包部署在同一个 tomcat 的使用场景

（4）app.properties

确保 classpath:/META-INF/app.properties 文件存在，并且其中内容形如：

```yml
app.id=YOUR-APP-ID
```

注：app.id 是用来标识应用身份的唯一 id，格式为 string。

### 15. Apollo Meta Server

[Apollo Meta Server](https://www.apolloconfig.com/#/zh/usage/java-sdk-user-guide?id=_122-apollo-meta-server)

Apollo 支持应用在不同的环境有不同的配置，所以需要在运行提供给 Apollo 客户端当前环境的 Apollo Meta Server 信息。

默认情况下，meta server 和 config service 是部署在同一个 JVM 进程，所以 meta server 的地址就是 config service 的地址。

为了实现 meta server 的高可用，推荐通过 SLB（Software Load Balancer）做动态负载均衡。Meta server 地址也可以填入 IP，如http://1.1.1.1:8080,http://2.2.2.2:8080，不过生产环境还是建议使用域名（走slb），因为机器扩容、缩容等都可能导致IP列表的变化。

1.0.0 版本开始支持以下方式配置 apollo meta server 信息，按照优先级从高到低分别为：

（1）通过 Java System Property apollo.meta

可以通过 Java 的 System Property apollo.meta 来指定

在 Java 程序启动脚本中，可以指定

```s
-Dapollo.meta=http://config-service-url
```

如果是运行 jar 文件，需要注意格式是

```s
java -Dapollo.meta=http://config-service-url -jar xxx.jar
```

也可以通过程序指定，如

```java
System.setProperty("apollo.meta", "http://config-service-url");
```

（2）通过 Spring Boot 的配置文件

可以在 Spring Boot 的 application.properties 或 bootstrap.properties 中指定

```yml
apollo.meta=http://config-service-url
```

该配置方式不适用于多个 war 包部署在同一个 tomcat 的使用场景

（3）通过操作系统的 System Environment `APOLLO_META`

可以通过操作系统的 System Environment APOLLO_META 来指定

注意 key 为全大写，且中间是`_`分隔

（4）通过 server.properties 配置文件

可以在 server.properties 配置文件中指定

```yml
apollo.meta=http://config-service-url
```

对于 Mac/Linux，文件位置为/opt/settings/server.properties
对于 Windows，文件位置为 C:\opt\settings\server.properties

（5）通过 app.properties 配置文件

可以在 classpath:/META-INF/app.properties 指定

```s
apollo.meta=http://config-service-url
```

（6）通过 Java system property `${env}_meta`

如果当前 env 是 dev，那么用户可以配置

```s
-Ddev_meta=http://config-service-url
```

使用该配置方式，那么就必须要正确配置 Environment

（7）通过操作系统的 System Environment `${ENV}_META` (1.2.0 版本开始支持)

如果当前 env 是 dev，那么用户可以配置操作系统的 System Environment

```s
DEV_META=http://config-service-url
```

注意 key 为全大写

使用该配置方式，那么就必须要正确配置 Environment

（8）通过 apollo-env.properties 文件

用户也可以创建一个 apollo-env.properties，放在程序的 classpath 下，或者放在 spring boot 应用的 config 目录下

使用该配置方式，那么就必须要正确配置 Environment

文件内容形如：

```yml
dev.meta=http://1.1.1.1:8080
fat.meta=http://apollo.fat.xxx.com
uat.meta=http://apollo.uat.xxx.com
pro.meta=http://apollo.xxx.com
```

如果通过以上各种手段都无法获取到 Meta Server 地址，Apollo 最终会 fallback 到http://apollo.meta作为Meta Server 地址

### 16. 配置查看权限

[配置查看权限](https://www.apolloconfig.com/#/zh/usage/apollo-user-guide?id=_61-%e9%85%8d%e7%bd%ae%e6%9f%a5%e7%9c%8b%e6%9d%83%e9%99%90)

从 1.1.0 版本开始，apollo-portal 增加了查看权限的支持，可以支持配置某个环境只允许项目成员查看私有 Namespace 的配置。

这里的项目成员是指：

- 项目的管理员
- 具备该私有 Namespace 在该环境下的修改或发布权限

配置方式很简单，用超级管理员账号登录后，进入管理员工具 - 系统参数页面新增或修改 configView.memberOnly.envs 配置项即可。

### 17. 配置访问密钥

[配置访问密钥](https://www.apolloconfig.com/#/zh/usage/apollo-user-guide?id=_62-%e9%85%8d%e7%bd%ae%e8%ae%bf%e9%97%ae%e5%af%86%e9%92%a5)

Apollo 从 1.6.0 版本开始增加访问密钥机制，从而只有经过身份验证的客户端才能访问敏感配置。如果应用开启了访问密钥，客户端需要配置密钥，否则无法获取配置。

配置方式按照优先级从高到低分别为：

1，通过 Java System Property apollo.access-key.secret(1.9.0+) 或者 apollo.accesskey.secret(1.9.0 之前)

可以通过 Java 的 System Property apollo.access-key.secret(1.9.0+) 或者 apollo.accesskey.secret(1.9.0 之前)来指定

在 Java 程序启动脚本中，可以指定-Dapollo.access-key.secret=1cf998c4e2ad4704b45a98a509d15719(1.9.0+) 或者 -Dapollo.accesskey.secret=1cf998c4e2ad4704b45a98a509d15719(1.9.0 之前)

如果是运行 jar 文件，需要注意格式是 java -Dapollo.access-key.secret=1cf998c4e2ad4704b45a98a509d15719 -jar xxx.jar(1.9.0+) 或者 java -Dapollo.accesskey.secret=1cf998c4e2ad4704b45a98a509d15719 -jar xxx.jar(1.9.0 之前)

也可以通过程序指定，如 System.setProperty("apollo.access-key.secret", "1cf998c4e2ad4704b45a98a509d15719");(1.9.0+) 或者 System.setProperty("apollo.accesskey.secret", "1cf998c4e2ad4704b45a98a509d15719");(1.9.0 之前)

2，通过 Spring Boot 的配置文件

可以在 Spring Boot 的 application.properties 或 bootstrap.properties 中指定 apollo.access-key.secret=1cf998c4e2ad4704b45a98a509d15719(1.9.0+) 或者 apollo.accesskey.secret=1cf998c4e2ad4704b45a98a509d15719(1.9.0 之前)

3，通过操作系统的 System Environment

还可以通过操作系统的 System Environment APOLLO_ACCESS_KEY_SECRET(1.9.0+) 或者 APOLLO_ACCESSKEY_SECRET(1.9.0 之前)来指定

注意 key 为全大写

4，通过 app.properties 配置文件

可以在 classpath:/META-INF/app.properties 指定 apollo.access-key.secret=1cf998c4e2ad4704b45a98a509d15719(1.9.0+) 或者 apollo.accesskey.secret=1cf998c4e2ad4704b45a98a509d15719(1.9.0 之前)

### 18. Metrics

[Metrics](https://www.apolloconfig.com/#/zh/design/apollo-design?id=_52-metrics)

从 1.5.0 版本开始，Apollo 服务端支持通过/prometheus 暴露 prometheus 格式的 metrics，如 http://${someIp:somePort}/prometheus

### 19. 客户端获取配置（Java API 样例）

[获取 Apollo 配置](https://www.apolloconfig.com/#/zh/design/apollo-core-concept-namespace?id=_54-%e4%be%8b%e5%ad%90)

当应用使用下面的语句获取配置时，我们称之为获取应用自身的配置，也就是应用自身的 application namespace 的配置。

```java
Config config = ConfigService.getAppConfig();
```

配置发布后，就能在客户端获取到了，以 Java 为例，获取配置的示例代码如下。

```java
Config config = ConfigService.getAppConfig();
Integer defaultRequestTimeout = 200;
Integer requestTimeout = config.getIntProperty("requestTimeout", defaultRequestTimeout);
```

以 FX.Hermes.Producer 为例，hermes producer 是 hermes 发布的公共组件。当使用下面的语句获取配置时，我们称之为获取公共组件的配置。

```java
Config config = ConfigService.getConfig("FX.Hermes.Producer");
```

对自定义 namespace 的配置获取，稍有不同，需要程序传入 namespace 的名字。

```java
Config config = ConfigService.getConfig("FX.Hermes.Producer");
Integer defaultSenderBatchSize = 200;
Integer senderBatchSize = config.getIntProperty("sender.batchsize", defaultSenderBatchSize);
```

### 20. 客户端监听配置变化

[客户端监听 Namespace 配置变化](https://www.apolloconfig.com/#/zh/design/apollo-introduction?id=_436-%e5%ae%a2%e6%88%b7%e7%ab%af%e7%9b%91%e5%90%acnamespace%e9%85%8d%e7%bd%ae%e5%8f%98%e5%8c%96)
当应用使用下面的语句获取配置时，我们称之为获取应用自身的配置，也就是应用自身的 application namespace 的配置。

```java
Config config = ConfigService.getAppConfig();
```

通过上述获取配置代码，应用就能实时获取到最新的配置了。

不过在某些场景下，应用还需要在配置变化时获得通知，比如数据库连接的切换等，所以 Apollo 还提供了监听配置变化的功能，Java 示例如下：

```java
Config config = ConfigService.getAppConfig();
config.addChangeListener(new ConfigChangeListener() {
  @Override
  public void onChange(ConfigChangeEvent changeEvent) {
    for (String key : changeEvent.changedKeys()) {
      ConfigChange change = changeEvent.getChange(key);
      System.out.println(String.format(
        "Found change - key: %s, oldValue: %s, newValue: %s, changeType: %s",
        change.getPropertyName(), change.getOldValue(),
        change.getNewValue(), change.getChangeType()));
     }
  }
});
```

以 FX.Hermes.Producer 为例，hermes producer 是 hermes 发布的公共组件。当使用下面的语句获取配置时，我们称之为获取公共组件的配置。

```java
Config config = ConfigService.getConfig("FX.Hermes.Producer");
```

对自定义 namespace 的配置获取，稍有不同，需要程序传入 namespace 的名字。

```java
Config config = ConfigService.getConfig("FX.Hermes.Producer");
config.addChangeListener(new ConfigChangeListener() {
  @Override
  public void onChange(ConfigChangeEvent changeEvent) {
    System.out.println("Changes for namespace " + changeEvent.getNamespace());
    for (String key : changeEvent.changedKeys()) {
      ConfigChange change = changeEvent.getChange(key);
      System.out.println(String.format(
        "Found change - key: %s, oldValue: %s, newValue: %s, changeType: %s",
        change.getPropertyName(), change.getOldValue(),
        change.getNewValue(), change.getChangeType()));
     }
  }
});
```

### 21. Namespace 的格式有哪些？

[Namespace 的格式有哪些？](https://www.apolloconfig.com/#/zh/design/apollo-core-concept-namespace?id=_3-namespace%e7%9a%84%e6%a0%bc%e5%bc%8f%e6%9c%89%e5%93%aa%e4%ba%9b%ef%bc%9f)

配置文件有多种格式，例如：properties、xml、yml、yaml、json 等。同样 Namespace 也具有这些格式。在 Portal UI 中可以看到“application”的 Namespace 上有一个“properties”标签，表明“application”是 properties 格式的。

注 1：非 properties 格式的 namespace，在客户端使用时需要调用 ConfigService.getConfigFile(String namespace, ConfigFileFormat configFileFormat)来获取，如果使用 Http 接口直接调用时，对应的 namespace 参数需要传入 namespace 的名字加上后缀名，如 datasources.json。

注 2：apollo-client 1.3.0 版本开始对 yaml/yml 做了更好的支持，使用起来和 properties 格式一致：Config config = ConfigService.getConfig("application.yml");，Spring 的注入方式也和 properties 一致。

### 22. Namespace 的类型

[Namespace 的类型](https://www.apolloconfig.com/#/zh/design/apollo-core-concept-namespace?id=_5-namespace%e7%9a%84%e7%b1%bb%e5%9e%8b)

Namespace 类型有三种：

- 私有类型

私有类型的 Namespace 具有 private 权限。

- 公共类型

公共类型的 Namespace 具有 public 权限。公共类型的 Namespace 相当于游离于应用之外的配置，且通过 Namespace 的名称去标识公共 Namespace，所以公共的 Namespace 的名称必须全局唯一。

- 关联类型

关联类型又可称为继承类型，关联类型具有 private 权限。关联类型的 Namespace 继承于公共类型的 Namespace，用于覆盖公共 Namespace 的某些配置。

### 23. 关联类型使用场景

[关联类型](https://www.apolloconfig.com/#/zh/design/apollo-core-concept-namespace?id=_53-%e5%85%b3%e8%81%94%e7%b1%bb%e5%9e%8b)

例如公共的 Namespace 有两个配置项

```yml
k1 = v1
k2 = v2
```

然后应用 A 有一个关联类型的 Namespace 关联了此公共 Namespace，且覆盖了配置项 k1，新值为 v3。那么在应用 A 实际运行时，获取到的公共 Namespace 的配置为：

```yml
k1 = v3
k2 = v2
```

举一个实际使用的场景。假设 RPC 框架的配置（如：timeout）有以下要求：

- 提供一份全公司默认的配置且可动态调整
- RPC 客户端项目可以自定义某些配置项且可动态调整

如果把以上两点要求去掉“动态调整”，那么做法很简单。在 rpc-client.jar 包里有一份配置文件，每次修改配置文件然后重新发一个版本的 jar 包即可。同理，客户端项目修改配置也是如此。

如果只支持客户端项目可动态调整配置。客户端项目可以在 Apollo “application” Namespace 上配置一些配置项。在初始化 service 的时候，从 Apollo 上读取配置即可。这样做的坏处就是，每个项目都要自定义一些 key，不统一。

那么如何完美支持以上需求呢？答案就是结合使用 Apollo 的公共类型的 Namespace 和关联类型的 Namespace。RPC 团队在 Apollo 上维护一个叫“rpc-client”的公共 Namespace，在“rpc-client” Namespace 上配置默认的参数值。rpc-client.jar 里的代码读取“rpc-client”Namespace 的配置即可。如果需要调整默认的配置，只需要修改公共类型“rpc-client” Namespace 的配置。如果客户端项目想要自定义或动态修改某些配置项，只需要在 Apollo 自己项目下关联“rpc-client”，就能创建关联类型“rpc-client”的 Namespace。然后在关联类型“rpc-client”的 Namespace 下修改配置项即可。这里有一点需要指出的，那就是 rpc-client.jar 是在应用容器里运行的，所以 rpc-client 获取到的“rpc-client” Namespace 的配置是应用的关联类型的 Namespace 加上公共类型的 Namespace。

### 24. 设置内存中的配置项是否保持和页面上的顺序一致

[配置顺序和页面上看到的一致](https://www.apolloconfig.com/#/zh/usage/java-sdk-user-guide?id=_1243-%e8%ae%be%e7%bd%ae%e5%86%85%e5%ad%98%e4%b8%ad%e7%9a%84%e9%85%8d%e7%bd%ae%e9%a1%b9%e6%98%af%e5%90%a6%e4%bf%9d%e6%8c%81%e5%92%8c%e9%a1%b5%e9%9d%a2%e4%b8%8a%e7%9a%84%e9%a1%ba%e5%ba%8f%e4%b8%80%e8%87%b4)

适用于 1.6.0 及以上版本

默认情况下，apollo client 内存中的配置存放在 Properties 中（底下是 Hashtable），不会刻意保持和页面上看到的顺序一致，对绝大部分的场景是没有影响的。不过有些场景会强依赖配置项的顺序（如 spring cloud zuul 的路由规则），针对这种情况，可以开启 OrderedProperties 特性来使得内存中的配置顺序和页面上看到的一致。

配置方式按照优先级从高到低分别为：

1，通过 Java System Property apollo.property.order.enable

可以通过 Java 的 System Property apollo.property.order.enable 来指定

在 Java 程序启动脚本中，可以指定-Dapollo.property.order.enable=true

如果是运行 jar 文件，需要注意格式是 java -Dapollo.property.order.enable=true -jar xxx.jar

也可以通过程序指定，如 System.setProperty("apollo.property.order.enable", "true");

2，通过 Spring Boot 的配置文件

可以在 Spring Boot 的 application.properties 或 bootstrap.properties 中指定 apollo.property.order.enable=true

3，通过 app.properties 配置文件

可以在 classpath:/META-INF/app.properties 指定 apollo.property.order.enable=true

### 25. 自定义缓存路径

[自定义缓存路径](https://www.apolloconfig.com/#/zh/usage/java-sdk-user-guide?id=_1231-%e8%87%aa%e5%ae%9a%e4%b9%89%e7%bc%93%e5%ad%98%e8%b7%af%e5%be%84)

1.0.0 版本开始支持以下方式自定义缓存路径，按照优先级从高到低分别为：

（1）通过 Java System Property apollo.cache-dir(1.9.0+) 或者 apollo.cacheDir(1.9.0 之前)

可以通过 Java 的 System Property apollo.cache-dir(1.9.0+) 或者 apollo.cacheDir(1.9.0 之前)来指定

在 Java 程序启动脚本中，可以指定-Dapollo.cache-dir=/opt/data/some-cache-dir(1.9.0+) 或者 apollo.cacheDir=/opt/data/some-cache-dir(1.9.0 之前)

如果是运行 jar 文件，需要注意格式是 java -Dapollo.cache-dir=/opt/data/some-cache-dir -jar xxx.jar(1.9.0+) 或者 java -Dapollo.cacheDir=/opt/data/some-cache-dir -jar xxx.jar(1.9.0 之前)

也可以通过程序指定，如 System.setProperty("apollo.cache-dir", "/opt/data/some-cache-dir");(1.9.0+) 或者 System.setProperty("apollo.cacheDir", "/opt/data/some-cache-dir");(1.9.0 之前)

（2）通过 Spring Boot 的配置文件

可以在 Spring Boot 的 application.properties 或 bootstrap.properties 中指定 apollo.cache-dir=/opt/data/some-cache-dir(1.9.0+) 或者 apollo.cacheDir=/opt/data/some-cache-dir(1.9.0 之前)

（3）通过操作系统的 System EnvironmentAPOLLO_CACHE_DIR(1.9.0+) 或者 APOLLO_CACHEDIR(1.9.0 之前)

可以通过操作系统的 System Environment APOLLO_CACHE_DIR(1.9.0+) 或者 APOLLO_CACHEDIR(1.9.0 之前)来指定

注意 key 为全大写，且中间是\_分隔

（4）通过 server.properties 配置文件

可以在 server.properties 配置文件中指定 apollo.cache-dir=/opt/data/some-cache-dir(1.9.0+) 或者 apollo.cacheDir=/opt/data/some-cache-dir(1.9.0 之前)

对于 Mac/Linux，默认文件位置为/opt/settings/server.properties

对于 Windows，默认文件位置为 C:\opt\settings\server.properties

注：本地缓存路径也可用于容灾目录，如果应用在所有 config service 都挂掉的情况下需要扩容，那么也可以先把配置从已有机器上的缓存路径复制到新机器上的相同缓存路径

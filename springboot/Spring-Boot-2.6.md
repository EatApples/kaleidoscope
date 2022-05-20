# Spring-Boot-2.6

## 1. 从 Spring Boot 2.5 升级

### 1.1 Spring Boot 2.4 的弃用

在 Spring Boot 2.4 中弃用的类、方法和属性已在此版本中删除。请确保在升级之前您没有调用已弃用的方法。

### 1.2 默认禁止循环引用

现在默认禁止 bean 之间的循环引用。

如果您的应用程序由于某个原因而无法启动，BeanCurrentlyInCreationException 强烈建议您更新配置以打破依赖循环。

如果您无法这样做，可以通过设置 spring.main.allow-circular-references 为 true，或使用新的 setter 方法再次允许循环引用 SpringApplication，SpringApplicationBuilder 这将恢复 2.5 的行为并自动尝试打破依赖循环。

### 1.3 基于 PathPattern 的 Spring MVC 路径匹配策略

将请求路径与已注册的 Spring MVC 处理程序映射匹配的默认策略已从 AntPathMatcher 更改为 PathPatternParser。

如果您正在使用 Spring Security，您应该检查您的使用 mvcMatchers 以确保您的匹配器继续满足您的需求。使用 AntPathMatcher,authorizeRequests.mvcMatchers("hello").permitAll()将授予对 的访问权限/hello。更精确的匹配 PathPatternParser 需要使用 authorizeRequests.mvcMatchers("/hello").permitAll()（注意前导/）来代替。

如果需要将默认切换回 AntPathMatcher，可以设置 spring.mvc.pathmatch.matching-strategy 为 ant-path-matcher。

执行器端点现在也使用 PathPattern 基于 URL 匹配。请注意，执行器端点的路径匹配策略无法通过配置属性进行配置。如果您使用的是 Actuator 和 Springfox，这可能会导致您的应用程序无法启动。有关更多详细信息，请参阅此 [Springfox 问题](this Springfox issue for further details)。

### 1.4 Actuator Env InfoContributor 默认禁用

该 env 信息的贡献者现在默认情况下禁用。要启用它，请设置 management.info.env.enabled 为 true。

### 1.5 应用启动

ApplicationStartup 登录的启动步骤 spring.boot.application.running 已重命名为 spring.boot.application.ready。

如果您正在处理从 FlightRecorderApplicationStartup 或 BufferingApplicationStartup 生成的文件您将需要使用新名称。

### 1.6 网络资源配置

Resources 直接注入不再有效，因为此配置已在 WebProperties. 如果您需要访问此信息，则需要改为注入 WebProperties。

### 1.7 依赖管理删除

（1）JBoss 事务 SPI

org.jboss:jboss-transaction-spi 的依赖管理已被删除。

如果您正在使用 org.jboss:jboss-transaction-spi，您应该定义自己的依赖管理来满足您的应用程序的需求。

（2）Nimbus DS

依赖管理 com.nimbusds:oauth2-oidc-sdk 和 com.nimbusds:nimbus-jose-jwt 已被删除。

如果您正在使用 Spring Security，您应该依赖它将作为传递依赖项引入的版本。

如果您不使用 Spring Security，您应该定义自己的依赖管理来满足您的应用程序需求。

（3）HAL 浏览器

org.webjars:hal-browser 的依赖管理已被删除。

如果您正在使用，org.webjars:hal-browser 您应该定义自己的依赖管理来满足您的应用程序的需求。

### 1.8 普罗米修斯版本属性

控制 Prometheus 版本的属性已从 prometheus-pushgateway.version 更改为 prometheus-client.version。

这是为了反映一个事实，即该属性管理 Prometheus 客户端中每个模块的版本，而不仅仅是 pushgateway。

### 1.9 嵌入式 Mongo

要使用嵌入式 mongo，现在必须设置 spring.mongodb.embedded.version 该属性。

这有助于确保嵌入式支持使用的 MongoDB 版本与您的应用程序将在生产中使用的 MongoDB 版本相匹配。

### 1.10 Oracle 数据库驱动程序依赖管理

Oracle 数据库驱动程序的依赖管理已得到简化。

如果您仍然依赖旧的 com.oracle.ojdbcgroupId，您需要升级到 com.oracle.database.jdbcgroup，因为我们已经删除了对前者的依赖管理。

### 1.11 删除了与 Vault 相关的 Flyway 属性

Flyway 的 7.12 版本将与 Vault 相关的设置移动到了一个闭源扩展。不幸的是，这会阻止 Spring Boot 配置它们。其结果是，相应的 spring.flyway.vault-secrets，spring.flyway.vault-token 和 spring.flyway.vault-url 属性已被删除。

如果您是 Flyway Teams 用户，则可以通过配置设置 FlywayConfigurationCustomizerbean，FluentConfigiguration.getExtensionConfiguration 和 VaultApiExtension。

### 1.12 WebFlux 会话属性

该 spring.webflux.session 属性组已被弃用，搬迁到 server.reactive.session。

旧属性将继续工作，但如果可能，您应该迁移到新属性。

### 1.13 Elasticsearch 属性整合

用于配置 Elasticsearch 客户端的配置属性已被整合。

以前，用于配置阻塞高级 REST 客户端和反应式 REST 客户端的许多常见属性在 spring.elasticsearch.rest 和 spring.data.elasticsearch.clients.reactive。

如果您使用的是阻塞 REST 客户端，下表列出了旧属性及其替换：

| 弃用的属性                                            | 替代品                                                      |
| ----------------------------------------------------- | ----------------------------------------------------------- |
| spring.elasticsearch.rest.uris                        | spring.elasticsearch.uris                                   |
| spring.elasticsearch.rest.username                    | spring.elasticsearch.username                               |
| spring.elasticsearch.rest.password                    | spring.elasticsearch.password                               |
| spring.elasticsearch.rest.connection-timeout          | spring.elasticsearch.connection-timeout                     |
| spring.elasticsearch.rest.read-timeout                | spring.elasticsearch.socket-timeout                         |
| spring.elasticsearch.rest.sniffer.interval            | spring.elasticsearch.restclient.sniffer.interval            |
| spring.elasticsearch.rest.sniffer.delay-after-failure | spring.elasticsearch.restclient.sniffer.delay-after-failure |

如果您使用的是反应式客户端，下表列出了旧属性及其替换：

| 弃用的属性                                                   | 替代品                                             |
| ------------------------------------------------------------ | -------------------------------------------------- |
| spring.data.elasticsearch.client.reactive.endpoints          | spring.elasticsearch.uris                          |
| spring.data.elasticsearch.client.reactive.use-ssl            | spring.elasticsearch.uris（使用 HTTPS 时需要配置） |
| spring.data.elasticsearch.client.reactive.username           | spring.elasticsearch.username                      |
| spring.data.elasticsearch.client.reactive.password           | spring.elasticsearch.password                      |
| spring.data.elasticsearch.client.reactive.connection-timeout | spring.elasticsearch.connection-timeout            |
| spring.data.elasticsearch.client.reactive.socket-timeout     | spring.elasticsearch.socket-timeout                |
| spring.data.elasticsearch.client.reactive.max-in-memory-size | spring.elasticsearch.webclient.max-in-memory-size  |

### 1.14 @Persistent 不再考虑 Spring Data Couchbase

为了使默认行为与 Spring Data Couchbase 保持一致，@Persistent 不再考虑 -annotated 类型。如果您依赖于该注释，@Document 则可以改用。

### 1.15 Maven 构建信息的默认时间

Maven 插件的构建信息支持现在使用 project.build.outputTimestamp 属性值作为默认构建时间。

如果未设置该属性，则使用之前的构建会话的开始时间。和以前一样，可以通过将时间设置为 off 来完全禁用时间。

### 1.16 记录和 @ConfigurationProperties

如果您使用@ConfigurationProperties 的是 Java 16 记录并且该记录只有一个构造函数，则不再需要用@ConstructorBinding. 如果您的记录具有多个构造函数，@ConstructorBinding 则仍应使用该构造函数来标识用于属性绑定的构造函数。

### 1.17 延迟 OpenID Connect 发现

对于使用 spring-security-oauth2-resource-server 作为 OpenID 的资源服务器，连接 issuer-uri 时，Spring Boot 现在自动配置 aSupplierJwtDecoder 而不是 NimbusJwtDecoder。

这启用了 Spring Security 的惰性 OIDC 发现支持，从而优化了启动时间。同样，对于反应式应用程序， aReactiveSupplierJwtDecoder 是自动配置的。

### 1.18 最低要求变更

无。

## 2. 新特性和值得注意的

注意：检查[配置更改日志](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6.0-Configuration-Changelog)以获取配置更改的完整概述。

### 2.1 Servlet 支持 SameSite Cookie 属性

您现在可以使用该 server.servlet.session.cookie.same-site 属性为 servlet 应用程序配置会话 SameSite 的属性。这适用于自动配置的 Tomcat、Jetty 和 Undertow 服务器。

此外， 如果您想将 SameSite 属性应用于其他 cookie ，则可以使用该 CookieSameSiteSupplier 接口。有关更多详细信息和一些示例代码，请参阅[更新的文档](https://docs.spring.io/spring-boot/docs/2.6.0/reference/html//web.html#web.servlet.embedded-container.customizing.samesite)。

### 2.2 反应式服务器会话属性

响应式服务器（以前在 spring.webflux.session 下）支持的会话属性已在此版本中扩展。新属性 server.reactive.session 在 servlet 版本下可用，现在提供对等的 servlet 版本。

### 2.3 可插入的清理规则

Spring Boot 清理存在于/env 和/configprops 端点中的敏感值。虽然可以通过配置属性来配置哪些属性需要清理，但用户可能希望根据 PropertySource 属性的来源应用清理规则。例如，Spring Cloud Vault 使用 Vault 来存储加密值并将它们加载到 Spring 环境中。

由于所有值都是加密的，因此将整个属性源中的每个键的值清空是有意义的。可以通过添加 @Bean 类型来配置此类清理自定义 SanitizingFunction。

### 2.4 Java 运行时信息

该 info 端点现在可以暴露在 Java 运行时信息 java 的关键，如下面的例子：

```json
{
  "java": {
    "vendor": "BellSoft",
    "version": "17",
    "runtime": {
      "name": "OpenJDK Runtime Environment",
      "version": "17+35-LTS"
    },
    "jvm": {
      "name": "OpenJDK 64-Bit Server VM",
      "vendor": "BellSoft",
      "version": "17+35-LTS"
    }
  }
}
```

要在 info 端点的响应中公开此信息，请将 management.info.java.enabled 属性设置为 true。

### 2.5 构建信息属性排除

现在可以排除特定属性被添加到 build-info.propertiesSpring Boot Maven 或 Gradle 插件生成的文件中。

Maven 用户可以排除标准 group，artifact，name，version 或 time 使用属性<excludeInfoProperties>标签。例如，要排除该 version 属性，可以使用以下配置：

```xml
<configuration>
	<excludeInfoProperties>
		<excludeInfoProperty>version</excludeInfoProperty>
	</excludeInfoProperties>
</configuration>
```

### 2.6 健康检查支持

（1）主端口或管理端口上的附加路径

健康组可以在主端口或管理端口的附加路径上可用。这在 Kubernetes 等云环境中很有用，在这种环境中，出于安全目的，为执行器端点使用单独的管理端口是很常见的。拥有单独的端口可能会导致不可靠的健康检查，因为即使健康检查成功，主应用程序也可能无法正常工作。

典型的配置会将所有执行器端点都放在一个单独的端口上，并将用于活动和准备状态的健康组放在主端口上的附加路径上。

（2）复合 Contributor Include/Exclude 支持

健康组可以配置为包含/排除 CompositeHealthContributor. 这可以通过指定组件的完全限定路径来完成。例如，spring 嵌套在名为 的组合内的组件 test 可以使用 test/spring.

### 2.7 指标支持

（1）应用启动

自动配置公开了与应用程序启动相关的两个指标：

- application.started.time：启动应用程序所需的时间。
- application.ready.time：应用程序准备好为请求提供服务所需的时间。

（2）磁盘空间

DiskSpaceMetrics 现在是自动配置的。 disk.free 并 disk.total 为当前工作目录标识的分区提供指标。

要更改使用的路径，请定义您自己的 DiskSpaceMetrics bean。

（3）任务执行和调度

只要底层 ThreadPoolExecutor 可用，ExecutorServiceMetrics 现在可以为所有 ThreadPoolTaskExecutor 和 ThreadPoolTaskScheduler bean 自动配置。

指标用从其 bean 名称派生的执行器名称进行标记。

（4）Jetty Connection and SSL

JettyConnectionMetrics 现在是自动配置的。此外，当 server.ssl.enabled 设置为 true 时 ， JettySslHandshakeMetrics 也会自动配置。

（5）导出到 Dynatrace v2 API

添加了对将指标导出到 Dynatrace v2 API 的支持。

在主机上运行本地 OneAgent 时，只需要依赖 io.micrometer:micrometer-registry-dynatrace 即可。如果没有本地 OneAgent，则必须配置 management.metrics.export.dynatrace.uri 和 management.metrics.export.dynatrace.api-token 属性。可以使用 management.metrics.export.dynatrace.v2 属性配置特定于 v2 API 的其他设置。有关详细信息，请参阅更新后的[参考文档](https://docs.spring.io/spring-boot/docs/2.6.0/reference/html//actuator.html#actuator.metrics.export.dynatrace)。

### 2.8 Docker 镜像构建支持

（1）附加 Image 标签

Maven 和 Gradle 插件现在支持在使用 tags 配置参数构建生成的图像后将附加标签应用于生成的图像。

有关更多详细信息，请参阅更新的 Gradle 和 Maven 参考文档。

（2）网络配置

Maven 插件 spring-boot:build-image 和 Gradle bootBuildImage 任务中添加了一个配置参数 network。此参数可用于配置运行 Cloud Native Buildpacks 构建器进程的容器所使用的网络驱动程序。

（3）缓存配置

Maven 和 Gradle 插件现在支持自定义卷的名称，这些卷用于缓存由 buildpacks usingbuildCache 和 launchCacheconfiguration 参数贡献给构建的图像的层。

有关更多详细信息，请参阅更新的 Gradle 和 Maven 参考文档。

### 2.9 Spring Data Envers 的自动配置

现在提供了 Spring Data Envers 的自动配置。要使用它，请添加依赖项 org.springframework.data:spring-data-envers 并更新您的 JPA 存储库。

### 2.10 Redis 连接池

Redis（Jedis 和 Lettuce）现在将在 commons-pool2 类路径上自动启用池。

可以设置 spring.redis.jedis.pool.enabled 或 spring.redis.lettuce.pool.enabled 为 false 禁用池化。

### 2.11 spring-rabbit-stream 的自动配置

spring-rabbit-stream 已添加 Spring AMQP 新模块的自动配置。

当 spring.rabbitmq.listener.type 属性设置为 stream 时，StreamListenerContainer 是自动配置的。这些 spring.rabbitmq.stream.\*属性可用于配置对代理的访问，spring.rabbitmq.listener.stream.native-listener 并可用于启用本机侦听器支持。

### 2.12 支持 Kafka SSL 属性中的 PEM 格式

以前，Kafka 仅支持 SSL 的基于文件的密钥和信任存储。使用 KIP-651，现在可以使用 PEM 格式。Spring Boot 添加了以下属性，允许使用 PEM 格式配置 SSL 证书和私钥：

- spring.kafka.ssl.key-store-key
- spring.kafka.ssl.key-store-certificate-chain
- spring.kafka.ssl.trust-store-certificates

### 2.13 改进了 Maven 插件启动目标的配置

Maven 插件的 start 目标已经从命令行变得更加可配置。

它的 wait 和 maxAttempts 属性可以分别使用 spring-boot.start.wait 和指定 spring-boot.start.maxAttempts 来设置。

### 2.14 自动配置的 Spring Web 服务服务器测试

引入@WebServiceServerTest 了可用于测试 Web 服务@Endpointbean 的新注释。

注释创建一个包含@Endpointbean 的测试切片，并自动配置一个 MockWebServiceClient 可用于测试您的 Web 服务端点的 bean。

### 2.15 基于消息源的 Bean 验证消息插值

在约束消息中解析{parameters}时可在应用程序使用 MessageSource。

这允许您将应用程序的 messages.properties 文件用于 Bean 验证消息。一旦解析了参数，就使用 Bean Validation 的默认插值器完成消息插值。

### 2.16 使用 WebTestClient 测试 Spring MVC

开发人员可以使用 WebTestClient 在模拟环境中测试 WebFlux 应用程序，或针对实时服务器测试任何 Spring Web 应用程序。

此更改还支持 WebTestClient 模拟环境中的 Spring MVC：使用注释的类@AutoConfigureMockMvc 可以注入 WebTestClient. 这使我们的支持变得完整，您现在可以使用单个 API 来驱动您的所有 Web 测试！

### 2.17 Spring 集成 PollerMetadata 属性

Spring Integration PollerMetadata（每秒轮询无限数量的消息）现在可以使用 spring.integration.poller.\*配置属性进行自定义。

### 2.18 支持 Log4j2 的复合配置

现在支持 Log4j2 的复合配置。要使用它，请使用逗号分隔列表配置该 logging.log4j2.config.override 属性，将覆盖主配置的辅助配置文件。

主要配置来源于 Spring Boot 的默认值，一个众所周知的标准位置，例如 log4j2.xml，或者 logging.config 像之前一样由属性指定的位置。

### 2.19 依赖升级

Spring Boot 2.5 迁移到几个 Spring 项目的新版本：

```md
[Spring Security 5.6](https://docs.spring.io/spring-security/reference/5.6.0/whats-new.html)

[Spring Data 2021.1](https://github.com/spring-projects/spring-data-commons/wiki/Release-Train-2021.1-%28Q%29-Release-Notes)

[Spring HATEOAS 1.4](https://spring.io/blog/2021/11/22/spring-hateoas-1-4-released)

[Spring Kafka 2.8](https://docs.spring.io/spring-kafka/docs/2.8.x/reference/html/#spring-kafka-intro-new)

[Spring AMQP 2.4](https://docs.spring.io/spring-amqp/docs/2.4.x/reference/html/#changes-in-2-4-since-2-3)

[Spring Session 2021.1.0](https://github.com/spring-projects/spring-session-bom/wiki/Spring-Session-2021.1-Release-Notes)
```

还更新了许多第三方依赖项，其中一些更值得注意的是：

```md
[Apache Kafka 3.0](https://www.confluent.io/blog/apache-kafka-3-0-major-improvements-and-new-features/)

[Artemis 2.19](https://issues.apache.org/jira/secure/ReleaseNote.jspa?projectId=12315920&version=12350519)

[Cassandra Driver 4.13](https://docs.datastax.com/en/developer/java-driver/4.13/)

Commons DBCP 2.9

Commons Pool 2.11

[Couchbase Client 3.2.2](https://docs.couchbase.com/java-sdk/current/project-docs/sdk-release-notes.html#version-3-2-3-2-november-2021)

[Elasticsearch 7.15](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/release-highlights.html)

[Flyway 8.0.5](https://flywaydb.org/documentation/learnmore/releaseNotes#8.0.5)

[Hibernate 5.6](https://in.relation.to/2021/10/13/hibernate-orm-560-final/)

[JUnit Jupiter 5.8](https://junit.org/junit5/docs/current/release-notes/#release-notes-5.8.1)

[Jedis 3.7](https://github.com/redis/jedis/releases/tag/v3.7.0)

[Kafka 3.0](https://downloads.apache.org/kafka/3.0.0/RELEASE_NOTES.html)

[Kotlin 1.6](https://blog.jetbrains.com/kotlin/2021/11/kotlin-1-6-0-is-released/)

[Liquibase 4.5](https://github.com/liquibase/liquibase/releases/tag/v4.5.0)

[Micrometer 1.8](https://github.com/micrometer-metrics/micrometer/releases/tag/v1.8.0)

[Mockito 4.0](https://github.com/mockito/mockito/releases/tag/v4.0.0)

[MongoDB 4.4](https://docs.mongodb.com/manual/release-notes/4.4/)

[Postgresql 42.3](https://www.postgresql.org/about/news/postgresql-jdbc-4230-released-2333/)

[QueryDSL 5.0](https://github.com/querydsl/querydsl/releases/tag/QUERYDSL_5_0_0)

SnakeYAML 1.29

[Thymeleaf Layout Dialect 3.0](https://ultraq.github.io/thymeleaf-layout-dialect/migrating-to-3.0/)
```

### 2.20 其他更新

除了上面列出的更改之外，还有许多小的调整和改进，包括：

现在的故障分析 NoSuchMethodError 包括有关加载调用类的位置的信息。

ClientResourcesBuilderCustomizer 现在可以定义一个 bean 来自定义 Lettuce 的 ClientResources 遗嘱，并保留默认的自动配置。

对于 Flyway 的配置属性 detectEncoding，failOnMissingLocations 和 ignoreMigrationPatterns 配置设置已被添加。

自定义 ResourceLoader 可以在创建 SpringApplicationBulder 时被提供给由所述应用程序使用。

现在可以定义 WebSessionIdResolver 来自定义自动配置 WebSessionManager 的解析器将使用的解析器 。

现在任何 RSocketConnectorConfigurer bean 都会自动应用于自动配置的 RSocketRequester.Builder.

现在 spring-boot-configuration-processor 可以为 Lombok 的@Value 生成元数据.

可以将新的配置属性 server.tomcat.reject-illegal-header 设置为 true 以将 Tomcat 配置为接受非法标头。

使用 Stackdriver 时，现在可以通过设置 management.metrics.export.stackdriver.resource-labels.\*配置属性在监视器资源上配置标签。

@EntityScan 现在在其 basePackages 属性中支持逗号分隔值。

添加了一个新的配置属性，server.netty.idle-timeout 可用于控制 Reactor Netty 的空闲超时。

Devtools 加载其全局设置的位置现在可以使用 spring.devtools.home 系统属性或 SPRING_DEVTOOLS_HOME 环境变量进行配置。

在 RabbitTemplateConfigurer setter 方法上现在是 public

该 heapdump 端点现在支持 OpenJ9 ，它会产生 PHD 格式堆转储。

WebFlux 中的多部分支持现在支持新的配置属性，例如 spring.webflux.multipart.\*

现在任何 ContainerCustomizer bean 来自动配置 Spring AMQP 的 MessageListenerContainer

Jackson 的默认宽大处理可以使用该 spring.jackson.default-leniency 属性进行配置。

分发统计的到期时间和缓冲区长度现在是可配置的。

Lettuce 的命令延迟指标现在是自动配置的。

磁盘空间指标可以使用该 management.metrics.system.diskspace.paths 属性配置一个或多个路径。

用户可以通过提供 RedisStandaloneConfigurationbean 来控制 Redis 自动配置。

当自动配置 H2 控制台时，现在会记录所有可用数据源的 URL。

新的配置属性 spring.integration.management.default-logging-enabled 可用于通过将其值设置为 false 来禁用 Spring Integration 的默认日志记录。

UserDetailsService 的自动配置现在将在 AuthenticationManagerProvider bean 存在的情况下退出。

## 3. Spring Boot 2.6 中的弃用

（1）AbstractDataSourceInitializer 类已被弃用，取而代之的是 DataSourceScriptDatabaseInitializer。另外，AbstractDataSourceInitializer 的子类也已被弃用，取而代之的是新的基于 DataSourceScriptDatabaseInitializer 的类。

（2）SpringPhysicalNamingStrategy 类已被弃用，取而代之的是 Hibernate 5.5 的 CamelCaseToUnderscoresNamingStrategy 类。

（2）AbstractApplicationContextRunner 类中的三个方法已被弃用，取而代之的是新的基于 RunnerConfiguration 的类。

（4）SpringApplicationRunListener 中的 started 和 running 方法已被弃用，取而代之的是接受 Duration 参数的新方法：

（5）ApplicationStartedEvent 和 ApplicationReadyEvent 中的构造函数也已被替换为接受 Duration 参数的版本：

（6）EnvironmentEndpoint.sanitize 被标识弃用了。

# 参考资料

#### 1. What's new in Spring Boot 2.4

https://spring.io/blog/2021/01/17/what-s-new-in-spring-boot-2-4
https://www.bilibili.com/video/BV1B5411E7iq

#### 2. 厉害了！SpringBoot2.5 重磅发布，你知道有哪些新特性？

https://www.bilibili.com/video/BV17B4y1u73T

#### 3. Spring Boot 2.X Release Notes

https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes
https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.4-Release-Notes
https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.5-Release-Notes
https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes

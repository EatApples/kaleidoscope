# Spring-Boot-2.5

## 1. 从 Spring Boot 2.4 升级

### 1.1 SQL 脚本数据源初始化

Spring Boot 2.5 中重新设计了用于支持 schema.sql 和 data.sql 脚本的底层方法。 spring.datasource.*与 DataSource 初始化相关的属性已被弃用，取而代之的是新 spring.sql.init.*属性。这些属性还可用于初始化通过 R2DBC 访问的 SQL 数据库。

（1）schema.sql 和 data.sql 文件

对于 Spring Boot 2.5.1 及更高版本，新的 SQL 初始化属性支持检测 JDBC 和 R2DBC 的嵌入式数据源。

默认情况下，SQL 数据库初始化仅在使用嵌入式内存数据库时执行。

要始终初始化 SQL 数据库，无论其类型如何，请设置 spring.sql.init.mode 为 always. 同样，要禁用初始化，请设置 spring.sql.init.mode 为 never。

（2）单独的凭证

新的基于脚本的 SQL 数据库初始化不支持对架构 (DDL) 和数据 (DML) 更改使用单独的凭据。这降低了复杂性并使其功能与 Flyway 和 Liquibase 保持一致。

如果您需要单独的架构和数据初始化凭据，请定义您自己的 org.springframework.jdbc.datasource.init.DataSourceInitializerbean。

（3）Hibernate 和 data.sql

默认情况下，data.sql 脚本现在在 Hibernate 初始化之前运行。这使基于脚本的基本初始化行为与 Flyway 和 Liquibase 的行为保持一致。

如果要用于 data.sql 填充由 Hibernate 创建的模式，请设置 spring.jpa.defer-datasource-initialization 为 true.

虽然不建议混合使用数据库初始化技术，但这也将允许您 schema.sql 在通过 data.sql.

（4）初始化排序

某些知名类型的 Bean ，例如 JdbcTemplate，将被排序，以便在数据库初始化后初始化它们。

如果您有一个 DataSource 直接使用的 bean ，请注释它的类或@Bean 方法，@DependsOnDatabaseInitialization 以确保它在数据库初始化后也被初始化。

### 1.2 Flyway and Liquibase JDBC URLs

如果您当前定义了一个 spring.flyway.url 或 spring.liquibase.url 您可能需要提供额外的 username 和 password 属性。

在 Spring Boot 的早期版本中，这些设置是从 spring.datasource 属性派生的，但这对于提供自己的 DataSource bean 的人来说是有问题的。

### 1.3 Spring Data JPA

Spring Data JPA 引入了一种新 getById 方法来取代 getOne. 如果您发现您的应用程序正在抛出一个 LazyLoadingException 请将任何现有 getById 方法重命名为 getXyzById（其中 xyz 是任意字符串）。

有关更多详细信息，请阅读更新的 Spring Data JPA [参考文档](https://docs.spring.io/spring-data/jpa/docs/2.6.0-RC1/reference/html/#new-features.2-5-0)。

### 1.4 Spring Data Solr

在 2021.0.0 从 Spring Data 中删除之后，Spring Data Solr 的自动配置已在此版本中删除。

### 1.5 安全信息端点

/info 默认情况下，执行器端点不再通过网络公开。此外，如果 Spring Security 在类路径上并且您的应用程序没有自定义安全配置，则端点默认需要经过身份验证的访问。

请参阅有关公开和保护执行器端点的文档以更改这些新的默认值。

### 1.6 任务调度协调与 Spring 集成

Spring Integration 现在重用一个可用的 TaskScheduler 而不是配置它自己的。在依赖自动配置的典型应用程序设置中，这意味着 Spring Integration 使用池大小为 1 的自动配置的任务调度程序。

要恢复 Spring Integration 的默认 10 个线程，请使用该 spring.task.scheduling.pool.size 属性。

### 1.7 默认表达式语言 (EL) 实现

Spring Boot 的 web 和验证启动器中包含的 EL 实现已更改。

现在使用 Tomcat 的实现 (org.apache.tomcat.embed.tomcat-embed-el) 代替 Glassfish ( org.glassfish:jakrta.el)的参考实现。

### 1.8 默认错误视图中的消息

message 默认错误视图中的属性现在被删除，而不是在未显示时为空白。如果解析错误响应 JSON，则可能需要处理缺失项。

server.error.include-message 如果您希望包含消息，您仍然可以使用该属性。

### 1.9 日志关闭钩子

我们现在默认为基于 jar 的应用程序注册一个日志关闭挂钩，以确保在 JVM 退出时释放日志资源。如果您的应用程序部署为战争，那么关闭挂钩不会注册，因为 servlet 容器通常处理日志记录问题。

大多数应用程序都需要关闭挂钩。但是，如果您的应用程序具有复杂的上下文层次结构，那么您可能需要禁用它。您可以使用该 logging.register-shutdown-hook 属性来执行此操作。

### 1.10 Gradle 默认 jar 和 war 任务

Spring Boot Gradle 插件不再自动禁用标准 Gradlejar 和 war 任务。相反，我们现在将 classifier 应用于这些任务。

如果您更喜欢禁用这些任务，[参考文档](https://docs.spring.io/spring-boot/docs/2.5.0/gradle-plugin/reference/htmlsingle/#packaging-executable-and-plain)包括更新的示例。

### 1.11 Cassandra 限流属性

Spring Boot 不再为 spring.data.cassandra.request.throttler 属性提供默认值。

如果您依赖 max-queue-size, max-concurrent-requests,max-requests-per-second 或者 drain-interval 您应该设置对您的应用程序有意义的值。

### 1.12 自定义 jOOQ DefaultConfiguration

为了简化 jOOQ 的定制，现在可以定义 DefaultConfiguration 一个实现的 bean DefaultConfigurationCustomizer。

应该使用此定制器回调来支持定义一个或多个\*Providerbean，现在已弃用对它们的支持。

### 1.13 Groovy 3

Groovy 的默认版本已升级到 3.x。

如果您在使用 Groovy 和 Spock，您还应该升级到最新的 Groovy 3.0 兼容版本的 Spock 2.0。或者，使用 groovy.version 降级回 Groovy 2.5。

### 1.14 最低要求变更

使用 Gradle 构建的项目现在需要 Gradle 6.8 或更高版本。

### 1.15 Hibernate Validator 6.2

Hibernate Validate 的默认版本已升级到 6.2.x。

Hibernate Validator 6.2 更改了表达式语言用于插入约束消息的方式。有关更多详细信息，请参阅 Hibernate Validator 团队的[这篇博文](https://in.relation.to/2021/01/06/hibernate-validator-700-62-final-released/)。

### 1.16 Spring Boot 2.3 和 2.4 的弃用

为反映 Spring Boot 版本兼容性策略，Spring Boot 2.3 中弃用的代码已在 Spring Boot 2.5 中删除。Spring Boot 2.4 中弃用的代码仍然存在，并计划在 Spring Boot 2.6 中删除。

## 2. 新特性和值得注意的

注意：检查[配置更改日志](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.5-Configuration-Changelog)以获取配置更改的完整概述。

### 2.1 环境变量前缀

现在可以为系统环境变量指定前缀，以便您可以在同一环境中运行多个不同的 Spring Boot 应用程序。

使用 SpringApplication.setEnvironmentPrefix(…​)设置要绑定属性时要使用的前缀。

例如，以下将添加 myapp 前缀：

```java
SpringApplication application = new SpringApplication(MyApp.class);
application.setEnvironmentPrefix("myapp");
application.run(args);
```

所有属性现在都需要一个带前缀的版本。例如，要更改服务器端口，您可以设置 MYAPP_SERVER_PORT.

### 2.2 基于 TCP 的 HTTP/2 (h2c)

所有四个嵌入式 Web 容器现在都支持 HTTP/2 over TCP (h2c)，无需任何手动定制。

要启用 h2c，请设置 server.http2.enabled 为 true 并保留 server.ssl.enabled 设置为 false（其默认值）。

与现有的 h2 支持一样，根据所使用的嵌入式 Web 服务器，h2c 的使用可能需要额外的依赖项。有关详细信息，请参阅[参考文档](https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/howto.html#howto.webserver.configure-http2)。

### 2.3 通用数据源初始化

如果您编写初始化数据源的代码，现在可以使用新的通用机制。这种机制现在也在内部用于为 Flyway、Liquibase 和基于脚本的初始化设置正确的 bean 依赖项。

大多数开发人员不需要直接使用新机制。但是，如果您正在为数据访问库开发第三方启动器，您可能需要提供一个 DependsOnDataSourceInitializationDetector.

有关详细信息，请参阅更新的[参考文档](https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/howto.html#howto-initialize-a-database-configuring-dependencies-initializer-detection)。

### 2.4 使用 R2DBC 初始化数据库

添加了对通过 R2DBC 访问的 SQL 数据库的基于脚本的初始化的支持。

默认情况下，名为 schema.sql 和的类路径上的脚本 data.sql 将自动应用于数据库。可以使用 spring.sql.init.\*配置属性自定义初始化。

有关更多详细信息，请参阅[参考文档](https://docs.spring.io/spring-boot/docs/2.5.0/reference/html/howto.html#howto-initialize-a-database-using-basic-scripts)。

### 2.5 Liquibase 数据源

如果您定义了一个用于 Liquibase 的自定义数据源，我们现在使用 SimpleDriverDataSource. 我们之前使用了一个池化数据源，这对于数据库初始化来说是不必要且效率低下的。

### 2.6 分层 WARs

Spring Boot Maven 和 Gradle 插件现在允许您创建分层 WAR 以与 Docker 映像一起使用。分层 WAR 的工作方式类似于早期版本的 Spring Boot 中提供的分层 JAR 支持。查看 Gradle 和 Maven 参考文档以获取更多详细信息。

### 2.7 Docker 镜像构建支持

（1）自定义构建包

Maven 和 Gradle 插件现在都支持使用自定义 Buildpack。您可以将该 buildpacks 属性设置为指向目录、tar.gz 文件、特定构建器引用或 Docker 映像。

有关更多详细信息，请参阅更新的 Gradle 和 Maven 参考文档。

（2）绑定
Maven 和 Gradle 插件现在都支持可以传递给 buildpack 构建器的卷绑定。这些允许您绑定本地路径或卷以供 buildpack 使用。

有关更多详细信息，请参阅更新的 Gradle 和 Maven 参考文档。

（3）War 支持

Maven 和 Gradle 插件现在都可以将可执行的 war 文件打包到 Docker 映像中。如果要为战争创建 Docker 映像，则应使用现有的 mvn spring-boot:image 或./gradlew bootBuildImage 命令。

### 2.8 普罗米修斯的 OpenMetrics

该/actuator/prometheus 驱动器现在终端可以同时提供标准普罗米修斯以及 OpenMetrics 响应。返回的响应将取决于随 HTTP 请求提供的接受标头。

### 2.9 Spring 数据存储库的指标

Actuator 现在将为 Spring Data 存储库生成 Micrometer 指标。默认情况下，指标命名为 spring.data.repository.invocations。要了解更多信息，请参阅参考文档的相关部分。

### 2.10 @Timed 指标与 WebFlux

将其功能与 Spring MVC 的功能保持一致，@Timed 现在可用于手动启用 WebFlux 控制器和功能处理程序处理的请求的计时。要使用手动计时，请设置 management.metrics.web.server.request.autotime.enabled 为 false。

### 2.11 MongoDB 指标

使用 Actuator 时，Mongo 连接池的指标和客户端发送的命令现在会自动生成。要了解有关 MongoDB 指标的更多信息，请参阅参考文档的相关部分。

### 2.12 Quartz 的执行器端点

一个/quartz 端点被添加到执行机构。它提供了有关 Quartz 作业和触发器的详细信息。有关更多详细信息，请参阅执行器 API 文档的相关部分。

### 2.13 GET 请求 actuator/startup

执行器的 startup 端点现在支持 GET 请求。与 POST 请求不同，GET 对端点的请求不会耗尽事件缓冲区，事件将继续保存在内存中。

### 2.14 抽象路由数据源健康检查

Actuator 的运行状况端点现在显示 AbstractRoutingDataSource. 每个目标 DataSource 都使用其路由键命名。

和以前一样，要忽略健康端点中的路由数据源，请设置 management.health.db.ignore-routing-data-sources 为 true。

### 2.15 Java 16 支持

此版本提供支持并针对 Java 16 进行了测试。Spring Boot 2.5 仍然与 Java 8 兼容。

### 2.16 Gradle 7 支持

Spring Boot Gradle 插件支持 Gradle 7.0.x 并对其进行了测试。

### 2.17 Jetty 10 支持

Spring Boot 2.5 现在可以使用 Jetty 10 作为嵌入式 Web 服务器。由于 Jetty 10 需要 Java 11，我们的默认 Jetty 版本将保持为 9。

要升级到 Jetty 10，请使用该 jetty.version 属性覆盖版本。您还应该排除 org.eclipse.jetty.websocket:websocket-server 和 org.eclipse.jetty.websocket:javax-websocket-server-impl 来自，spring-boot-starter-jetty 因为它们是特定于 Jetty 9 的。

### 2.18 文档更新

该项目发布的 HTML 文档具有更新的外观和一些新功能。您现在可以通过将鼠标悬停在示例上并单击“复制”图标，轻松地将代码片段复制到剪贴板。此外，许多示例现在包括可以根据需要显示或隐藏的完整导入语句。

我们现在还在每个文档的顶部有一个“深色主题”切换器。

### 2.19 其他优化

除了上面列出的更改之外，还有许多小的调整和改进，包括：

management.endpoints.web.cors.allowed-origin-patterns 现在可用于为 Actuator 端点配置允许的原始模式(#24608)

HttpSessionIdListenerbean 现在自动注册到 servlet 上下文(#24879)

Couchbase 现在 ObjectMapper 默认使用自动配置(#24616)

ElasticsearchSniffer 现在在其 elasticsearch-rest-client-sniffer 模块位于类路径上时自动配置(#24174)

spring.data.cassandra.controlconnection.timeout 现在可用于配置 Cassandra 控制连接的超时时间(#24189)

spring.kafka.listener.only-log-record-metadata 现在可用于配置尝试重试时记录的内容(#24582)

支持 Apache Phoenix，自动检测 jdbc:phoenixJDBC URL (#24114)

Rabbit 的密钥存储和信任存储算法的配置属性(#24076)

该/actuator 发现页面现在可以使用禁用 management.endpoints.web.discovery.enabled 属性。

在/actuator/configprops 与 actuator/env 端点现在已经 additional-keys-to-sanitize 可以用来消毒键属性。

如果要自定义 JMX 执行器端点的名称，现在可以使用 EndpointObjectNameFactory 。

DataSourceBuilder.derivedFrom(…​)添加了一种新方法，允许您构建 DataSource 从现有方法派生的新方法。

当 Spring Security 在类路径上时，配置属性现在可以绑定到 RSAPublicKey 和 RSAPrivateKey。

ConnectionFactorySpring AMQP 使用的 RabbitMQ 现在可以使用 ConnectionFactoryCustomizerbean 进行自定义。

现在可以使用新的 spring.datasource.embedded-database-connection 配置属性来控制自动配置的嵌入式数据库。它可以设置 EmbeddedDatabaseConnection 为 none 的任何值 ，包括 完全禁用自动配置的嵌入式数据库。

CloudPlatform 现在可以自动检测 Azure 应用服务。

server.tomcat.keep-alive-timeout 可用于配置 Tomcat 在关闭保持连接之前等待另一个请求的时间。

server.tomcat.max-keep-alive-requests 可用于控制在关闭前保持活动连接可以发出的最大请求数。

spring.webflux.session.cookie.same-site 可用于配置 WebFlux 的 SameSite cookie 策略。默认是宽松的。

Apache HttpClient 5 现在自动配置为与 WebClient 一起使用。

ApplicationEnvironment 引入了一个新类，它应该可以提高一个小的性能提升。

您现在可以使用该 spring.netty.leak-detection 属性配置 Netty 内存。

### 2.20 依赖升级

Spring Boot 2.5 迁移到几个 Spring 项目的新版本：

```md
[Spring Data 2021.0](https://spring.io/blog/2021/04/20/what-s-new-in-spring-data-2021-0)

[Spring Integration 5.5](https://docs.spring.io/spring-integration/docs/5.5.x/reference/html/whats-new.html#whats-new)

[Spring Security 5.5](https://docs.spring.io/spring-security/site/docs/5.5.x/reference/html5/#new)

Spring Session 2021.0

Spring HATEOAS 1.3

[Spring Kafka 2.7.0](https://spring.io/blog/2021/04/14/spring-for-apache-kafka-2-7-0-available)
```

还更新了许多第三方依赖项，其中一些更值得注意的是：

```md
Kotlin 1.5

Groovy 3.0

Flyway 7.7

Liquibase 4.3

Jackson 2.12

Kafka 2.7

Cassandra Driver 4.10

Embedded Mongo 3.0

Hibernate Validator 6.2

Jersey 2.33

Mockito 3.7

MongoDB 4.2

JUnit Jupiter 5.7

Elasticsearch 7.12
```

## 3. Spring Boot 2.5 中的弃用

在 Spring Boot 2.5 中进行了以下显着的弃用

（1）ActuatorMediaType 并 ApiVersion 在 org.springframework.boot.actuate.endpoint.http 有利于当量的在 org.springframework.boot.actuate.endpoint

（2）支持实现 jOOQ 的\*Provider 回调接口或 Settings 已被弃用的 bean 。DefaultConfigurationCustomizer 应该使用 A 来代替。

（3）EntityManagerFactoryDependsOnPostProcessor 在 org.springframework.boot.autoconfigure.data.jpa 已迁往 org.springframework.boot.autoconfigure.orm.jpa

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

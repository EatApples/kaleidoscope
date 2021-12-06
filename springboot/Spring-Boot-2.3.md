# Spring-Boot-2.3

## 1. 从 Spring Boot 2.2 升级

### 1.1 最低要求的变化

Spring Boot 现在需要：

- Gradle 6.3+（如果您使用 Gradle 构建）。5.6.x 也受支持，但已弃用
- Jetty 9.4.22+（如果您使用 Jetty 作为嵌入式容器）

### 1.2 validation 不再包含在 Web 启动器中

如果您的应用程序正在使用验证功能，例如您发现 `javax.validation.*` 未解析导入，则需要自己添加启动器。

对于 Maven 构建，您可以使用以下方法执行此操作：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 1.3 默认唯一 DataSource 名称

默认情况下，在启动时自动生成 DataSource 的唯一标识。 这会影响 H2 控制台的使用，因为数据库 URL 不再引用 testdb。

您可以通过设置 spring.datasource.generate-unique-name=false 为来禁用此行为。

### 1.4 Spring Data Neumann

Spring Boot 2.3 附带一个主要版本的 Spring Data。如果您使用 Cassandra、Couchbase、Elasticsearch、MongoDB 或 JDBC，则需要格外小心。

（1）Cassandra

此版本切换到 Cassandra v4，带来了许多向后不兼容的更改。

如果您依赖于 ClusterBuilderCustomizer 自定义 Cluster，则此概念在 v4 中不再存在，并已被两个更具体的自定义程序取代：

- DriverConfigLoaderBuilderCustomizer 自定义驱动程序的属性。请将此用于尚未公开的任何属性。

- CqlSessionBuilderCustomizer 自定义 CqlSession（前 Session）。

Cassandra v4 驱动程序不再具有从接触点自动进行本地 DC 推断的功能。因此，必须使用默认的负载平衡策略设置“本地数据中心”属性，并且联系点必须位于该数据中心。spring.data.cassandra.local-datacenter 添加了新属性来设置本地数据中心。

（2）Couchbase

此版本切换到 Couchbase SDK v3，带来了许多向后不兼容的更改。

- 要连接到群集，你现在应该使用 spring.couchbase.connection-string，而不是前者 spring.couchbase.bootstrap-hosts

- 基于角色的访问控制现已通用化

- Spring Boot 不再自动配置 Bucke，但您可以使用 ClusterAPI 轻松实现

- Endpoints IO 配置在 spring.couchbase.env.io

* 如果您要扩展 CouchbaseConfiguration 以自定义环境，请 ClusterEnvironmentBuilderCustomizer 以更惯用的方式使用它

* 如果您将 Couchbase 与 Spring Data 一起使用，则需要提供存储桶名称。

（3）Elasticsearch

已弃用的 Native Elasticsearch 传输已被删除，因为 Elasticsearch 和 Spring Data 本身在其下一个版本中都不支持它。

此版本中还删除了对 Jest 库的支持。

Spring Boot 现在默认使用 Elasticsearch 7.5+。

（4）MongoDB

此版本切换到 MongoDB 4 并协调反应式和命令式驱动程序。如果您使用的是 starter，这对您来说应该是非常透明的。

通过这种协调，如果您使用，将不再提供非反应性基础结构 spring-boot-starter-data-mongodb-reactive。

如果您需要在启动时使用命令式基础结构（例如 MongoOperations），请考虑添加 spring-boot-starter-data-mongodb。

（5）Neo4j

Neo4j 的视图拦截器中的打开会话现在默认为禁用。

如果需要再次启用它，请使用 standardspring.data.neo4j.open-in-view 属性。

Neo4j 运行状况指示器的详细信息现在包含 version 和 edition 的服务器，如以下示例所示：

```json
neo4j: {
  status: "UP",
  details: {
    edition: "community",
    version: "4.0.0"
  }
}
```

（6）JDBC

在其[新功能](https://docs.spring.io/spring-data/jdbc/docs/2.0.0.RELEASE/reference/html/#new-features.2-0-0)中，Spring Data JDBC 2.0 现在默认情况下引用标识符。

这种行为可以通过调用被禁用 setForceQuote(false)的 RelationalMappingContext。

### 1.5 Micrometer

此版本升级到 Micrometer 1.5，带来许多弃用：

- 服务级别协议已重命名为“服务级别目标”，并且边界用 double 而不是表示 long。

- Wavefront 指标现在通过导出 WavefrontSender。读取和连接超时属性废弃。

### 1.6 Jackson

此版本升级到 Jackson 2.11，其中包括对 java.util.Date 和 java.util.Calendar 的默认格式的更改。

### 1.7 Spring Cloud Connectors 启动器已被删除

Spring Cloud Connectors starter 在 2.2 中被弃用，取而代之的是 Java CFEnv。

这个 starter 已经被移除，Spring Cloud Connectors 依赖不再包含在 Spring Boot 的托管依赖中。

### 1.8 嵌入式 Servlet Web 服务器线程配置

用于配置嵌入式 Servlet Web 服务器（Jetty、Tomcat 和 Undertow）使用的线程的配置属性已移至专用 threads 组。

属性现在可以发现 server.jetty.threads，server.tomcat.threads 和 server.undertow.threads。

旧属性仍保留为弃用形式以简化迁移。

### 1.9 对默认错误页面内容的更改

默认情况下，错误消息和任何绑定错误不再包含在默认错误页面中。这降低了将信息泄露给客户端的风险。

server.error.include-message 并且 server.error.include-binding-errors 可以分别用于控制消息的包含和绑定错误。

支持的值是 always，on-param 和 never。

### 1.10 磁盘空间运行状况指示器

自动配置的磁盘空间运行状况指示器不再需要应用程序启动时所监视的路径存在。

一条不存在的路径将被检测为具有零可用空间，从而导致指示器的响应下降。

### 1.11 自动创建 developmentOnlyGradle 配置

该 developmentOnly 配置主要用于声明对 Spring Boot 的 DevTools 的依赖，现在由 Spring Boot 的 Gradle 插件自动创建。

任何手动配置都 developmentOnly 应该从您的 Gradle 构建脚本中删除，因为它的存在会导致构建失败并显示消息 cannot add a configuration with name 'developmentOnly' as a configuration with that name already exists。

### 1.12 移除 Maven 站点插件管理

Spring Boot 的构建不再使用站点插件（maven-site-plugin），并且其插件管理已删除。如果您依赖于 Spring Boot 的托管版本，则应该添加自己的插件管理。

### 1.13 删除 Maven Exec 插件自定义配置

如果您从继承 spring-boot-starter-parent，它将不再配置 Maven 的 exec 插件（exec-maven-plugin）来使用来设置主类 start-class。

如果您依赖于此，则可以按以下方式恢复该行为：

```xml
<plugin>
   <groupId>org.codehaus.mojo</groupId>
   <artifactId>exec-maven-plugin</artifactId>
   <configuration>
       <mainClass>${start-class}</mainClass>
   </configuration>
</plugin>
```

### 1.14 ApplicationContextRunner 默认禁用 bean 覆盖

为了与 保持一致 SpringApplication，ApplicationContextRunner 现在默认禁用 bean 覆盖。

如果您需要使用 bean 覆盖进行测试，withAllowBeanDefinitionOverriding 可以启用它。

### 1.15 激活多个配置文件 @ActiveProfiles

现在，@ActiveProfiles 注释支持包含逗号的配置文件名称。这意味着像这样的注释@ActiveProfiles("p1,p2")会将提供的值 p1,p2 视为单个配置文件名称。

要激活多个配置文件，请像中提供每个配置文件名称作为单独的值@ActiveProfiles({"p1","p2"})。

### 1.16 WebServerInitializedEvent 和 ContextRefreshedEvent

作为引入对优雅关闭支持的一部分，Web 服务器初始化现在在应用程序上下文刷新处理结束时执行，而不是在刷新处理完成后立即执行。

因此，WebServerInitializedEvent 现已在 ContextRefreshedEvent 之前发布。

### 1.17 从 Spring Boot 2.2 弃用

在此发行版中删除了 Spring Boot 2.2 中不推荐使用的大多数类，方法和属性。请确保升级之前不调用不推荐使用的方法。

许多属性已被重命名或弃用。您可以使用该 spring-boot-properties-migrator 模块来识别那些属性。一旦添加为项目的依赖项，它不仅会分析应用程序的环境并在启动时显示诊断信息，还会在运行时为您临时迁移属性。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-properties-migrator</artifactId>
	<scope>runtime</scope>
</dependency>
```

## 2. 新特性和值得注意的

注意：检查[配置更改日志](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3.0-Configuration-Changelog)以获取配置更改的完整概述。

### 2.1 Java 14 支持

Spring Boot 2.3 添加了对 Java 14 的支持。也支持 Java 8 和 11。

### 2.2 使用 Cloud Native Buildpacks 构建 OCI 镜像

支持使用 Cloud Native Buildpacks 构建 Docker 镜像，并已通过 spring-boot:build-image goal 和 bootBuildImage task 添加到 Maven 和 Gradle 插件中。

默认情况下使用 Paketo Java buildpack 创建镜像。

### 2.3 构建分层 jar 以包含在 Docker 映像中

Maven 和 Gradle 插件已添加了对将 jar 文件的内容分层的支持。分层根据 jar 包的内容更改频率将其分隔。这种分离允许构建更有效的 Docker 镜像。可以重用未更改的现有图层，并将已更改的图层放置在顶部。

根据您的应用程序，您可能需要调整图层的创建方式和添加新图层的方式。这可以通过使用配置来完成，该配置描述了如何将 jar 分为几层，以及这些层的顺序。

当您创建分层 jar 时，该 spring-boot-jarmode-layertoolsjar 将默认添加为您的 jar 的依赖项（这可以通过构建配置禁用）。使用类路径上的这个 jar，您可以在特殊模式下启动您的应用程序，该模式允许引导代码运行与您的应用程序完全不同的东西，例如，提取图层的东西。要查看可用选项，请使用-Djarmode=layertools 以下示例中所示的方式启动 fat jar ：

```s
$ java -Djarmode=layertools -jar my-app.jar
Usage:
  java -Djarmode=layertools -jar my-app.jar

Available commands:
  list     List layers from the jar that can be extracted
  extract  Extracts layers from the jar for image creation
  help     Help about any command
```

### 2.4 jar 解压缩时的可预测的类路径顺序

使用 Maven 和 Gradle 构建的 jar 现在包含索引文件。当 jar 解压时，此索引文件用于确保类路径的顺序与直接执行 jar 时的顺序相同。

### 2.5 支持配置文件的通配符位置

加载配置文件时，Spring Boot 现在支持通配符位置。默认情况下，`config/*/jar`外部的通配符位置受支持。

当有多个配置属性源时，这在 Kubernetes 等环境中很有用。例如，如果您有单独的 mysql 和 redis 配置，则将它们放在/config，即/config/mysql/application.properties 和中可以自动选择它们/config/redis/application.properties。

### 2.6 优雅关机

所有四个嵌入式 Web 服务器（Jetty、Reactor Netty、Tomcat 和 Undertow）以及反应式和基于 Servlet 的 Web 应用程序都支持正常关闭。

设置 server.shutdown=graceful 即可启用，应用关闭时，Web 服务器将不再允许新请求，并将等待活动请求完成的宽限期。可以使用 spring.lifecycle.timeout-per-shutdown-phase 配置宽限期。

有关更多详细信息，请参阅[参考文档](https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/html/spring-boot-features.html#boot-features-graceful-shutdown)。

### 2.7 Liveness 和 Readiness 探针

Spring Boot 现在具有有关应用程序可用性的内置知识，可以跟踪应用程序是否处于活动状态以及是否已准备好处理流量。

可以将运行状况端点配置为通过配置 management.health.probes.enabled=true 属性公开应用程序的活动性（/actuator/health/liveness）和就绪状态（/actuator/health/readiness）。

在 Kubernetes 上运行时，这是自动完成的。

要了解有关此功能的[更多信息](https://spring.io/blog/2020/03/25/liveness-and-readiness-probes-with-spring-boot)，请查看此博客文章及其链接的参考文档。

### 2.8 Spring Data Neumann

Spring Boot 2.3 附带了一个主要的 Spring Data 版本。请参阅[Spring Data Neumann](https://spring.io/blog/2020/05/12/spring-data-neumann-goes-ga) 了解更多信息。

当 r2dbc 在类路径上时，ConnectionFactory 将以与 jdbc 类似的方式自动配置 DataSource。如果 Spring Data 在类路径上，则存储库也将像往常一样自动配置。

R2DBC 支持还添加了连接工厂的运行状况指示器，ConnectionPool metrics 和测试部分 @DataR2dbcTest。

### 2.9 WebFlux 应用程序的可配置基本路径

现在可以配置 WebFlux 应用程序的所有 Web 处理程序的基本路径。使用该 spring.webflux.base-path 属性来执行此操作。

### 2.10 Web 应用程序中的日期时间转换

现在可以通过应用程序属性配置 Web 应用程序中的时间和日期时间值的转换。这补充了对格式化日期值的现有支持。

对于 MVC，属性分别为 spring.mvc.format.time 和 spring.mvc.format.date-time。

对于 WebFlux，属性分别是 spring.webflux.format.time 和 spring.webflux.format.date-time。

除了采用典型的格式设置模式外，用于配置日期，时间和日期时间格式的属性现在还支持值 iso。设置后，将应用相应的 ISO-8601 格式。

iso 下列属性支持这些值：

```yml
spring.mvc.format.date

spring.mvc.format.date-time

spring.mvc.format.time

spring.webflux.format.date

spring.webflux.format.date-time

spring.webflux.format.time
```

### 2.11 Actuator 改进

（1）配置属性的端到端可追溯性

/actuator/configprops 端点提供有关配置属性的端到端信息，以使其行为与环境端点保持一致。例如，在添加在 application.properties 添加 server.server-header=Spring Boot，端点将显示以下内容：

```log
"serverHeader": {
  "origin": "class path resource [application.properties]:2:22",
  "value": "Spring Boot"
},
```

（2）指标端点中的名称按字母顺序排列

可用的指标名称/actuator/metrics/现在按字母顺序排列，这样可以更轻松地找到您要查找的内容。

（3）无查询数据源健康指标
在没有验证查询的情况下，数据源 HealthIndicator 现在以无查询模式运行，java.sql.Connection#isValid 用于验证连接。

（4）为 Web MVC 和 WebFlux 指标提供其他标签
除了默认为 MVC 和 WebFlux 提供的标记外，现在还可以提供这些标记。可以使用 WebMvcTagsContributor @BeanMVC 和 WebFluxTagsContributor @BeanWebFlux 进行贡献。

（5）自动配置 Wavefront 的发送器

Wavefront 的自动配置已更新，可以定义一个 WavefrontSenderbean。这允许通过单个连接将指标和跟踪发布到 Wavefront。

（6）原生 Kafka 指标
Kafka 指标是为自动配置 ConsumerFactory 和创建的消费者和生产者本地发布的 ProducerFactory。

要为由自定义工厂创建的组件生成度量标准，应添加一个侦听器，如以下示例所示：

```java
factory.addListener(new MicrometerConsumerListener<>(meterRegistry));
```

注意：如果仅出于收集 Kafka 指标的目的而启用 JMX 支持，则不再需要此功能。

### 2.12 RSocket 对 Spring 集成的支持

Spring Boot 现在为 Spring Integration 的 RSocket 支持提供自动配置。

如果 spring-integration-rsocket 可用，开发人员可以使用`spring.rsocket.server.*`属性配置 RSocket 服务器，并使其使用 IntegrationRSocketEndpoint 或 RSocketOutboundGateway 组件来处理传入的 RSocket 消息。

### 2.13 绑定到 Period

如果一个属性需要表达一段时间，你可以使用一个 java.time.Period 属性来实现。与 Duration 支持类似，支持一种简单的格式（即 10w 表示持续 10 周）以及元数据支持。

### 2.14 Web 服务的切片测试

@WebServiceClientTest 添加了一个新注释以支持 Web 服务的“切片”测试。

### 2.15 依赖升级

Spring Boot 2.3 依赖的几个 Spring 项目的新版本：

- Spring Data Neumann
- Spring HATEOAS 1.1
- Spring Integration 5.3
- Spring Kafka 2.5
- Spring Security 5.3
- Spring Session Dragonfruit

请注意，Spring Boot 2.3 与 Spring Boot 2.2 建立在相同的 Spring Framework 和 Reactor 生成之上。

许多第三方依赖项也已更新，其中一些更值得注意的是：

```s
Artemis 2.12

AssertJ 3.16

Cassandra Driver 4.6

Couchbase Client 3.0

Elasticsearch 7.6

Flyway 6.4

Hibernate Validator 6.1

Infinispan 10.1

Jackson 2.11

JUnit Jupiter 5.6

Kafka 2.5

Kotlin 1.3.72

Lettuce 5.3

Micrometer 1.5

Mockito 3.3

MongoDB 4.0

QueryDSL 4.3

```

### 2.16 其他改进

除了上面列出的更改之外，还进行了许多小的调整和改进，包括：

- 我们在 JPA 支持中更新了默认配置，以改善测试体验
- spring-boot-autoconfigure-processor 的输出现在是可重复的，从而使其与 Gradle 的构建缓存更好地配合工作
- Couchbase 的类型密钥可以通过 spring.data.couchbase.type-key 进行配置
- OAuth2 参数绑定现在可用于@WebMvcTest
- 可以使用 server.jetty.max-queue-capacity 配置 Jetty 的后备队列
- 可以使用 spring.liquibase.tag 来配置 Liquibase 的标签支持。现在可以通过该 spring.liquibase.clear-checksums 属性清除当前变更日志中的所有校验和
- Gradle 元数据现已发布
- DataSourceBuilder 可以用来配置 SimpleDriverDataSource
- DataSource 指标现在有描述。
- 可以使用 spring.main.cloud-platform 覆盖云平台的自动检测
- 当请求具有认证时，现在支持缓存来自启动器的 HTTP 端点的响应

- 现在，Maven 支持创建 jar 中的 project.build.outputTimestamp 属性，从而使其输出可重现
- Maven 插件的 Javadoc 现在已发布
- 提供了一个定制器接口 rsocketmessagehandlercustimizer，用于定制自动配置的 RSocketMessageHandler
- 提供了一个定制器界面，DefaultCookieSerializerCustomizer 用于定制自动配置的 DefaultCookieSerializer
- 现在可以通过设置 server.servlet.register-default-servlet=false 为来禁用默认 servlet 的自动配置
- 添加了一个新条件@ConditionalOnWarDeployment。它可用于检测应用程序何时作为战争部署到 Servlet 容器或应用程序服务器
- 属性迁移器处理所有已弃用的属性，而不仅仅是具有错误级别的属性
- 销毁 war 的 ServletContext 时，将注销 JDBC 驱动程序。
- Redis 的哨兵密码可以使用 spring.redis.sentinel.password 配置

## 3. Spring Boot 2.3 中的弃用

（1）spring.http 相关属性已经被迁移到，server.servlet.encoding., spring.mvc. and spring.codec.

```s
spring.http.encoding.* -> server.servlet.encoding.*
spring.http.log-request-details -> spring.mvc.log-request-details (Spring MVC) or spring.codec.log-request-details (Spring WebFlux)
spring.http.converters.preferred-json-mapper -> spring.mvc.converters.preferred-json-mapper
```

（2）使用 SpringApplication#refresh(ConfigurableApplicationContext) ，而不是 SpringApplication#refresh(ApplicationContext)

（3）在 ON_TRACE_PARAM 与所使用的 server.error.include-stacktrace 属性已被重命名为 ON_PARAM

（4）使用 org.springframework.boot.autoconfigure.elasticsearch.RestClientBuilderCustomizer，而不是 org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientBuilderCustomizer

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

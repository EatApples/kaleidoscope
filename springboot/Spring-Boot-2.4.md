# Spring-Boot-2.4

## 1. 从 Spring Boot 2.3 升级

### 1.1 版本方案变更

从 2.4 开始，Spring Boot 采用了 the new Spring versioning scheme——这意味着你应该从  2.3.5.RELEASE 开始更新 build.gradle/pom.xml 文件中的 Spring Boot 版本到 2.4.0。

### 1.2 JUnit 5 的老式引擎从 spring-boot-starter-test 移除

如果您升级到 Spring Boot 2.4，并看到诸如 org.junit.Test 的 JUnit 类的测试编译错误。这可能是因为 JUnit 5 的老式引擎已经从 spring-boot-starter-test 中移除。

老式引擎允许用 JUnit 4 编写的测试由 JUnit 5 运行。

如果您不想将您的测试迁移到 JUnit 5，并且希望继续使用 JUnit 4，那么添加一个对老式引擎的依赖，如下面的 Maven 示例所示:

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 1.3 配置文件处理（应用程序属性和 YAML 文件）

Spring Boot 2.4 改变了处理 application.properties 和 application.yml 文件的处理方式。

如果你只有一个简单的 application.properties 或者 application.ym 文件，你的升级应该是无缝的。但是，如果您有一个更复杂的设置(带有特定于概要文件的属性，或者概要文件激活属性)，那么如果您想使用新特性，您可能需要做 [some changes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-Config-Data-Migration-Guide)。

如果你只想要 Spring Boot 2.3 兼容的逻辑，你可以在你的 application.properties 或者 application.yml 文件设置一个 spring.config.use-legacy-processing 属性为 true。

### 1.4 配置数据导入

通过 spring.config.location 和 spring.config.import(在本版本中引入)明确指定配置位置，如果文件或文件夹不存在，将不再静默失败。如果您想导入一个位置，但如果找不到它也不介意跳过它，那么现在应该在它前面加上 optional:

例如, spring.config.location=optional:/etc/config/application.properties，如果它存在将从/etc/config/导入 application.properties 文件 ,并且如果它不存在将跳过。

如果你想把所有的位置都当作 optional ，你可以设置 spring.config.on-not-found=ignore 在 SpringApplication.setDefaultProperties(…)或在系统/环境变量设置。

### 1.5 嵌入式数据库检测

嵌入式数据库逻辑经过了改进，只有在内存中才认为数据库是嵌入式的。如果在 H2、HSQL 和 Derby 中使用基于文件的持久性或服务器模式，则此更改有两个后果:

（1）sa 用户名不再设置。如果您依赖于该行为，则需要设置  spring.datasource.username=sa 在您的配置。

（2）这样的数据库将不再在启动时被初始化，因为它们不再被认为是嵌入式的。您可以像往常一样使用 pring.datasource.initialization-mode 调优它。

### 1.6 用户定义的 MongoClientSettings 不再自定义

如果您提供自己的 MongoClientSettings bean，则它不再由自动配置自定义。如果您依赖该行为，特别是与嵌入式 Mongo 结合使用，请考虑将定制器应用于您自己的 bean，如下例所示：

```java
@Bean
public MongoClientSettings userDefinedMongoClientSettings(MongoProperties properties, Environment environment) {
   MongoClientSettings.Builder builder = MongoClientSettings.builder();
   //...
   new MongoPropertiesClientSettingsBuilderCustomizer(properties, environment).customize(builder);
   return builder.build();
}
```

### 1.7 Logback 配置属性

特定于 Logback 的 Logging properties 已被重命名，以反映它们是特定于 Logback 的事实。以前的名称已被弃用。

下面 Spring Boot properties 已经被改变:

```yml
logging.pattern.rolling-file-name → logging.logback.rollingpolicy.file-name-pattern

logging.file.clean-history-on-start → logging.logback.rollingpolicy.clean-history-on-start

logging.file.max-size → logging.logback.rollingpolicy.max-file-size

logging.file.total-size-cap → logging.logback.rollingpolicy.total-size-cap

logging.file.max-history → logging.logback.rollingpolicy.max-history
```

以及它们映射到的系统环境属性:

```yml
ROLLING_FILE_NAME_PATTERN → LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN

LOG_FILE_CLEAN_HISTORY_ON_START → LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START

LOG_FILE_MAX_SIZE → LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE

LOG_FILE_TOTAL_SIZE_CAP → LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP

LOG_FILE_MAX_HISTORY → LOGBACK_ROLLINGPOLICY_MAX_HISTORY
```

### 1.8 默认 Servlet 注册

Spring Boot 2.4 将不再注册 servlet 容器提供的 DefaultServlet。在大多数应用程序中，它不被使用，因为 Spring MVC 的 DispatcherServlet 是唯一需要的 servlet。

如果您发现仍然需要默认的 servlet，你可以设置 server.servlet.register-default-servlet=true。

### 1.9 默认情况下，HTTP 跟踪不再包含 cookie 标头

默认情况下，Cookie 请求头和 Set-Cookie 响应头不再包含在 HTTP traces 。

为了恢复 Spring Boot 2.3 的行为，设置 management.trace.http.include 进 cookies, errors, request-headers, response-headers

### 1.10 Undertow 转发路径

默认情况下，当请求被转发时，Undertow 保留原始的请求 URL。这个版本覆盖了 Undertow 默认值以符合 Servlet 规范。

之前的 Undertow 默认行为可以通过设置属性 server.undertow.preserve-path-on-forward=true 来恢复。

### 1.11 Neo4j

这个版本对 Neo4j 的支持进行了重大的调整。在 spring.data.neo4j.\*中有许多属性已经移除，同时也移除了对 Neo4j OGM 的支持。

Neo4j 驱动程序的配置是通过 spring.neo4j.\*命名空间完成的。尽管 URI 和基本身份验证仍然以一种弃用的方式支持。

想要了解更多关于这一变化以及 Spring Data Neo4j 6 带来了什么，[check the documentation](https://docs.spring.io/spring-data/neo4j/docs/6.0.x/reference/html/)。

### 1.12 Hazelcast 4

此版本升级到 Hazelcast 4，同时保持与 Hazelcast 的兼容性 3.2.x。如果您还没有准备好切换到 Hazelcast 4，您可以使用 hazelcast.version 构建中的属性降级。

### 1.13 Elasticsearch RestClient

低级 Elasticsearch RestClient bean 将不再由 Spring Boot 自动配置。RestHighLevelClient bean 仍然是自动配置的。

大多数用户不需要使用低级客户端，也不应该受到此更改的影响。

### 1.14 R2DBC

R2DBC 的核心基础设施已经转移到 Spring 框架，并提供了一个新的 spring-r2dbc 模块。

如果您正在使用这个基础设施，请确保将已弃用的访问迁移到新的核心支持。

### 1.15 Flyway

Flyway 7 的升级包括了回调顺序的[some changes](https://github.com/flyway/flyway/issues/2785)。这将是一个突破性的变化，任何人依赖注册订单，我们支持通过@Order 和 Ordered。

如果您正在使用 Flyway 5，请确保在升级到 Spring Boot 2.4 之前升级到 Flyway 6，因为 Flyway 只为一个特性版本保留模式升级。

### 1.16 删除 Flatten Maven 插件的插件管理

Spring Boot 的构建不再使用 Flatten Maven 插件(flatten-maven-plugin)，它的插件管理也被删除了。

如果你依赖 Spring Boot 的托管版本，你应该添加你自己的插件管理。

### 1.17 exec-maven-plugin 版本管理

exec-maven-plugin 的版本管理已经被删除。如果您正在使用这个插件，请确保在您自己的 pluginManagement 中指定一个版本。

### 1.18 Spring Boot Gradle 插件

Spring Boot Gradle PluginbootJar 任务的 DSL 已更新，以便 mainClass 可以使用 Property<String>。如果您当前使用 mainClassName，例如：

```yml
bootJar {
mainClassName 'com.example.ExampleApplication'
}
```

您应该将其更改为 mainClass：

```yml
bootJar {
mainClass 'com.example.ExampleApplication'
}
```

### 1.19 集成测试中的指标导出

@SpringBootTest 不再配置可用的监控系统，只提供内存中的 MeterRegistry。如果您将度量作为集成测试的一部分导出，那么您可以将@AutoConfigureMetrics 添加到您的测试中，以恢复以前的行为。

### 1.20 Spring Boot 2.2 和 2.3 的弃用

为反映 Spring Boot 版本兼容性策略，Spring Boot 2.2 中弃用的代码已在 Spring Boot 2.4 中删除。Spring Boot 2.3 中弃用的代码仍然存在，并计划在 Spring Boot 2.5 中删除。

## 2. 新特性和值得注意的

注意：检查[配置更改日志](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.4.0-Configuration-Changelog)以获取配置更改的完整概述。

### 2.1 Spring Framework 5.3

Spring Boot 2.4 使用 Spring Framework 5.3。Spring Framework wiki 中有一个[what’s new section](https://github.com/spring-projects/spring-framework/wiki/What%27s-New-in-Spring-Framework-5.x)部分详细介绍了这个新版本。

### 2.2 Spring Data 2020.0

Spring Boot 2.4 包含了 Spring Data 发布系列的 2020.0 版本(代码名为 Ockham)。有关发布细节，请参阅[Spring Data wiki](https://github.com/spring-projects/spring-data-commons/wiki/Release-Train-Ockham-%282020.0.0%29)。

### 2.3 Neo4j

这个版本带来了对反应性存储库的支持，并依赖于一个单独的 Neo4j 驱动程序的自动配置。因此，现在可以在有或没有 Spring Data.的情况下使用 Neo4j。

Neo4j 的健康检查使用驱动程序，只要配置了 Neo4j 驱动程序，就可以进行健康检查。

如果您希望将@Transactional 与响应式访问一起使用，您现在需要自己配置 Neo4jReactiveTransactionManager bean。

```java
@Bean(ReactiveNeo4jRepositoryConfigurationExtension.DEFAULT_TRANSACTION_MANAGER_BEAN_NAME)
public ReactiveTransactionManager reactiveTransactionManager(Driver driver,
      ReactiveDatabaseSelectionProvider databaseNameProvider) {
    return new ReactiveNeo4jTransactionManager(driver, databaseNameProvider);
}

```

### 2.4 R2DBC

AR2dbcEntityTemplate 可用于通过实体简化 Reactive R2DBC 的使用

### 2.5 Java 15 支持

Spring Boot 2.4 现在完全支持（并针对）Java 15。支持的最低版本仍然是 Java 8。

### 2.6 自定义属性名称支持

使用构造函数绑定时，属性的名称派生自参数名称。如果您想使用 java 保留关键字，这可能会成为一个问题。对于这种情况，现在可以使用@Name 注释，类似于:

```java
@ConfigurationProperties(prefix = "sample")
@ConstructorBinding
public class SampleConfigurationProperties {

  private final String importValue;

  public SampleConfigurationProperties(@Name("import") String importValue) {
    this.importValue = importValue;
  }

}

```

上面的示例展示了一个 sample.import property。

### 2.7 默认启用分层 jar

这个版本启用分层 jar，并默认包含 layertools。这应该可以使用开箱即用的构建包提高生成映像的效率，并让您在制作自定义[Dockerfile](https://docs.spring.io/spring-boot/docs/2.4.0/reference/html/spring-boot-features.html#layering-docker-images)受益于该特性。

### 2.8 导入其他应用程序配置

只要您还没有设置 spring.config.use-legacy-processing 为 true，您现在就可以直接从主文件 application.properties 或.yaml 文件导入其他属性和 yaml 文件 application.yml。

您可以使用该 spring.config.import 属性来指定一个或多个应导入 Spring 的附加配置文件 Environment。有关更多详细信息，请参阅参考指南的[这一部分](https://docs.spring.io/spring-boot/docs/2.4.0/reference/html/spring-boot-features.html#boot-features-external-config-files-importing)。

我们发布了一篇简短的博客，解释了我们进行这些[更改的原因](https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)。

### 2.9 卷挂载配置目录树

该 spring.config.import 属性还可用于导入 Kubernetes 常用的配置树。配置树是提供键/值对的另一种方式。每对都在自己的文件中声明，文件名构成属性键，文件内容提供值。

有关完整示例，请参阅更新的[参考文档](https://docs.spring.io/spring-boot/docs/2.4.0/reference/html/spring-boot-features.html#boot-features-external-config-files-configtree)。

默认情况下，从配置树加载的属性会修剪尾随的换行符。

### 2.10 导入没有文件扩展名的配置文件

某些云平台只允许您批量挂载没有文件扩展名的文件。如果您有这样的约束，现在可以通过向 Spring Boot 提供有关内容类型的提示来导入这些文件。例如，spring.config.import=/etc/myconfig[.yaml]将加载/etc/myconfig 为 YAML。

### 2.11 源链

使用新的 getParent()方法更新了 Origin 接口。这允许我们提供一个完整的来源链，可以准确地显示一个项目的来源。

例如，您可以使用 spring.config.import 在你的 application.properties 来导入第二个文件。从第二个文件加载的属性的起源将有一个指向原始导入声明的父元素。

您可以通过查看 actuator/env 或 actuator/configprops 执行器端点的输出来自己尝试一下。

### 2.12 Startup Endpoint

startup 现在提供了一个新的执行器端点，可显示有关您的应用程序启动的信息。端点可以帮助您识别启动时间比预期更长的 bean。

这项工作建立在最近添加到 Spring Framwork 5.3 的应用程序启动跟踪功能的基础上。您可以在 Spring Framework 参考文档中阅读有关该功能的[更多信息](https://docs.spring.io/spring-framework/docs/5.3.x/reference/html/core.html#context-functionality-startup)。

[此处](https://docs.spring.io/spring-boot/docs/2.4.0/actuator-api/htmlsingle/#startup)记录了新的执行器 API 。

### 2.13 Docker/Buildpack 支持

（1）发布镜像

Maven 插件 spring-boot:build-image 目标和 Gradle 插件 bootBuildImage 任务现在能够将生成的图像发布到 Docker 注册表。有关配置用于发布图像的插件的更多详细信息，请参阅 Maven 和 Gradle 插件文档。

（2）验证
使用 Spring Boot 的 buildpack 支持时，您现在可以为构建器或运行映像使用经过身份验证的私有 Docker 注册表。支持用户名/密码和基于令牌的身份验证。

在 Maven 的和摇篮的文档进行了更新，以显示新的配置。

（3）Paketo Buildpack 默认值
Maven 插件 spring-boot:build-image 目标和 Gradle 插件 bootBuildImage 任务默认使用的镜像构建器已升级为最新的 Paketo 镜像。Paketo 镜像注册表已从 Google Container Registry 更改为 Docker Hub，以提高可访问性。

### 2.14 Maven Buildpack 支持

在 spring-boot:build-image 现在的 Maven 目标，把所有的项目模块的依赖关系，在“应用”层。这意味着如果您的构建中有多个项目模块，它们现在都将在同一层中结束。

XML 模式也已更新，以允许使用新元素<includeModuleDependencies/>和<excludeModuleDependencies/>元素自定义图层。

有关详细信息，请参阅更新的 [Maven 文档](https://docs.spring.io/spring-boot/docs/2.4.0/maven-plugin/reference/htmlsingle/#repackage-layers-configuration)。

### 2.15 Gradle Buildpack 支持

bootBuildImage Gradle 任务现在将所有项目模块的依赖关系放在“application”层。这意味着，如果您的构建中有多个项目模块，它们现在将全部结束在同一层。

在自定义层时，还可以在 DSL 中使用 includeProjectDependencies()和 excludeProjectDependencies()。

有关详细信息，请参阅更新的 [Gradle 文档](https://docs.spring.io/spring-boot/docs/2.4.0/gradle-plugin/reference/htmlsingle#packaging-layered-jars)。

### 2.16 Redis 缓存指标

如果你正在使用 Redis 缓存，你现在可以通过 Micrometer 公开缓存统计信息。记录的度量包括数量放置、获取和删除，以及点击/错过。挂起的请求数和锁等待时间也会被记录。

要启用该特性，设置 spring.cache.redis.enable-statistics=true

### 2.17 Web 配置属性

添加了属性以支持使用 Spring MVC 或 Spring WebFlux 配置 Web 区域设置和资源位置。新属性是：

```yml
spring.web.locale
spring.web.locale-resolver
spring.web.resources.*
```

添加了一个新属性以支持使用 servlet 或反应式 Web 堆栈配置执行器管理端点：

```yml
management.server.base-path
```

这些 Spring MVC 和 servlet 特定属性已被弃用，取而代之的是支持任一 Web 堆栈的新属性：

```yml
spring.mvc.locale
spring.mvc.locale-resolver
spring.resources.*
management.server.servlet.context-path
```

### 2.18 @WebListeners 允许以 servlet 和过滤器的方式注册

Servlet@WebListener 类现在以这样一种方式注册，它们可以自己注册 servlet 和过滤器。

早期版本的 Spring Boot 使用对 javax.servlet.Registration.Dynamic. 这意味着 Servlet 规范 (4.4) 的以下部分适用：

如果 ServletContext 传递给 ServletContextListener 的 contextInitialized 方法，其中 ServletContextListener 既未在 web.xml 或 web-fragment.xml 中声明，也未使用 @WebListener 进行注释，则必须为 ServletContext 中定义的所有方法抛出 UnsupportedOperationException 以编程配置 servlet、过滤器和听众。

从 Spring Boot 2.4 开始，我们不再使用动态注册，因此调用 event.getServletContext().addServlet(…​)和 event.getServletContext.addFilter(…​)从 ServletContextListener.contextInitialized 方法中是安全的。

### 2.19 Cassandra 的切片测试

使用@DataCassandraTest 可以使用额外的测试片来测试依赖 Cassandra 的组件。通常，默认情况下只配置 Cassandra 存储库和所需的基础设施。

下面是一个使用 Testcontainers 和@DynamicPropertSource 的例子：

```java
@DataCassandraTest(properties = "spring.data.cassandra.local-datacenter=datacenter1")
@Testcontainers(disabledWithoutDocker = true)
class SampleDataCassandraTestIntegrationTests {

	@Container
	static final CassandraContainer<?> cassandra = new CassandraContainer<>().withStartupAttempts(5)
			.withStartupTimeout(Duration.ofMinutes(2));

	@DynamicPropertySource
	static void cassandraProperties(DynamicPropertyRegistry registry) {
		registry.add("spring.data.cassandra.contact-points",
				() -> cassandra.getHost() + ":" + cassandra.getFirstMappedPort());
	}

	...

}
```

### 2.20 Flyway 7

此版本升级到 Flyway 7，带来了一些额外的特性。对于开源版本，我们添加了以下 spring.flyway 属性：

```yml
url
user
password
```

如果您使用的是“团队”版本，您还可以使用：

```yml
cherry-pick
jdbc-properties
oracle-kerberos-cache-file
oracle-kerberos-config-file
skip-executing-migrations
```

### 2.21 H2 控制台的 web 管理员密码的配置属性

引入了一个新的配置属性，spring.h2.console.settings.web-admin-password 用于配置 H2 控制台的 Web 管理员密码。密码控制对控制台首选项和工具的访问。

### 2.22 Apache Cassandra 的基于 CqlSession 的健康指标

引入了新的 CqlSession 基于健康指标的 CassandraDriverHealthIndicator 和 CassandraDriverReactiveHealthIndicator。当 Cassandra 的 Java 驱动程序位于类路径上时，这些指标之一将被自动配置。现有的基于 Spring Data Cassandra 的健康指标已被弃用。

### 2.23 使用 Prometheus 过滤抓取

Actuator 的 Prometheus 端点，/actuator/prometheus 现在支持一个 includedNames 查询参数，该参数可用于过滤响应中包含的样本。有关更多详细信息，请参阅执行器 [API 文档](https://docs.spring.io/spring-boot/docs/2.4.0/actuator-api/htmlsingle#prometheus-retrieving-names)。

### 2.24 Spring Security SAML 配置属性

已添加属性以允许配置 SAML2 信赖方注册的解密凭据和断言消费者服务 (ACS)。属性位于以下标题下：

spring.security.saml2.relyingparty.registration.decryption.\*

spring.security.saml2.relyingparty.registration.acs.\*

### 2.25 故障分析

即使没有创建 ApplicationContext，FailureAnalizers （故障分析器）也将被考虑。这也允许他们分析环境处理过程中抛出的任何异常。

注意，除非创建了 ApplicationContext，否则不会使用任何实现 BeanFactoryAware 或 EnvironmentAware 的分析器。

### 2.26 jar 优化

当生成可运行的 Spring Boot jar 时，空的启动器依赖项将被自动删除。由于大多数启动器只提供可传递的依赖项，所以将它们打包到最终 jar 中没有什么意义。

Spring Boot 注释处理器也被删除了，而且它们只在构建过程中有用。它们是 spring-boot- autoconfiguration -processor 和 spring-boot-configuration-processor。

如果您有自己的不包含代码的 starter POMs，您可以将 Spring-Boot-Jar-Type 的条目添加到它的 MANIFEST.MF 中，值为"dependencies-starter"。如果您想过滤掉一个注释处理器，您可以添加值为“annotation-processor”的相同属性。

### 2.27 其他优化

除了上面列出的更改之外，还有许多小的调整和改进，包括：

运行应用程序的 JVM 版本现在已在启动时记录。

自动从logging.config 的值中修剪尾随空格。

R2DBC 池支持公开了额外的配置属性。

异常处理 LdapTemplate 可以配置为忽略某些异常。

ISO 偏移日期时间格式支持 MVC 和 Webflux。

添加配置属性以选择加入新 PathPatternParser 的替代 AntPathMatcher 解析和匹配请求映射路径模式。

@DurationUnit, @DataSizeUnit, 并且@PeriodUnit 可以使用 注释构造函数参数@ConstructorBinding。

自动配置的 RabbitConnectionFactory 检查是否存在 CredentialsProvider 和 CredentialsRefreshService。

可以仅使用排除项来定义健康组。

AbstractRoutingDataSource 可以在使用 management.health.db.ignore-routing-data-sources.

可以配置 SAML 依赖方的 localEntityIdTemplate。

HTTP 跟踪是具有纳秒精度的度量。

FailureAnalyzer 当缺少 Liquibase 更改日志时，专用消息会提供有意义的消息。

Netty 的请求解码器可以使用 server.netty.\*属性进行自定义。

与 Spring Boot 版本管理的 Liquibase 版本一致的 Liquibase Maven 插件的插件管理。

Prometheus PushGateway 的基本身份验证支持。

当 Jedis 和 Lettuce 都可用时，允许选择 Jedis 使用 spring.redis.client-type.

允许使用 禁用 Redis 集群动态源刷新 spring.redis.lettuce.cluster.refresh.dynamic-sources。

参考文档现在包括 Properties 与 YAML 用于所有配置举例。

现在可以使用该 spring.rsocket.fragment-size 属性自定义 RSocketServer 的片段大小。

Logback 和 Log4j 日志记录使用的字符集现在可以使用属性 logging.charset.console 和 logging.charset.file.

当使用 Gradle 6.7 或更高版本构建 Spring Boot 应用程序时，支持 Gradle 的配置缓存。

### 2.28 依赖升级

Spring Boot 2.4 迁移到几个 Spring 项目的新版本：

```md
Spring AMQP 2.3 ([what’s new](https://docs.spring.io/spring-amqp/reference/html/#whats-new))

Spring Batch 4.3 ([what’s new](https://docs.spring.io/spring-batch/docs/current/reference/html/whatsnew.html#whatsNew))

Spring Data 2020.0 ([changelog](https://github.com/spring-projects/spring-data-commons/wiki/Release-Train-Ockham-%282020.0.0%29))

Spring Framework 5.3 ([what’s new](https://github.com/spring-projects/spring-framework/wiki/What%27s-New-in-Spring-Framework-5.x) | [upgrading](https://github.com/spring-projects/spring-framework/wiki/Upgrading-to-Spring-Framework-5.x#upgrading-to-version-53))

Spring Integration 5.4 ([what’s new](https://docs.spring.io/spring-integration/docs/current/reference/html/whats-new.html#whats-new))

Spring HATEOAS 1.2 ([migration guide](https://docs.spring.io/spring-hateoas/docs/1.2.0/reference/html/#migrate-to-1.0))

Spring Kafka 2.6 ([what’s new](https://docs.spring.io/spring-kafka/reference/html/#spring-kafka-intro-new))

Spring Retry 1.3

Spring Security 5.4 ([what’s new](https://docs.spring.io/spring-security/site/docs/5.4.0/reference/html5/#new))

Spring Session 2020.0
```

还更新了许多第三方依赖项，其中一些更值得注意的是：

```md
Artemis 2.13

AssertJ 3.18

Cassandra Driver 4.7

Elasticsearch 7.9

Flyway 7

Jersey 2.31

JUnit 5.7

Liquibase 3.10

Lettuce 6.0 ([release notes](https://github.com/lettuce-io/lettuce-core/releases/tag/6.0.0.RELEASE))

Micrometer 1.6 ([release notes](https://github.com/micrometer-metrics/micrometer/releases/tag/v1.6.0))

Mockito 3.4

MongoDB 4.1

Oracle Database 19.7

Reactor 2020.0 ([release notes](https://github.com/reactor/reactor/releases/tag/2020.0.0))

RSocket 1.1

Undertow 2.2
```

## 3. Spring Boot 2.4 中的弃用

（1）ConfigFileApplicationListener 已弃用，取而代之的是 ConfigDataEnvironmentPostProcessor。

（2）与 contextClass 相关的 SpringApplicationBuilder 和 SpringApplication 方法已经被弃用，取而代之的是使用 contextFactory。

（3）CloudFoundryVcapEnvironmentPostProcessor  的一些方法已被弃用，以处理 EnvironmentPostProcessor 更新(这些应该会影响大多数用户)。

（4）BuildLog 构建包支持类已经弃用了一些方法，并用提供更多细节的替代方法替换了它们。

（5）LoggingSystemProperties  中的 Logback 常量已被弃用，取而代之的是 LogbackLoggingSystemProperties。

（6）UndertowServletWebServerFactory 中的 isEagerInitFilters/setEagerInitFilters 方法已被 isEagerFilterInit/setEagerFilterInit 替换。

（7）为了支持 BootstrapContext, ApplicationEnvironmentPreparedEvent、ApplicationStartingEvent 和 SpringApplicationRunListener 中的一些方法已经被弃用。

（8）用于支持 buildpack 的 BuildLog 已经更新，以支持更多数据(大多数用户不会直接使用这个类)。

（9）一些 Spring MVC 和 servlet 特定属性已被弃用（请参阅上面的 Web 配置属性部分）。

（10）使用 Spring Data Cassandra 的健康指标已被弃用，取而代之的是使用原始驱动程序的健康指标。

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

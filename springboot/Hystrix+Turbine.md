### 1. Rest+Ribbon

> @HystrixCommand(fallbackMethod = "hiError")

> @LoadBalanced

### 2. Hystrix

较底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用达到一个阀值（hystrix 是5秒20次） 断路器将会被打开

熔断器默认错误率阈值为50%。

连接断路器的状态也暴露在呼叫应用程序的 /health 端点中。

在引入spring-boot-starter-actuator依赖后，Spring Boot应用会暴露出 /hystrix.stream 端点以供监控工具读取该应用的Hystrix Metrics数据。但是默认情况下，该Endpoint每间500ms就会向建立连接的客户端发送metrics数据，频率太高了，浪费CPU和带宽资源。在Hystrix Dashboard主页中虽然有让你输入delay的输入框，但是该参数根本不起作用！

经过抓包，发现在设置delay参数后，实际向应用发的请求是

http://localhost:9000/hystrix.stream?delay=2000

我们通过curl发送同样的请求，发现delay参数确实被无视了，应用依然每隔500ms就向我们推送一次断路器数据。通过查阅源码，发现控制该行为的地方在 **HystrixDashboardStream** 类中。

因为这个类是在应用启动时就进行初始化的，且 dataEmissionIntervalInMs 已经被声明成了private static final，所以这个参数是在应用启动时唯一确定好了，根本无法动态修改！坑！

@HystrixCommand 配置的 fallbackMethod 可以再次使用 @HystrixCommand。而且 fallbackMethod 可对异常进行处理。

默认情况下，Hystrix 会让相同组名的命令使用同一个线程池。

Hystrix 当然可以有缓存啦。

指定隔离策略：
```java
@HystrixCommand(fallbackMethod = "stubMyService",
    commandProperties = {
      @HystrixProperty(name="execution.isolation.strategy", value="SEMAPHORE")
    }
)
```

貌似不起作用：

> hystrix.metrics.enabled： true

启用Hystrix指标轮询。默认为true。

> hystrix.metrics.polling-interval-ms： 2000

后续轮询度量之间的间隔。默认为2000 ms。

### 3. Feign

Feign是自带断路器的，在D版本的Spring Cloud中，它没有默认打开。需要在配置文件中配置打开它，在配置文件加以下代码：

> feign.hystrix.enabled=true

Spring Cloud Netflix将所有Feign实例标记为@Primary，所以Spring Framework将知道要注入哪个bean。在某些情况下，这可能是不可取的。要关闭此行为，将@FeignClient的primary属性设置为false。
```java
@FeignClient(name = "hello", primary = false)
public interface HelloClient {
        // methods here
}
```

当使用包含Ribbon客户端的Hystrix命令时，您需要确保您的Hystrix超时配置为长于配置的Ribbon超时，包括可能进行的任何潜在的重试。例如，如果您的Ribbon连接超时为一秒钟，并且Ribbon客户端可能会重试该请求三次，那么您的Hystrix超时应该略超过三秒钟

Ribbon是一个客户端负载均衡器，它可以很好地控制HTTP和TCP客户端的行为。Feign已经使用Ribbon。

Spring Cloud根据需要，使用FeignClientsConfiguration为每个已命名的客户端创建一个新的集合ApplicationContext。这包含（除其他外）feign.Decoder，feign.Encoder和feign.Contract。

Spring Cloud可以通过使用@FeignClient声明额外的配置（在FeignClientsConfiguration之上）来完全控制假客户端。例：
```java
@FeignClient(name = "stores", configuration = FooConfiguration.class)
public interface StoreClient {
    //..
}
```
在这种情况下，客户端由FeignClientsConfiguration中的组件与FooConfiguration中的任何组件组成（后者将覆盖前者）。

配置 ribbon 的参数：

（1）全局配置
```
ribbon.<key>=<value>
```
（2）指定服务配置
```
<client>.ribbon.<key>=<value>
```

<client>为Feign 设置的 name 或 value 值

手动创建Feign客户端：

在某些情况下，可能需要以上述方法不可能自定义您的Feign客户端。在这种情况下，您可以使用 Feign Builder API 创建客户端

```java
@Import(FeignClientsConfiguration.class)
class FooController {

	private FooClient fooClient;

	private FooClient adminClient;

    @Autowired
	public FooController(
			Decoder decoder, Encoder encoder, Client client) {
		this.fooClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
				.target(FooClient.class, "http://PROD-SVC");
		this.adminClient = Feign.builder().client(client)
				.encoder(encoder)
				.decoder(decoder)
				.requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
				.target(FooClient.class, "http://PROD-SVC");
    }
}
```
### 4. Dashboard

Dashboard主要展示了2类信息，一是HystrixCommand的执行情况，二是线程池的状态，包括线程池名，大小，当前活跃线程说，最大活跃线程数，排队队列大小等。

### 5. Turbine

在复杂的分布式系统中，相同服务的结点经常需要部署上百甚至上千个，很多时候，运维人员希望能够把相同服务的节点状态以一个整体集群的形式展现出来，这样可以更好的把握整个系统的状态。 为此，Netflix又提供了一个开源项目（Turbine）来提供把多个hystrix.stream的内容聚合为一个数据源供Dashboard展示。

Turbine有2种用法，其一是内嵌Turbine到你的项目中；另外一个是把Turbine当做一个独立的Module。不管哪种用法，配置文件都是一致的。 Turbine默认会在classpath下查找配置文件：config.properties， 该文件中会配置：

（1）Turbine在监控哪些集群：
> turbine.aggregator.clusterConfig=cluster-1,cluster-2

指定聚合哪些集群，多个使用”,”分割，默认为default。可使用
**http://.../turbine.stream?cluster={clusterConfig之一}** 访问


（2）Turbine怎样获取到节点的监控信息（hystrix.stream）：
```
turbine.instanceUrlSuffix.<cluster-name> = :/HystrixDemo/hystrix.stream
```

（3） 集群下有哪些节点：
> turbine.ConfigPropertyBasedDiscovery.cluster-1.instances=localhost:8080,localhost:8081


（4） 配置Eureka中的serviceId列表，表明监控哪些服务
> turbine.appConfig


（5）turbine.clusterNameExpression
```
（1）clusterNameExpression 指定集群名称，默认表达式appName；此时 turbine.aggregator.clusterConfig 需要配置想要监控的应用名称；

（2）当 clusterNameExpression: default时，turbine.aggregator.clusterConfig 可以不写，因为默认就是 default；

（3）当 clusterNameExpression: metadata['cluster'] 时，假设想要监控的应用配置了 eureka.instance.metadata-map.cluster: ABC，则需要配置，同时 turbine.aggregator.clusterConfig: ABC
```

Spring Cloud Hystrix（参数详解），以下条目 6-10 来自
> http://blog.csdn.net/aichidemao2017/article/details/79036419

GitHub

>https://github.com/Netflix/Hystrix/wiki

### 6. Execution 相关的属性的配置：
```
hystrix.command.default.execution.isolation.strategy 隔离策略，默认是Thread, 可选Thread｜Semaphore

thread 通过线程数量来限制并发请求数，可以提供额外的保护，但有一定的延迟。一般用于网络调用
semaphore 通过semaphore count来限制并发请求数，适用于无网络的高并发请求
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds 命令执行超时时间，默认1000ms

hystrix.command.default.execution.timeout.enabled 执行是否启用超时，默认启用true
hystrix.command.default.execution.isolation.thread.interruptOnTimeout 发生超时是是否中断，默认true
hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests 最大并发请求数，默认10，该参数当使用ExecutionIsolationStrategy.SEMAPHORE策略时才有效。如果达到最大并发请求数，请求会被拒绝。理论上选择semaphore size的原则和选择thread size一致，但选用semaphore时每次执行的单元要比较小且执行速度快（ms级别），否则的话应该用thread。
semaphore应该占整个容器（tomcat）的线程池的一小部分。
Fallback相关的属性
这些参数可以应用于Hystrix的THREAD和SEMAPHORE策略

hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests 如果并发数达到该设置值，请求会被拒绝和抛出异常并且fallback不会被调用。默认10
hystrix.command.default.fallback.enabled 当执行失败或者请求被拒绝，是否会尝试调用hystrixCommand.getFallback() 。默认true
```

### 7. Circuit Breaker相关的属性
```
hystrix.command.default.circuitBreaker.enabled 用来跟踪circuit的健康性，如果未达标则让request短路。默认true
hystrix.command.default.circuitBreaker.requestVolumeThreshold 一个rolling window内最小的请求数。如果设为20，那么当一个rolling window的时间内（比如说1个rolling window是10秒）收到19个请求，即使19个请求都失败，也不会触发circuit break。默认20
hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds 触发短路的时间值，当该值设为5000时，则当触发circuit break后的5000毫秒内都会拒绝request，也就是5000毫秒后才会关闭circuit。默认5000
hystrix.command.default.circuitBreaker.errorThresholdPercentage错误比率阀值，如果错误率>=该值，circuit会被打开，并短路所有请求触发fallback。默认50
hystrix.command.default.circuitBreaker.forceOpen 强制打开熔断器，如果打开这个开关，那么拒绝所有request，默认false
hystrix.command.default.circuitBreaker.forceClosed 强制关闭熔断器 如果这个开关打开，circuit将一直关闭且忽略circuitBreaker.errorThresholdPercentage
```

### 8. Metrics相关参数
```
hystrix.command.default.metrics.rollingStats.timeInMilliseconds 设置统计的时间窗口值的，毫秒值，circuit break 的打开会根据1个rolling window的统计来计算。若rolling window被设为10000毫秒，则rolling window会被分成n个buckets，每个bucket包含success，failure，timeout，rejection的次数的统计信息。默认10000
hystrix.command.default.metrics.rollingStats.numBuckets 设置一个rolling window被划分的数量，若numBuckets＝10，rolling window＝10000，那么一个bucket的时间即1秒。必须符合rolling window % numberBuckets == 0。默认10
hystrix.command.default.metrics.rollingPercentile.enabled 执行时是否enable指标的计算和跟踪，默认true
hystrix.command.default.metrics.rollingPercentile.timeInMilliseconds 设置rolling percentile window的时间，默认60000
hystrix.command.default.metrics.rollingPercentile.numBuckets 设置rolling percentile window的numberBuckets。逻辑同上。默认6
hystrix.command.default.metrics.rollingPercentile.bucketSize 如果bucket size＝100，window＝10s，若这10s里有500次执行，只有最后100次执行会被统计到bucket里去。增加该值会增加内存开销以及排序的开销。默认100
hystrix.command.default.metrics.healthSnapshot.intervalInMilliseconds 记录health 快照（用来统计成功和错误率）的间隔，默认500ms
Request Context 相关参数
hystrix.command.default.requestCache.enabled 默认true，需要重载getCacheKey()，返回null时不缓存
hystrix.command.default.requestLog.enabled 记录日志到HystrixRequestLog，默认true

```
### 9. Collapser Properties 相关参数
```
hystrix.collapser.default.maxRequestsInBatch 单次批处理的最大请求数，达到该数量触发批处理，默认Integer.MAX_VALUE
hystrix.collapser.default.timerDelayInMilliseconds 触发批处理的延迟，也可以为创建批处理的时间＋该值，默认10
hystrix.collapser.default.requestCache.enabled 是否对HystrixCollapser.execute() and HystrixCollapser.queue()的cache，默认true
```

### 10. ThreadPool 相关参数
```
线程数默认值10适用于大部分情况（有时可以设置得更小），如果需要设置得更大，那有个基本得公式可以follow：
requests per second at peak when healthy × 99th percentile latency in seconds + some breathing room
每秒最大支撑的请求数 (99%平均响应时间 + 缓存值)
比如：每秒能处理1000个请求，99%的请求响应时间是60ms，那么公式是：
1000 （0.060+0.012）

基本得原则时保持线程池尽可能小，他主要是为了释放压力，防止资源被阻塞。
当一切都是正常的时候，线程池一般仅会有1到2个线程激活来提供服务

hystrix.threadpool.default.coreSize 并发执行的最大线程数，默认10
hystrix.threadpool.default.maxQueueSize BlockingQueue的最大队列数，当设为－1，会使用SynchronousQueue，值为正时使用LinkedBlcokingQueue。该设置只会在初始化时有效，之后不能修改threadpool的queue size，除非reinitialising thread executor。默认－1。
hystrix.threadpool.default.queueSizeRejectionThreshold 即使maxQueueSize没有达到，达到queueSizeRejectionThreshold该值后，请求也会被拒绝。因为maxQueueSize不能被动态修改，这个参数将允许我们动态设置该值。if maxQueueSize == -1，该字段将不起作用
hystrix.threadpool.default.keepAliveTimeMinutes 如果corePoolSize和maxPoolSize设成一样（默认实现）该设置无效。如果通过plugin（https://github.com/Netflix/Hystrix/wiki/Plugins）使用自定义实现，该设置才有用，默认1.
hystrix.threadpool.default.metrics.rollingStats.timeInMilliseconds 线程池统计指标的时间，默认10000
hystrix.threadpool.default.metrics.rollingStats.numBuckets 将rolling window划分为n个buckets，默认10
```

### 11. Turbine与RabbitMQ结合

spring-cloud-starter-turbine-amqp

@EnableTurbineStream

spring-cloud-netflix-hystrix-amqp

消息代理异步，存疑？

### 12. 配置快速失败

> spring.cloud.config.failFast=true

另外还有重试次数、重试间隔设置

### 13. 如何让应用产生hystrix.stream？

需要actuator hystrix的相关jar包，需要 **@EnableCircuitBreaker** 相关注解：

（1）网关服务zuul本来就有，不用额外配置

（2）使用feign调用的服务，需要打开
> feign.hystrix.enabled: true

(3)非feign的springboot项目，使用resttemple调用服务时，需要以上相关包和相关注解，还需要 **@HystrixCommand** 来使用hystrix来支持。

（4）另外需要有调用任意hystrix接口，不然没有hystrix调用，访问hystrix.stream会一直ping，hystrix监控界面一直loading，查看hystrix.stream是没数据。

### 14. Hystrix配置

（1）Command 配置

Command配置源码在**HystrixCommandProperties**，构造Command时通过Setter进行配置。

```
//使用命令调用隔离方式,默认:采用线程隔离,ExecutionIsolationStrategy.THREAD  
private final HystrixProperty<ExecutionIsolationStrategy> executionIsolationStrategy;   
//使用线程隔离时，调用超时时间，默认:1秒  
private final HystrixProperty<Integer> executionIsolationThreadTimeoutInMilliseconds;   
//线程池的key,用于决定命令在哪个线程池执行  
private final HystrixProperty<String> executionIsolationThreadPoolKeyOverride;   
//使用信号量隔离时，命令调用最大的并发数,默认:10  
private final HystrixProperty<Integer> executionIsolationSemaphoreMaxConcurrentRequests;  
//使用信号量隔离时，命令fallback(降级)调用最大的并发数,默认:10  
private final HystrixProperty<Integer> fallbackIsolationSemaphoreMaxConcurrentRequests;   
//是否开启fallback降级策略 默认:true   
private final HystrixProperty<Boolean> fallbackEnabled;   
// 使用线程隔离时，是否对命令执行超时的线程调用中断（Thread.interrupt()）操作.默认:true  
private final HystrixProperty<Boolean> executionIsolationThreadInterruptOnTimeout;   
// 统计滚动的时间窗口,默认:5000毫秒circuitBreakerSleepWindowInMilliseconds  
private final HystrixProperty<Integer> metricsRollingStatisticalWindowInMilliseconds;  
// 统计窗口的Buckets的数量,默认:10个,每秒一个Buckets统计  
private final HystrixProperty<Integer> metricsRollingStatisticalWindowBuckets; // number of buckets in the statisticalWindow  
//是否开启监控统计功能,默认:true  
private final HystrixProperty<Boolean> metricsRollingPercentileEnabled;   
// 是否开启请求日志,默认:true  
private final HystrixProperty<Boolean> requestLogEnabled;   
//是否开启请求缓存,默认:true  
private final HystrixProperty<Boolean> requestCacheEnabled; // Whether request caching is enabled.  
```

（2）熔断器（Circuit Breaker）配置

Circuit Breaker配置源码在 **HystrixCommandProperties**，构造Command时通过Setter进行配置,每种依赖使用一个Circuit Breaker。

```
// 熔断器在整个统计时间内是否开启的阀值，默认20秒。也就是10秒钟内至少请求20次，熔断器才发挥起作用  
private final HystrixProperty<Integer> circuitBreakerRequestVolumeThreshold;   
//熔断器默认工作时间,默认:5秒.熔断器中断请求5秒后会进入半打开状态,放部分流量过去重试  
private final HystrixProperty<Integer> circuitBreakerSleepWindowInMilliseconds;   
//是否启用熔断器,默认true. 启动  
private final HystrixProperty<Boolean> circuitBreakerEnabled;   
//默认:50%。当出错率超过50%后熔断器启动.  
private final HystrixProperty<Integer> circuitBreakerErrorThresholdPercentage;  
//是否强制开启熔断器阻断所有请求,默认:false,不开启  
private final HystrixProperty<Boolean> circuitBreakerForceOpen;   
//是否允许熔断器忽略错误,默认false, 不开启  
private final HystrixProperty<Boolean> circuitBreakerForceClosed;  
```

（3）命令合并(Collapser)配置

Command配置源码在 **HystrixCollapserProperties**，构造Collapser时通过Setter进行配置。

```
//请求合并是允许的最大请求数,默认: Integer.MAX_VALUE  
private final HystrixProperty<Integer> maxRequestsInBatch;  
//批处理过程中每个命令延迟的时间,默认:10毫秒  
private final HystrixProperty<Integer> timerDelayInMilliseconds;  
//批处理过程中是否开启请求缓存,默认:开启  
private final HystrixProperty<Boolean> requestCacheEnabled;  
```

（4）线程池(ThreadPool)配置

```
/**
配置线程池大小,默认值10个.
建议值:请求高峰时99.5%的平均响应时间 + 向上预留一些即可
*/  
HystrixThreadPoolProperties.Setter().withCoreSize(int value)  
/**
配置线程值等待队列长度,默认值:-1
建议值:-1表示不等待直接拒绝,测试表明线程池使用直接决绝策略+ 合适大小的非回缩线程池效率最高.所以不建议修改此值。
当使用非回缩线程池时，queueSizeRejectionThreshold,keepAliveTimeMinutes 参数无效
*/  
HystrixThreadPoolProperties.Setter().withMaxQueueSize(int value)  
```

### 15. Hystrix给了我们三种key来用于隔离。

+ CommandKey，针对相同的接口一般CommandKey值相同，目的是把HystrixCommand，HystrixCircuitBreaker，HytrixCommandMerics以及其他相关对象关联在一起，形成一个原子组。采用原生接口的话，默认值为类名；采用注解形式的话，默认值为方法名。

+ CommandGroupKey，对CommandKey分组，用于真正的隔离。相同CommandGroupKey会使用同一个线程池或者信号量。一般情况相同业务功能会使用相同的CommandGroupKey。

+ ThreadPoolKey，如果说CommandGroupKey只是逻辑隔离，那么ThreadPoolKey就是物理隔离，当没有设置ThreadPoolKey的时候，线程池或者信号量的划分按照CommandGroupKey，当设置了ThreadPoolKey，那么线程池和信号量的划分就按照ThreadPoolKey来处理，相同ThreadPoolKey采用同一个线程池或者信号量

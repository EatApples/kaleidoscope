### 一 SpringBoot 定时任务的线程池
#### 1 一句话总结
SpringBoot 定时任务默认newSingleThreadScheduledExecutor，使用定时任务需要配置线程池大小。

#### 2 背景
我们在项目中引入了 spring-boot-admin，在启动应用的时候发现其中的定时任务根本就没有执行。

#### 3 解决步骤
搜索发现了spring-boot-admin 的 ISSSUE  [Scheduled task with cron not working as expected](https://github.com/codecentric/spring-boot-admin/issues/478#issuecomment-311749930)，从中找到一种解决方案：

> Ok. I fix it according to this: https://stackoverflow.com/a/34646639/2811722.

```java
@Bean
public ThreadPoolTaskScheduler taskScheduler(){
    ThreadPoolTaskScheduler taskScheduler = new ThreadPoolTaskScheduler();
    taskScheduler.setPoolSize(20);
    return  taskScheduler;
}

```

然后从 stackoverflow 的回复中发现了问题的端倪：
> If you do not provide a pool-size attribute, the default thread pool will only have a single thread.

spring-boot-admin 的开发者回复：
>Since the scheduler provided by the SBA client is very similiar to the default one from @EnableScheduling and doesn't interfere when you are providing a custom one, I tend to not fix this issue.

原来与 spring-boot-admin 无关，根源在 SpringBoot。

#### 4 原理
SpringBoot 定时任务默认执行器为：**newSingleThreadScheduledExecutor**。

从 **org.springframework.scheduling.config.ScheduledTaskRegistrar** 可以发现：
```java
if (this.taskScheduler == null) {
            this.localExecutor = Executors.newSingleThreadScheduledExecutor();
            this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
}
```
由于只有一个执行线程，那些通过 **@Scheduled** 注解的定时任务会按照程序加载次序顺序执行。如果当前任务阻塞或陷入死循环，其他任务就一直不能运行。

> 如果某个定时任务执行未完成会出现什么现象呢？那此任务一直无法执行完成，无法设置下次任务执行时间，之后会导致此任务后面的所有定时任务无法继续执行，也就会出现所有的定时任务“失效”现象。所以应用springBoot定时任务的方法中，一定不要出现“死循环”、“http持续等待无响应”现象，否则会导致定时任务程序无法正常运行。再就是非特殊需求情况下可以把定时任务“分散”下。

#### 5 推荐解决方案：
通过实现 **SchedulingConfigurer** 接口来设置线程池大小。示例如下：
```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {

    /**
     * 并行任务
     */
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        TaskScheduler taskScheduler = MytaskScheduler();
        taskRegistrar.setTaskScheduler(taskScheduler);
    }

    /**
     * 并行任务使用策略：多线程处理
     *
     * @return ThreadPoolTaskScheduler 线程池
     */
    @Bean(destroyMethod = "shutdown")
    public ThreadPoolTaskScheduler MytaskScheduler() {

        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(20);
        scheduler.setThreadNamePrefix("task-");
        scheduler.setAwaitTerminationSeconds(60);
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        scheduler.initialize();
        return scheduler;
    }

}
```

使用
> @bean(destroyMethod="shutdown")

这样是为了确保当 SpringBoot 应用上下文关闭的时候任务执行器也被正确地关闭。

### 二 SpringBoot将本地jar包打入jar包中
#### 1 引入本地包
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-core</artifactId>
    <version>1.3.6.RELEASE.REVIEW</version>
    <scope>system</scope>
    <systemPath>${basedir}/lib/spring-cloud-netflix-core-1.3.6.RELEASE.REVIEW.jar</systemPath>
</dependency>
```

#### 2 将本地jar包打入jar包中
```xml
<plugins>
      <plugin>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-maven-plugin</artifactId>
          <configuration>
                <includeSystemScope>true</includeSystemScope>
          </configuration>
      </plugin>
</plugins>
```

### 三 随机端口
为Spring Cloud的应用实用随机端口非常简单，主要有两种方法：

（1）设置server.port=0，当应用启动的时候会自动的分配一个随机端口，但是该方式在注册到Eureka的时候会一个问题：所有实例都使用了同样的实例名（如：Lenovo-test:hello-service:0），这导致只出现了一个实例。所以，我们还需要修改实例ID的定义，让每个实例的ID不同，比如使用随机数来配置实例ID：
```
server.port=0
eureka.instance.instance-id=${spring.application.name}:${random.int}
```

（2）除了上面的方法，实际上我们还可以直接使用random函数来配置server.port。这样就可以指定端口的取值范围，比如：
```
server.port=${random.int[10000,19999]}
```
由于默认的实例ID会由server.port拼接，而此时server.port设置的随机值会重新取一次随机数，所以使用这种方法的时候不需要重新定义实例ID的规则就能产生不同的实例ID了。

### 四 如何解决Eureka Server不踢出已关停的节点的问题
如果在Eureka Server的首页看到以下这段提示，则说明Eureka已经进入了保护模式。

```
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

What happens during network outages between Peers?

In the case of network outages between peers, following things may happen

+ The heartbeat replications between peers may fail and the server detects this situation and enters into a self-preservation mode protecting the current state.

+ Registrations may happen in an orphaned server and some clients may reflect new registrations while the others may not. The situation autocorrects itself after the network connectivity is restored to a stable state. When the peers are able to communicate fine, the registration information is automatically transferred to the servers that do not have them.

The bottom line is, during the network outages, the server tries to be as resilient as possible, but there is a possibility of clients having different views of the servers during that time.

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。

在开发过程中，我们常常希望Eureka Server能够迅速有效地踢出已关停的节点，但是新手由于Eureka自我保护模式，以及心跳周期长的原因，常常会遇到Eureka Server不踢出已关停的节点的问题。解决方法如下：

(1) Eureka Server端：配置关闭自我保护，并按需配置Eureka Server清理无效节点的时间间隔。
```yml
eureka.server.enable-self-preservation          # 设为false，关闭自我保护
eureka.server.eviction-interval-timer-in-ms     # 清理间隔（单位毫秒，默认是60*1000）
```
(2) Eureka Client端：配置开启健康检查，并按需配置续约更新时间和到期时间。

```yml
eureka.client.healthcheck.enabled                       # 开启健康检查（需要spring-boot-starter-actuator依赖）
eureka.instance.lease-renewal-interval-in-seconds       # 续约更新时间间隔（默认30秒）
eureka.instance.lease-expiration-duration-in-seconds    # 续约到期时间（默认90秒）
```

示例：

服务器端配置：
```yml
eureka:
  server:
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 4000
```
客户端配置：
```yml
eureka:
  client:
    healthcheck:
      enabled: true
  instance:
    lease-expiration-duration-in-seconds: 30
    lease-renewal-interval-in-seconds: 10
```
注意：更改Eureka更新频率将打破服务器的自我保护功能，生产环境下不建议自定义这些配置。

### 参考资料
#### 1. Scheduled task with cron not working as expected #478 https://github.com/codecentric/spring-boot-admin/issues/478#issuecomment-311749930

#### 2. SpringBoot Schedule 配置
https://www.cnblogs.com/slimer/p/6222485.html

#### 3. 深度解析Java8 – ScheduledThreadPoolExecutor源码解析
http://ju.outofmemory.cn/entry/99456

#### 4. springBoot中@Scheduled执行原理解析
http://blog.csdn.net/gaodebao1/article/details/51789225

#### 5. springboot怎么使用maven打包时将本地jar包一块打进去
https://blog.csdn.net/qq_30698633/article/details/78331920

#### 6. Spring Cloud中，Eureka常见问题总结
http://www.itmuch.com/spring-cloud-sum-eureka/

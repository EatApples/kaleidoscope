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

# 一 SpringBoot 定时任务的线程池
## 1 一句话总结
SpringBoot 定时任务默认newSingleThreadScheduledExecutor，使用定时任务需要配置线程池大小。

## 2 背景
我们在项目中引入了 spring-boot-admin，在启动应用的时候发现其中的定时任务根本就没有执行。

## 3 解决步骤
搜索发现了spring-boot-admin 的 ISSSUE  [Scheduled task with cron not working as expected](https://github.com/codecentric/spring-boot-admin/issues/478#issuecomment-311749930)，从中找到一种解决方案：

> Ok. I fix it according to this: https://stackoverflow.com/a/34646639/2811722.

```
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

## 4 原理
SpringBoot 定时任务默认执行器为：**newSingleThreadScheduledExecutor**。

从 **org.springframework.scheduling.config.ScheduledTaskRegistrar** 可以发现：
```
if (this.taskScheduler == null) {
            this.localExecutor = Executors.newSingleThreadScheduledExecutor();
            this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
}
```
由于只有一个执行线程，那些通过 **@Scheduled** 注解的定时任务会按照程序加载次序顺序执行。如果某些定时任务“恰好”交替执行，那么另一些定时任务就可能被阻塞。更有甚者，如果当前任务阻塞或陷入死循环，其他任务就一直不能运行。

## 5 推荐解决方案：
通过实现 **SchedulingConfigurer** 接口来设置线程池大小。示例如下：
```
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
        return scheduler;
    }

}
```

使用
> @bean(destroyMethod="shutdown")

这样是为了确保当 SpringBoot 应用上下文关闭的时候任务执行器也被正确地关闭。

# 参考资料
1 https://github.com/codecentric/spring-boot-admin/issues/478#issuecomment-311749930

2 https://www.cnblogs.com/slimer/p/6222485.html

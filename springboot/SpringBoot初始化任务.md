## 初始化任务

&#8195;&#8195;我们在开发中可能会有这样的情景：需要在应用程序启动之初做一些事情。比如读取配置文件，初始化数据库连接之类的。

&#8195;&#8195;SpringBoot 给我们提供了两个接口来帮助我们实现这种需求。这两个接口分别为 CommandLineRunner 和 ApplicationRunner。它们的执行时机为容器启动完成的时候，即在所有Spring Beans 都初始化之后，在 SpringApplication.run() 运行之前执行。

&#8195;&#8195;这两个接口中有一个 run 方法，我们只需要实现这个方法即可。这两个接口的不同之处在于：ApplicationRunner 中 run 方法的参数为 ApplicationArguments，而 CommandLineRunner 接口中run方法的参数为 String 数组。

### 一 准备工作

&#8195;&#8195;这里了解下 @Order注解。如果有多个实现类，而你需要它们按一定顺序执行的话，可以在实现类上加上 **@Order** 注解。
>@Order(value=整数值)

SpringBoot会按照 @Order 中的 value 值从小到大依次执行。

### 二 代码示例及说明

&#8195;&#8195;因为 CommandLineRunnerTask 上的 @Order 注解值比 ApplicationRunnerTask 上的 @Order 注解值小，如果这两个任务在同一个应用中，则 CommandLineRunnerTask 先启动。

&#8195;&#8195;注意：因为 SpringBoot 加载是单线程执行，如果在初始化任务中有阻塞的操作（死循环、等待IO操作等），则之后的动作不会执行。所以，初始化任务中一定不能阻塞。

#### 2.1 CommandLineRunner

&#8195;&#8195;注意使用 @Component 注解任务实现类。
```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(value = 1)
public class CommandLineRunnerTask implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {

        // 这里运行的任务不能阻塞

    }

}
```

#### 2.2 ApplicationRunner
```java
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Order(value = 2)
public class ApplicationRunnerTask implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {

        // 这里运行的任务不能阻塞

    }

}
```

### 三 参考资料
#### 3.1 CommandLineRunner或者ApplicationRunner接口
https://www.jianshu.com/p/5d4ffe267596

#### 3.2 Spring Boot 启动原理解析
https://www.cnblogs.com/zheting/p/6707035.html

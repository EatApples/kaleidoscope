### 1. 插件结构

Java Agent 只会启用 plugins 目录下的所有插件，bootstrap-plugins 目录以及 optional-plugins 目录下的插件不会启用。如需启用引导插件或可选插件，只需将 JAR 包移到 plugins 目录下，如需禁用某款插件，只需从 plugins 目录中移除即可。

### 2. Spring 注解插件 Spring-annotation-plugin

这个插件可以实现对被@Bean, @Service, @Component and @Repository 注解标注的 bean 的所有方法的追踪。

为什么这个插件是可选的？
追踪 bean 的所有方法会创建大量的 span，这会导致耗费更多的 CPU、内存和网络带宽。 当前如果你想追踪尽可能多的方法，请确保系统负载可以支撑更多的请求。

### 3. 性能诊断

https://skyapm.github.io/document-cn-translation-of-skywalking/zh/8.0.0/ui/
并不是所有的 SkyWalking 生态系统代理都支持此特性，7.0.0 中的 java 代理默认支持此特性。

### 1. 在线代码级性能剖析，补全分布式追踪的最后一块“短板”

https://skywalking.apache.org/zh/2020-03-23-using-profiling-to-fix-the-blind-spot-of-distributed-tracing/

### 2. Skywalking 系列博客 9-Skywalking 集群部署

https://www.itmuch.com/skywalking/cluster/

### 3. Skywalking 系列博客 5-apm-customize-enhance-plugin 插件使用教程

https://www.itmuch.com/skywalking/apm-customize-enhance-plugin/

### 4. SkyWalking 极简入门

https://skywalking.apache.org/zh/2020-04-19-skywalking-quick-start/

### 5. SkyWalking 方法级 trace 粒度实现 @Trace 和 apm-customize-enhance-plugin 介绍

https://blog.csdn.net/zxh1991811/article/details/115379470

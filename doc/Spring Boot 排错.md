# 1，Fiegn Client
## 问题：

Fiegn Client with Spring Boot: RequestParam.value() was empty on parameter 3

## 原因（内容来自stackoverflow）：

Both Spring MVC and Spring cloud feign are using same ParameterNameDiscoverer - named DefaultParameterNameDiscoverer to find parameter name. It tries to find the parameter names with the following step.

First, it uses StandardReflectionParameterNameDiscoverer. It tries to find the variable name with reflection. It is only possible when your classes are compiled with -parameter.

Second, if it fails, it uses LocalVariableTableParameterNameDiscoverer. It tries to find the variable name from the debugging info in the class file with ASM libraries.

The difference between Spring MVC and Feign occurs here. Feign uses above annotations (like @RequestParam) on methods of Java interfaces. But, we use these on methods of Java classes when using Spring MVC. Unfortunately, javac compiler omits the debug information of parameter name from class file for java interfaces. That's why feign fails to find parameter name without -parameter.

Namely, if you compile your code with -parameter, both Spring MVC and Feign will succeed to acquire parameter names. But if you compile without -parameter, only Spring MVC will succeed.

As a result, it's not a bug. it's a limitation of Feign at this moment as I think.

## 解决方案：

最好的做法是通过 @RequestParam 注解指定具体的参数名称

# 2，spring boot自动注入

## 问题：
spring boot自动注入出现Consider defining a bean of type 'xxx' in your configuration

## 解决方案：
将接口与对应的实现类放在与 application 启动类的同一个目录或者他的子目录下，这样注解可以被扫描到，这是最省事的办法 

或在指定的 application 类上加上 @SpringBootApplication(scanBasePackages = {"com.xxx.yyy"}) 


# 3，spring boot 避免加载不必要的自动化配置

```
@ComponentScan(basePackages = { "com..yyy.zzz" }, excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = aaaa.class),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = bbbb.class) })
```

# 4，eureka 报错
```
2017-10-14 07:41:28.315 [Eureka-EvictionTimer] INFO  c.netflix.eureka.registry.AbstractInstanceRegistry -
                                Running the evict task with compensationTime 0ms
2017-10-14 07:41:36.654 [TaskBatchingWorker-target_localhost-13] ERROR c.netflix.eureka.cluster.ReplicationTaskProcessor -
                                Network level connection to peer localhost; retrying after delay
com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: 拒绝连接 (Connection refused)
        at com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpClient4Handler.java:187)
        at com.netflix.eureka.cluster.DynamicGZIPContentEncodingFilter.handle(DynamicGZIPContentEncodingFilter.java:48)
        at com.netflix.discovery.EurekaIdentityHeaderFilter.handle(EurekaIdentityHeaderFilter.java:27)
        at com.sun.jersey.api.client.Client.handle(Client.java:652)
        at com.sun.jersey.api.client.WebResource.handle(WebResource.java:682)
        at com.sun.jersey.api.client.WebResource.access$200(WebResource.java:74)
        at com.sun.jersey.api.client.WebResource$Builder.post(WebResource.java:570)
        at com.netflix.eureka.transport.JerseyReplicationClient.submitBatchUpdates(JerseyReplicationClient.java:116)
        at com.netflix.eureka.cluster.ReplicationTaskProcessor.process(ReplicationTaskProcessor.java:71)
        at com.netflix.eureka.util.batcher.TaskExecutors$BatchWorkerRunnable.run(TaskExecutors.java:187)
        at java.lang.Thread.run(Thread.java:748)
Caused by: java.net.ConnectException: 拒绝连接 (Connection refused)
        at java.net.PlainSocketImpl.socketConnect(Native Method)
        at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
        at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        at java.net.Socket.connect(Socket.java:589)
        at org.apache.http.conn.scheme.PlainSocketFactory.connectSocket(PlainSocketFactory.java:121)
        at org.apache.http.impl.conn.DefaultClientConnectionOperator.openConnection(DefaultClientConnectionOperator.java:180)
        at org.apache.http.impl.conn.AbstractPoolEntry.open(AbstractPoolEntry.java:144)
        at org.apache.http.impl.conn.AbstractPooledConnAdapter.open(AbstractPooledConnAdapter.java:134)
        at org.apache.http.impl.client.DefaultRequestDirector.tryConnect(DefaultRequestDirector.java:610)
        at org.apache.http.impl.client.DefaultRequestDirector.execute(DefaultRequestDirector.java:445)
        at org.apache.http.impl.client.AbstractHttpClient.doExecute(AbstractHttpClient.java:835)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:118)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56)
        at com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpClient4Handler.java:173)
        ... 10 common frames omitted

```

# 原因：

```

配置了 
# 表示是否注册自身到eureka服务器    
eureka.client.register-with-eureka: false
# 是否从eureka上获取注册信息  
eureka.client.fetch-registry: false

```

来自：http://blog.csdn.net/tianyaleixiaowu/article/details/78184793

把server端yml里配置 register-with-eureka: false 的那两行注释给放开，看看eureka的server忽略自己后，是否能完成服务发现的高可用。可以看到和上面的最终结果是一样的，都是server1关闭后，server2依旧能进行client的发现。区别在于unavailable-replicas

	

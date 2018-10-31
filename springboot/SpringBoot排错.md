### 1. Fiegn Client
Feign是一种声明式、模板化的HTTP客户端

Feign是简化Java HTTP客户端开发的工具（java-to-httpclient-binder），它的灵感来自于Retrofit、JAXRS-2.0和WebSocket。Feign的初衷是降低统一绑定Denominator到HTTP API的复杂度，不区分是否为restful

Feign通过配置注入一个模板化请求进行工作。只需在发送之前关闭它，参数就可以被直接的运用到模板中。然而这也限制了Feign，只支持文本形式的API，它可以在响应请求方面来简化系统

Feign将方法签名中方法参数对象序列化为请求参数放到HTTP请求中的过程，是由编码器(Encoder)完成的。同理，将HTTP响应数据反序列化为java对象是由解码器(Decoder)完成的

Feign在默认情况下使用的是JDK原生的URLConnection发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用HTTP的persistence connection

简而言之：
```
feign 采用的是接口加注解
feign 整合了ribbon
```

#### 1.1 问题：

Fiegn Client with Spring Boot: RequestParam.value() was empty on parameter 3

#### 1.2 原因（内容来自stackoverflow）：

Both Spring MVC and Spring cloud feign are using same ParameterNameDiscoverer - named DefaultParameterNameDiscoverer to find parameter name. It tries to find the parameter names with the following step.

First, it uses StandardReflectionParameterNameDiscoverer. It tries to find the variable name with reflection. It is only possible when your classes are compiled with -parameter.

Second, if it fails, it uses LocalVariableTableParameterNameDiscoverer. It tries to find the variable name from the debugging info in the class file with ASM libraries.

The difference between Spring MVC and Feign occurs here. Feign uses above annotations (like @RequestParam) on methods of Java interfaces. But, we use these on methods of Java classes when using Spring MVC. Unfortunately, javac compiler omits the debug information of parameter name from class file for java interfaces. That's why feign fails to find parameter name without -parameter.

Namely, if you compile your code with -parameter, both Spring MVC and Feign will succeed to acquire parameter names. But if you compile without -parameter, only Spring MVC will succeed.

As a result, it's not a bug. it's a limitation of Feign at this moment as I think.

Feign 将方法签名中方法参数对象序列化为请求参数放到 HTTP 请求中的过程，是由编码器(Encoder)完成的。同理，将 HTTP 响应数据反序列化为 java 对象是由解码器(Decoder)完成的。

默认情况下，Feign 会将标有 @RequestParam 注解的参数转换成字符串添加到 URL 中，将没有注解的参数通过 Jackson 转换成 json 放到请求体中。注意，如果在 @RequetMapping 中的 method 将请求方式指定为 POST，那么所有未标注解的参数将会被忽略

在Spring Cloud环境下，Feign 的 Encoder *只会用来编码没有添加注解的参数*


#### 1.3 解决方案：

最好的做法是通过 @RequestParam 注解指定具体的参数名称

### 2. spring boot自动注入

#### 2.1 问题：
```
spring boot自动注入出现Consider defining a bean of type 'xxx' in your configuration
```
#### 2.2 解决方案：
将接口与对应的实现类放在与 application 启动类的同一个目录或者他的子目录下，这样注解可以被扫描到，这是最省事的办法

或在指定的 application 类上加上 @SpringBootApplication(scanBasePackages = {"com.xxx.yyy"})

TODO:
### 3. spring boot 避免加载不必要的自动化配置

```
@ComponentScan(basePackages = { "com..yyy.zzz" }, excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = aaaa.class),
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, value = bbbb.class) })
```

### 4. eureka 报错
#### 4.1 问题：
```java
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

#### 4.2 原因：

```yml

配置了
# 表示是否注册自身到eureka服务器    
eureka.client.register-with-eureka: false
# 是否从eureka上获取注册信息  
eureka.client.fetch-registry: false

```

来自：http://blog.csdn.net/tianyaleixiaowu/article/details/78184793

把server端yml里配置 register-with-eureka: false 的那两行注释给放开，看看eureka的server忽略自己后，是否能完成服务发现的高可用。可以看到和上面的最终结果是一样的，都是server1关闭后，server2依旧能进行client的发现。区别在于unavailable-replicas

### 5. CSRF
#### 5.1 问题
```
POST访问报错：Invalid CSRF Token 'null' was found on the request parameter '_csrf' or header 'X-CSRF-TOKEN
```
#### 5.2 原因  
Spring Security 4.0之后，引入了CSRF，默认是开启。

不得不说，CSRF和RESTful技术有冲突。CSRF默认支持的方法： GET|HEAD|TRACE|OPTIONS，不支持POST

CSRF（Cross-site request forgery）跨站请求伪造，也被称为“One Click Attack” 或者Session Riding，攻击方通过伪造用户请求访问受信任站点

#### 5.3 解决方案
```
 @CrossOrigin(methods = { RequestMethod.GET, RequestMethod.POST }, origins = "*")

```
### 6. JSONObject引发的惨案
#### 6.1 问题
classNotFound

#### 6.2 原因
（1）JSONObject 纯正来源于 org.json，但项目中没有引入
```xml
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20171018</version>
</dependency>
```
（2）项目能编译是因为starter-test带入了android-json
```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
</dependency>
```
然而<scope>test</scope>让它只在编译时生效，运行时就没了，所以classNotFound

（3）很不幸的是在其他项目中，POM中又引入了
```xml
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
</dependency>
```
意味着 org.json 只在编译时生效

#### 6.3 解决方案：
```
（1）去掉starter-test的<scope>test</scope>，这样只因为一个对象，带入了全家桶，丑陋
（2）引入org.json，删除 starter-test，最小依赖，觉得可行，最终采用这种
（3）构造Map对象，使用 ObjectMapper 进行转换
（4）直接构造JSON串
```

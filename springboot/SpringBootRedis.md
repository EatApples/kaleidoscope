## Redis使用

&#8195;&#8195;Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。Spring Boot 提供了 Jedis 客户端库的基本自动配置和 Spring Data Redis 提供的抽象。

### 一 准备工作
&#8195;&#8195;Maven 中引入相关依赖
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 二 代码示例及说明
#### 2.1 单机 Redis 使用
##### 2.1.1 配置文件中添加 Redis 相关属性
```yml
spring:
  redis:
    #指定分片,分片模式是一种轻量级集群
    database: 0
    #地址
    host: 127.0.0.1
    #端口
    port: 6379
    #密码
    password: 123456
    #超时时间
    timeout: 3000
    #连接池设置
    pool:
      # 连接池最大连接数（使用负值表示没有限制）
      max-active: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池中的最大空闲连接
      max-idle: 8
      # 连接池中的最小空闲连接
      min-idle: 0
```
##### 2.1.2 增加 Redis 的配置类

&#8195;&#8195;这里通过 @Bean 注解，由 SpringBoot 读取先前配置文件中添加的 Redis 相关属性，产生了 StringRedisTemplate 对象。

&#8195;&#8195;注意配置类上添加 @Configuration 注解。
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;

@Configuration
public class RedisConfig {

    @Bean(name = "redisTemplate")
    public StringRedisTemplate redisTemplate(RedisConnectionFactory connectionFactory) {

        return new StringRedisTemplate(connectionFactory);
    }

}

```

##### 2.1.3 在调用类中使用 StringRedisTemplate

&#8195;&#8195;在调用类中使用 @Autowired 注解，获得 StringRedisTemplate 对象，就能使用 Redis 的相关操作了。

```java
    @Autowired
    @Qualifier("redisTemplate")
    public StringRedisTemplate redisTemplate;
```
&#8195;&#8195;例如：
```java
//String 相关操作
redisTemplate.opsForValue().XXXX
//Set 相关操作
redisTemplate.opsForSet().XXXX
//List相关操作
redisTemplate.opsForList().XXXX
//Hash 相关操作
redisTemplate.opsForHash().XXXX
```

#### 2.2 集群 Redis 使用
##### 2.2.1 配置文件中添加 Redis 集群相关属性
```yml
#节点，多个节点用,分隔
spring.redis.cluster.nodes: 127.0.0.1:6379,127.0.0.2:6379
#密码
spring.redis.cluster.password: 12345
#最大连接数, 默认8个,一些低版本的包是maxActive，如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted(耗尽)。
spring.redis.cluster.maxTotal: 100
#控制一个pool最多有多少个状态为idle(空闲的)的jedis实例。
spring.redis.cluster.maxIdle: 20
#控制一个pool最少有多少个状态为idle(空闲的)的jedis实例。
spring.redis.cluster.minIdle: 2
#等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException；
spring.redis.cluster.maxWait: 10000
#在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的
spring.redis.cluster.testOnBorrow: true
#jedis调用returnObject方法时，是否进行有效检查
spring.redis.cluster.testOnReturn: false
#读取超时
spring.redis.cluster.timeout: 5000
#连接超时
spring.redis.cluster.connectionTimeout: 5000
#最大尝试次数
spring.redis.cluster.maxAttempts: 3
```

##### 2.2.2 增加集群 Redis 的配置类
&#8195;&#8195;SpringBoot默认支持单机 Redis 的配置解析，没有支持集群 Redis 的配置解析。这里使用 @ConfigurationProperties 注解来达到自动解析。@ConfigurationProperties 注解，它可以把同类的配置信息自动封装成实体类。

&#8195;&#8195;@ConfigurationProperties 和 @Value 两个都能把配置文件的值直接注入到代码，但它们有很大区别：

| Feature           | @ConfigurationProperties | @Value |
| ----------------- | ------------------------ | ------ |
| Relaxed binding   | Yes                      | No     |
| Meta-data support | Yes                      | No     |
| SpEL evaluation   | No                       | Yes    |

&#8195;&#8195;首先，我们建立集群Redis的配置实体类。这些属性在后面配置Redis集群连接时会用到。

```java
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

// 特别注意：@ConfigurationProperties(prefix = "spring.redis.cluster")中spring.redis.cluster要和配置文件中的前缀对应
@Component
@ConfigurationProperties(prefix = "spring.redis.cluster")
public class ClusterConfigurationProperties {

    List<String> nodes;
    private String password;
    private int maxTotal;
    private int maxIdle;
    private int minIdle;
    private int maxWait;
    private boolean testOnBorrow;
    private boolean testOnReturn;
    private int timeout;
    private int connectionTimeout;
    private int maxAttempts;

    public List<String> getNodes() {

        return nodes;
    }

    public void setNodes(List<String> nodes) {

        this.nodes = nodes;
    }

    public String getPassword() {

        return password;
    }

    public void setPassword(String password) {

        this.password = password;
    }

    public int getMaxTotal() {

        return maxTotal;
    }

    public void setMaxTotal(int maxTotal) {

        this.maxTotal = maxTotal;
    }

    public int getMaxIdle() {

        return maxIdle;
    }

    public void setMaxIdle(int maxIdle) {

        this.maxIdle = maxIdle;
    }

    public int getMinIdle() {

        return minIdle;
    }

    public void setMinIdle(int minIdle) {

        this.minIdle = minIdle;
    }

    public int getMaxWait() {

        return maxWait;
    }

    public void setMaxWait(int maxWait) {

        this.maxWait = maxWait;
    }

    public boolean isTestOnBorrow() {

        return testOnBorrow;
    }

    public void setTestOnBorrow(boolean testOnBorrow) {

        this.testOnBorrow = testOnBorrow;
    }

    public boolean isTestOnReturn() {

        return testOnReturn;
    }

    public void setTestOnReturn(boolean testOnReturn) {

        this.testOnReturn = testOnReturn;
    }

    public int getTimeout() {

        return timeout;
    }

    public void setTimeout(int timeout) {

        this.timeout = timeout;
    }

    public int getConnectionTimeout() {

        return connectionTimeout;
    }

    public void setConnectionTimeout(int connectionTimeout) {

        this.connectionTimeout = connectionTimeout;
    }

    public int getMaxAttempts() {

        return maxAttempts;
    }

    public void setMaxAttempts(int maxAttempts) {

        this.maxAttempts = maxAttempts;
    }

}

```

&#8195;&#8195;然后，建立集群Redis的配置类，用于创建 StringRedisTemplate 对象。


```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;

import redis.clients.jedis.JedisPoolConfig;

@Configuration
public class RedisConfig {

    @Autowired
    private ClusterConfigurationProperties clusterProperties;

    /**
     * 初始化连接池
     *
     */
    private JedisPoolConfig getJedisPoolConfig() {

        JedisPoolConfig poolConfig = new JedisPoolConfig();
        poolConfig.setMaxTotal(clusterProperties.getMaxTotal());
        poolConfig.setMaxIdle(clusterProperties.getMaxIdle());
        poolConfig.setMinIdle(clusterProperties.getMinIdle());
        poolConfig.setMaxWaitMillis(clusterProperties.getMaxWait());
        poolConfig.setTestOnBorrow(clusterProperties.isTestOnBorrow());
        poolConfig.setTestOnReturn(clusterProperties.isTestOnReturn());
        return poolConfig;
    }

    /**
     * 配置集群连接信息
     *
     */
    @Bean
    public RedisConnectionFactory jedisConnectionFactory() {

        // 集群节点配置
        RedisClusterConfiguration rconfig = new RedisClusterConfiguration(clusterProperties.getNodes());
        // 集群的连接
        JedisConnectionFactory jedis = new JedisConnectionFactory(rconfig);
        // 设置密码
        jedis.setPassword(clusterProperties.getPassword());
        // 连接池配置
        jedis.setPoolConfig(getJedisPoolConfig());
        return jedis;
    }

    /**
     *
     * 返回Redis操作模板类
     */
    @Bean(name = "redisTemplate")
    public StringRedisTemplate redisTemplate(RedisConnectionFactory connectionFactory) {

        return new StringRedisTemplate(connectionFactory);
    }

}
```

##### 2.2.3 在调用类中使用 StringRedisTemplate

&#8195;&#8195;在调用类中使用 @Autowired 注解，获得 StringRedisTemplate 对象，就能使用 Redis 的相关操作了。

```java
    @Autowired
    @Qualifier("redisTemplate")
    public StringRedisTemplate redisTemplate;
```
&#8195;&#8195;例如：
```java
//String 相关操作
redisTemplate.opsForValue().XXXX
//Set 相关操作
redisTemplate.opsForSet().XXXX
//List相关操作
redisTemplate.opsForList().XXXX
//Hash 相关操作
redisTemplate.opsForHash().XXXX
```

### 三 参考资料
#### 3.1 SpringBoot 下 Redis 的配置使用（集群）
http://blog.csdn.net/kokjuis/article/details/78338481

#### 3.2 SpringBoot 使用 @ConfigurationProperties
https://blog.csdn.net/yingxiake/article/details/51263071

#### 3.3 Spring Boot中使用Redis数据库
http://blog.didispace.com/springbootredis/

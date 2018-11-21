### Eureka的核心代码

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>Dalston.SR5</version>
    <type>pom</type>
</dependency>
```
从D版本的依赖查到依赖的 netflix 版本是 1.3.6
```
org.springframework.cloud » spring-cloud-netflix-dependencies   1.3.6.RELEASE
```
进而查到依赖的原生 eureka 版本是 1.6.2
```
com.netflix.eureka » eureka-client  1.6.2
com.netflix.eureka » eureka-core    1.6.2
com.netflix.ribbon » ribbon-eureka  2.2.2
```

### eureka 原生版 1.6.2
+ 官网文档
https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance
+ 源码标注
https://github.com/YunaiV/eureka/tree/master
+ 源码解析
http://www.iocoder.cn/categories/Eureka/?github
+ Spring Cloud Eureka
https://xujin.org/categories/Spring-Cloud-Eureka/


### 扩展阅读
#### 1. Eureka 源码解析 —— 应用实例注册发现（六）之全量获取
http://www.iocoder.cn/Eureka/instance-registry-fetch-all/

#### 2. 深入了解EurekaClient的注册过程
https://blog.csdn.net/weixin_40615418/article/details/78731080#itme1

#### 3. Eureka REST operations
https://github.com/Netflix/eureka/wiki/Eureka-REST-operations

#### 4. 深度剖析服务发现组件Netflix Eureka
https://blog.csdn.net/jek123456/article/details/74171039

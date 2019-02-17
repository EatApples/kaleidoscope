### 1. `Spring application`中配置项的优先级

```yml
ApplicationPreparedEvent:[bootstrapProperties]                                  # 来自configServer的值
ApplicationPreparedEvent:[commandLineArgs]                                      # 命令行参数
ApplicationPreparedEvent:[servletConfigInitParams]
ApplicationPreparedEvent:[servletContextInitParams]
ApplicationPreparedEvent:[systemProperties]                                     # 系统配置参数
ApplicationPreparedEvent:[systemEnvironment]                                    # 系统环境变量
ApplicationPreparedEvent:[random]
ApplicationPreparedEvent:[applicationConfig: [classpath:/application-test.yml]] # 指定生效的`application-XX.yml`
ApplicationPreparedEvent:[applicationConfig: [classpath:/application.yml]]      # 本地的`application.yml`
ApplicationPreparedEvent:[springCloudClientHostInfo]
ApplicationPreparedEvent:[applicationConfig: [classpath:/bootstrap.yml]]        # 本地的`bootstrap.yml`
ApplicationPreparedEvent:[defaultProperties]                                    # 默认配置
ApplicationPreparedEvent:[Management Server]
```
`Spring`想要得到一个配置的值，就按照上面的顺序一个个去找，找到就直接返回。

由于`ConfigServer`处于最高优先级，本地项目不管怎么设置都不能覆盖（当然可以改变配置项的加载次序，但最好不要这样做，见参考资料）。

上面的排序是通过implements ApplicationListener<ApplicationPreparedEvent> 然后打印出来的。测试程序如下：

```java
@Component
public class ApplicationPreparedEventListener implements ApplicationListener<ApplicationPreparedEvent> {

    @Override
    public void onApplicationEvent(ApplicationPreparedEvent event) {

        // 初始化完成
        ConfigurableApplicationContext context = ((ApplicationPreparedEvent) event).getApplicationContext();
        ConfigurableEnvironment environment = context.getEnvironment();
        MutablePropertySources propertySources = environment.getPropertySources();
        if (propertySources != null) {
            Iterator<PropertySource<?>> iterator = propertySources.iterator();
            while (iterator.hasNext()) {
                PropertySource<?> propertySource = (PropertySource<?>) iterator.next();
                System.err.println("ApplicationPreparedEvent:[" + propertySource.getName() + "]");
            }
        }

    }
}
```
### 2. 我的解释

（1）`Spring`中属性的配置有多个来源，按照加载次序生效。也就是说，对于某个属性的值，哪个先出现，就以先出现的为准，后面出现的值不会覆盖之前的生效值。

（2）如果没有添加`config`相关依赖，则不能使用配置中心，因为`bootstrap.yml`不会被加载（与配置中心相关的某个配置只能放在`bootstrap.yml`中，见后文）。
```xml
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```
或
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

（3）如果使用配置中心，则需要添加添加`config`相关依赖。

（3.1）应用程序则会首先加载`bootstrap.yml`，读取关于配置中心的相关配置：
```yml
spring.cloud.config.name: this-is-your-application-name
spring.cloud.config.profile: pro
spring.cloud.config.label: master
spring.cloud.config.uri: http://127.0.0.1:8080
```
以上`4`个属性的加载非常有意思，只有这`4`个属性全部集齐，才能~~召唤神龙~~访问配置中心!

**注意**：

+ `spring.cloud.config.uri`配置只能放在`bootstrap.yml`中（所以必须添加`config`相关依赖，才能使用配置中心）

+ `spring.cloud.config.label`与`spring.cloud.config.profile`不一定要放在`bootstrap.yml`中，只要按照加载次序能加载到就行。

+ `spring.cloud.config.name`这个属性也不一定要放在`bootstrap.yml`中，只要按照加载次序能加载到就行。而且，如果该属性不配置，则以`spring.application.name`为准。也就是说，有乔选~~乔~~`spring.cloud.config.name`，无乔选~~鲨~~`spring.application.name`。

如果这`4`个属性不能全部加载到，则配置中心失效。

（3.2）然后从配置中心读取`application.yml`相关属性。

（3.3）之后再从本地读取`application.yml`相关属性。

（3.4）最后再从`bootstrap.yml`读取其他属性。

（4）如果使用`--spring.profiles.active=XX`参数，则指定生效的`application-XX.yml`在配置中心读取`application.yml`相关属性之后，本地读取`application.yml`相关属性之前加载。如果没有使用配置中心，则指定生效的`application-XX.yml`是最高优先级。

### 3. 我的总结

基本的加载次序为：
1. 配置中心的`application.yml`
2. 指定生效的`application-XX.yml`
3. 本地的`application.yml`
4. 本地的`bootstrap.yml`

相同的属性，不同的值，以第一次加载为准！

### 4. 我的建议
（1）如果项目组使用配置中心，则除了在配置中心的`application.yml`设置属性外，项目本地也保留一份与远端配置相同的本地的`application.yml`，这样是为了防止配置中心失效（概率极低）而不能启动项目

（2）如果项目组不使用配置中心，请将配置放在本地的`application.yml`中而不是`bootstrap.yml`中，因为读取`bootstrap.yml`文件需要额外添加`config`相关依赖

（3）如果项目组不使用配置中心，指定生效的`application-XX.yml`可用来同一个应用部署到不同环境，在启动时通过`--spring.profiles.active=XX`参数来指定生效的配置文件

**示例**：假如现在同一个应用需要部署到测试与生产，同时本地是第三套环境，则在资源路径添加3个文件：
+ `application.yml`（本地）
+ `application-test.yml`（测试）
+ `application-pro.yml`（生产）

本地启动时，不需要添加任何参数，使用的就是本地的配置；

在上测试时，指定`--spring.profiles.active=test`，则测试的配置生效；

在上生产时，指定`--spring.profiles.active=pro`，则生产的配置生效。

### 参考资料
#### 1. 【SpringBoot】SpringCloud Config Server实践
https://blog.csdn.net/ssrc0604hx/article/details/52802392

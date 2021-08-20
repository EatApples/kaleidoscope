# 实现一个 starter 有四个要素：

作者：古时的风筝
链接：https://juejin.cn/post/6844904167786446855
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

（1）starter 命名 ;

（2）自动配置类，用来初始化相关的 bean ;

（3）指明自动配置类的配置文件 spring.factories ;

（4）自定义属性实体类，声明 starter 的应用配置属性 ;

### 1. 给 starter 起个名字

也就是我们使用它的时候在 pom 中引用的 artifactId。命名有有规则的，官方规定：

官方的 starter 的命名格式为 spring-boot-starter-{name} ，例如上面提到的 spring-boot-starter-actuator。

非官方的 starter 的命名格式为 {name}-spring-boot-starter。

### 2. 引入自动配置包及其它相关依赖包

实现 starter 主要依赖自动配置注解，所以要在 pom 中引入自动配置相关的两个 jar 包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>

```

除此之外，依赖的其他包当然也要引进来。

### 3. 创建 spring.factories 文件

在 resource/META-INF 目录下创建名称为 spring.factories 的文件。

为什么在这里？当 Spring Boot 启动的时候，会在 classpath 下寻找所有名称为 spring.factories 的文件，然后运行里面的配置指定的自动加载类，将指定类(一个或多个)中的相关 bean 初始化。

例如本例中的配置信息是这样的：

```s
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
first,\
second
```

等号前面是固定的写法，后面就是我们自定义的自动配置类了，如果有多个的话，用英文逗号分隔开。

### 4. 编写自动配置类

自动配置类是用来初始化 starter 中的相关 bean 的。可以说是实现 starter 最核心的功能。

@EnableConfigurationProperties：启用配置类。
@ConditionalOnClass：只有在 classpath 中找到类的情况下，才会解析此自动配置类，否则不解析。
@Bean：实例化一个 bean
@ConditionalOnMissingBean：与 @Bean 配合使用，只有在当前上下文中不存在某个 bean 的情况下才会执行所注解的代码块
@ConditionalOnProperty：当应用配置文件中有相关的配置才会执行其所注解的代码块

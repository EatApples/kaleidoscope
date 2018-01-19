# Part III. Using Spring Boot
## 13. Build systems

### 13.1 Dependency management

### 13.2 Maven

+ JDK16以上
+ UTF-8编码
+ 添加spring-boot-starter-parent

```
<!-- Inherit defaults from Spring Boot -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.7.RELEASE</version>
</parent>
```

+ 设置JDK的版本
```
<properties>
	<java.version>1.8</java.version>
</properties>
```
+ 使用打包JAR插件
```
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

### 13.3 Gradle
### 13.4 Ant

### 13.5 Starters

## 14. Structuring your code

### 14.1 Using the “default” package
使用 @ComponentScan, @EntityScan or @SpringBootApplication 注解来触发依赖注入

### 14.2 Locating the main application class

之前用户使用的是3个注解注解他们的main类，分别是@Configuration、@EnableAutoConfiguration、@ComponentScan。由于这些注解一般都是一起使用，spring boot提供了一个统一的注解 @SpringBootApplication。

@SpringBootApplication = (默认属性)@Configuration + @EnableAutoConfiguration + @ComponentScan

@Configuration：提到@Configuration就要提到他的搭档@Bean。使用这两个注解就可以创建一个简单的spring配置类，可以用来替代相应的xml配置文件

@EnableAutoConfiguration：能够自动配置spring的上下文，试图猜测和配置你想要的bean类，通常会自动根据你的类路径和你的bean定义自动配置

@ComponentScan：会自动扫描指定包下的全部标有@Component的类，并注册成bean，当然包括@Component下的子注解@Service,@Repository,@Controller

## 15. Configuration classes
### 15.1 Importing additional configuration classes

@ComponentScan(指定包类扫描)

### 15.2 Importing XML configuration

@ImportResource 加载 XML

## 16. Auto-configuration

@EnableAutoConfiguration 或 @SpringBootApplication

### 16.1 Gradually replacing auto-configuration

### 16.2 Disabling specific auto-configuration

排除不想自动配置的类
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})


## 17. Spring Beans and dependency injection

@ComponentScan 找 beans(@Component,@Service, @Repository, @Controller etc.) + @Autowired 注入

### Spring 注释 @Autowired 和@Resource 的区别
```
（1）@Autowired和@Resource都可以用来装配bean，都可以写在字段上，或者方法上。
（2）@Autowired属于Spring的；@Resource为JSR-250标准的注释，属于J2EE的。
（3）@Autowired默认按类型装配，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，
例如：@Autowired(required=false) ，
如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下：
	@Autowired()
	@Qualifier("baseDao")
	private BaseDao baseDao;

（4）@Resource，默认安装名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名进行安装名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
例如：
@Resource(name="baseDao")
private BaseDao baseDao;
（5）推荐使用：@Resource注解在字段上，这样就不用写setter方法了，并且这个注解是属于J2EE的，减少了与spring的耦合。这样代码看起就比较优雅。
```

### @Component, @Repository, @Service的区别
@Service, @Controller , @Repository = {@Component + 一些特定的功能}。这个就意味着这些注解在部分功能上是一样的。

|注解         |含义                                        |
|-------------|--------------------------------------------|
|@Component	  |最普通的组件，可以被注入到spring容器进行管理|
|@Repository  |作用于持久层                                |
|@Service	  |作用于业务逻辑层                            |
|@Controller  |作用于表现层（spring-mvc的注解）            |

## 18. Using the @SpringBootApplication annotation

@SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan(默认属性)

## 19. Running your application

### 19.1 Running from an IDE

### 19.2 Running as a packaged application

$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n -jar target/myproject-0.0.1-SNAPSHOT.jar

### 19.3 Using the Maven plugin

### 19.4 Using the Gradle plugin

### 19.5 Hot swapping

## 20. Developer tools

Maven.
```
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

### 20.1 Property defaults

### 20.2 Automatic restart

工具 spring-boot-devtools 使程序在文件修改后自动重启（热重启）。

原理是：使用两个类加载器，未变的部分使用base classloader，改变的部分使用restart classloader。

资源文件（/META-INF/maven, /META-INF/resources, /resources, /static, /public 或 /templates）不会触发热重启，但会热加载。

如果不想触发默认路径热加载，使用：
spring.devtools.restart.exclude=static/**,public/**

如果想保留默认，禁用其他路径，则使用：
spring.devtools.restart.additional-exclude=

如果想启用不在默认路径的，使用：
spring.devtools.restart.additional-paths=

完全禁用热重启：

```
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```

因为热重启使用两个类加载器，在某些情况可能有问题，可以建一个文件：
META-INF/spring-devtools.properties

以 restart.exclude. 为前缀的将会排除，以 restart.include. 为前缀的将会包含。

例子（支持正则）：
```
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

### 20.3 LiveReload

不想热加载的，使用：
spring.devtools.livereload.enabled=false

### 20.4 Global settings

全局使用spring-boot-devtools

```
vi ~/.spring-boot-devtools.properties
spring.devtools.reload.trigger-file=.reloadtrigger
```

### 20.5 Remote applications

### 21. Packaging your application for production
添加 spring-boot-actuator 以使用扩展功能

### 22. What to read next

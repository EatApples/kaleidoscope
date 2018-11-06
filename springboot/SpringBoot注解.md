### 什么是注解
注解也叫元数据，例如我们常见的@Override和@Deprecated，注解是JDK1.5版本开始引入的一个特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。它主要的作用有以下四方面：

生成文档，通过代码里标识的元数据生成javadoc文档。
编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

### 一般注解可以分为三类：

1. 一类是Java自带的标准注解，包括@Override、@Deprecated和@SuppressWarnings，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告，用这些注解标明后编译器就会进行检查。

2. 一类为元注解，元注解是用于定义注解的注解，包括@Retention、@Target、@Inherited、@Documented，
@Retention用于标明注解被保留的阶段，
@Target用于标明注解使用的范围，
@Inherited用于标明注解可继承，
@Documented用于标明是否生成javadoc文档。

3. 一类为自定义注解，可以根据自己的需求定义注解，并可用元注解对自定义注解进行注解。

Java内置的注解直接使用即可，但很多时候我们需要自己定义一些注解，例如常见的spring就用了大量的注解来管理对象之间的依赖关系。

### 如何获取注解参数

这里强调一下，Annotation是被动的元数据，永远不会有主动行为，但凡Annotation起作用的场合都是有一个执行机制/调用者通过反射获得了这个元数据然后根据它采取行动

注解本身不做任何事情，只是像xml文件一样起到配置作用。注解代表的是某种业务意义，注解背后处理器的工作原理如上源码实现：首先解析所有属性，判断属性上是否存在指定注解，如果存在则根据搜索规则取得bean，然后利用反射原理注入。如果标注在字段上面，也可以通过字段的反射技术取得注解，根据搜索规则取得bean，然后利用反射技术注入。

看来如果想用自己的注解类，还是得编写自己的解析处理类

### 1. @SpringBootApplication
使用 @ComponentScan, @EntityScan or @SpringBootApplication 注解来触发依赖注入

之前用户使用的是3个注解注解他们的main类，分别是@Configuration、@EnableAutoConfiguration、@ComponentScan。由于这些注解一般都是一起使用，spring boot提供了一个统一的注解 @SpringBootApplication。

> @SpringBootApplication = (默认属性) @Configuration + @EnableAutoConfiguration+ @ComponentScan

### 2. @Configuration
@Configuration：提到@Configuration就要提到他的搭档@Bean。使用这两个注解就可以创建一个简单的spring配置类，可以用来替代相应的xml配置文件

### 3. @EnableAutoConfiguration
@EnableAutoConfiguration：能够自动配置spring的上下文，试图猜测和配置你想要的bean类，通常会自动根据你的类路径和你的bean定义自动配置
排除不想自动配置的类
> @EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})

### 4. @ComponentScan
@ComponentScan：会自动扫描指定包下的全部标有@Component的类，并注册成bean，当然包括@Component下的子注解 @Service，@Repository，@Controller

> @ComponentScan 找 beans(@Component,@Service, @Repository, @Controller etc.) + @Autowired 注入

### 5. @ImportResource
@ImportResource 加载 XML

### 6. Spring 注释 @Autowired 和@Resource 的区别
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

PS：Java变量的初始化顺序为：静态变量或静态语句块–>实例变量或初始化语句块–>构造方法–>@Autowired

### 7. @Component, @Service, @Controller , @Repository 的区别
> @Service, @Controller , @Repository = {@Component + 一些特定的功能}。

这个就意味着这些注解在部分功能上是一样的。

|注解         |含义                                        |
|-------------|--------------------------------------------|
|@Component	  |最普通的组件，可以被注入到spring容器进行管理|
|@Repository  |作用于持久层                                |
|@Service	  |作用于业务逻辑层                            |
|@Controller  |作用于表现层（spring-mvc的注解）            |

### 8. handler method 参数绑定常用的注解
@see
http://blog.csdn.net/walkerjong/article/details/7946109

handler method 参数绑定常用的注解，根据他们处理的Request的不同内容部分分为四类：（主要讲解常用类型）
+ 处理request uri 部分（这里指uri template中variable，不含queryString部分）的注解：@PathVariable;
+ 处理request header部分的注解：@RequestHeader, @CookieValue;
+ 处理request body部分的注解：@RequestParam,  @RequestBody;
+ 处理attribute类型是注解： @SessionAttributes, @ModelAttribute;

### 9. @PathVariable
当使用@RequestMapping URI template 样式映射时，即someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上

若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称

### 10. @RequestParam
+ 常用来处理简单类型的绑定，通过Request.getParameter() 获取的String可直接转换为简单类型的情况（ String--> 简单类型的转换操作由ConversionService配置的转换器来完成）；因为使用request.getParameter()方式获取参数，所以可以处理get 方式中queryString的值，也可以处理post方式中 body data的值；
+ 用来处理Content-Type: 为 application/x-www-form-urlencoded编码的内容，提交方式GET、POST；
+ 该注解有两个属性： value、required； value用来指定要传入值的id名称，required用来指示参数是否必须绑定；

```
问题：
Fiegn Client with Spring Boot: RequestParam.value() was empty on parameter 3

解释：
Both Spring MVC and Spring cloud feign are using same ParameterNameDiscoverer - named DefaultParameterNameDiscoverer to find parameter name. It tries to find the parameter names with the following step.

First, it uses StandardReflectionParameterNameDiscoverer. It tries to find the variable name with reflection. It is only possible when your classes are compiled with -parameter.

Second, if it fails, it uses LocalVariableTableParameterNameDiscoverer. It tries to find the variable name from the debugging info in the class file with ASM libraries.

The difference between Spring MVC and Feign occurs here. Feign uses above annotations (like @RequestParam) on methods of Java interfaces. But, we use these on methods of Java classes when using Spring MVC. Unfortunately, javac compiler omits the debug information of parameter name from class file for java interfaces. That's why feign fails to find parameter name without -parameter.

Namely, if you compile your code with -parameter, both Spring MVC and Feign will succeed to acquire parameter names. But if you compile without -parameter, only Spring MVC will succeed.

As a result, it's not a bug. it's a limitation of Feign at this moment as I think.

解决方案：
最好的做法是通过@RequestParam注解指定具体的参数名称
```
### 11. @RequestBody
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等

1，@RequestBody需要把所有请求参数作为json解析，因此，不能包含key=value这样的写法在请求url中，所有的请求参数都是一个json

2，直接通过浏览器输入url时，@RequestBody获取不到json对象，需要用java编程或者基于ajax的方法请求，将Content-Type设置为application/json

### 12. @Value
默认值设置：

@Value("${PROPERTY:DEFAULT_VALUE}")
　
### 13. @Configuration 和 @Component
首先看一下Spring官方文档是怎么说的：
```
The @Bean methods in a Spring component are processed differently than their counterparts inside a Spring @Configuration class. The difference is that @Component classes are not enhanced with CGLIB to intercept the invocation of methods and fields. CGLIB proxying is the means by which invoking methods or fields within @Bean methods in @Configuration classes creates bean metadata references to collaborating objects; such methods are not invoked with normal Java semantics but rather go through the container in order to provide the usual lifecycle management and proxying of Spring beans even when referring to other beans via programmatic calls to @Bean methods. In contrast, invoking a method or field in an @Bean method within a plain @Component class has standard Java semantics, with no special CGLIB processing or other constraints applying.

在Component中(@Component标注的类，包括@Service,@Repository, @Controller)使用@Bean注解和在@Configuration中使用是不同的。在@Component类中使用方法或字段时不会使用CGLIB增强(及不使用代理类：调用任何方法，使用任何变量，拿到的是原始对象，后面会有例子解释)。而在@Configuration类中使用方法或字段时则使用CGLIB创造协作对象（及使用代理：拿到的是代理对象）;当调用@Bean注解的方法时它不是普通的Java语义，而是从容器中拿到由Spring生命周期管理、被Spring代理甚至依赖于其他Bean的对象引用。在@Component中调用@Bean注解的方法和字段则是普通的Java语义，不经过CGLIB处理。
```

一句话概括就是 @Configuration 中所有带 @Bean 注解的方法都会被动态代理，因此调用该方法返回的都是同一个实例。

但是对于@Component，@Component 注解并没有通过cglib来代理@Bean方法的调用，因此调用带@Bean注解的方法态返回的都是新的实例。

从定义来看， @Configuration 注解本质上还是 @Component，因此 <context:component-scan/> 或者 @ComponentScan 都能处理@Configuration 注解的类。

@Configuration 标记的类必须符合下面的要求：
```
配置类必须以类的形式提供（不能是工厂方法返回的实例），允许通过生成子类在运行时增强（cglib 动态代理）。
配置类不能是 final 类（没法动态代理）。
配置注解通常为了通过 @Bean 注解生成 Spring 容器管理的类。
配置类必须是非本地的（即不能在方法中声明，不能是 private）。
任何嵌套配置类都必须声明为static。
@Bean方法不能创建进一步的配置类（也就是返回的bean如果带有@Configuration，也不会被特殊处理，只会作为普通的 bean）。
```

### 14. @Scope
```
@Scope(ConfigurableBeanFactory.SCOPE_SINGLETON) //默认类型是单例
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) //可生成多个对象
```

### 15. REST参数解析
```
@QueryParam 用于从请求 URL 的查询组件中提取查询参数
如果需要为参数设置默认值，可以使用 @DefaultValue
@FormParam 顾名思义是处理 HTML表单请求的
@MatrixParam 从 URL 路径提取信息
@HeaderParam 从 HTTP 头部提取信息
@CookieParam从关联在 HTTP 头部的 cookies 里提取信息
@BeanParam 允许注入参数到一个 bean
@Context 一般可以用于获得一个Java类型关联请求或响应的上下文
```
### 参考资料
#### 1. @Bean在@Configuration和在@Component中的区别
https://blog.csdn.net/ttjxtjx/article/details/49866011

#### 2. Spring @Configuration 和 @Component 区别
https://blog.csdn.net/isea533/article/details/78072133?locationNum=7&fps=1

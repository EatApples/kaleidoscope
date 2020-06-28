### SpringBoot 自动装配原理

SpringBoot 所有自动配置类都是在启动的时候进行扫描并加载，通过 spring.factories 可以找到自动配置类的路径，但是不是所有存在于 spring.factories 中的配置都进行加载，而是通过@ConditionalOnClass 注解进行判断条件是否成立（只要导入相应的 stater，条件就能成立），如果条件成立则加载配置类，否则不加载该配置类。

spring 的配置文件目录可以放在
/config
/(根目录)
resource/config/
resource/

SpringBoot 默认可以加载以下三种配置文件：
application.yml
application.yaml
application.properties

SpringBoot 的注解扫描的默认规则是 SpringBoot 的入口类所在包及其子包。

@ConfigurationProperties 这个注解，可以将 yml 文件中写好的值注入到我们类的属性中

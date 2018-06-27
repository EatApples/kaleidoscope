### Spring application中配置项的优先级
来自：https://www.cnblogs.com/zhangjianbin/p/7443194.html
```yml
bootstrapProperties   # 来自configServer的值
commandLineArgs       # 命令行参数
servletConfigInitParams                                         
servletContextInitParams                                        
systemProperties                                                
systemEnvironment                       
random      
applicationConfig: [classpath:/application.yml]
springCloudClientHostInfo
applicationConfig: [classpath:/bootstrap.yml]
defaultProperties
Management Server
```
Spring想要得到一个配置的值，就按照上面的顺序一个个去找，找到就直接返回。由于ConfigServer处于最优先级，本地项目不管怎么设置都不能覆盖。

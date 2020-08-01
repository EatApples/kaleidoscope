### 关于 Springboot 配置文件的设置

# 指定配置文件的搜索路径，按理说是路径，不应该直接定位到文件

--spring.config.location=PATH

# 指定生效的文件后缀，即 application-LABEL，貌似只认 application 为前缀的文件

--spring.profiles.active=LABEL

# 可以通过 JVM 参数的方式指定

-Dspring.config.location=PATH
-Dspring.profiles.active=LABEL

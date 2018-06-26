### logback 按天循环日志配置

```xml

<?xml version="1.0" encoding="UTF-8"?>
<!-- logback 按天循环日志配置 -->
<configuration>
	<!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径 -->
	<property name="LOG_HOME" value="./log" />
	<property name="LOG_NAME" value="MyLog" />
	<!-- 控制台输出 -->
	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
		<layout class="ch.qos.logback.classic.PatternLayout">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}
				-%msg%n</pattern>
		</layout>
	</appender>
	<!-- 按照每天生成日志文件 -->
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">

		<rollingPolicy
			class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<!-- 每天一归档 -->
			<fileNamePattern>${LOG_HOME}/${LOG_NAME}.%d{yyyy-MM-dd}.%i.log
			</fileNamePattern>
			<!-- 单个日志文件最多 10MB, 30天的日志周期 -->
			<maxFileSize>10MB</maxFileSize>
			<maxHistory>30</maxHistory>
		</rollingPolicy>

		<layout class="ch.qos.logback.classic.PatternLayout">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}
				-%msg%n
			</pattern>
		</layout>

	</appender>

	<!-- 日志输出级别 -->
	<root level="INFO">
		<appender-ref ref="STDOUT" />
		<appender-ref ref="FILE" />
	</root>

</configuration>


```
SpringBoot 中默认 logback 的配置文件：
>org/springframework/boot/logging/logback/base.xml

在 spring-boot-1.5.2.RELEASE 中 org.springframework.boot.logging.logback 下

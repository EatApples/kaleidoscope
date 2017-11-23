# logback 按天循环日志配置

```
	<!-- 按照每天生成日志文件 -->
	<appender name="FILE"
		class="ch.qos.logback.core.rolling.RollingFileAppender">

		<rollingPolicy
			class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<!-- 每天一归档 -->
			<fileNamePattern>${LOG_HOME}/pplication.log.%d{yyyy-MM-dd}.%i.log
			</fileNamePattern>
			<!-- 单个日志文件最多 10MB, 30天的日志周期 -->
			<maxFileSize>10MB</maxFileSize>
			<maxHistory>30</maxHistory>
		</rollingPolicy>

		<layout class="ch.qos.logback.classic.PatternLayout">
			<!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
			<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} -
				%msg%n
			</pattern>
		</layout>

	</appender>
	
```
原先的版本 <spring.boot.version>1.5.9.RELEASE</spring.boot.version> 最近有安全问题，升级为 1.5.11.RELEASE。

Spring Cloud与Spring Boot版本匹配关系

| Spring Cloud     | Spring Boot                                    |
| ---------------- | ---------------------------------------------- |
| Finchley         | 兼容Spring Boot 2.0.x，不兼容Spring Boot 1.5.x |
| Dalston和Edgware | 兼容Spring Boot 1.5.x，不兼容Spring Boot 2.0.x |
| Camden           | 兼容Spring Boot 1.4.x，也兼容Spring Boot 1.5.x |
| Brixton          | 兼容Spring Boot 1.3.x，也兼容Spring Boot 1.4.x |
| Angel            | 兼容Spring Boot 1.2.x                          |

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<!-- 项目名称配置，请自定义修改 -->
	<groupId>com.creditease</groupId>
	<artifactId>SpringCloud-demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<!-- 基本配置，开始 -->
	<properties>
		<java.version>1.8</java.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<spring.boot.version>1.5.11.RELEASE</spring.boot.version>
		<spring.cloud.version>Dalston.SR5</spring.cloud.version>
	</properties>

	<dependencyManagement>
		<dependencies>

			<dependency>
				<!-- Import dependency management from Spring Boot -->
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-starter-parent</artifactId>
				<version>${spring.boot.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

			<dependency>
				<!-- Import dependency management from Spring Cloud -->
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring.cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>

		</dependencies>
	</dependencyManagement>
	<!-- 基本配置，结束 -->


	<dependencies>
		<!-- 基本依赖，开始 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-client</artifactId>
		</dependency>
		<!-- 基本依赖，结束 -->

		<!-- 此处添加个性化依赖 -->

	</dependencies>

	<!-- 配合 bootstrap.yml 使用配置中心 -->
	<profiles>
		<profile>
			<!-- 测试环境 -->
			<id>test</id>
      <!-- 一些配置属性 -->
			<properties>

			</properties>
      <!-- 默认生效 -->
			<activation>
				<activeByDefault>true</activeByDefault>
			</activation>

		</profile>
		<profile>
			<!-- 生产环境 -->
			<id>pro</id>
      <!-- 一些配置属性 -->
			<properties>

			</properties>
		</profile>
	</profiles>

	<!-- 打包配置 -->
	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>true</filtering>
			</resource>
		</resources>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>


</project>

```
### 参考资料
#### 1. Spring Cloud与Spring Boot版本匹之间的关系
https://blog.csdn.net/yyzc2/article/details/79299805

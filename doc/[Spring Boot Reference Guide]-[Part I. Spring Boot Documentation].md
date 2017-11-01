# Part II. Getting started
## 8. Introducing Spring Boot
使用 jar -jar 运行即可

## 9. System Requirements
推荐用JDK18

### 9.1 Servlet containers
容器对应的JDK版本

|Names          |Servlet Version  |Java Version |
----------------------------------------------
|Tomcat 8       |3.1              |Java 7+      |
|Tomcat 7       |3.0              |Java 6+      |
|Jetty 9.3      |3.1              |Java 8+      |
|Jetty 9.2      |3.1              |Java 7+      |
|Jetty 8        |3.0              |Java 6+      |
|Undertow 1.3   |3.1              |Java 7+      |

## 10. Installing Spring Boot
先看下JDK版本

$ java -version

### 10.1 Installation instructions for the Java developer
装个 Maven

### 10.2 Installing the Spring Boot CLI

### 10.3 Upgrading from an earlier version of Spring Boot

## 11. Developing your first Spring Boot application
### 11.1 Creating the POM
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

	<modelVersion>4.0.0</modelVersion>
	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>1.5.7.RELEASE</version>
	</parent>
	<!-- Additional lines to be added here... -->

</project>
```

### 11.2 Adding classpath dependencies
```
<dependencies>
	<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```

### 11.3 Writing the code
```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;
@RestController
@EnableAutoConfiguration
public class Example {
	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}
	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```

### 11.4 Running the example

### 11.5 Creating an executable jar
在 POM 中 dependencies 之后添加：
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

## 12. What to read next
开始使用吧
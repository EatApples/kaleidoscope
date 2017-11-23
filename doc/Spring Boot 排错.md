# Fiegn Client
## 问题：

Fiegn Client with Spring Boot: RequestParam.value() was empty on parameter 3

## 原因（内容来自stackoverflow）：

Both Spring MVC and Spring cloud feign are using same ParameterNameDiscoverer - named DefaultParameterNameDiscoverer to find parameter name. It tries to find the parameter names with the following step.

First, it uses StandardReflectionParameterNameDiscoverer. It tries to find the variable name with reflection. It is only possible when your classes are compiled with -parameter.

Second, if it fails, it uses LocalVariableTableParameterNameDiscoverer. It tries to find the variable name from the debugging info in the class file with ASM libraries.

The difference between Spring MVC and Feign occurs here. Feign uses above annotations (like @RequestParam) on methods of Java interfaces. But, we use these on methods of Java classes when using Spring MVC. Unfortunately, javac compiler omits the debug information of parameter name from class file for java interfaces. That's why feign fails to find parameter name without -parameter.

Namely, if you compile your code with -parameter, both Spring MVC and Feign will succeed to acquire parameter names. But if you compile without -parameter, only Spring MVC will succeed.

As a result, it's not a bug. it's a limitation of Feign at this moment as I think.

## 解决方案：

最好的做法是通过 @RequestParam 注解指定具体的参数名称

# spring boot自动注入

## 问题：
spring boot自动注入出现Consider defining a bean of type 'xxx' in your configuration

## 解决方案：
将接口与对应的实现类放在与 application 启动类的同一个目录或者他的子目录下，这样注解可以被扫描到，这是最省事的办法 

或在指定的 application 类上加上 @SpringBootApplication(scanBasePackages = {"com.XXX.YYY"}) 

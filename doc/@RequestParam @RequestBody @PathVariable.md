# 链接
http://blog.csdn.net/walkerjong/article/details/7946109

handler method 参数绑定常用的注解，根据他们处理的Request的不同内容部分分为四类：（主要讲解常用类型）
+ 处理request uri 部分（这里指uri template中variable，不含queryString部分）的注解：@PathVariable;
+ 处理request header部分的注解：@RequestHeader, @CookieValue;
+ 处理request body部分的注解：@RequestParam,  @RequestBody;
+ 处理attribute类型是注解： @SessionAttributes, @ModelAttribute;

# @PathVariable 
当使用@RequestMapping URI template 样式映射时，即someUrl/{paramId}, 这时的paramId可通过 @Pathvariable注解绑定它传过来的值到方法的参数上

若方法参数名称和需要绑定的uri template中变量名称不一致，需要在@PathVariable("name")指定uri template中的名称

# @RequestParam 
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
# @RequestBody
该注解常用来处理Content-Type: 不是application/x-www-form-urlencoded编码的内容，例如application/json, application/xml等

1，@RequestBody需要把所有请求参数作为json解析，因此，不能包含key=value这样的写法在请求url中，所有的请求参数都是一个json

2，直接通过浏览器输入url时，@RequestBody获取不到json对象，需要用java编程或者基于ajax的方法请求，将Content-Type设置为application/json

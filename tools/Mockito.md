### 1. 常用的 Mockito 方法

| 方法名                                                                                                    | 描述                                          |
| --------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| Mockito.mock(classToMock)                                                                                 | 模拟对象                                      |
| Mockito.verify(mock)                                                                                      | 验证行为是否发生                              |
| Mockito.when(methodCall).thenReturn(value1).thenReturn(value2)                                            | 触发时第一次返回 value1，第 n 次都返回 value2 |
| Mockito.doThrow(toBeThrown).when(mock).[method]                                                           | 模拟抛出异常。                                |
| Mockito.mock(classToMock,defaultAnswer) 使用默认 Answer                                                   | 模拟对象                                      |
| Mockito.when(methodCall).thenReturn(value)                                                                | 参数匹配                                      |
| Mockito.doReturn(toBeReturned).when(mock).[method]                                                        | 参数匹配（直接执行不判断）                    |
| Mockito.when(methodCall).thenAnswer(answer))                                                              | 预期回调接口生成期望值                        |
| Mockito.doAnswer(answer).when(methodCall).[method]                                                        | 预期回调接口生成期望值（直接执行不判断）      |
| Mockito.spy(Object)                                                                                       | 用 spy 监控真实对象,设置真实对象行为          |
| Mockito.doNothing().when(mock).[method]                                                                   | 不做任何返回                                  |
| Mockito.doCallRealMethod().when(mock).[method] //等价于 Mockito.when(mock.[method]).thenCallRealMethod(); | 调用真实的方法                                |
| reset(mock)                                                                                               | 重置 mock                                     |

作者：PC_Repair
链接：https://www.jianshu.com/p/ecbd7b5a2021
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 2. 使用 Mockito 修改私有属性

https://blog.csdn.net/tcctcszhanghao/article/details/107079445

```java
public class Whitebox {
    /**
     *
     * @param target    目标对象
     * @param fieldName 属性
     * @param value     mock 值
     */
    public static void setInternalState(Object target, String fieldName, Object value) {
        try {
            Field field = target.getClass().getDeclaredField(fieldName);
            field.setAccessible(true); // 粗爆的改成可访问，不管现有修饰
            field.set(target, value);
        } catch (Exception e) {
            Assert.fail("Cannot change value of " + fieldName + " against target " + target);
        }
    }
}
```

### 3. Mockito 3.6.0 中文文档

https://changingfond.github.io/mockito-zh-doc.html

### 4. MockHttpServletRequest+MockHttpServletResponse+MockFilterChain

引入

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <scope>test</scope>
</dependency>
```

```java
MockHttpServletRequest servletRequest = new MockHttpServletRequest();
ServletResponse servletResponse = new MockHttpServletResponse();
FilterChain filterChain = new MockFilterChain();
```

### 5. Mockito 示例

```java
// mock 属性
Whitebox.setInternalState(XXX, "YYY", ZZZ);
// mock 对象
HttpServletRequest request = Mockito.mock(HttpServletRequest.class);
// 描述行为
Mockito.when(request.getParameter(ArgumentMatchers.anyString())).thenReturn("test");
// 注入
Injector.inject(被注入实例, (Class<?>) 被注入实例.class, "属性", (Object) 注入对象);
// HandlerMethod
HandlerMethod handler = Mockito.mock(HandlerMethod.class);
Mockito.when(handler.getMethod()).thenReturn(NoLoginExample.class.getDeclaredMethod("some_method", null));
Mockito.when(handler.getMethod()).thenReturn(MethodExample.class.getDeclaredMethod("some_method", null));

// StringRedisTemplate
StringRedisTemplate redisTemplate = Mockito.mock(StringRedisTemplate.class, Mockito.RETURNS_DEEP_STUBS);
```

使用注解的类：

```java
public class MethodExample {
    @Login
    public void some_method() {

    }
}

public class NoLoginExample {

    public void some_method() {

    }

}
```

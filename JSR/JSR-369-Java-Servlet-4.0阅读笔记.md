### 1. 什么是 Servlet？

Servlet 是基于 Java 技术的 web 组件，容器托管的，用于生成动态内容。像其他基于 Java 的组件技术一样，Servlet 也是基于平台无关的 Java 类格式，被编译为平台无关的字节码，可以被基于 Java 技术的 web server动态加载并运行。容器，有时候也叫做 servlet 引擎，是 web server 为支持 servlet 功能扩展的部分。客户端通过 Servlet 容器实现的请求/应答模型与 Servlet 交互。

### 2. 什么是 Servlet 容器？

Servlet 容器是 web server 或 application server 的一部分，提供基于请求/响应发送模型的网络服务，解码基于 MIME 的请求，并且格式化基于 MIME 的响应。 Servlet 容器也包含了管理 Servlet 生命周期

所有 Servlet 容器必须支持基于 HTTP 协议的请求/响应模型，比如像基于 HTTPS（ HTTP over SSL）协议的请求/应答模型可以选择性的支持。容器必须实现的 HTTP 协议版本包含 HTTP/1.0 和 HTTP/1.1。

Java SE 6 是构建 Servlet 容器最低的 Java 平台版本。

### 3. Servlet 接口
目前有 GenericServlet 和 HttpServlet 这两个类实现了 Servlet 接口。大多数情况下，开发者只需要继承 HttpServlet 去实现自己的 Servlet 即可。

Web 应用程序的并发请求处理通常需要 Web 开发人员去设计适合多线程执行的 Servlet，从而保证 service方法能在一个特定时间点处理多线程并发执行。（注：即 Servlet 默认是线程不安全的，需要开发人员处理多线程问题）

### 4. 多线程问题

Servlet 容器可以并发的发送多个请求到 Servlet 的 service 方法。为了处理这些请求， Servlet 开发者必须为service 方法的多线程并发处理做好充足的准备。一个替代的方案是开发人员实现 SingleThreadModel 接口，由容器保证一个 service 方法在同一个时间点仅被一个请求线程调用，但是此方案是不推荐的。

Servlet 容器可以通过串行化访问 Servlet 的请求，或者维护一个 Servlet 实例池完成该需求。如果 Web 应用中的 Servlet 被标注为分布式的，容器应该为每一个分布式应用程序的 JVM 维护一个 Servlet 实例池。

对于那些没有实现 SingleThreadModel 接口的 Servlet，但是它的 service 方法（或者是那些 HttpServlet 中通过 service 方法分派的 doGet、 doPost 等分派方法）是通过 synchronized 关键词定义的， Servlet 容器不能使用实例池方案，并且只能使用序列化请求进行处理。强烈推荐开发人员不要去通过 service 方法（或者那些由 Service 分派的方法），因为这将严重影响性能。

SingleThreadModel 接口的作用是保证一个特定 servlet 实例的 service 方法在一个时刻仅能被一个线程执行，一定要注意，此保证仅适用于每一个 servlet 实例，因此容器可以选择池化这些对象。

### 5. 加载和实例化

Servlet 容器负责加载和实例化 Servlet。加载和实例化可以发生在容器启动时，或者延迟初始化直到容器决定有请求需要处理时。当 Servlet 引擎启动后， servlet 容器必须定位所需要的 Servlet 类。

一旦一个 Servlet 对象实例化完毕，容器接下来必须在处理客户端请求之前初始化该 Servlet 实例。初始化的目的是以便 Servlet能读取持久化配置数据，初始化一些代价高的资源（比如 JDBC API 连接），或者执行一些一次性的动作。容器通过调用 Servlet 实例的 init 方法完成初始化， init 方法定义在 Servlet 接口中，并且提供一个唯一的 ServletConfig 接口实现的对象作为参数，该对象每个 Servlet 实例一个。

### 6. 异步处理

Servlet 3.0 引入了异步处理请求的能力，使线程可以返回到容器，从而执行更多的任务。当开始异步处理请求时，另一个线程或回调可以或者产生响应，或者调用完成（ complete）或请求分派（ dispatch），这样，它可以在容器上下文使用 AsyncContext.dispatch 方法运行。

### 7. HTTP 协议参数

ServletRequest 接口的下列方法可访问这些参数：

+ getParameter

+ getParameterNames

+ getParameterValues

+ getParameterMap

### 8. 请求路径元素

使用 HttpServletRequest 接口中的下面方法来访问这些信息：

+ getContextPath

+ getServletPath

+ getPathInfo

重要的是要注意，除了请求 URI 和路径部分的 URL 编码差异外，下面的等式永远为真：

requestURI = contextPath + servletPath + pathInfo

### 9. 非阻塞 IO
Web 容器中的非阻塞请求处理有助于提高对改善 Web 容器可扩展性不断增加的需求，增加 Web 容器可同时处理请求的连接数量。servlet 容器的非阻塞 IO 允许开发人员在数据可用时读取数据或在数据可写时写数据。

### 10. request 对象的生命周期

每个 request 对象只在 servlet 的 service 方法的作用域内，或过滤器的 doFilter方法的作用域内有效，除非该组件启用了异步处理并且调用了 request 对象的 startAsync 方法。在发生异步处理的情况下， request 对象一直有效，直到调用 AsyncContext 的 complete 方法。容器通常会重复利用 request 对象，以避免创建 request对象的性能开销。

### 11. Reload 注意事项

尽管容器供应商不需要实现类的重新加载（ reload）模式以便易于开发，但是任何此类的实现必须确保所有 servlet 及它们使用的类（ Servlet 使用的系统类异常可能使用的是一个不同的 class loader）在一个单独的 class loader 范围内被加载。为了保证应用像开发人员预期的那样工作，该要求是必须的。

### 12. 包装请求和响应

过滤器的核心概念是包装请求或响应，以便它可以覆盖行为执行过滤任务。

过滤器是一种代码重用的技术，它可以改变 HTTP 请求的内容，响应，及 header 信息。过滤器通常不产生响应或像 servlet 那样对请求作出响应，而是修改或调整到资源的请求，修改或调整来自资源的响应。

### Tomcat与支持的Servlet规范
| Servlet Spec | Latest Released Version | Supported Java Versions                |
| ------------ | ----------------------- | -------------------------------------- |
| 4.0          | 9.0.11                  | 8 and later                            |
| 3.1          | 8.5.33                  | 7 and later                            |
| 3.0          | 7.0.90                  | 6 and later(7 and later for WebSocket) |

### 扩展阅读
#### 1. Tomcat与支持的Servlet规范
http://tomcat.apache.org/whichversion.html

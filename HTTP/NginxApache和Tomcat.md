### Nginx、Apache 和 Tomcat：

Nginx：由俄罗斯程序员 Igor Sysoev 所开发的轻量级、高并发 HTTP 服务器。
Apache HTTP Server Project：一个 Apache 基金会下的 HTTP 服务项目，和 Nginx 功能类似。
Apache Tomcat：是 Apache 基金会下的另外一个项目，是一个 Application Server。
更准确的说是一个 Servlet 应用容器，与 Apache HTTP Server 和 Nginx 相比，Tomcat 能够动态的生成资源并返回到客户端。
Apache HTTP Server 和 Nginx 本身不支持生成动态页面，但它们可以通过其他模块来支持（例如通过 Shell、PHP、Python 脚本程序来动态生成内容）。

# HTTP Server

一个 HTTP Server 关心的是 HTTP 协议层面的传输和访问控制，所以在 Apache/Nginx 上你可以看到代理、负载均衡等功能。
客户端通过 HTTP Server 访问服务器上存储的资源（HTML 文件、图片文件等等）。
通过 CGI 技术，也可以将处理过的内容通过 HTTP Server 分发，但是一个 HTTP Server 始终只是把服务器上的文件如实的通过 HTTP 协议传输给客户端。

# 应用服务器

而应用服务器，则是一个应用执行的容器。它首先需要支持开发语言的运行（对于 Tomcat 来说，就是 Java），保证应用能够在应用服务器上正常运行。
其次，需要支持应用相关的规范，例如类库、安全方面的特性。对于 Tomcat 来说，就是需要提供 JSP/Sevlet 运行需要的标准类库、Interface 等。
为了方便，应用服务器往往也会集成 HTTP Server 的功能，但是不如专业的 HTTP Server 那么强大。
所以应用服务器往往是运行在 HTTP Server 的背后，执行应用，将动态的内容转化为静态的内容之后，通过 HTTP Server 分发到客户端。

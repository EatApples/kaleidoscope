# cookie+session

### 记住我功能：cookie 实现

通常，我们可以从很多网站的登录界面中看到“请记住我”这样的选项，如果你勾选了它之后再登录，那么在下一次访问该网站的时候就不需要进行重复而繁琐的登录动作了，而这个功能就是通过 Cookie 实现的。

### sessionid 如何产生？

sessionid 是一个会话的 key，浏览器第一次访问服务器会在服务器端生成一个 session，有一个 sessionid 和它对应。tomcat 生成的 sessionid 叫做 jsessionid。

### sessionid 由谁产生？

session 在访问 tomcat 服务器 HttpServletRequest 的 getSession(true)的时候创建，tomcat 的 ManagerBase 类提供创建 sessionid 的方法：随机数+时间+jvmid；

### session 保存在哪里？

存储在服务器的内存中，tomcat 的 StandardManager 类将 session 存储在内存中，也可以持久化到 file，数据库，memcache，redis 等。客户端只保存 sessionid 到 cookie 中，而不会保存 session，session 销毁只能通过 invalidate 或超时，关掉浏览器并不会关闭 session。

### session 的销毁

超时；程序调用 HttpSession.invalidate()；程序关闭；

本文作者：@Ryan Miao
本文链接：https://www.cnblogs.com/woshimrf/p/5317776.html
版权声明： 本博客所有文章除特别声明外，均采用 CC BY-NC-SA 3.0 许可协议。转载请注明出处！
http://www.cnblogs.com/sharpxiajun/p/3395607.html
http://lavasoft.blog.51cto.com/62575/275589/

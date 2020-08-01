### SO_REUSEPORT

SO_REUSEPORT 干的其实是大众期望 SO_REUSEADDR 能够干的事，将多个 socket 绑定到同一 ip 和端口。并且它要求所有绑定同一 ip/port 的 socket 都设置了 SO_REUSEPORT。不过可能有的操作系统并没有这个 option。

Linux 最新 SO_REUSEPORT 特性：
（1）允许多个套接字 bind()/listen() 同一个 TCP/UDP 端口，每一个线程拥有自己的服务器套接字
（2）在服务器套接字上没有了锁的竞争
（3）内核层面实现负载均衡
（4）安全层面，监听同一个端口的套接字只能位于同一个用户下面

作者：小忍甜甜圈
链接：https://www.jianshu.com/p/a23b7e8a4c6a
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

https://blog.csdn.net/yaokai_assultmaster/article/details/68951150
https://www.cnblogs.com/Anker/p/7076537.html

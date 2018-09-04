## 单机最大tcp连接数
来自：http://wanshi.iteye.com/blog/1256282

### 如何标识一个TCP连接

在确定最大连接数之前，先来看看系统如何标识一个tcp连接。系统用一个4四元组来唯一标识一个TCP连接：{local ip, local port,remote ip,remote port}。

### client最大tcp连接数

client每次发起tcp连接请求时，除非绑定端口，通常会让系统选取一个空闲的本地端口（local port），该端口是独占的，不能和其他tcp连接共享，因此client端的tcp连接数取决于/受限于端口数量。tcp端口的数据类型是unsigned short，因此本地端口个数最大只有65536，端口0有特殊含义，不能使用，这样可用端口最多只有65535，所以在全部作为client端的情况下，一个client最大tcp连接数为65535，这些连接可以连到不同的server ip。

这里感觉有误。一个套接字只能建立一个连接，无论对于 server 还是 client。
https://blog.lilydjwg.me/2015/8/19/tcp-fun.180084.html

### server最大tcp连接数

server通常固定在某个本地端口上监听，等待client的连接请求。不考虑地址重用（unix的SO_REUSEADDR选项）的情况下，即使server端有多个ip，本地监听端口也是独占的，因此server端tcp连接4元组中只有remote ip（也就是client ip）和remote port（客户端port）是可变的，因此最大tcp连接为客户端ip数×客户端port数，对IPV4，不考虑ip地址分类等因素，最大tcp连接数约为2的32次方（ip数）×2的16次方（port数），也就是server端单机最大tcp连接数约为2的48次方。

### 实际的tcp连接数

上面给出的是理论上的单机最大连接数，在实际环境中，受到机器资源、操作系统等的限制，特别是sever端，其最大并发tcp连接数远不能达到理论上限。在unix/linux下限制连接数的主要因素是内存和允许的文件描述符个数（每个tcp连接都要占用一定内存，每个socket就是一个文件描述符），另外1024以下的端口通常为保留端口。

对server端，通过增加内存、修改最大文件描述符个数等参数，单机最大并发TCP连接数超过10万,甚至上百万 是没问题的，国外 Urban Airship 公司在产品环境中已做到 50 万并发 。在实际应用中，对大规模网络应用，还需要考虑C10K ，c100k问题。

### 结论
单机最大tcp连接数，不论作为客户端还是服务器，最大为TCP连接：{local ip, local port,remote ip,remote port} 的种数。

（1）作为服务端，最多开启65535个端口。最多建立65535个服务（每个服务占用一个端口）。（资源足够的情况下）每个端口最多建立 2^48 个TCP连接（2^32 IP * 2^16 PORT）。

（2）作为客户端，最多开启65535个端口。对于同一个端口，（资源足够的情况下）最多连接 2^48 个服务器。

TCP 连接会占用系统资源（文件描述符），有时候甚至会导致系统假死（不能发起或者处理 TCP 请求）。

# CLOSE_WAIT 被动关闭，程序代码错误经常导致连接不能释放。

20170208 大量 Http 请求 close_wait 的问题
https://blog.csdn.net/gisredevelopment/article/details/54930040

（1）在发起 get 或者 post 请求时，设置 Connection 属性为 close，而非 keep-alive，如：httpGet.setHeader("Connection","close");
（2）代码中的 http 连接都没有关闭，所以导致出现大量的 close_wait

但是，当超过 net.ipv4.tcp_keepalive_time 时间，对端如果已经释放，那么本端的 TCP 也会释放。

# TIME_WAIT 主动关闭，会等待 2MSL。

解决 TIME_WAIT 过多造成的问题
https://www.cnblogs.com/dadonggg/p/8778318.html

在高并发短连接的 TCP 服务器上，当服务器处理完请求后立刻主动正常关闭连接。这个场景下会出现大量 socket 处于 TIME_WAIT 状态。如果客户端的并发量持续很高，此时部分客户端就会显示连接不上。

（1）高并发可以让服务器在短时间范围内同时占用大量端口，而端口有个 0~65535 的范围，并不是很多，刨除系统和其他服务要用的，剩下的就更少了。
（2）在这个场景中，短连接表示“业务处理+传输数据的时间 远远小于 TIMEWAIT 超时的时间”的连接。

综合这两个方面，持续的到达一定量的高并发短连接，会使服务器因端口资源不足而拒绝为一部分客户服务。同时，这些端口都是服务器临时分配，无法用 SO_REUSEADDR 选项解决这个问题。

编辑内核文件/etc/sysctl.conf，加入以下内容：

net.ipv4.tcp_syncookies = 1 表示开启 SYN Cookies。当出现 SYN 等待队列溢出时，启用 cookies 来处理，可防范少量 SYN 攻击，默认为 0，表示关闭；
net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将 TIME-WAIT sockets 重新用于新的 TCP 连接，默认为 0，表示关闭；
net.ipv4.tcp_tw_recycle = 1 表示开启 TCP 连接中 TIME-WAIT sockets 的快速回收，默认为 0，表示关闭。
net.ipv4.tcp_fin_timeout 修改系默认的 TIMEOUT 时间

然后执行 /sbin/sysctl -p 让参数生效

简单来说，就是打开系统的 TIME_WAIT 重用和快速回收。

如果以上配置调优后性能还不理想，可继续修改一下配置：

vi /etc/sysctl.conf
net.ipv4.tcp_keepalive_time = 1200 #表示当 keepalive 起用的时候，TCP 发送 keepalive 消息的频度。缺省是 2 小时，改为 20 分钟。
net.ipv4.ip_local_port_range = 1024 65000 #表示用于向外连接的端口范围。缺省情况下很小：32768 到 61000，改为 1024 到 65000。
net.ipv4.tcp_max_syn_backlog = 8192 #表示 SYN 队列的长度，默认为 1024，加大队列长度为 8192，可以容纳更多等待连接的网络连接数。
net.ipv4.tcp_max_tw_buckets = 5000 #表示系统同时保持 TIME_WAIT 套接字的最大数量，如果超过这个数字，TIME_WAIT 套接字将立刻被清除并打印警告信息。
默认为 180000，改为 5000。对于 Apache、Nginx 等服务器，上几行的参数可以很好地减少 TIME_WAIT 套接字数量，但是对于 Squid，效果却不大。此项参数可以控制 TIME_WAIT 套接字的最大数量，避免 Squid 服务器被大量的 TIME_WAIT 套接字拖死。

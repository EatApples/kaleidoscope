### 1 Zabbix 简介
Zabbix 是由 Alexei Vladishev 开发的一种网络监视、管理系统，基于 Server-Client 架构。

可用于监视各种网络服务、服务器和网络机器等状态。

使用各种 Database-end 如 MySQL, PostgreSQL, SQLite, Oracle 或 IBM DB2 储存资料。

Server 端基于 C语言、Web 管理端 frontend 则是基于 PHP 所制作的。

Zabbix 可以使用多种方式监视。可以只使用 Simple Check 不需要安装 Client 端，亦可基于 SMTP 或 HTTP ... 各种协定做死活监视。在客户端如 UNIX, Windows 中安装 Zabbix Agent 之后，可监视 CPU Load、网络使用状况、硬盘容量等各种状态。而就算没有安装 Agent 在监视对象中，Zabbix 也可以经由 SNMP、TCP、ICMP、利用 IPMI、SSH、telnet 对目标进行监视。另外，Zabbix 包含 XMPP 等各种 Item 警示功能。

Zabbix 的授权是属于 GPLv2。

主要是由 Alexei Vladishev 所设立的 Zabbix SIA 做开发与维护。


### 2 进程介绍
+ zabbix_agentd

客户端守护进程，此进程收集客户端数据，例如 cpu 负载、内存、硬盘使用情况等

+ zabbix_get

zabbix 工具，单独使用的命令，通常在 server 或者 proxy 端执行获取远程客户端信息的命令。通常用户排错。例如在 server 端获取不到客户端的内存数据，我们可以使用 zabbix_get 获取客户端的内容的方式来做故障排查。

+ zabbix_sender
zabbix 工具，用于发送数据给 server 或者 proxy，通常用于耗时比较长的检查。很多检查非常耗时间，导致zabbix 超时。于是我们在脚本执行完毕之后，使用 sender 主动提交数据。

+ zabbix_server

zabbix 服务端守护进程。zabbix_agentd、zabbix_get、zabbix_sender、zabbix_proxy、zabbix_java_gateway的数据最终都是提交到 server 备注：当然不是数据都是主动提交给 zabbix_server,也有的是 server 主动去取数据。

+ zabbix_proxy

zabbix 代理守护进程。功能类似 server，唯一不同的是它只是一个中转站，它需要把收集到的数据提交/被提交到 server 里。

+ zabbix_java_gateway

zabbix2.0 之后引入的一个功能。顾名思义：Java 网关，类似 agentd，但是只用于 Java 方面。需要特别注意的是，它只能主动去获取数据，而不能被动获取数据。它的数据最终会给到 server 或者 proxy。

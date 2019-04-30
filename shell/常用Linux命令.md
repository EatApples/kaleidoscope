### 启用 Shell 脚本调试模式的方法
```sh
-v （verbose 的简称） - 告诉 Shell 读取脚本时显示所有行，激活详细模式。
-n （noexec 或 no ecxecution 简称） - 指示 Shell 读取所有命令然而不执行它们，这个选项激活语法检查模式。
-x （xtrace 或 execution trace 简称） - 告诉 Shell 在终端显示所有执行的命令和它们的参数。 这个选项是启用 Shell 跟踪模式。
```

### find
```sh
#找到N天前的文件
find 对应目录 -mtime +天数 -type f -name "文件名"
```

### exec
```sh
# -exec 必须由一个 ; 结束，而因为通常 shell 都会对 ; 进行处理，所以用 \; 防止这种情况
# {}是占位符，用来替换前一个命令的输出
前一个命令 -exec 后一个命令 {} \;
```

### xsrgs
```sh
# xargs命令每次只获取一部分文件而不是全部
前一个命令 | xargs 后一个命令
```

### scp
```sh
# scp [参数] [原路径] [目标路径]
# 复制文件  
scp local_file remote_username@remote_ip:remote_folder  

# 复制目录
scp -r local_folder remote_username@remote_ip:remote_folder  
```

### netstat
```sh
# 显示网卡列表
netstat -i

# 显示网络统计信息
netstat -s

# 显示关于路由表的信息
netstat -r

# 找出运行在指定端口的进程
netstat -apn | grep XX

# 找出监听端口的进程
nestat -tunlp | grep XX

```
### ss
```sh
# 显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效
# -n, --numeric	不解析服务名称
# -a, --all	显示所有套接字（sockets）
# -l, --listening	显示监听状态的套接字（sockets）
# -t, --tcp	仅显示 TCP套接字（sockets）
# -u, --udp	仅显示 UCP套接字（sockets）
# -4, --ipv4           仅显示IPv4的套接字（sockets）
# -6, --ipv6           仅显示IPv6的套接字（sockets）
# -s, --summary	显示套接字（socket）使用概况
# -p, --processes	显示使用套接字（socket）的进程
# 显示TCP连接
ss -t -a

#显示所有UDP Sockets
ss -u -a

# 显示 Sockets 摘要
ss -s

# 列出所有打开的网络连接端口
ss -l

# 查看进程使用的socket
ss -lp

# 匹配远程地址和端口号
ss dst ADDRESS_PATTERN

# 匹配本地地址和端口号
ss src ADDRESS_PATTERN
```

### traceroute
```sh
# 我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径
traceroute -n HOSTNAME
```

### ping
```sh
# -i 秒数：设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次
# -c 数目：在发送指定数目的包后停止
ping -c 10 127.0.0.1
```

### route
```sh
# Flags标志说明：
# U Up表示此路由当前为启动状态
# H Host，表示此网关为一主机
# G Gateway，表示此网关为一路由器
route -n
```

### ifconfig
```sh
# 第一行：连接类型：Ethernet（以太网）HWaddr（硬件mac地址）
# 第二行：网卡的IP地址、子网、掩码
# 第三行：UP（代表网卡开启状态）RUNNING（代表网卡的网线被接上）MULTICAST（支持组播）MTU:1500（最大传输单元）：1500字节
# 第四、五行：接收、发送数据包情况统计
# 第七行：接收、发送数据字节数统计信息

ifconfig
```

### lsof
```sh
# lsof（list open files）是一个列出当前系统打开文件的工具
# 用于查看你进程开打的文件，打开文件的进程，进程打开的端口(TCP、UDP)。找回/恢复删除的文件。是十分方便的系统监视工具，因为 lsof 需要访问核心内存和各种文件，所以需要root用户执行。
# lsof输出各列信息的意义如下：
# COMMAND：进程的名称
# PID：进程标识符
# PPID：父进程标识符（需要指定-R参数）
# USER：进程所有者
# PGID：进程所属组
# FD：文件描述符，应用程序通过文件描述符识别该文件。如cwd、txt等
# 一般在标准输出、标准错误、标准输入后还跟着文件状态模式：r、w、u等
# TYPE：文件类型，如DIR、REG等，常见的文件类型
# （1）DIR：表示目录
# （2）CHR：表示字符类型
# （3）BLK：块设备类型
# （4）UNIX： UNIX 域套接字
# （5）FIFO：先进先出 (FIFO) 队列
# （6）IPv4：网际协议 (IP) 套接字
# DEVICE：指定磁盘的名称
# SIZE：文件的大小
# NODE：索引节点（文件在磁盘上的标识）
# NAME：打开文件的确切名称

# 列出某个程序进程所打开的文件信息
lsof -c mysql

# 查看谁正在使用某个文件，也就是说查找某个文件相关的进程
lsof 文件名

# 列出谁在使用某个端口
lsof -i tcp:3306
```

### crontab
```sh
# -u user：用来设定某个用户的crontab服务，例如，“-u ixdba”表示设定ixdba用户的crontab服务，此参数一般有root用户来运行
# file：file是命令文件的名字,表示将file做为crontab的任务列表文件并载入crontab。如果在命令行中没有指定这个文件，crontab命令将接受标准输入（键盘）上键入的命令，并将它们载入crontab
# -e：编辑某个用户的crontab文件内容。如果不指定用户，则表示编辑当前用户的crontab文件
# -l：显示某个用户的crontab文件内容，如果不指定用户，则表示显示当前用户的crontab文件内容
# -r：从/var/spool/cron目录中删除某个用户的crontab文件，如果不指定用户，则默认删除当前用户的crontab文件

crontab [-u user] file
crontab [-u user] [ -e | -l | -r ]
```
### watch
```sh
# watch可以帮你监测一个命令的运行结果，省得你一遍遍的手动运行
# 可以将命令的输出结果输出到标准输出设备，多用于周期性执行命令/定时执行命令
# -n或--interval  watch缺省每2秒运行一下程序，可以用-n或-interval来指定间隔的时间。
# -d或--differences  用-d或--differences 选项watch 会高亮显示变化的区域。 而-d=cumulative选项会把变动过的地方(不管最近的那次有没有变动)都高亮显示出来。
# -t 或-no-title  会关闭watch命令在顶部的时间间隔,命令，当前时间的输出。

# 查看系统内存的变化
watch -n 1 -d 'free -m'
```

###
```sh
```

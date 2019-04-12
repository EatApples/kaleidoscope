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

### 

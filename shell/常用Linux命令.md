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

### date
```sh
# %H 小时(以00-23来表示)。
# %M 分钟(以00-59来表示)。
# %S 秒(以本地的惯用法来表示)。
# %d 日期(以01-31来表示)。
# %m 月份(以01-12来表示)。
# %Y 年份(以四位数来表示)。
date +"%Y%m%d%H%M%S"
# 设置当前时间，只有root权限才能设置，其他只能查看。设置具体时间，不会对日期做更改
date -s 01:01:01
```

### df
```sh
# linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。
# -a 全部文件系统列表
# -h 方便阅读方式显示
# -i 显示inode信息
df [选项] [文件]
df -h
df -i
```

### du
```sh
# -a或-all  显示目录中个别文件的大小。   
# -s或--summarize  仅显示总计，只列出最后加总的值。
# -h或--human-readable  以K，M，G为单位，提高信息的可读性。
# 显示每个文件和目录的磁盘使用空间
du [选项][文件]
# 方便阅读的格式显示
du -h 文件
# 文件和目录都显示
du -ah 文件
# 按照空间大小排序
du|sort -nr|more
# 输出当前目录下各个子目录所使用的空间
du -h  --max-depth=1
```

### exec
```sh
# -exec 必须由一个 ; 结束，而因为通常 shell 都会对 ; 进行处理，所以用 \; 防止这种情况
# {}是占位符，用来替换前一个命令的输出
前一个命令 -exec 后一个命令 {} \;
```

### find
```sh
#找到N天前的文件
find 对应目录 -mtime +天数 -type f -name "文件名"
find 对应目录 -mtime +天数 -type d -name "目录名"
```

### grep
```sh
#grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户
ps -ef | grep java
#查看具体cpu
ps -mp PID -o THREAD,tid,time
```

### gzip
```sh
# gzip不仅可以用来压缩大的、较少使用的文件以节省磁盘空间，还可以和tar命令一起构成Linux操作系统中比较流行的压缩文件格式。据统计，gzip命令对文本文件有60%～70%的压缩率。
# -d或--decompress或----uncompress 　解开压缩文件
# -r或--recursive 　递归处理，将指定目录下的所有文件及子目录一并处理
# -v或--verbose 　显示指令执行过程
gzip[参数][文件或者目录]
# 把当前目录下的每个文件压缩成.gz文件
gzip *
# 每个压缩的文件解压，并列出详细的信息
gzip -dv *
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

### less
```sh
# less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。
# -i  忽略搜索时的大小写
# -m  显示类似more命令的百分比
# -N  显示每行的行号
# -o <文件名> 将less 输出的内容在指定文件中保存起来
# -s  显示连续空行为一行
# /字符串：向下搜索“字符串”的功能
# ?字符串：向上搜索“字符串”的功能
# n：重复前一个搜索（与 / 或 ? 有关）
# Q  退出less 命令
# 空格键 滚动一页
# 回车键 滚动一行
# [pagedown]： 向下翻动一页
# [pageup]：   向上翻动一页
less -mN 文件名
```

### ln
```sh
# Linux文件系统中，有所谓的链接(link)，我们可以将其视为档案的别名，而链接又可分为两种 : 硬链接(hard link)与软链接(symbolic link)，硬链接的意思是一个档案可以有多个名称，而软链接的方式则是产生一个特殊的档案，该档案的内容是指向另一个档案的位置。硬链接是存在同一个文件系统中，而软链接却可以跨越不同的文件系统。
# 软链接：（引用的引用，间接引用）
# 1.软链接，以路径的形式存在。类似于Windows操作系统中的快捷方式
# 2.软链接可以 跨文件系统 ，硬链接不可以
# 3.软链接可以对一个不存在的文件名进行链接
# 4.软链接可以对目录进行链接
# 硬链接:（直接引用）
# 1.硬链接，以文件副本的形式存在。但不占用实际空间。
# 2.不允许给目录创建硬链接
# 3.硬链接只有在同一个文件系统中才能创建
# 这里有两点要注意：
# 第一，ln命令会保持每一处链接文件的同步性，也就是说，不论你改动了哪一处，其它的文件都会发生相同的变化；
# 第二，ln的链接又分软链接和硬链接两种，软链接就是ln –s 源文件 目标文件，它只会在你选定的位置上生成一个文件的镜像，不会占用磁盘空间，硬链接 ln 源文件 目标文件，没有参数-s， 它会在你选定的位置上生成一个和源文件大小相同的文件，无论是软链接还是硬链接，文件都保持同步变化。
ln [参数][源文件或目录][目标文件或目录]
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

### ping
```sh
# -i 秒数：设定间隔几秒送一个网络封包给一台机器，预设值是一秒送一次
# -c 数目：在发送指定数目的包后停止
ping -c 10 127.0.0.1
```

### rmdir
```sh
# 该命令从一个目录中删除一个或多个子目录项，删除某目录时也必须具有对父目录的写权限
rmdir 目录名
# -p 递归删除目录dirname，当子目录删除后其父目录为空时，也一同被删除
rmdir -p 目录名
```

### route
```sh
# Flags标志说明：
# U Up表示此路由当前为启动状态
# H Host，表示此网关为一主机
# G Gateway，表示此网关为一路由器
route -n
```

### scp
```sh
# scp [参数] [原路径] [目标路径]
# 复制文件  
scp local_file remote_username@remote_ip:remote_folder  

# 复制目录
scp -r local_folder remote_username@remote_ip:remote_folder  
```

### sh
```sh
#启用 Shell 脚本调试模式的方法
# -v （verbose 的简称） - 告诉 Shell 读取脚本时显示所有行，激活详细模式。
# -n （noexec 或 no ecxecution 简称） - 指示 Shell 读取所有命令然而不执行它们，这个选项激活语法检查模式。
# -x （xtrace 或 execution trace 简称） - 告诉 Shell 在终端显示所有执行的命令和它们的参数。 这个选项是启用 Shell 跟踪模式。

sh -n XX.sh
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

### tar
```sh
# 用来压缩和解压文件。tar本身不具有压缩功能，调用压缩功能实现的
# -A 新增压缩文件到已存在的压缩
# -r 添加文件到已经压缩的文件
# -x 从压缩的文件中提取文件
# -z 支持gzip解压文件
# -j 支持bzip2解压文件
# -Z 支持compress解压文件
# -v 显示操作过程
# 可选参数如下：
# -f 指定压缩文件
tar[必要参数][选择参数][文件]
# 解包
tar -xvf FileName.tar
# 打包
tar -cvf FileName.tar DirName
```

### traceroute
```sh
# 我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径
traceroute -n HOSTNAME
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

### wc
```sh
#统计指定文件中的字节数、字数、行数，并将统计结果显示输出。
# -c 统计字节数。
# -l 统计行数。
# -m 统计字符数。这个标志不能与 -c 标志一起使用。
# -w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串。

ls -al | wc -l
```

### which
```sh
# 我们经常在linux要查找某个文件，但不知道放在哪里了，可以使用下面的一些命令来搜索：
# which 指令会在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果
# whereis 命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息
# locate 指令和find找寻档案的功能类似，但locate是透过update程序将硬盘中的所有档案和目录资料先建立一个索引数据库，在 执行loacte时直接找该索引，查询速度会较快，索引数据库一般是由操作系统管理，但也可以直接下达update强迫系统立即修改索引数据库。
# find  实际搜寻硬盘查询文件名称。
which 可执行文件名称
```

### xsrgs
```sh
# xargs命令每次只获取一部分文件而不是全部
前一个命令 | xargs 后一个命令
```

### 资料来源
#### 1. 每日一linux命令
https://www.cnblogs.com/peida/tag/%E6%AF%8F%E6%97%A5%E4%B8%80linux%E5%91%BD%E4%BB%A4/

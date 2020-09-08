### 1. 日期显示

```sh
date +"%Y%m%d%H%M%S"

%H 小时，24小时制（00~23）
%I 小时，12小时制（01~12）
%k 小时，24小时制（0~23）
```

### 2. 设置文本格式

```
set fileformats=unix
或
set ff=unix
```

### 3. CURL

```
curl -h 来查看请求参数的含义
     -v 显示请求的信息
     -X 选项指定其它协议

curl -v -X [GET|POST|PUT|DELETE] "IP:PORT/PATH"
```

- put 请求

```sh
curl -v -X PUT -d "param1=value1&param2=value2" "IP:PORT/PATH"
```

- delete 请求

```sh
curl -v -X DELETE "IP:PORT/PATH"
```

- get 请求

```sh
curl -v "IP:PORT/PATH"
curl "http://www.baidu.com"     如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
curl -i "http://www.baidu.com"  显示全部信息
curl -l "http://www.baidu.com"  只显示头部信息
curl -v "http://www.baidu.com"  显示get请求全过程解析
wget "http://www.baidu.com"     也可以
```

- post 请求

```sh
curl -d "param1=value1&param2=value2" "http://www.baidu.com"
```

- json 格式的 post 请求

```sh
curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13521389587","password":"test"}' http://domain/apis/users.json
```

### 4. 在 Linux 上可用以下语句看了一下服务器的 TCP 状态(连接状态数量统计)：

```sh
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

netstat -tunlp | grep PORT
```

### 5. 环境变量

- login shell：当你通过终端输入用户名和密码，然后进入到 terminal，这时候进入的 shell 环境就叫做是 login shell，例如，通过 ssh 远程进入到主机。

- no-login shell：顾名思义就是不需要输入用户名密码而进入的 shell 环境，例如你已经登陆了你的桌面电脑，这时候在应用管理器中找到 terminal 图标，然后双击打开终端，也就是通过像 gnome，KDE 这种桌面环境而进入的终端，这时候你进入的 shell 环境就是所谓的 no-login shell 环境。

简而言之，就是把你想通过 login shell 运行的 shell 命令放入到 .bash_profile 中，把想通过 no-login shell 运行的 shell 命令放入到 .bashrc 文件中。

~/.bash_profile 是交互式、login 方式进入 bash 运行的

~/.bashrc 是交互式 non-login 方式进入 bash 运行的

通常二者设置大致相同，所以通常前者会调用后者。

.bash_profile 只在会话开始时被读取一次，而.bashrc 则每次打开新的终端时，都会被读取。

### 6. 机器信息

```sh
# 查看物理CPU的个数
cat /proc/cpuinfo |grep "physical id"|sort |uniq|wc -l

# 查看逻辑CPU的个数
cat /proc/cpuinfo |grep "processor"|wc -l

# 查看CPU是几核
cat /proc/cpuinfo |grep "cores"|uniq

# 查看CPU的主频
cat /proc/cpuinfo |grep MHz|uniq

# 查看当前操作系统内核信息
cat /proc/version
uname -a
cat /etc/issue
cat /etc/issue | grep Linux

# 查看操作系统位数
getconf LONG_BIT
```

### 7. 处理命令行参数的一个样例

```
while [ "$1" != "" ]; do
    case $1 in
        -s  )   shift
        SERVER=$1 ;;
        -d  )   shift
        DATE=$1 ;;
    --paramter|p ) shift
        PARAMETER=$1;;
        -h|help  )   usage # function call
                exit ;;
        * )     usage # All other parameters
                exit 1
    esac
    shift
done
```

### 8. 命令行菜单的一个样例

```sh
#!/bin/bash
# Bash Menu Script Example

PS3='Please enter your choice: '
options=("Option 1" "Option 2" "Option 3" "Quit")
select opt in "${options[@]}"
do
    case $opt in
        "Option 1")
            echo "you chose choice 1"
            ;;
        "Option 2")
            echo "you chose choice 2"
            ;;
        "Option 3")
            echo "you chose choice $REPLY which is $opt"
            ;;
        "Quit")
            break
            ;;
        *) echo "invalid option $REPLY";;
    esac
done
```

### 9. free -m 内存解析

```
total：内存总数
used：已经使用的内存数
free：空闲的内存数
shared：多个进程共享的内存总额，当前已经废弃不用，总是0
-buffers/cache：(已用)的内存数，即used-buffers-cached
+buffers/cache：(可用)的内存数，即free+buffers+cached
```

### 10. 查看线程数

```
top -Hp PID
cat /proc/PID/status
```

### 11. ulimit -a

从事分布式服务器开发工作的都会遇到，linux 下 open_file 的值默认是 1024；max user processes 的值默认是 4096，在实际用于中，这两个值严重不足，常常需要调整这两个值。

查看系统配置

```
sysctl -a | grep keepalive
```

用户最大进程数

```
max user processes (-u) 1024
```

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
```

### 3. CURL
+ get请求
```sh
curl "http://www.baidu.com"     如果这里的URL指向的是一个文件或者一幅图都可以直接下载到本地
curl -i "http://www.baidu.com"  显示全部信息
curl -l "http://www.baidu.com"  只显示头部信息
curl -v "http://www.baidu.com"  显示get请求全过程解析
wget "http://www.baidu.com"     也可以
```

+ post请求
```sh
curl -d "param1=value1&param2=value2" "http://www.baidu.com"
```

+ json格式的post请求
```sh
curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13521389587","password":"test"}' http://domain/apis/users.json
```

### 4. 在 Linux 上可用以下语句看了一下服务器的TCP状态(连接状态数量统计)：
```sh
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

netstat -tunlp | grep PORT
```

### 5. 环境变量
+ login shell：当你通过终端输入用户名和密码，然后进入到terminal，这时候进入的shell环境就叫做是login shell，例如，通过ssh远程进入到主机。

+ no-login shell：顾名思义就是不需要输入用户名密码而进入的shell环境，例如你已经登陆了你的桌面电脑，这时候在应用管理器中找到terminal图标，然后双击打开终端，也就是通过像gnome，KDE这种桌面环境而进入的终端，这时候你进入的shell环境就是所谓的no-login shell环境。

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

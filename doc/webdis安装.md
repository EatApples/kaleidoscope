webdis是一个简单的 Web 服务器，提供了 HTTP 接口来访问 Redis 服务器，项目地址：
 
https://github.com/nicolasff/webdis/

# 先装 yum install libevent-devel

1. 下载安装包
新版本是libevent-2.0.10-stable。（如果你的系统已经安装了libevent，可以不用安装）

官网：http://www.monkey.org/~provos/libevent/

下载：http://www.monkey.org/~provos/libevent-2.0.10-stable.tar.gz

2. 解压 
$ tar zxvf libevent-2.0.10-stable.tar.gz

3. 进入目录
$ cd libevent-2.0.10-stable

4. 切换到root
$ su root

5. 安装gcc
$ yum install gcc

6. 设置安装路径
$ ./configure -prefix=/usr

7. 编译
$ make

8. 安装
$ make install

/usr/local/lib 目录下应该可以看见大量的动态链接库了

# 下载 webdis
https://github.com/nicolasff/webdis/

$ cd webdis
$ make
$ ./webdis &

# 遇到问题：/webdis: error while loading shared libraries: libevent-2.0.so.5: cannot open shared object file: No such file or directory

可能的原因有两个： 
（1）你忘了执行上面提到的ln -s....命令,这是因为运行时动态库的搜索路径默认是/lib以及/usr/lib路径。

（2）如果还是不行, 运行命令 ldconfig 生效。 ldconfig 通常在系统启动时运行,而当用户安装了一个新的动态链接库时,就需要手工运行这个命令进行更新。

# 解决方法： ldconfig

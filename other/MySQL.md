## MySQL 相关
### 1. Windows 安装 MySQL 报缺少 data 目录
查看日志发现缺少 data 目录，官网给出的解决方式是使用命令 (mysqld --initialize) 先初始化 data 目录，生成 data 目录，然后在执行 mysql 服务启动命令。

（1）修改 my.ini，配置好 data 目录，放到 bin 路径下；

（2）执行：
> mysqld --initialize --user=mysql --console

会自动生成 data 目录，还有一个随机密码，记下来。

（3）执行：
> mysqld --console

启动 mysql ，登录 mysql -uroot -p 输入刚才记录的密码，如果忘记密码,

> mysqld --skip-grant-tables

注意：在输入此命令之前先在任务管理器中结束 mysqld.exe 进程，确保 mysql 服务器端已结束运行
```
show databases;
use mysql;
show tables;
select * from user;
select user,host,authentication_string from user;
update user set authentication_string=password('password') where user='user' and host='localhost';
```

（4）杀掉mysqld进程。安装服务，执行：
> mysqld --install MySQL --defaults-file="/my.ini"

> net start mysql


### 2. Windows 卸载 MySQL
发现MYSQL有问题时，最便捷的方法，是先把 mysql 卸载掉，然后重装，重新配置，具体方法如下：
（1）卸载 MySQL，清理掉安装目录和 Windows 目录下的 my.ini 文件。

（2）检查任务管理器中是否还有 mysql 进程，如果有，可以把 mysqld.exe 杀掉，或者先杀掉再卸载也可以。

（3）在cmd命令窗口，执行：sc delete mysql，该命令是清理注册服务命令。

### 3. Linux 重装 MySQL

（1）移除老版本
```
    rpm -qa | grep -i mysql
    yum -y remove mysql-libs*
    yum remove mysql mysql-server mysql-libs mysql-server;
```

（2）安装
```
    rpm -ivh MySQL-server-5.6.35-1.el6.x86_64.rpm
    rpm -ivh MySQL-client-5.6.35-1.el6.x86_64.rpm
```

（3）修改配置文件位置

>    cp /usr/share/mysql/my-default.cnf /etc/my.cnf

（4）初始化MySQL及设置密码
```
    /usr/bin/mysql_install_db
    service mysql start
    cat .mysql_secret #查看初始密码
    ln -s /usr/local/mysql/bin/mysql /usr/bin
    mysql -uroot –p初始密码
    SET PASSWORD = PASSWORD('新密码');
```

### 4. MySQL 忘记密码
```
# /etc/init.d/mysql stop
# mysqld_safe --user=mysql --skip-grant-tables --skip-networking &

新开SHELL
# mysql -u root mysql
mysql> UPDATE user SET Password=PASSWORD('新密码') where USER='root';
mysql> FLUSH PRIVILEGES;
mysql> quit

重启
# /etc/init.d/mysql restart
# mysql -uroot
Enter password: <输入新设的密码>

```

### 5. 允许远程登陆
```
    use mysql;
    select host,user,password from user;
    UPDATE user SET password=password('新密码') where USER='root'
    update user set host='%' where user='root' and host='localhost';
    flush privileges;
```

### 6. select 的默认值
```
    select
    case when exists( SQL 语句)
    then (SQL 语句 )
    else '默认值'
    end
```

### 7. 释放磁盘空间
（1）drop table table_name 立刻释放磁盘空间，不管是Innodb和MyISAM。

（2）truncate table table_name 立刻释放磁盘空间 ，不管是 Innodb和MyISAM 。truncate table 其实有点类似于drop table 然后creat，只不过这个 create table 的过程做了优化，比如表结构文件之前已经有了等等。所以速度上应该是接近drop table的速度。

（3）delete from table_name 删除表的全部数据，对于 MyISAM 会立刻释放磁盘空间（应该是做了特别处理，也比较合理），InnoDB 不会释放磁盘空间。

（4）对于 delete from table_name where xxx 带条件的删除, 不管是innodb还是MyISAM都不会释放磁盘空间。

（5）delete 操作以后，使用 optimize table table_name 会立刻释放磁盘空间。不管是innodb还是myisam。所以要想达到释放磁盘空间的目的，delete 以后执行optimize table 操作。

（6）delete from 表，虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以使用这部分空间。


### 8. 往 MySQL 插入中文
当向 MySQL 数据库插入一条带有中文的数据出现乱码时，可以使用语句
>show variables like 'character%';

来查看当前数据库的相关编码集。

可以看到 MySQL 有六处使用了字符集，分别为：client 、connection、database、results、server、system。

其中与服务器端相关：database、server、system（永远无法修改，就是utf-8）；
与客户端相关：connection、client、results 。

```
connection  为连接数据库的字符集设置类型，如果程序没有指明连接数据库使用的字符集类型则按照服务器端默认的字符集设置。
client  为客户端使用的字符集。
results 为数据库给客户端返回时使用的字符集设定，如果没有指明，使用服务器默认的字符集。

database    为数据库服务器中某个库使用的字符集设定，如果建库时没有指明，将使用服务器安装时指定的字符集设置。
server  为服务器安装时指定的默认字符集设定。
system  为数据库系统使用的字符集设定。

1、看数据库字符集
show create database test;

2、看数据表字符集
show create table t_data;

```

了解了上面的信息我们来分析下乱码的原因，问题出在了当前的 CMD 客户端窗口，因为当前的 CMD 客户端输入采用 GBK 编码，而数据库的编码格式为 UTF-8，编码不一致导致了乱码产生。而当前 CMD 客户端的编码格式无法修改，所以只能修改 connection、 client、results 的编码集来告知服务器端当前插入的数据采用 GBK 编码，而服务器的数据库虽然是采用 UTF-8 编码，但却可以识别通知服务器端的 GBK 编码数据并将其自动转换为 UTF-8 进行存储。可以使用如下语句来快速设置与客户端相关的编码集：
>set names gbk;

设置完成后即可解决客户端插入数据或显示数据的乱码问题了，但我们马上会发现这种形式的设置只会在当前窗口有效，当窗口关闭后重新打开 CMD 客户端的时候又会出现乱码问题；那么，如何进行一个一劳永逸的设置呢？在 MySQL 的安装目录下有一个 my.ini 配置文件，通过修改这个配置文件可以一劳永逸的解决乱码问题。在这个配置文件中 [mysql] 与客户端配置相关，[mysqld] 与服务器配置相关。默认配置如下：
```
[mysql]
default-character-set=utf8
[mysqld]
character-set-server=utf8
```
这时只需要将下的默认编码 default-character-set=utf8 改为 default-character-set=gbk ，重新启动 MySQL 服务即可。

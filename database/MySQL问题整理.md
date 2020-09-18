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

（4）杀掉 mysqld 进程。安装服务，执行：

> mysqld --install MySQL --defaults-file="/my.ini"

> net start mysql

### 2. Windows 卸载 MySQL

发现 MYSQL 有问题时，最便捷的方法，是先把 mysql 卸载掉，然后重装，重新配置，具体方法如下：
（1）卸载 MySQL，清理掉安装目录和 Windows 目录下的 my.ini 文件。

（2）检查任务管理器中是否还有 mysql 进程，如果有，可以把 mysqld.exe 杀掉，或者先杀掉再卸载也可以。

（3）在 cmd 命令窗口，执行：sc delete mysql，该命令是清理注册服务命令。

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

> cp /usr/share/mysql/my-default.cnf /etc/my.cnf

（4）初始化 MySQL 及设置密码

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

（1）drop table table_name 立刻释放磁盘空间，不管是 Innodb 和 MyISAM。

（2）truncate table table_name 立刻释放磁盘空间 ，不管是 Innodb 和 MyISAM 。truncate table 其实有点类似于 drop table 然后 creat，只不过这个 create table 的过程做了优化，比如表结构文件之前已经有了等等。所以速度上应该是接近 drop table 的速度。

（3）delete from table_name 删除表的全部数据，对于 MyISAM 会立刻释放磁盘空间（应该是做了特别处理，也比较合理），InnoDB 不会释放磁盘空间。

（4）对于 delete from table_name where xxx 带条件的删除, 不管是 innodb 还是 MyISAM 都不会释放磁盘空间。

（5）delete 操作以后，使用 optimize table table_name 会立刻释放磁盘空间。不管是 innodb 还是 myisam。所以要想达到释放磁盘空间的目的，delete 以后执行 optimize table 操作。

（6）delete from 表，虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以使用这部分空间。

### 8. 往 MySQL 插入中文

当向 MySQL 数据库插入一条带有中文的数据出现乱码时，可以使用语句

> show variables like 'character%';

来查看当前数据库的相关编码集。

可以看到 MySQL 有六处使用了字符集，分别为：client 、connection、database、results、server、system。

其中与服务器端相关：database、server、system（永远无法修改，就是 utf-8）；
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

> set names gbk;

设置完成后即可解决客户端插入数据或显示数据的乱码问题了，但我们马上会发现这种形式的设置只会在当前窗口有效，当窗口关闭后重新打开 CMD 客户端的时候又会出现乱码问题；那么，如何进行一个一劳永逸的设置呢？在 MySQL 的安装目录下有一个 my.ini 配置文件，通过修改这个配置文件可以一劳永逸的解决乱码问题。在这个配置文件中 [mysql] 与客户端配置相关，[mysqld] 与服务器配置相关。默认配置如下：

```
[mysql]
default-character-set=utf8
[mysqld]
character-set-server=utf8
```

这时只需要将下的默认编码 default-character-set=utf8 改为 default-character-set=gbk ，重新启动 MySQL 服务即可。

### 9. 连接失效

```java
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

错误分析：数据库连接已经关闭或者失效后仍然在执行操作，导致 mysql 服务没返回数据

```
1，客户端连接池中连接已经失效。但是连接池还没有检测到，当操作数据库时，启用该连接，抛出该错误

2，mysql服务器，已关闭该连接，但客户端连接池中该连接，尚未检测到。当用该连接操作数据库时，抛出该错
```

处理办法：

```
1，每次连接操作数据库时，检测该连接的有效性

2，缩短监控空闲线程的时间
```

### 10. SQL 优化

#### Oracle：

explain plan for( SQL );

select \* from table(dbms_xplan.display);

#### Mysql：

explain

#### 执行次序

```
（8）SELECT （9）DISTINCT <select_list>
（1）FROM <left_table>
（3）<join_type> JOIN <right_table>
（2）ON <join_condition>
（4）WHERE <where_condition>
（5）GROUP BY <group_by_list>
（6）WITH {CUBE|ROLLUP}
（7）HAVING <having_condition>
（10）ORDER BY <order_by_list>
（11）LIMIT <limit_number>
```

（1）FROM 连接
首先，对 SELECT 语句执行查询时，对FROM 关键字两边的表执行连接，会形成笛卡尔积，这时候会产生一个虚表VT1(virtual table)

（2）ON 过滤
然后对 FROM 连接的结果进行 ON 筛选，创建 VT2，把符合记录的条件存在 VT2 中。

（3）JOIN 连接
第三步，如果是 OUTER JOIN(left join、right join) ，那么这一步就将添加外部行，如果是 left join 就把 ON 过滤条件的左表添加进来，如果是 right join ，就把右表添加进来，从而生成新的虚拟表 VT3。

（4）WHERE 过滤
第四步，是执行 WHERE 过滤器，对上一步生产的虚拟表引用 WHERE 筛选，生成虚拟表 VT4。

WHERE 和 ON 的区别
+ 如果有外部列，ON 针对过滤的是关联表，主表(保留表)会返回所有的列;
+ 如果没有添加外部列，两者的效果是一样的;

应用
+ 对主表的过滤应该使用 WHERE;
+ 对于关联表，先条件查询后连接则用 ON，先连接后条件查询则用 WHERE;

（5）GROUP BY
根据 group by 字句中的列，会对 VT4 中的记录进行分组操作，产生虚拟机表 VT5。如果应用了group by，那么后面的所有步骤都只能得到的 VT5 的列或者是聚合函数（count、sum、avg等）。

with
数据的展示方式

（6）HAVING
紧跟着 GROUP BY 字句后面的是 HAVING，使用 HAVING 过滤，会把符合条件的放在 VT6

（7）SELECT
第七步才会执行 SELECT 语句，将 VT6 中的结果按照 SELECT 进行刷选，生成 VT7

（8）DISTINCT
在第八步中，会对 TV7 生成的记录进行去重操作，生成 VT8。事实上如果应用了 group by 子句那么 distinct 是多余的，原因同样在于，分组的时候是将列中唯一的值分成一组，同时只为每一组返回一行记录，那么所以的记录都将是不相同的。

（9）ORDER BY
应用 order by 子句。按照 order_by_condition 排序 VT8，此时返回的一个游标，而不是虚拟表。sql 是基于集合的理论的，集合不会预先对他的行排序，它只是成员的逻辑集合，成员的顺序是无关紧要的。

- 嵌套子查询可以优化为连接查询；

- 连接表时，先用 where 条件对表进行过滤，然后再连接；

- 建立合适的索引，必要时建立联合索引；

### 11. 清理 DB

SELECT CONCAT('TRUNCATE TABLE ',table_schema,'.',TABLE_NAME, ';')
FROM INFORMATION_SCHEMA.TABLES 
WHERE table_schema in ('DB-NAME');

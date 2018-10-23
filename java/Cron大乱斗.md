### `quartz` 的 `cron` 与 `Spring` 的 `cron` 与 `linux` 的 `cron`

### 1. `quartz` 的 `cron`
### 2. `Spring` 的 `cron`
### 3. `linux` 的 `cron`
#### 3.1 `crond`
crond 是 linux 系统中用来定期执行命令/脚本或指定程序任务的一种服务或软件，一般情况下，我们安装完 Centos5/6 linux 操作系统之后，默认便会启动 crond 任务调度服务。

crond 服务会定期（默认每分钟检查一次）检查系统中是否有要执行的任务工作，如果有，便会根据其预先设定的定时任务规则自动执行该定时任务工作，这个 crond 定时任务服务就相当于我们平时早起使用的闹钟一样。

需要将 crond 设置为系统启动后自动启动的服务，可以在 /etc/rc.d/rc.local 中，在末尾加上 service crond start。

crond 是Linux的内置服务，但它不自动起来，可以用以下的方法启动、关闭这个服务：
```
service crond status //系统是否启用了crond服务

/sbin/service crond start //启动服务

/sbin/service crond stop //关闭服务

/sbin/service crond restart //重启服务

/sbin/service crond reload //重新载入配置
```

#### 3.2 `crontab`
crontab 是一个很方便的在 unix/linux 系统上定时(循环)执行某个任务的程序使用 cron服务，用 service crond status 查看 cron 服务状态，如果没有启动则 service crond start 启动它，cron 服务是一个定时执行的服务，可以通过 crontab 命令添加或者编辑需要定时执行的任务：

```
usage:	crontab [-u user] file
	crontab [-u user] [ -e | -l | -r ]
		(default operation is replace, per 1003.2)
	-e	(edit user's crontab)
	-l	(list user's crontab)
	-r	(delete user's crontab)
	-i	(prompt before deleting user's crontab)
	-s	(selinux context)

```

crontab file的格式:
```
MIN HOUR DAY MONTH DAYOFWEEK COMMAND

crontab 文件中的行由 6 个字段组成，不同字段间用空格或 tab 键分隔。前 5 个字段指定命令要运行的时间
       分钟 (0-59)
       小时 (0-23)
       日期 (1-31)
       月份 (1-12)
       星期几（0-6，其中 0 代表星期日）
第 6 个字段是一个要在适当时间执行的字符串

在以上各个字段中，还可以使用以下特殊字符：
（1）星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
（2）逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
（3）中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
（4）正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

```


#### 3.3 crontab定时任务不执行的原因
+ （1）`crond 服务未启动`

crontab 不是 Linux 内核的功能，而是依赖一个 crond 服务，这个服务可以启动当然也可以停止。如果停止了就无法执行任何定时任务了，解决的方法是打开它。

+ （2）`权限问题`：比如：脚本没有 x 执行权限，也有可能 crontab 任务所属的用户对某个目录没有写权限，也会失败。

+ （3）`路径问题`：有的命令在 shell 中执行正常，但是在 crontab 执行却总是失败。有可能是因为crontab 使用的 sh 未正确识别路径。

+ （4）`时差问题`：因为服务器与客户端时差问题，所以 crontab 的时间以服务器时间为准。

+ （5）`变量问题`：有时候命令中含有变量，但 crontab 执行时却没有，也会造成执行失败。

问题排查：
```
先手动执行定时任务以此来判断脚本是否有问题

service crond status // 系统是否启用了crond 服务

ps -ef |grep crond   // crond 进程是否存在

tail -f /var/log/cron // 查看执行日志。如果cron任务不小心删除了，可以通过这里恢复
```

### 扩展阅读
#### 1. quartz-misfire 错失、补偿执行
https://www.cnblogs.com/skyLogin/p/6927629.html

#### 2. crontab定时任务不执行的原因
https://blog.csdn.net/u011734144/article/details/54576469

#### 3. Linux Crontab 设置的定时任务没有启动的排查
https://blog.csdn.net/u013850277/article/details/54344805

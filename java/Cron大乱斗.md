### `quartz` 的 `cron` 与 `Spring` 的 `cron` 与 `linux` 的 `cron`

一句话总结：

就 cron 表达式语法来说，Spring 的 cron 与 Quartz 的 cron 是一样的， 而 Linux 的 cron 没有“秒”级别的控制，其他与 Spring（Quartz）类似。

至于 cron 表达式的实现，Quartz 比 Spring 的功能要强。 Spring 的 cron 的底层依赖于 ScheduledThreadPoolExecutor，基于 Future 来做周期控制。

感觉Linux 的 cron “最二”，没有并行控制：到点就起，无论上一次是否执行完。

### 1. `quartz` 的 `cron`
misfire产生的条件是：到了该触发执行时上一个执行还未完成，且线程池中没有空闲线程可以使用（或有空闲线程可以使用但job设置为@DisallowConcurrentExecution）且过期时间已经超过misfireThreshold就认为是misfire了，错失触发了

+ withMisfireHandlingInstructionDoNothing
```
——不触发立即执行
——等待下次Cron触发频率到达时刻开始按照Cron频率依次执行
```

+ withMisfireHandlingInstructionIgnoreMisfires
```
——以错过的第一个频率时间立刻开始执行
——重做错过的所有频率周期后
——当下一次触发频率发生时间大于当前时间后，再按照正常的Cron频率依次执行
```

+ withMisfireHandlingInstructionFireAndProceed（默认）
```
——以当前时间为触发频率立刻触发一次执行
——然后按照Cron频率依次执行
```

### 2. `Spring` 的 `cron`

Spring 融合 quartz 框架，Spring 的 cron 表达式语法与 Quartz 的一样。

cron一共有7位，但是最后一位是年，可以留空，所以我们可以写6位，有两种形式：
```
Seconds Minutes Hours DayofMonth Month DayofWeek Year
或
Seconds Minutes Hours DayofMonth Month DayofWeek
```
* 第一位，表示秒，取值0-59
* 第二位，表示分，取值0-59
* 第三位，表示小时，取值0-23
* 第四位，日期天/日，取值1-31
* 第五位，日期月份，取值1-12
* 第六位，星期，取值1-7，星期一，星期二...，注：不是第1周，第二周的意思。另外：1表示星期天，2表示星期一。
* 第7为，年份，可以留空，取值1970-2099

cron中，还有一些特殊的符号，含义如下：
```
(*)星号：可以理解为每的意思，每秒，每分，每天，每月，每年...
(?)问号：问号只能出现在日期和星期这两个位置，表示这个位置的值不确定，每天3点执行，所以第六位星期的位置，我们是不需要关注的，就是不确定的值。同时：日期和星期是两个相互排斥的元素，通过问号来表明不指定值。比如，1月10日，比如是星期1，如果在星期的位置是另指定星期二，就前后冲突矛盾了。
(-)减号：表达一个范围，如在小时字段中使用“10-12”，则表示从10到12点，即10,11,12
(,)逗号：表达一个列表值，如在星期字段中使用“1,2,4”，则表示星期一，星期二，星期四
(/)斜杠：如：x/y，x是开始值，y是步长，比如在第一位（秒） 0/15就是，从0秒开始，每15秒，最后就是0，15，30，45，60    另：*/y，等同于0/y
```

下面列举几个例子：
```
0 0 3 * * ?     每天3点执行
0 15 3 * * ?    每天3点15分执行
0 5 3 ? * *     每天3点5分执行，与上面作用相同
0 5/10 3 * * ?  每天3点的 5分，15分，25分，35分，45分，55分这几个时间点执行
0 10 3 ? * 1    每周星期天，3点10分 执行，注：1表示星期天
0 10 3 ? * 1#3  每个月的第三个星期，星期天 执行，#号只能出现在星期的位置
```

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

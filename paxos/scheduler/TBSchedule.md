## 一 分布式开源调度框架TBSchedule原理与应用
@see：https://blog.csdn.net/taosir_zhang/article/details/50728362
### 第一部分 TBSchedule基本概念及原理
#### 1. 概念介绍
TBSchedule是一个支持分布式的调度框架，能让一种批量任务或者不断变化的任务，被动态的分配到多个主机的JVM中，不同的线程组中并行执行。基于ZooKeeper的纯Java实现，由Alibaba开源。

#### 2. 工作原理
TBSchesule对分布式的支持包括调度机的分布式和执行机的分布式，其网络部署架构图如下：
![](../pic/TBSchedule.png)

#####　2.1 数据存储
执行机和调度机均以ZooKeeper为注册中心，所有数据以节点及节点内容的形式注册，通过定时汇报主机状态保持存活在ZooKeeper上。

1）执行机部署启动，会在ZooKeeper上创建永久根节点schedule.zookeeper.address，其后所有的操作均在该根节点下进行。

查看执行机注册后情况：可以看到根节点下面有3个永久子节点，
strategy存储调度机创建的策略信息，baseTaskType存储调度机创建的任务信息，factory存储执行机注册的主机信息。每台执行机启动后，都会在factory下创建一个临时顺序子节点，该节点名是由TBSchedule源码生成的主机唯一表示。

2）调度机部署启动，这时不会对ZooKeeper节点做任何操作。打开调度机配置面板：

配置好ZooKeeper接入点，点击管理主页，进入调度任务管理面板：

输入各项参数创建新任务后，此时会在baseTaskType下面创建任务名称永久子节点（调度机所有都宕机重启后，仍能保持数据的完整性），而当前节点的内容就是配置的各项参数。

3）创建调度策略，控制调度机调度状态。
创建完成调度策略后开启调度，此过程会在对应的任务节点strategy下创建永久子节点并写入策略数据，在该子节点下创建表示调度机的临时顺序子节点并写入调度策略数据。

同时会在baseTaskType/IScheduleTaskDealSingleTest下创建下创建两层永久子节点并注册调度主机数据。

##### 2.2 分布式高可用高效率保障
1）调度机的高可用有保障，多调度机向注册中心注册后，共享调度任务，且同一调度任务仅由一台调度机执行调度，当前调度机异常宕机后，其余的调度机会接上。

2）执行机的高可用有保障，多执行机向注册中心注册后，配置执行机单线程（多机总线程为1）执行任务，调度机会随机启动一台执行机执行，当前执行异常机宕机后，调度机会会新调度一台执行机。

3）执行机的并行高效保障，配置执行机多线程且划分多任务子项后，各任务子项均衡分配到所有执行机，各执行机均执行，多线程数据一致性协调由任务项参数区分。

4）弹性扩展失效转移保障，运行中的执行机宕机，或新增执行机，调度机将在下次任务执行前重新分配任务项，不影响正常执行机任务（崩溃的执行机当前任务处理失效）；运行中的调度机宕机或动态新增调度机，不影响执行机当前任务，调度机宕机后动态切换。

#### 3. 源码分析
#####　3.1 执行机注册节选
从Spring配置文件可以看到，执行机注册的入口在TBScheduleManagerFactory的init方法

init方法将配置参数封装到Properties对象后开始初始化，连接上ZooKeeper并启动一个新的线程进行节点数据处理

跟踪代码可以看到新线程调用的实际处理方法是：

上述几个节点创建完成，并向ZooKeeper注册监听，当有数据变化时获得通知（任务执行/暂停）。到这里，就完成了执行机到ZooKeeper的注册监听过程。


#### 3.2 调度任务创建节选
任务创建提交保存为入口，将参数封装到ScheduleTaskType对象中，调用节点创建和更新方法：

如果是更新的话，就不会再创建任务永久节点了，直接修改任务节点内容即可。

#### 3.3 策略创建节选
策略创建提交保存为入口，将参数封装到ScheduleStrategy对象中，调用节点创建和更新方法：

如果是更新的话，就不会再创建任务永久节点了，直接修改任务节点内容即可。

#### 3.4 调度控制节选
修改节点数据，通过ZooKeeper的事件通知机制，让执行机获得变更通知。

#### 4. 与其他开源调度框架对比
1）Quartz：Java事实上的定时任务标准。但Quartz关注点在于定时任务而非数据，并无一套根据数据处理而定制化的流程。虽然Quartz可以基于数据库实现作业的高可用，缺少分布式并行执行作业的功能。

2）Crontab：Linux系统级的定时任务执行器，缺乏分布式和集中管理功能。

3）elastic-job：当当网最近开源项目，功能跟TBSchedule几乎一样（批斗TBSchedule文档缺失严重），一台服务器只能开启一个任务实例，基于Ip不基于IpPort，单机难调试集群功能。

4）TBSchedule：淘宝早期开源，稳定性可以保证。


### 第二部分 TBSchedule分布式调度示例

## 淘宝开源项目TbSchedule的使用

@see https://blog.csdn.net/yuchao2015/article/details/53033628

#### 1. TBSchedule源码下载
#### 2. 引入源码Demo开发示例
#### 3. 控制台配置任务调度
#### 4. selectTasks方法参数说明

taskParameter：对应控制台自定义参数，可自定义传入做逻辑上的操作

taskQueueNum：对应控制台任务项数量

taskItemList：集合中TaskItemDefine的id值对应任务项值，多线程处理时，根据任务项协调数据一致性和完整性

eachFetchDataNum：对应控制台每次获取数量，由于子计时单元开始后，会不断的去取数据进行处理，直到取不到数据子计时才停止，等待下一个子计时开始。可以限制每次取数，防止一次性数据记录过大，内存不足。

ownSign：环境参数，一般没什么用

#### 5. 创建调度策略参数说明

策略名称：策略标示，可任意填写

任务类型：一般保持默认Schedule

任务名称：对应任务栏被调度任务名称

任务参数：一般不用，保持默认

单JVM最大线程组数量：单个JVM允许开启的线程数

最大线程组数量：多处理机情况下的线程总数限制（总线程为2，任务项线程为4是没有意义的）

IP地址：127.0.0.1或者localhost会在所有机器上运行，注意多处理机若没有根据任务子项划分数据处理，会导致多处理机重复处理数据，谨慎配置


#### 6. 创建任务参数说明

任务名称：策略调度的标示，一旦创建保存，不可更改

任务处理的SpringBean：注册到spring的任务bean，如iScheduleTaskDealSingleTest

心跳频率/假定服务死亡时间/处理模式/没有数据时休眠时长/执行结束时间：一般保持默认即可

线程数：处理该任务的线程数，在没有划分多任务项的情况下，多线程是没有意义的，且线程数量大于任务项也是没有意义的（线程数小于等于任务项），注意如果开启多线程，必须对数据做任务项过滤

单线程组最大任务项：配置单JVM处理的最大任务项数量，多任务项情况下，可按需限制，一般默认，多执行机会均衡分配

每次获取数量：子计时单元开始，线程会不断的去获取数据（每次获取的限制）并处理数据，直到获取不到数据子计时才结束（方法内不用就可以随意配置）

每次执行数量：//还没测试过（可能是将获取的数量拆分多次执行）

每次处理完休眠时间：子计时单元开始，只要有数据，就会不停的获取不停的处理，这个时间设置后，子计时单元开始每次获取执行后，不管还有没有待数据，都先歇会儿再获取处理

自定义参数：可自定义控制任务逻辑操作

任务项：这项很重要，在多线程情况下，划分任务项是有意义的，但是要注意必须通过任务项参数，协调待处理数据，否则多线程会重复处理


## 二 官方文档
@see http://code.taobao.org/p/tbschedule/wiki/index/


### 1、调度器的设计目标
1、tbschedule的目的是让一种批量任务或者不断变化的任务，能够被动态的分配到多个主机的JVM中，不同的线程组中并行执行。所有的任务能够被不重复，不遗漏的快速处理。

2、调度的Manager可以动态的随意增加和停止

3、可以通过JMX控制调度服务的创建和停止

4、可以指定调度的时间区间：

PERMIT_RUN_START_TIME ：允许执行时段的开始时间crontab的时间格式.以startrun:开始，则表示开机立即启动调度

PERMIT_RUN_END_TIME ：允许执行时段的结束时间crontab的时间格式,如果不设置，表示取不到数据就停止

PERMIT_RUN_START_TIME ='0 * * * * ?' 表示在每分钟的0秒开始

PERMIT_RUN_END_TIME ='20 * * * * ?' 表示在每分钟的20秒终止

就是每分钟的0-20秒执行，其它时间休眠
格式信息请参照： http://dogstar.javaeye.com/blog/116130

### 2、主要概念
#### TaskType任务类型:
是任务调度分配处理的单位，例如：
1、将一张表中的所有状态为STS=’N’的所有数据提取出来发送给其它系统，同时将修改状态STS=’Y’,就是一种任务。TaskType=’DataDeal’

2、将一个目录以所有子目录下的所有文件读取出来入库，同时把文件移到对应的备份目录中，也是一种任务。TaskType=’FileDeal’。

3、可以为一个任务类型自定义一个字符串参数由应用自己解析。例如:"AREA=杭州,YEAR>30"

#### ScheduleServer任务处理器

1、是由一组线程【1..n个线程】构成的任务处理单元，每一任务处理器有一个唯一的全局标识，
   一般以IP$UUID[例如192.168.1.100$0C78F0C0FA084E54B6665F4D00FA73DC]的形式出现。 一个任务类型的数据可以由1..n个任务处理器同时进行。

2、这些任务处理器可以在同一个JVM中，也可以分布在不同主机的JVM中。任务处理器内部有一个心跳线程，用于确定Server的状态和任务的动态分配，
   有一组工作线程，负责处理查询任务和具体的任务处理工作。

3、目前版本所有的心跳信息都是存放在Zookeeper服务器中的，所有的Server都是对等的，当一个Server死亡后，其它Server会接管起拥有的任务队列，
   期间会有几个心跳周期的时延。后续可以用类似ConfigerServer类的存储。

4、现有的工作线程模式分为Sleep模式和NotSleep模式。缺省是缺省是NOTSLEEP模式。在通常模式下，在通常情况下用Sleep模式。
   在一些特殊情况需要用NotSleep模式。两者之间的差异在后续进行描述。

#### TaskItem任务项
是对任务进行的分片划分。例如：

1、将一个数据表中所有数据的ID按10取模，就将数据划分成了0、1、2、3、4、5、6、7、8、9供10个任务项。

2、将一个目录下的所有文件按文件名称的首字母(不区分大小写)，就划分成了A、B、C、D、E、F、G、H、I、J、K、L、M、N、O、P、Q、R、S、T、U、V、W、X、Y、Z供26个队列。

3、将一个数据表的数据ID哈希后按1000取模作为最后的HASHCODE,我们就可以将数据按[0,100)、[100,200) 、[200,300)、[300,400) 、[400,500)、[500,600)、[600,700)、[700,800)、[800,900)、 [900,1000)划分为十个任务项，当然你也可以划分为100个任务项，最多是1000个任务项。

任务项是进行任务分配的最小单位。一个任务项只能由一个ScheduleServer来进行处理。但一个Server可以处理任意数量的任务项。

例如任务被划分为了10个队列，可以只启动一个Server，所有的任务项都有这一个Server来处理；也可以启动两个Server，每个Sever处理5个任务项；但最多只能启动10个Server，每一个ScheduleServer只处理一个任务项。如果在多，则第11个及之后的Server将不起作用，处于休眠状态。

4、可以为一个任务项自定义一个字符串参数由应用自己解析。例如:"TYPE=A,KIND=1"

#### TaskDealBean任务处理类
是业务系统进行数据处理的实现类。要求实现Schedule的接口IScheduleTaskDealMulti或者IScheduleTaskDealSingle。

接口主要包括两个方法。一个是根据调度器分配到的队列查询数据的接口，一个是进行数据处理的接口。

#### 运行时间
 1、可以指定任务处理的时间间隔，例如每天的1：00－3：00执行，或者每个月的第一天执行、每一个小时的第一分钟执行等等。间格式与crontab相同。如果不指定就表示一直运行。PERMIT_RUN_START_TIME,PERMIT_RUN_END_TIME

 2、可以指定如果没有数据了，休眠的时间间隔。SLEEP_TIME_NODATA 单位秒

 3、可以指定每处理完一批数据后休眠的时间间隔.SLEEP_TIME_INTERVAL 单位                                 

#### OwnSign环境区域
是对运行环境的划分，进行调度任务和数据隔离。例如：开发环境、测试环境、预发环境、生产环境。

不同的开发人员需要进行数据隔离也可以用OwnSign来实现，避免不同人员的数据冲突。缺省配置的环境区域OwnSign='BASE'。

例如：TaskType='DataDeal',配置的队列是0、1、2、3、4、5、6、7、8、9。缺省的OwnSign='BASE'。

此时如果再启动一个测试环境，则Schedule会动态生成一个TaskType='DataDeal-Test'的任务类型，环境会作为一个变量传递给业务接口，

由业务接口的实现类，在读取数据和处理数据的时候进行确定。业务系统一种典型的做法就是在数据表中增加一个OWN_SIGN字段。

在创建数据的时候根据运行环境填入对应的环境名称，在Schedule中就可以环境的区分了。

#### 调度策略
是指某一个任务在调度集群上的分布策略，可以制定：
1、可以指定任务的机器IP列表。127.0.0.1和localhost表示所有机器上都可以执行

2、可以指定每个机器上能启动的线程组数量，0表示没有限制

3、可以指定所有机器上运行的线程组总数。

### 3、业务接口说明
包含三个业务接口，：
1、IScheduleTaskDeal 调度器对外的基础接口，是一个基类，并不能被直接使用

2、IScheduleTaskDealSingle 单任务处理的接口,继承 IScheduleTaskDeal

3、IScheduleTaskDealMulti 可批处理的任务接口,继承 IScheduleTaskDeal

#### IScheduleTaskDeal 调度器对外的基础接口
```java
public interface IScheduleTaskDeal<T> {
/**
 * 根据条件，查询当前调度服务器可处理的任务
 * @param taskParameter 任务的自定义参数
 * @param ownSign 当前环境名称
 * @param taskQueueNum 当前任务类型的任务队列数量
 * @param taskQueueList 当前调度服务器，分配到的可处理队列
 * @param eachFetchDataNum 每次获取数据的数量
 * @return
 * @throws Exception
 */
public List<T> selectTasks(String taskParameter,String ownSign,int taskQueueNum,List<TaskItemDefine> taskItemList,int eachFetchDataNum) throws Exception;

/**
 * 获取任务的比较器,只有在NotSleep模式下需要用到
 * @return
 */
public Comparator<T> getComparator();
}                                    
```
#### IScheduleTaskDealSingle 单任务处理的接口
```java
public interface IScheduleTaskDealSingle<T> extends IScheduleTaskDeal<T> {
  /**
   * 执行单个任务
   * @param task Object
   * @param ownSign 当前环境名称
   * @throws Exception
   */
  public boolean execute(T task,String ownSign) throws Exception;

}     
```

#### IScheduleTaskDealMulti 可批处理的任务接口
```java
public interface IScheduleTaskDealMulti<T>  extends IScheduleTaskDeal<T> {

/**
 * 	执行给定的任务数组。因为泛型不支持new 数组，只能传递OBJECT[]
 * @param tasks 任务数组
 * @param ownSign 当前环境名称
 * @return
 * @throws Exception
 */
  public boolean execute(Object[] tasks,String ownSign) throws Exception;
}   
```

### 4、Sleep模式和NotSleep模式的区别
1、ScheduleServer启动的工作线程组线程是共享一个任务池的。

2、在Sleep的工作模式：当某一个线程任务处理完毕，从任务池中取不到任务的时候，检查其它线程是否处于活动状态。如果是，则自己休眠；如果其它线程都已经因为没有任务进入休眠，当前线程是最后一个活动线程的时候，就调用业务接口，获取需要处理的任务，放入任务池中，同时唤醒其它休眠线程开始工作。

3、在NotSleep的工作模式：当一个线程任务处理完毕，从任务池中取不到任务的时候，立即调用业务接口获取需要处理的任务，放入任务池中。

4、Sleep模式在实现逻辑上相对简单清晰，但存在一个大任务处理时间长，导致其它线程不工作的情况。

5、在NotSleep模式下，减少了线程休眠的时间，避免大任务阻塞的情况，但为了避免数据被重复处理，增加了CPU在数据比较上的开销。同时要求业务接口实现对象的比较接口。

6、如果对任务处理不允许停顿的情况下建议用NotSleep模式，其它情况建议用sleep模式。

## 三 tbschedule与spring整合
@see https://blog.csdn.net/chinabestchina/article/details/76269642

### 一、tbschedule核心知识点

1、tbschedule是淘宝开源的，能够让批量任务或变化的任务，被动态的分配到不同主机（可分布式）的jvm，不同的线程组中并行执行。

所有任务能够不重复，不遗漏的执行。

2、tbschedule的任务、策略等调度数据是存储在zookeeper中的。

3、tbschedule的执行是基于jdk的Timer和TimerTask实现的。

4、tbschedule中的任务，是依附于策略而运行的。也就是说，任务定义了要执行的行为，包括任务名称、取数据和数据处理的bean、每次取数的数量、执行的开始与结束时间、任务项等信息，而策略定义了要执行的任务、在哪台机器上执行、所有机器最大线程组，单个机器线程组数等，并控制任务的执行与停止。

### 二、重要概念或类介绍

#### 1、ZKManager

ZKManager就是最基本的zookeeper会话管理类，内容包括zookeeper的创建、会话的连接或重连接、关闭会话等。

#### 2、TBScheduleManagerFactory

TBScheduleManagerFactory是tbschedule管理类，包含的功能有：

a)配置zookeeper，并创建zookeeper会话

zookeeper的配置信息有：zkConnectString, rootPath, userName, password, zkSessionTimeout, isCheckParentPath

b)调度任务和调度策略的管理器生成

ScheduleDataManager4ZK，调度任务管理器（对应在zookeeper中的数据），在此进行初始化和生成。

ScheduleStrategyDataManager4ZK，调度策略管理器（对应在zookeeper中的数据），在此进行初始化和生成。

c)调度服务的重启、停止等

如stopServer(String strategyName)、stopAll()、reStart()等。

#### 3、ScheduleServer

任务处理器（可以理解为线程组），由一组线程（n个线程）组成，每个任务处理器有全局唯一的标识符，

一般以IP$UUID[例如192.168.1.100$0C78F0C0FA084E54B6665F4D00FA73DC]的形式出现，

一个任务类型的数据可以n个任务处理器处理。内分为Sleep模式和NotSleep模式：

a）sleep模式，当一线程处理完任务，同时从任务池取不到任务时，若其它线程仍工作，则自己休眠，

若其它线程已休眠，则新调取需要处理的数据，同时唤醒其它休眠线程处理数据;

b)NotSleep模式，当一线程处理完任务，同时从任务池取不到任务时，则新调取需要处理的数据，

同时唤醒其它休眠线程处理数据;

#### 4、TaskItem

任务项，也就是将待处理的任务（数据），进行分片划分，

如：可以按数据的id按10取模，这样就将数年数据划分成了0、1、2、3、4、5、6、7、8、9共10个任务项;

也可按数据的首字母分成了A、B、C、D、E、F、G、H、I、J、K、L、M、N、O、P、Q、R、S、T、U、V、W、X、Y、Z供26个任务项。

这个可以根据需要自行定义的。

#### 5、TaskDealBean

自定义的任务处理类，需要实现Schedule的接口IScheduleTaskDealMulti（批处理）或者IScheduleTaskDealSingle（单任务处理），

内部主要有两个方法，一个是筛选需当前任务处理器处理的数据，另一个是处理已筛选好的数据。

#### 6、OwnSign

环境，指定运行环境，如：开发环境、测试环境、预发环境、生产环境。在筛选当前任务处理器需处理的数据时，会传入该参数。

#### 7、ScheduleTaskType

任务的配置类，包括运行的线程数（threadNumber）、运行时间，任务项分组、没数据时的休眠时间、每次取数的量等

#### 8、ScheduleStrategy

策略的配置类，所有机器的最大线程组数（assignNum），单个jvm的线程数（numOfSingleServer），运行机器（IPList）等信息。

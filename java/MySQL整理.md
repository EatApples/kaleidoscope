### MySQL 是怎样运行的：从根儿上理解 MySQL
```
来源：掘金小册《MySQL 是怎样运行的：从根儿上理解 MySQL》
作者：小孩子4919，公众号「我们都是小青蛙」作者
```
### 1. 客户端连接
```
mysql -h127.0.0.1 -uroot -P3307 -p
```

### 2. MySQL存储引擎

| 存储引擎  | 描述                                 |
| --------- | ------------------------------------ |
| ARCHIVE   | 用于数据存档（行被插入后不能再修改） |
| BLACKHOLE | 丢弃写操作，读操作会返回空内容       |
| CSV       | 在存储数据时，以逗号分隔各个数据项   |
| FEDERATED | 用来访问远程表                       |
| InnoDB    | 具备外键支持功能的事务存储引擎       |
| MEMORY    | 置于内存的表                         |
| MERGE     | 用来管理多个MyISAM表构成的表集合     |
| MyISAM    | 主要的非事务处理存储引擎             |
| NDB       | MySQL集群专用存储引擎                |

```
// 查看当前服务器程序支持的存储引擎
SHOW ENGINES;

// 创建表时指定存储引擎
CREATE TABLE 表名(
    建表语句;
) ENGINE = 存储引擎名称;

// 修改表的存储引擎
ALTER TABLE 表名 ENGINE = 存储引擎名称;
```

### 3. 存储引擎的各种功能

| Feature                               | MyISAM | Memory  | InnoDB | Archive | NDB   |
| ------------------------------------- | ------ | ------- | ------ | ------- | ----- |
| B-tree indexes                        | yes    | yes     | yes    | no      | no    |
| Backup/point-in-time recovery         | yes    | yes     | yes    | yes     | yes   |
| Cluster database support              | no     | no      | no     | no      | yes   |
| Clustered indexes                     | no     | no      | yes    | no      | no    |
| Compressed data                       | yes    | no      | yes    | yes     | no    |
| Data caches                           | no     | N/A     | yes    | no      | yes   |
| Encrypted data                        | yes    | yes     | yes    | yes     | yes   |
| Foreign key support                   | no     | no      | yes    | no      | yes   |
| Full-text search indexes              | yes    | no      | yes    | no      | no    |
| Geospatial data type support          | yes    | no      | yes    | yes     | yes   |
| Geospatial indexing support           | yes    | no      | yes    | no      | no    |
| Hash indexes                          | no     | yes     | no     | no      | yes   |
| Index caches                          | yes    | N/A     | yes    | no      | yes   |
| Locking granularity                   | Table  | Table   | Row    | Row     | Row   |
| MVCC                                  | no     | no      | yes    | no      | no    |
| Query cache support                   | yes    | yes     | yes    | yes     | yes   |
| Replication support                   | yes    | Limited | yes    | yes     | yes   |
| Storage limits                        | 256TB  | RAM     | 64TB   | None    | 384EB |
| T-tree indexes                        | no     | no      | no     | no      | yes   |
| Transactions                          | no     | no      | yes    | no      | yes   |
| Update statistics for data dictionary | yes    | yes     | yes    | yes     | yes   |

### 4. 配置文件的优先级
MySQL将按照给定的顺序依次读取各个配置文件，如果该文件不存在则忽略。

值得注意的是，如果我们在多个配置文件中设置了相同的启动选项，那以最后一个配置文件中的为准。

如果在同一个配置文件中，在这些组里出现了同样的配置项，那么，将以最后一个出现的组中的启动选项为准。

如果同一个启动选项既出现在命令行中，又出现在配置文件中，那么以命令行中的启动选项为准！

配置是覆盖型的。

### 5. 系统变量
系统变量比较牛逼的一点就是，对于大部分系统变量来说，它们的值可以在服务器程序运行过程中进行动态修改而无需停止并重启服务器。不过系统变量有作用范围之分，具体来说作用范围分为这两种：

GLOBAL：全局变量，影响服务器的整体操作。

SESSION：会话变量，影响某个客户端连接的操作。（注：SESSION有个别名叫LOCAL）

通过启动选项设置的系统变量的作用范围都是GLOBAL的，也就是对所有客户端都有效的。

在服务器程序运行期间通过客户端程序设置系统变量的语法：
```
SET [GLOBAL|SESSION] 系统变量名 = 值;
```
或者写成这样也行：
```
SET [@@(GLOBAL|SESSION).]var_name = XXX;
```

如果在设置系统变量的语句中省略了作用范围，默认的作用范围就是SESSION。也就是说SET 系统变量名 = 值和SET SESSION 系统变量名 = 值是等价的。

### 6. 查看系统变量
我们可以使用下列命令查看MySQL服务器程序支持的系统变量以及它们的当前值：

```
SHOW VARIABLES [LIKE 匹配的模式];
```
由于系统变量实在太多了，如果我们直接使用SHOW VARIABLES查看的话就直接刷屏了，所以通常都会带一个LIKE过滤条件来查看我们需要的系统变量的值

既然系统变量有作用范围之分，那我们的SHOW VARIABLES语句查看的是什么作用范围的系统变量呢？

答：默认查看的是SESSION作用范围的系统变量。

当然我们也可以在查看系统变量的语句上加上要查看哪个作用范围的系统变量，就像这样：

```
SHOW [GLOBAL|SESSION] VARIABLES [LIKE 匹配的模式];
```

### 7. 状态变量
为了让我们更好的了解服务器程序的运行情况，MySQL服务器程序中维护了好多关于程序运行状态的变量，它们被称为状态变量。比方说Threads_connected表示当前有多少客户端与服务器建立了连接。

由于状态变量是用来显示服务器程序运行状况的，所以它们的值只能由服务器程序自己来设置，我们程序员是不能设置的。

与系统变量类似，状态变量也有GLOBAL和SESSION两个作用范围的，所以查看状态变量的语句可以这么写：

```
SHOW [GLOBAL|SESSION] STATUS [LIKE 匹配的模式];
```

### 8. 一些重要的字符集

| 字符集     | 字符个数           | 字节数 | 备注                   |
| ---------- | ------------------ | ------ | ---------------------- |
| ASCII      | 128                | 1      |                        |
| ISO 8859-1 | 256                | 1      | 别名latin1，兼容ASCII  |
| GB2312     | 7445=6763+682      | 1或2   | 兼容ASCII时，使用1字节 |
| GBK        | GB2312扩充         | 1或2   | 兼容GB2312             |
| utf8       | 所有字符           | 1～4   | 变长编码               |
| utf16      | 所有字符           | 2或4   | 变长编码               |
| utf32      | 所有字符           | 4      |                        |
| utf8mb3    | 阉割过的utf8字符集 | 1～3   | MySQL中字符集          |
| utf8mb4    | 正宗的utf8字符集   | 1～4   | MySQL中字符集          |


有一点需要大家十分的注意，在MySQL中utf8是utf8mb3的别名，所以之后在MySQL中提到utf8就意味着使用1~3个字节来表示一个字符，如果大家有使用4字节编码一个字符的情况，比如存储一些emoji表情啥的，那请使用utf8mb4。

MySQL支持好多好多种字符集，查看当前MySQL中支持的字符集可以用下边这个语句：
```
SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
```

### MySQL中字符集的转换

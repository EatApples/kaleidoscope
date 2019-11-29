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
| **Data caches**                       | no     | N/A     | yes    | no      | yes   |
| Encrypted data                        | yes    | yes     | yes    | yes     | yes   |
| **Foreign key support**               | no     | no      | yes    | no      | yes   |
| Full-text search indexes              | yes    | no      | yes    | no      | no    |
| Geospatial data type support          | yes    | no      | yes    | yes     | yes   |
| Geospatial indexing support           | yes    | no      | yes    | no      | no    |
| **Hash indexes**                      | no     | yes     | no     | no      | yes   |
| Index caches                          | yes    | N/A     | yes    | no      | yes   |
| **Locking granularity**               | Table  | Table   | Row    | Row     | Row   |
| **MVCC**                              | no     | no      | yes    | no      | no    |
| Query cache support                   | yes    | yes     | yes    | yes     | yes   |
| Replication support                   | yes    | Limited | yes    | yes     | yes   |
| Storage limits                        | 256TB  | RAM     | 64TB   | None    | 384EB |
| T-tree indexes                        | no     | no      | no     | no      | yes   |
| **Transactions**                      | no     | no      | yes    | no      | yes   |
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
由于系统变量实在太多了，如果我们直接使用SHOW VARIABLES查看的话就直接刷屏了，所以通常都会带一个LIKE过滤条件来查看我们需要的系统变量的值。

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
字符集指的是某个字符范围的编码规则。

比较规则是针对某个字符集中的字符比较大小的一种规则。

在MySQL中，一个字符集可以有若干种比较规则，其中有一个默认的比较规则，一个比较规则必须对应一个字符集。

查看MySQL中查看支持的字符集和比较规则的语句如下：
```
SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
SHOW COLLATION [LIKE 匹配的模式];
```

MySQL有四个级别的字符集和比较规则:服务器级别，数据库级别，表级别，列级别

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

### 9. MySQL中字符集的转换

从发送请求到接收结果过程中发生的字符集转换：

（1）客户端使用操作系统的字符集编码请求字符串，向服务器发送的是经过编码的一个字节串。

（2）服务器将客户端发送来的字节串采用character_set_client代表的字符集进行解码，将解码后的字符串再按照character_set_connection代表的字符集进行编码。

（3）如果character_set_connection代表的字符集和具体操作的列使用的字符集一致，则直接进行相应操作，否则的话需要将请求中的字符串从character_set_connection代表的字符集转换为具体操作的列使用的字符集之后再进行操作。

（4）将从某个列获取到的字节串从该列使用的字符集转换为character_set_results代表的字符集后发送到客户端。

（5）客户端使用操作系统的字符集解析收到的结果集字节串。

character_set_client，服务器解码请求时使用的字符集。服务器认为客户端发送过来的请求是用character_set_client编码的。假设你的客户端采用的字符集和 character_set_client 不一样的话，这就会出现意想不到的情况。

character_set_connection，服务器处理请求时会把请求字符串从character_set_client转为character_set_connection。character_set_connection只是服务器在将请求的字节串从character_set_client转换为character_set_connection时使用，它是什么其实没多重要，但是一定要注意，该字符集包含的字符范围一定涵盖请求中的字符，要不然会导致有的字符无法使用character_set_connection代表的字符集进行编码。

character_set_results，服务器向客户端返回数据时使用的字符集。服务器将把得到的结果集使用character_set_results编码后发送给客户端。假设你的客户端采用的字符集和 character_set_results 不一样的话，这就可能会出现客户端无法解码结果集的情况。

我们通常都把 character_set_client 、character_set_connection、character_set_results 这三个系统变量设置成和客户端使用的字符集一致的情况，这样减少了很多无谓的字符集转换。为了方便我们设置，MySQL提供了一条非常简便的语句：
```
SET NAMES 字符集名;
```
这一条语句产生的效果和我们执行这3条的效果是一样的：
```
SET character_set_client = 字符集名;
SET character_set_connection = 字符集名;
SET character_set_results = 字符集名;
```

### 10. InnoDB页
当我们想从表中获取某些记录时，InnoDB存储引擎需要一条一条的把记录从磁盘上读出来么？

不，那样会慢死，InnoDB采取的方式是：将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，InnoDB中页的大小一般为 16 KB。也就是在一般情况下，一次最少从磁盘中读取16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中。

页是MySQL中磁盘和内存交互的基本单位，也是MySQL是管理存储空间的基本单位。

### 11. InnoDB行格式
我们平时是以记录为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为行格式或者记录格式。

InnoDB存储引擎有4种不同类型的行格式，分别是Compact、Redundant、Dynamic和Compressed行格式。

行溢出：一个页一般是16KB，当记录中的数据太多，当前页放不下的时候，会把多余的数据存储到其他页中，这种现象称为行溢出。

那发生行溢出的临界点是什么呢？也就是说在列存储多少字节的数据时就会发生行溢出？
MySQL中规定一个页中至少存放两行记录。你不用关注这个临界点是什么，只要知道如果我们一条记录的某个列中存储的数据占用的字节数非常多时，该列就可能成为溢出列。

（1）COMPACT行格式：一条完整的记录其实可以被分为记录的额外信息和记录的真实数据两大部分。
（1-1）记录的额外信息：这部分信息是服务器为了描述这条记录而不得不额外添加的一些信息，这些额外信息分为3类，分别是变长字段长度列表、NULL值列表和记录头信息。
（1-1-1）变长字段长度列表：在Compact行格式中，把所有变长字段的真实数据占用的字节长度都存放在记录的开头部位，从而形成一个变长字段长度列表，各变长字段数据占用的字节数按照列的顺序逆序存放，我们再次强调一遍，是逆序存放！变长字段长度列表中只存储值为 非NULL 的列内容占用的长度，值为 NULL 的列的长度是不储存的。
（1-1-2）NULL值列表：如果表中没有允许存储 NULL 的列，则 NULL值列表 也不存在了，否则将每个允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列，二进制位表示的意义如下：二进制位的值为1时，代表该列的值为NULL。二进制位的值为0时，代表该列的值不为NULL。
（1-1-3）记录头信息：除了变长字段长度列表、NULL值列表之外，还有一个用于描述记录的记录头信息，它是由固定的5个字节组成。5个字节也就是40个二进制位，不同的位代表不同的意思。
（1-2）记录的真实数据：记录的真实数据除了我们自己定义的列的数据以外，MySQL会为每个记录默认的添加一些列（也称为隐藏列），具体的列如下：

| 列名        | 是否必须 | 占用空间 | 描述                   |
| ----------- | -------- | -------- | ---------------------- |
| DB_ROW_ID   | 否       | 6字节    | 行ID，唯一标识一条记录 |
| DB_TRX_ID   | 是       | 6字节    | 事务ID                 |
| DB_ROLL_PTR | 是       | 7字节    | 回滚指针               |

注意：对于 CHAR(M) 类型的列来说，当列采用的是定长字符集时，该列占用的字节数不会被加到变长字段长度列表，而如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表。

（2）Redundant行格式：其实知道了Compact行格式之后，其他的行格式就是依葫芦画瓢了。我们现在要介绍的Redundant行格式是MySQL5.0之前用的一种行格式，也就是说它已经非常老了。下边我们从各个方面看一下Redundant行格式有什么不同的地方：
（2.1）字段长度偏移列表。没有了变长两个字，意味着Redundant行格式会把该条记录中所有列（包括隐藏列）的长度信息都按照逆序存储到字段长度偏移列表。多了个偏移两个字，这意味着计算列值长度的方式不像Compact行格式那么直观，它是采用两个相邻数值的差值来计算各个列值的长度。因为Redundant行格式并没有NULL值列表，所以在字段长度偏移列表中的各个列对应的偏移量处做了一些特殊处理 —— 将列对应的偏移量值的第一个比特位作为是否为NULL的依据，该比特位也可以被称之为NULL比特位。
（2.2）记录头信息。Redundant行格式的记录头信息占用6字节，48个二进制位。

对于Compact和Reduntant行格式来说，如果某一列中的数据非常多的话，在本记录的真实数据处只会存储该列的前768个字节的数据和一个指向其他页的地址，然后把剩下的数据存放到其他页中，这个过程也叫做行溢出，存储超出768字节的那些页面也被称为溢出页。

（3&4）Dynamic和Compressed行格式。这两种行格式类似于COMPACT行格式，只不过在处理行溢出数据时有点儿分歧，它们不会在记录的真实数据处存储字符串的前768个字节，而是把所有的字节都存储到其他页面中，只在记录的真实数据处存储其他页面的地址。另外，Compressed行格式会采用压缩算法对页面进行压缩。

指定行格式的语法
```
CREATE TABLE 表名 (列的信息) ROW_FORMAT=行格式名称  
ALTER TABLE 表名 ROW_FORMAT=行格式名称
```

在mysql中查询表的详细信息如下
```
use 库名
show table status like '表名';
```

#### 12. 
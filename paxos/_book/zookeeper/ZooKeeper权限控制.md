### Zookeeper ACL
ZooKeeper的权限管理亦即ACL控制功能通过Server、Client两端协调完成：

Server端：

一个ZooKeeper的节点（znode）存储两部分内容：数据和状态，状态中包含ACL信息。创建一个znode会产生一个ACL列表，列表中每个ACL包括：

+ 验证模式(scheme)

+ 具体内容(Id)（当scheme=“digest”时，Id为用户名密码，例如“root：J0sTy9BCUKubtK1y8pkbL7qoxSw=”）

+ 权限(perms)

#### 1.1 scheme
ZooKeeper提供了如下几种验证模式（scheme）：

+ digest：Client端由用户名和密码验证，譬如user:password，digest的密码生成方式是Sha1摘要的base64形式

+ auth：不使用任何id，代表任何已确认用户。

+ ip：Client端由IP地址验证，譬如172.2.0.0/24

+ world：固定用户为anyone，为所有Client端开放权限

+ super：在这种scheme情况下，对应的id拥有超级权限，可以做任何事情(cdrwa）

注意的是，exists操作和getAcl操作并不受ACL许可控制，因此任何客户端可以查询节点的状态和节点的ACL。

节点的权限（perms）主要有以下几种：

+ Create 允许对子节点Create操作

+ Read 允许对本节点GetChildren和GetData操作

+ Write 允许对本节点SetData操作

+ Delete 允许对子节点Delete操作

+ Admin 允许对本节点setAcl操作

Znode ACL权限用一个int型数字perms表示，perms的5个二进制位分别表示setacl、delete、create、write、read。比如0x1f=adcwr，0x1=----r，0x15=a-c-r

```java
const int ZOO_PERM_READ; //can read node’s value and list its children

const int ZOO_PERM_WRITE;// can set the node’s value

const int ZOO_PERM_CREATE; //can create children

const int ZOO_PERM_DELETE;// can delete children

const int ZOO_PERM_ADMIN; //can execute set_acl()

const int ZOO_PERM_ALL;// all of the above flags OR’d together
```
#### 1.2 ACL机制的缺陷
然而，ACL毕竟仅仅是访问控制，并非完善的权限管理，通过这种方式做多集群隔离，还有很多局限性：

ACL并无递归机制，任何一个znode创建后，都需要单独设置ACL，无法继承父节点的ACL设置。

除了ip这种scheme，digest和auth的使用对用户都不是透明的，这也给使用带来了很大的成本，很多依赖zookeeper的开源框架也没有加入对ACL的支持，例如hbase，storm。

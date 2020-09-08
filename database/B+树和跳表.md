### 问题：为什么使用 B+树，而不是跳表作为（数据库、文件系统等）索引？

或者说，为什么 MYSQL 使用 B+树作为 InnoDB 引擎的索引结构，而 REDIS 使用跳表作为 ZSET 的实现？

### 为什么 MySQL 使用 B+ 树

为何数据库索引使用 B+树而不是红黑树(或其他)？

要理解这个问题，我们先分析一下数据库的性质。

数据库的数据被分割为多个页以文件的形式储存在硬盘上的。因此我们每次进行数据库查询其实是在做磁盘 IO，而磁盘 IO 是时间开销较大的操作！数据库在进行索引查找的时候每次访问一个页都是一次磁盘 IO。因此我们需要选择一种能够尽量少做磁盘 IO 的数据结构来构建索引。

B+ 树之所以被选中主要是因为它的扇出率较大，树高较小。因而在进行索引搜索的时候需要进行的 IO 数量也较其他树的数量小。

B+ 树只有叶节点会存储数据，将树中的每一个叶节点通过指针连接起来就能实现顺序遍历，而遍历数据在关系型数据库中非常常见，所以这么选择是完全没有问题的。

基于上述原因，在应用场景需要选树的时候我们都会做如下的思考：基于内存操作的我们考虑红黑树，AVL 和 BST，基于磁盘操作的我们优先考虑 B 或 B+树。

MySQL 默认的存储引擎选择 B+ 树而不是 B 树的原因：

B 树能够在非叶节点中存储数据，但是这也导致在查询连续数据时可能会带来更多的随机 I/O，而 B+ 树的所有叶节点可以通过指针相互连接，能够减少顺序遍历时产生的额外随机 I/O。

### 为什么 Redis 使用跳表

跳表是一种采用了用空间换时间思想的数据结构。它会随机地将一些节点提升到更高的层次，以创建一种逐层的数据结构，以提高操作的速度。在理论上能够在 O(log(n))时间内完成查找、插入、删除操作。

跳表的性质

(1) 由很多层结构组成，level 是通过一定的概率随机产生的。

(2) 每一层都是一个有序的链表，默认是升序，也可以根据创建映射时所提供的 Comparator 进行排序，具体取决于使用的构造方法。

(3) 最底层(Level 1)的链表包含所有元素。

(4) 如果一个元素出现在 Level i 的链表中，则它在 Level i 之下的链表也都会出现。

(5) 每个节点包含两个指针，一个指向同一链表中的下一个元素，一个指向下面一层的元素

这是 Redis 作者 antirez 的理由：

```
There are a few reasons:

1) They are not very memory intensive. It's up to you basically. Changing parameters about the probability of a node to have a given number of levels will make then less memory intensive than btrees.

2) A sorted set is often target of many ZRANGE or ZREVRANGE operations, that is, traversing the skip list as a linked list. With this operation the cache locality of skip lists is at least as good as with other kind of balanced trees.

3) They are simpler to implement, debug, and so forth. For instance thanks to the skip list simplicity I received a patch (already in Redis master) with augmented skip lists implementing ZRANK in O(log(N)). It required little changes to the code.

```

（1）跳表比 B 树/B+树占用的内存更少

（2）以链表的形式遍历跳跃表，跳跃表的缓存局部性与其他类型的平衡树相当

（3）跳表更容易实现、调试等

简言之，跳表和时间复杂度几乎和红黑树一样，而且实现起来简单。

在 server 端，对并发和性能有要求的情况下，如何选择合适的数据结构（这里是跳跃表和红黑树）。

如果单纯比较性能，跳跃表和红黑树可以说相差不大，但是加上并发的环境就不一样了。

如果要更新数据，跳跃表需要更新的部分就比较少，锁的东西也就比较少，所以不同线程争锁的代价就相对少了。而红黑树有个平衡的过程，牵涉到大量的节点，争锁的代价也就相对较高了。性能也就不如前者了。

在并发环境下跳表有另外一个优势，红黑树在插入和删除的时候可能需要做一些平衡操作，这样的操作可能会涉及到整个树的其他部分，而跳表的操作显然更加局部性一些，锁需要盯住的节点更少，因此在这样的情况下性能好一些。

### B+树 VS 跳表

数据库常见的查询需求：
（1）等值
（2）范围
这 2 者都支持。B+树的结构和操作，跟跳表非常类似。理论上讲，对跳表稍加改造，也可以替代 B+树，作为数据库的索引实现的。

B+树发明于 1972 年，跳表发明于 1989 年，我们可以大胆猜想下，跳表的作者有可能就是受了 B+树的启发，才发明出跳表来的。不过，这个也无从考证了。

### 参考资料

#### 1. 为什么 MySQL 使用 B+ 树

https://draveness.me/whys-the-design-mysql-b-plus-tree/

#### 2. 为什么 MongoDB 使用 B 树

https://draveness.me/whys-the-design-mongodb-b-tree/

#### 3. Redis 的作者 antirez 回复为什么选用 skiplist

https://news.ycombinator.com/item?id=1171423

#### 4. AVL 树，红黑树，B 树，B+树，Trie 树都分别应用在哪些现实场景中？

作者：Kelvin
链接：https://www.zhihu.com/question/30527705/answer/754822718
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

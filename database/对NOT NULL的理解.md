### 对 NOT NULL 的理解
（1）数据库的底线是 NULL，这就是默认值

（2）用户设置的值高于默认值

（3）NOT NULL 的语义是，不能从SQL语句显示传 NULL 值，而不是指数据库强制该属性不能为NULL，原因见（1）

（4）可以通过属性判断来跳过 NOT NULL 约束，即插入一条记录，标记为 NOT NULL 的属性，由系统默认为 NULL

（5）为了符合预期，NOT NULL 属性应该添加默认值，即 DEFAULT XXX，原理见（2）

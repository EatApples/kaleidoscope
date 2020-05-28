### 对 NOT NULL 的理解
（1）数据库的底线是 NULL，NULL 是未初始化数据，一般通过 IS 来查询。不同的类型有不同的默认值（非 NULL）。

（2）用户设置的值高于默认值

（3）NOT NULL 的语义是，不能在数据库中设置为 NULL，而不是SQL中该属性不能为NULL，原因见（4）

（4）可以通过属性判断来跳过对SQL的 NOT NULL 约束（INSERT IGNORE），即插入一条记录，标记为 NOT NULL 的属性，由系统初始化为类型的默认值

https://dev.mysql.com/doc/refman/8.0/en/insert.html

Inserting NULL into a column that has been declared NOT NULL. For multiple-row INSERT statements or INSERT INTO ... SELECT statements, the column is set to the implicit default value for the column data type. This is 0 for numeric types, the empty string ('') for string types, and the “zero” value for date and time types. INSERT INTO ... SELECT statements are handled the same way as multiple-row inserts because the server does not examine the result set from the SELECT to see whether it returns a single row. (For a single-row INSERT, no warning occurs when NULL is inserted into a NOT NULL column. Instead, the statement fails with an error.)

（5）为了符合预期，NOT NULL 属性应该添加默认值，即 DEFAULT XXX，意思是使用自定义的默认值替换系统的初始化默认值，原理见（2）

（6）INSERT IGNORE 可能出现意料之外的结果，应该使用 INSERT INTO XXX VALUES(YYY) ON DUPLICATE KEY UPDATE id  = id 来替换;

INSERT IGNORE simply decides that it’s better to let the INSERT go through with all the missing non-nullable values set to an empty string — which is something I wasn’t expecting at all

https://odino.org/mysqls-insert-ignore-and-not-null-columns/

（7）NOT NULL 与 默认值，（5）和（6）满足一个即可
### 资料：
关于一次mysql的列属性not null的坑爹排查
https://blog.csdn.net/fyhailin/article/details/78538610
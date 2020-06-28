### NULL值不应该存储的原因

+ NULL值影响该列上是否使用索引

值为NULL的二级索引记录都被放在了B+树的最左边，这是因为设计InnoDB的大叔有这样的规定：We define the SQL null to be the smallest possible value of a field。
是否使用索引由执行计划决定，NULL值影响执行计划。
如果这个条数占整个记录条数的比例特别大，那么就趋向于使用全表扫描执行查询，否则趋向于使用这个索引执行查询。

+ NULL值存储需要额外信息

在COMPACT行格式中，需要存储记录的额外信息：NULL值列表。
允许存储NULL的列对应一个二进制位，二进制位按照列的顺序逆序排列，二进制位表示的意义如下：二进制位的值为1时，代表该列的值为NULL。二进制位的值为0时，代表该列的值不为NULL。

+ NULL值与业务语义可能不符

NULL值数据是否应该被查询，取决于业务的语义，但SQL中 NULL 值只能使用 IS NULL 或 IS NOT NULL 查询。
比如想查询一个名字不为张三的人：SELECT * FROM PERSON WHERE NAME !='张三'。如果 NAME 列允许为 NULL，那这些为 NULL 的数据该不该返回，由业务语义决定。然而，该SQL查询并不会返回 NAME 为 NULL 的数据。如果需要，则应该显示指定：SELECT * FROM PERSON WHERE NAME !='张三' OR NAME IS NULL。


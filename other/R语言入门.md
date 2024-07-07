### 1. 基本语法

R 是一种区分大小写的解释型语言

可以在命令提示符'>'后每次输入并执行一条命令

或者一次性执行卸载脚本文件中（文件为 xx.R）的一组命令

R 使用 '<-'（或者 '->'，箭头指向谁，就是向谁赋值），而不是传统的 '=' 作为赋值符号

注释由符号 '#' 开头，在 '#' 之后出现的任何文本都会被 R 解释器忽略

对象名称中的句点'.' 没有特殊意义，但是美元符号'$'有和其他语言中的句点类似的含义（对象中的子对象）

R 不提供多行注释或块注释功能，只能每行使用 '#' 分别注释

R 中的下标不从 0 开始，而是从 1 开始（其他语言一般从 0 开始）

R 变量无法被声明，它们在首次被赋值时生成

### 2. R 中的帮助函数

```r
# 查看函数foo的帮助
help(foo)

# 函数foo的使用示例
example(foo)

# 列出名称中含有foo的所有可用函数
apropos("foo",mode="function")

# 列出当前已加载包中所含的所有可用示例数据集
data()
```

### 3. 用于管理 R 工作空间的函数

```r
# 显示当前的工作目录
getwd()

# 修改当前的工作目录为foo，注意：setwd() 不会自动创建一个不存在的目录
setwd("foo")

# 创建新的目录foo
dir.create("foo")

# 列出当前工作空间中的对象
ls()

# 删除对象foo
rm(foo)
# 清空工作空间的所有对象，慎用
rm(list=ls())

# 显示最近使用过的x个命令，默认值为25
history(x)

# 保存工作空间到文件foo中，默认值为 .RData
save.image("foo")

# 读取一个工作空间到当前会话中，默认值为 .RData
load("foo")

#退出R
q()
```

### 4. 包的安装

```r
# 安装包foo
install.packages("foo")

# 更新已经安装的包
update.packages()

# 查看已经安装的包
installed.packages()

# 看看包的简要描述以及包中的函数名称和数据集名称的列表
help(package="package_name")

```

### 5. 常见错误

（1）使用了错误的大小写
help() √
Help() X

（2）忘记使用必要的引号
install.packages("gclus") √
install.packages(gclus) X

（3）在函数调用时忘记使用括号
help() √
help X

（4）在 windows 上，路径名中使用了 '\'
setwd("c:/foo") √
setwd("c:\\foo") √
setwd("c:\foo") X

（5）使用了一个尚未载入包中的函数

```r
# 载入工具包foo
library(foo)
# 然后再使用foo函数
```

### 6. 向量

```r
# 创建向量
c()

# 示例
a <- c(1, 2, 3, 4, 5, 6)
# 与上面的等价
a <- c(1:6)
# 访问向量a中的第二个和第四个元素
a[c(2,4)]
```

### 7. 矩阵

```r
# 创建矩阵
myymatrix <- matrix(vector, nrow=number_of_rows, ncol=number_of_columns,byrow=logical_value, dimnames=list(char_vector_rownames, char_vector_colnames))

# 示例
# 待填充的数据
cells <- c(1,26,24,68)
# 行名
rnames <- c("R1", "R2")
# 列名
cnames <- c("C1", "C2")
# 将cells中数据，按行，生成2X2的矩阵，并给定行列名
mymatrix <- matrix(cells, nrow=2, ncol=2, byrow=TRUE, dimnames=list(rnames, cnames))
# 上面的效果
mymatrix
 C1 C2
R1 1 26
R2 24 68

# 创建1个5X4的矩阵
y <- matrix(1:20, nrow=5, ncol=4)
# 使用下标和方括号悬着矩阵中的行、列或元素
# 矩阵y中的第i行
y[i,]
# 矩阵y中的第j列
y[,j]
# 矩阵y中第i行第j列的元素
y[i,j]
```

### 8. 数组

数据（array）与矩阵类似，到那时维度可以大于 2

数组是矩阵的一个自然推广

```r
# 创建数组
myarray <- array(vector, dimensions, dimnames)
```

### 9. 数据框

数据框的概念较矩阵来说更为一般

列向量 col1、col2 等可为任意类型（字符型、数值型或逻辑型）

```r
# 使用 data.frame()创建
mydata <- data.frame(col1, col2, col3,...)
# '$' 用来选取一个给定 frame 中的某个特定变量（就是选取特定的列）
y <- mydata$col1

# 使用 attach()和 detach() 来避免频繁的使用'mydata$'，下面的与上面的等价
# attach() 将 frame 添加到R的搜索路径
attach(mydata)
y <- col1
# detach() 将 frame 从 R的搜索路径移除
detach(mydata)

# 使用 with()，{} 等价 attach()和 detach()，赋值 '<-' 仅在括号内生效。如需外部生效，使用 '<<-'
with(mydata,{y <- col1 ; print(y)})
```

### 10. 因子

名义型变量是没有顺序之分的类别变量

有序型变量表示一种顺序关系，而非数量关系

连续型变量可以表现为某个范围内的任意值，并同时表示了顺序和数量

类别（名义型）变量和有序类别（有序型）变量在 R 中称为因子（factor）

函数 factor() 以 1 个整型向量的形式存储类别值，整数的取值范围是[1...k]（其中 k 是名义型变量中唯一值的个数），同时一个由字符串（原始值）组成的内部变量将映射到这些整数上

```r
diabetes <- c("Type1", "Type2", "Type1", "Type1")
# 将此向量存储为(1,2,1,1)，并在内部将其关联为 1 = Type1 和 2 = Type2 (具体赋值根据字母顺序而定)
diabetes <- factor(diabetes)

# 要表示有序型变量，需要指定参数 ordered = TRUE
status <- c("Poor", "Improved", "Excellent", "Poor")
status <- factor(status, ordered=TRUE)
# 通过指定 levels 选项来覆盖默认排序
status <- factor(status, order=TRUE, levels=c("Poor", "Improved", "Excellent"))

# 因子的使用
patientID <- c(1, 2, 3, 4)
age <- c(25, 34, 28, 52)
diabetes <- c("Type1", "Type2", "Type1", "Type1")
status <- c("Poor", "Improved", "Excellent", "Poor")
diabetes <- factor(diabetes)
status <- factor(status, order=TRUE)
patientdata <- data.frame(patientID, age, diabetes, status)
# 显示对象的结构
str(patientdata)
# 显示对象的统计概要
summary(patientdata)
```

### 11. 列表

列表是一些对象的有序集合

列表允许以一种简单的方式组织和重新调用不相干的信息

许多 R 函数的运行结果都是以列表的形式返回

```r
# 对象可以是目前讲到的任何结构
mylist <- list(object1, object2, ...)
```

### 12. 数据的输入

```r
# 使用键盘输入数据
mydata <- data.frame(age=numeric(0), gender=character(0), weight=numeric(0))
mydata <- edit(mydata)

# 从带分隔符的文本文件导入数据，具体参数，通过 help(read.table) 查看使用
mydataframe <- read.table(file, options)
# 示例
grades <- read.table("studentgrades.csv", header=TRUE, row.names="StudentID", sep=",",colClasses=c("character", "character", "character", "numeric", "numeric", "numeric"))

# 导入 excel 数据，file是文件所在路径，n是要导入的工作表序号
library(xlsx)
read.xlsx(file, n)
```

### 13. 处理数据对象的使用函数

```r
# 显示对象中元素的数量
length(object)
# 显示对象的维度
dim(object)
# 显示对象的结构
str(object)
# 显示对象的类型
class(object)
# 显示对象的模式
mode(object)
# 显示对象中各个成分的名称
names(object)
# 将对象合并入一个向量
c(object, object,...)
# 按列合并对象
cbind(object, object, ...)
# 按行合并对象
rbind(object, object, ...)
# 输出某个对象
object
# 列出对象的开始部分
head(object)
# 列出对象的最后部分
tail(object)
# 显示当前的对象列表
ls()
# 删除一个或多个对象
rm(object, object, ...)
# 删除当前工作环境的几乎所有对象
rm(list = ls())
# 编辑对象并另存为 newobject
newobject <- edit(object)
# 直接编辑对象
fix(object)
# 将 x 重复 n 次
rep(x, n)
# 连接对象
cat(... , file ="myfile", append =FALSE)
```

### 14. 简单绘图

```r
# 创建多个图形并随时查看每一个
# 在创建一副新图形之前打开一个新的图形窗口
dev.new()
statements to create graph 1
dev.new()
statements to create a graph 2

# 图形参数设置，在会话结束前一直有效
par(optionname=value, optionname=name,...)
# 示例
# 读取当前的设置
opar <- par(no.readonly=TRUE)
# 修改设置
par(lty=2, pch=17)
# 画图
plot(dose, drugA, type="b")
# 还原设置
par(opar)

```

### 15. 算术运算符

```r
# 加
+
# 减
–
# 乘
*
# 除˱
/
# 幂
^或**
# 取余
x%%y
# 整数除
x%/%y
```

### 16. 逻辑运算符

```r
# 小于
<
# 小于或等于
<=
# 大于
>
# 大于或等于
>=
# 严格等于
==
# 不等于
!=
# 非X
!x
# 或
x | y
# 与
x & y
# 测试x是否为TRUE
isTRUE(x)

```

### 17. 变量重命名

```r
# 交互式
fix(leadership)
# 编程方式
names(leadership)[2] <- "testDate"
```

### 18. 检查缺失值

```r
# 检查缺失值
is.na()
# 检查无穷
is.infinite()
# 检查非数（not a number）
is.nan()
```

### 19. 类型转换函数

```r
# 左边是判断，右边是转换
is.numeric()       as.numeric()
is.character()     as.character()
is.vector()        as.vector()
is.matrix()        as.matrix()
is.data.frame()    as.data.frame()
is.factor()        as.factor()
is.logical()       as.logical()

```

### 20. 排序

```r
order()

```

### 21. 数据集合并

```r
# inner join
total <- merge(dataframeA, dataframeB, by="ID")
# 横向合并
total <- cbind(A, B)
# 合并 frame
total <- rbind(dataframeA, dataframeB)
```

### 22. 数学函数

```r
# 绝对值
abs(x)
# 平方根
sqrt(x)
# 向上取整
ceiling(x)
# 向下取整
floor(x)
# 向0取整
trunc(x)
# 将 x 舍入为指定位的小数
round(x, digits=n)
# 将 x 舍入为指定的有效数字位数
signif(x, digits=n)
# 余弦 正弦 正切
cos(x) sin(x)  tan(x)
# 反余弦 反正弦 反正切
acos(x) asin(x) atan(x)
# 双曲余弦 双曲正弦 双曲正切
cosh(x) sinh(x) tanh(x)
# 反双曲余弦 反双曲正弦 反双曲正切
acosh(x) asinh(x) atanh(x)
# 以n为底的对数
log(x,base=n)
# 自然对数
log(x)
# 常用对数
log10(x)
# 指数
exp(x)

```

### 23. 数学函数

```r
# 平均数
mean(x)
# 中位数
median(x)
# 标准差
sd(x)
# 方差
var(x)
# 绝对中位差
mad(x)
# 分位数
quantile(x,probs)
# 值域
range(x)
# 求和
sum(x)
# 滞后差分
diff(x, lag=n)
# 最小值
min(x)
# 最大值
max(x)
# 为数据对象x按列进行中心化或标准化
scale(x,center=TRUE,scale=TRUE)
```

### 24. 将函数应用于矩阵或 frame

```r
# x 是数据对象，MARGIN 是维度的下标， FUN 是指定的函数，后续 ... 是传递给 FUN 的参数
apply(x, MARGIN, FUN, ...)

```

### 25. 循环、分支

```r
for (var in seq) statement
while (cond) statement
if (cond) statement
if (cond) statement1 else statement2
ifelse(cond, statement1, statement2)
switch(expr, ...)
```

### 26. 自定义函数

```r
myfunction <- function(arg1, arg2, ... ){
 statements
 return(object)
}
```

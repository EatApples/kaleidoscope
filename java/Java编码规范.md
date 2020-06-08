# Java Coding Guidelines: 75 Recommendations for Reliable and Secure Programs

https://wiki.sei.cmu.edu/confluence/display/java/Rec.%3A+All+Guidelines+with+Classification

Java™ 编码指南 由 75 条指南组成，分为以下五章，分别围绕以下原则进行组织：

第 1 章，安全性，介绍了确保 Java 应用程序安全性的准则。
第 2 章，防御性编程，包含防御性编程的准则，以便程序员可以编写保护自己免受意外情况影响的代码。
第 3 章，可靠性，为提高 Java 应用程序的可靠性和安全性提供了建议。
第 4 章，程序可理解性，提供有关使程序更具可读性和可理解性的建议。
第 5 章，程序员的误解，揭示了 Java 语言和编程概念经常被误解的情况。

# 规则 00.输入验证和数据清理（IDS）

**IDS00-J。** 防止 SQL 注入

使用 PreparedStatement，并使用占位符（而不是自己拼接 SQL）。

现在的 ORM 工具（Mybatis）应该已经封装了。

**IDS01-J。** 在验证字符串之前将其标准化

**IDS02-J。** 在验证路径名称之前对其进行规范化

**IDS03-J。** 不要记录未过滤的用户输入

因为你不知道用户会输入什么。

**IDS04-J。** 从 ZipInputStream 安全地提取文件

谨防 ZIP 炸弹。

**IDS05-J。** 使用 ASCII 的安全子集作为文件和路径名

**IDS06-J。** 从格式字符串中排除未经过滤的用户输入

**IDS07-J。** 清理传递给 Runtime.exec() 方法的不受信任的数据

使用系统调用（方法）替代。

**IDS08-J。** 清理正则表达式中包含的不受信任的数据

**IDS09-J。** 比较与语言环境相关的数据时，请指定适当的语言环境

Compliant Solution (String.equalsIgnoreCase())
Compliant Solution (Charset)

**IDS10-J。** 不要形成包含部分字符的字符串

```java
for (i = 0; i < string.length(); i += Character.charCount(ch)) {
    ch = string.codePointAt(i);
    if (!Character.isLetter(ch)) {
        break;
    }
}
```

**IDS11-J。** 验证之前执行任何字符串修改

重要的是在验证发生后不要修改字符串，因为这样做可能会使攻击者绕过验证。

不要做多余的事，验证就是验证，不要替换。

**IDS13-J。** 在文件或网络 IO 的两侧使用兼容的字符编码

**IDS14-J。** 不要相信隐藏表单域的内容

**IDS15-J。** 不允许敏感信息泄漏到信任边界之外

**IDS16-J。** 防止 XML 注入

**IDS17-J。** 防止 XML 外部实体攻击

# 规则 01.声明和初始化（DCL）

**DCL00-J。** 防止类初始化循环依赖

**DCL01-J。** 不要重复使用 Java 标准库中的公共标识符

容易造成误解。

**DCL02-J。** 在增强的 for 语句期间不要修改集合的元素

因为使用是迭代器，会有副作用。

# 规则 02.表达式（EXP）

**EXP00-J。** 不要忽略方法返回的值

不应该改变入参的值，即不要将入参作为出参。

使用返回值作为出参。

**EXP01-J。** 在需要对象的情况下，请勿使用 null

NPE 了解下。

**EXP02-J。** 不要使用 Object.equals()方法比较两个数组

Arrays.equals(arr1, arr2)

**EXP03-J。** 比较装箱的基本类型的值时不要使用相等运算符

比如 Integer 是对象类型，不要使用'=='，要用 equals

**EXP04-J。** 不要将参数传递给某些类型不同于集合参数类型的 Java Collections Framework 方法

类型提升和自动装箱会产生意外的对象类型。

**EXP05-J。** 不要在表达式后跟写或读取表达式中的同一对象

对象的状态可能改变，不再符合预期。

**EXP06-J。** 断言中使用的表达式不得产生副作用

**EXP07-J。** 防止由于弱引用而丢失有用数据

比如 ThreadLocal 应该设置为 static，否则 ThreadLocal 生命周期结束时，Thread 内部还持有 Value 引用，但是无法读取。

# 规则 03.数字类型和运算（NUM）

**NUM00-J。** 检测或防止整数溢出

**NUM01-J。** 不要对同一数据执行按位和算术运算

通常使用左移和右移运算符将数字乘以或除以 2 的幂。为了经常获得虚幻的速度，这种方法会损害代码的可读性和可移植性。

NUM01-EX0：可以使用按位运算来构造常量表达式。

**NUM02-J。** 确保除法和余数运算不会导致被零除错误

**NUM03-J。** 使用可以完全表示无符号数据可能范围的整数类型

**NUM04-J。** 如果需要精确计算，请不要使用浮点数

使用 BigDecimal 替代。

**NUM07-J。** 不要尝试与 NaN 进行比较

与数据库的 NULL 类似，请使用 isNaN 函数。

**NUM08-J。** 检查浮点输入是否有异常值

注意这些值：NaN, infinity, or -infinity

**NUM09-J。** 不要将浮点变量用作循环计数器

浮点数有精度损失。

**NUM10-J。** 不要从浮点数构造 BigDecimal 对象

使用 BigDecimal(String val)

**NUM11-J。** 不要比较或检查浮点值的字符串表示形式

使用 BigDecimal

**NUM12-J。** 确保将数字类型转换为较窄的类型不会导致数据丢失或解释错误

向下转型，不要超出表示范围。

**NUM13-J。** 将原始整数转换为浮点数时避免精度损失

**NUM14-J。** 正确使用移位运算符

注意符号右移'>>' 和 无符号右移 '>>>'

# 规则 04.字符和字符串（STR）

**STR00-J。** 不要形成包含可变宽度编码的部分字符的字符串

如果使用 byte 流来读取，则可变长度的编码数据可能被截断。可以将流读完，然后再转换。

或者使用 Reader r = new InputStreamReader(in, "UTF-8"); 来读取

**STR01-J。** 不要假定 Java 字符完全代表 Unicode 代码点

不同的编码使用的字节数不同，一个字符可能占用 2 个 char，4 个字节。

使用 string.codePointAt(i); 来获取字符，使用 Character.charCount(ch) 来统计字符所用 char 个数：

```java
public static String trim(String string) {
  int ch;
  int i;
  for (i = 0; i < string.length(); i += Character.charCount(ch)) {
    ch = string.codePointAt(i);
    if (!Character.isLetter(ch)) {
      break;
    }
  }
  return string.substring(i);
}
```

**STR02-J。** 比较与语言环境相关的数据时，请指定适当的语言环境

**STR03-J。** 不要将非字符数据编码为字符串

```java
BigInteger x = new BigInteger("530500452766");
String s = x.toString();  // Valid character data
byte[] byteArray = s.getBytes();
String ns = new String(byteArray);
x = new BigInteger(ns);
```

**STR04-J。** 在 JVM 之间通信字符串数据时使用兼容的字符编码

String result = new String(data, encoding);

# 规则 05.对象类型（OBJ）

**OBJ01-J。** 限制字段的可访问性

**OBJ02-J。** 更改超类时保留子类中的依赖项

开发人员对超类的特定于实现的细节进行了假设。即使这些假设最初是正确的，超类的实现细节也可能会更改，而不会发出警告。

使用组合而不是继承。

使用继承的话，对于不用的父类函数，最好显式覆盖（抛异常就行），以免错误调用。

**OBJ03-J。** 防止堆污染

因为泛型会类型擦除。使用时，请强制指定数据类型。PECS。

**OBJ04-J。** 提供具有复制功能的可变类，以安全地允许将实例传递给不受信任的代码

使用保护性拷贝。

**OBJ05-J。** 不要返回对私有可变类成员的引用

使用深度拷贝。

**OBJ06-J。** 防御性地复制可变输入和可变内部组件

如果入参可变，保护性地做个快照。一是防止外部更改，影响内部逻辑；二是防止内部更改，影响入参。

**OBJ07-J。** 敏感的类一定不能让自己被复制

信息不能泄露。

**OBJ08-J。** 不要从嵌套类中公开外部类的私有成员

嵌套类不得将外部类的私有成员暴露给外部类或包。

**OBJ09-J。** 比较类别而不是类别名称

```java
// Determine whether object auth has required/expected class name
 if (auth.getClass() == com.application.auth.DefaultAuthenticationHandler.class) {
   // ...
}


// Determine whether objects x and y have the same class
if (x.getClass() == y.getClass()) {
  // Objects have the same class
}
```

**OBJ10-J。** 不要使用公共静态非最终字段

```java
public static final FuncLoader m_functions;
// Initialize m_functions in a static initialization block


class DataSerializer implements Serializable {
  private static final long serialVersionUID = 1973473122623778747L;
}
```

**OBJ11-J。** 警惕让构造函数引发异常

安全发布与初始化（使用 final 关键字）。

**OBJ12-J。** 尊重基于对象的注释

**OBJ13-J。** 确保不暴露对可变对象的引用

**OBJ14-J。** 不要使用已释放的对象。

# 规则 06.方法（MET）

**MET00-J。** 验证方法参数

**MET01-J。** 永远不要使用断言来验证方法参数

有 2 点原因：

断言可被禁用，这样的话，校验就不再生效

校验失败应该抛有意义的异常，而断言失败不会引发适当的异常。

**MET02-J。** 不要使用过时或过时的类或方法

**MET03-J。** 执行安全检查的方法必须声明为私有或最终的

MET03-J-EX0：声明为 final 的类不受此规则约束，因为它们的成员方法不能被覆盖。

**MET04-J。** 不要增加覆盖或隐藏方法的可访问性

使用 private 或者 final 修饰。

**MET05-J。** 确保构造函数不要调用可重写的方法

使用 final 来修饰。

**MET06-J。** 不要在 clone()中调用可重写的方法

使用 final 来修饰。

**MET07-J。** 永远不要在子类中定义能隐藏父类的类方法的类方法

使用函数重写。

**MET08-J。** 覆盖 equals()方法时满足规范

（1）自反性
（2）对称性
（3）传递性
（4）一致性。如果对象没有修改（equals 方法中涉及到的信息不变），多次调用 equals，总是返回同一个结果
（5）非空性。对于任何非空引用值 x，x.equals (null)应该返回 false

**MET09-J。** 定义 equals()方法的类还必须定义 hashCode()方法

**MET10-J。** 实现 compareTo()方法时，请遵循常规约定

相等关系（自反性，对称性，传递性）或偏序关系（自反性，反对称性，传递性）

**MET11-J。** 确保比较操作中使用的键是不可变的

**MET12-J。** 不要使用终结器

**MET13-J。** 不要假设修改方法参数的值会有副作用
java 是 call by value。

java 的基本类型有 8 种（boolean,byte,char,short,int,long,float,double），函数调用时在栈上直接分配参数值的副本。
java 的引用类型有 4 种（类，接口，数组，泛型），函数调用时将参数的地址副本在栈上分配，地址副本和原地址所指向的堆中对象是同一个。

```java
void canNotChange(int i,int[] arr,SomeObject obj){
    i=-1;//并不会影响入参
    arr[0]=-1;//并不会影响入参，但是堆中的数据被改变了
    obj.setSomething(something);//并不会影响入参，但是堆中的数据被改变了
    arr=new int[10];//并不会影响入参
    obj=new SomeObject();//并不会影响入参
}
```

# 规则 07.异常行为（ERR）

**ERR00-J。** 不要忽略受检查的异常

记录或直接处理。

**ERR01-J。** 不允许异常对外暴露敏感信息

如果异常内容含有敏感信息，比如 java.sql.SQLException，java.io.FileNotFoundException 等，应该对异常做封装，不应该直接对外暴露。

**ERR02-J。** 记录日志时防止异常

使用日志组件记录日志，而不是标准输出或错误输出。

**ERR03-J。** 方法失败时，应该恢复先前对象状态

不要因为方法失败导致状态的不一致。

**ERR04-J。** 不要从 finally 块中 return

因为 finally 中的返回值会作为最终的返回。

**ERR05-J。** 不要让受检异常从 finally 块中逸出

如果 finally 块中直接 return，或抛出新的异常，则之前的 try 块中未被 catch 的异常 或 catch 块中新抛出的异常都会被忽略！

**ERR06-J。** 不要抛出未声明的检查异常

在 Java 语言规范中，所有异常都是 Throwable 类或其子类的实例。

Throwable 有两个重要的子类：Exception（异常）和 Error（错误）。而 Exception 又分为 RuntimeException 和其他类型。

Exception 和 Error 的区别是：Exception 涵盖程序可能需要捕获并处理的异常；Error 涵盖不应捕获的异常。

RuntimeException 与 Error 属于 Java 里的非检查异常，其他异常则属于检查异常。

简单理解：

（1）检查异常：程序编译时就知道了，不处理不行；

（2）RuntimeException：程序运行时抛出，有点意外，处不处理都可以（建议关键代码 catch Exception）；

（3）Error：鬼知道程序发生了什么？不知道怎么处理，也不应该处理。

注意：即使有这些区分，其实 catch 能捕获 Throwable！！！

受检异常从语义上看，说明该异常应该被处理。而错误或运行时异常从语义上看，是不应该被处理。

**ERR07-J。** 不要抛出 RuntimeException，Exception 或 Throwable

**ERR08-J。** 不要捕获 NullPointerException 或其任何父类

函数应该有前置条件约束，提前做入参检测而不是通过异常逻辑来处理：

（1）catch 逻辑的开销比简单的检测更多
（2）捕获的 NPE 可能是其他原因所致，不一定是预期的那一个
（3）一般抛出 NPE 时，表明程序处于不可用状态，很难恢复

**ERR09-J。** 不允许不受信任的代码终止 JVM

程序必须防止无意或恶意调用 System.exit()。此外，程序在强制终止时应执行必要的清除操作。

# 规则 08.可见性和原子性（VNA）

**VNA00-J。** 访问共享基本变量时确保可见性

为了确保最新更新的可见性，必须将变量声明为 volatile 或必须同步读取和写入。

**VNA01-J。** 确保对不可变对象的共享引用的可见性

一个常见的误解是，一旦更新了对不可变对象的共享引用，它们就会立即在多个线程中可见。例如，开发人员可能会错误地认为，包含仅引用不可变对象的字段的类本身就是不可变的，因此是线程安全的。

虽然不可变对象（final 修饰）本身不可变，但是不可变对象的引用是可变的，仍然需要保证其引用的可见性。

The problem is that, while the shared object is immutable, the reference used to access the shared object is itself shared and often mutable. Consequently, a correctly synchronized program must synchronize access to that shared reference, but often programs do not do this, because programmers do not recognize the need to do it.

**VNA02-J。** 确保对共享变量的复合操作是原子的

volatile 只保证单指令操作的可见性，如果是复合操作，不保证原子性。

**VNA03-J。** 不要假设对独立原子方法的一组调用是原子的

复合的原子操作不具有原子性！

**VNA04-J。** 确保对链式方法的调用是原子的

**VNA05-J。** 读写 64 位值时确保原子性

使用 volatile

### 规则 09.锁定（LCK）

**LCK00-J。** 使用私有的最终锁定对象来同步可能与不受信任的代码进行交互的类

**LCK01-J。** 不要在可能被重用的对象上同步

**LCK02-J。** 不要在 getClass()返回的类对象上同步

**LCK03-J。** 不要在高级并发对象的内在锁上同步

**LCK04-J。** 如果可访问后备集合，则不要在集合视图上同步

**LCK05-J。** 同步访问可以由不受信任的代码修改的静态字段

**LCK06-J。** 不要使用实例锁来保护共享的静态数据

**LCK07-J。** 通过以相同顺序请求和释放锁来避免死锁

**LCK08-J。** 确保在特殊情况下释放主动保持的锁

**LCK09-J。** 请勿在保持锁定状态下执行会阻塞的操作

**LCK10-J。** 使用正确形式的双重检查锁定习惯用语

**LCK11-J。** 使用不提交其锁定策略的类时，避免客户端锁定

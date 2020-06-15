# Java Coding Guidelines: 75 Recommendations for Reliable and Secure Programs

https://wiki.sei.cmu.edu/confluence/display/java/SEI+CERT+Oracle+Coding+Standard+for+Java

## 第二部分-规则

https://wiki.sei.cmu.edu/confluence/display/java/2+Rules

### 规则 00.输入验证和数据清理（IDS）

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

### 规则 01.声明和初始化（DCL）

**DCL00-J。** 防止类初始化循环依赖

**DCL01-J。** 不要重复使用 Java 标准库中的公共标识符

容易造成误解。

**DCL02-J。** 在增强的 for 语句期间不要修改集合的元素

因为使用是迭代器，会有副作用。

### 规则 02.表达式（EXP）

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

### 规则 03.数字类型和运算（NUM）

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

### 规则 04.字符和字符串（STR）

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

### 规则 05.对象类型（OBJ）

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

### 规则 06.方法（MET）

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

### 规则 07.异常行为（ERR）

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

### 规则 08.可见性和原子性（VNA）

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

**LCK00-J。** 使用 private final 对象来同步可能与不受信任的代码进行交互的类
使用同步块，而不是同步方法。

如果使用 synchronized 修饰方法，则有拒绝服务的风险：攻击者可以持有类或对象的内置锁（monitor）永不释放。

```java
public class SomeObject {
  private final Object lock = new Object(); // private final lock object

  public void changeValue() {
    synchronized (lock) { // Locks on the private Object
      // ...
    }
  }
}


public class SomeObject {
  private static final Object lock = new Object();

  public static void changeValue() {
    synchronized (lock) { // Locks on the private Object
      // ...
    }
  }
}
```

**LCK01-J。** 不要在可能被重用的对象上同步

```java

private final Object lock = new Object();

public void doSomething() {
  synchronized (lock) {
    // ...
  }
}
```

**LCK02-J。** 不要在 getClass()返回的类对象上同步

**LCK03-J。** 不要在高级并发对象的内在锁上同步

**LCK04-J。** 如果内置集合是可访问，则不要在集合视图上同步
同步对象可能改变。

**LCK05-J。** 类的的静态字段改变，也需要使用同步

**LCK06-J。** 不要使用对象锁来保护共享的静态数据

使用类锁而不是对象锁来同步静态数据。

**LCK07-J。** 通过以相同顺序请求和释放锁来避免死锁

**LCK08-J。** 确保在特殊情况下锁一定会被释放

```java
    lock.lock();
    try {
      //do something
    } finally {
      lock.unlock();
    }
```

**LCK09-J。** 请勿在保持锁定状态下执行会阻塞的操作

**LCK10-J。** 使用正确形式的双重检查锁定

```java
		private volatile FieldType field;
		FieldType getField(){
			FieldType result =  field;
			if(result == null){
				synchronized(this){
					result=field;
					if(result == null){
						field = result = computeFieldValue();
					}

				}
			}
			return result;
		}
```

**LCK11-J。** 使用包装类同步时，不应该依赖被包装类的同步逻辑
应该使用自定义的同步逻辑，以免这种隐性同步失效。

### 规则 10.线程 API（THI）

**THI00-J。** 不要调用 Thread.run()

start() 方法才会真正新建一个线程，然后该线程执行 run() 方法；而 run()方法只是一次当前线程的方法调用。

**THI01-J。** 不要调用 ThreadGroup 的方法
除了过时之外，有些方法还不是线程安全的。

**THI02-J。** 通知所有等待的线程，而不是单个线程
因为通知一个容易唤醒丢失。

**THI03-J。** 始终在循环内调用 wait()和 await()方法
因为调用这 2 个方法时，必须处于临界区。然而被唤醒时，等待条件不一定为真，仍需睡眠。

**THI04-J。** 确保执行阻塞操作的线程可以被终止

**THI05-J。** 不要使用 Thread.stop()终止线程
使用 Thread.interrupt()，或其他方式优雅地停止，否则可能出现与预期不符的情况。

### 规则 11.线程池（TPS）

**TPS00-J。** 使用线程池而不是新建一个线程

**TPS01-J。** 不要在有界线程池中执行相互依赖的任务
容易造成循环依赖而不能推进（活锁）。

**TPS02-J。** 确保提交给线程池的任务是可中断的
活性保证。出现异常情况，可以终止。

**TPS03-J。** 确保在线程池中执行的任务不会静默失败
关键代码 try-catch，或者注册异常处理方法。

**TPS04-J。** 确保在使用线程池时重新初始化 ThreadLocal 变量
因为线程池中，线程的生命周期不是 run() 完后就结束，而是会被重用。

在使用 ThreadLocal 变量时，如果不希望保留上一次的数据，线程需要在任务结束后，主动清理 ThreadLocal 中存储的内容。

### 规则 12.线程安全杂项（TSM）

**TSM00-J。** 不要用非线程安全的方法覆盖线程安全的方法

**TSM01-J。** 在对象构建期间，请勿 this 引用逃逸
this 是对象指针，不要对外暴露。

**TSM02-J。** 在类初始化期间不要使用后台线程
可能造成类初始化死循环和死锁。

**TSM03-J。** 不要发布部分初始化的对象

### 规则 13.输入输出（FIO）

**FIO00-J。** 不要对共享目录中的文件进行操作

**FIO01-J。** 创建具有适当访问权限的文件

**FIO02-J。** 检测并处理与文件相关的错误

**FIO03-J。** 终止前删除临时文件

**FIO04-J。** 在不再需要时释放资源

**FIO05-J。** 不要将缓冲区或其后备数组方法暴露给不受信任的代码

**FIO06-J。** 不要在单个字节或字符流上创建多个缓冲包装器

**FIO07-J。** 不要让外部进程阻塞 IO 缓冲区

**FIO08-J。** 区分从流读取的字符或字节与-1 的比较
因为 byte 或 char 与 int 比较时，会向上转型，导致与 -1 比较时，永远不会相等。

-1 是 int 型，对应的 16 进制为：0xFFFFFFFF
byte 的 -1，即 0xFF，会被提升为 0x000000FF
char 的 -1，即 0xFFFF，会被提升为 0x0000FFFF

```java
FileInputStream in;
// Initialize stream
int inbuff;
byte data;
while ((inbuff = in.read()) != -1) {
  data = (byte) inbuff;
  // ...
}


FileReader in;
// Initialize stream
int inbuff;
char data;
while ((inbuff = in.read()) != -1) {
  data = (char) inbuff;
  // ...
}
```

**FIO09-J。** 不要依赖 write()方法来输出 0 到 255 范围之外的整数
因为 write()的参数必须在 0 到 255 的范围内，超出时会被截断，导致与预期不符。

**FIO10-J。** 使用 read()填充数组时，请确保填充数组
某些数据流的长度小于缓冲区。

**FIO11-J。** 在未指定有效字符编码的情况下，请勿在字符串和字节之间进行转换

**FIO12-J。** 提供读写小端数据的方法
在 Java 中，数据以 big-endian 格式（也称为网络顺序）存储。即从最高有效位到最低有效位依次表示所有数据。

**FIO13-J。** 不要在信任范围之外记录敏感信息

**FIO14-J。** 在程序终止时执行适当的清理

请使用的 addShutdownHook()方法来协助执行清理操作。

```java
Runtime.getRuntime().addShutdownHook(new Thread() {
          public void run() {
            // Log shutdown and close all resources
          }
      });
```

**FIO15-J。** 不要重置 servlet 的输出流

```java
public void doGet(HttpServletRequest request, HttpServletResponse response)
  throws IOException, ServletException {

  try {
    // Do work that doesn't require the output writer
  } catch (IOException x) {
    response.sendError(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);
  }

  PrintWriter out = response.getWriter();
  try {
    out.println("<html>");

    // ... All work

  } catch (IOException ex) {
    out.println(ex.getMessage());
  } finally {
    out.flush();
  }
}
```

**FIO16-J。** 在验证路径名称之前对其进行规范化

### 规则 14.序列化（SER）

**SER00-J。** 在类演变过程中保证序列化兼容性
可以考虑使用 json。

**SER01-J。** 不要偏离序列化方法的正确签名
在对象序列化和反序列化期间需要特殊处理的类必须使用具有以下正确签名的特殊方法来实现：

```java
private void writeObject(java.io.ObjectOutputStream out)
    throws IOException;
private void readObject(java.io.ObjectInputStream in)
    throws IOException, ClassNotFoundException;
private void readObjectNoData()
    throws ObjectStreamException;
```

可序列化的类也可以实现 readResolve()和 writeReplace()方法。根据序列化规范[ Sun 2006 ] readResolve()和 writeReplace()方法文档：

对于 Serializable 和 Externalizable 类，该 readResolve 方法允许一个类替换/解析从流中读取的对象，然后将其返回给调用方。通过实现该 readResolve 方法，类可以直接控制自己反序列化的实例的类型和实例。

对于 Serializable 和 Externalizable 类，该 writeReplace 方法允许对象的类在写入对象之前在流中指定其自己的替换。通过实现该 writeReplace 方法，类可以直接控制要序列化的自身实例的类型和实例。

```java
  protected Object readResolve() {
    // ...
  }

  protected Object writeReplace() {
    // ...
  }
```

**SER02-J。** 在对象被发布到信任边界之外之前，签名然后加密对象

**SER03-J。** 不要序列化未加密的敏感数据

**SER04-J。** 不允许序列化和反序列化绕过安全管理器

**SER05-J。** 不要序列化内部类的实例

序列化在非静态上下文中声明的内部类，该内部类包含对封闭类实例的隐式引用，从而导致对其关联的外部类实例的序列化。

Java 编译器生成的用于实现内部类的合成字段取决于实现，并且在编译器之间可能有所不同。此类字段中的差异可能会破坏兼容性，并导致冲突的默认 serialVersionUID 值。分配给本地和匿名内部类的名称也取决于实现，并且在编译器之间可能有所不同。

因为内部类不能声明除编译时常量字段之外的静态成员，所以内部类不能使用该 serialPersistentFields 机制指定可序列化的字段。

由于与外部实例相关联的内部类没有零参数构造函数（此类内部类的构造函数隐式接受封闭实例作为前置参数），因此它们无法实现 Externalizable。该 Externalizable 接口要求实现对象使用 writeExternal()和 readExternal()方法手动保存和恢复其状态。

因此，程序不得序列化内部类。

**SER06-J。** 在反序列化期间对需要序列化的数据，进行保护性拷贝，防止代码注入

**SER07-J。** 对于具有特殊实现的类，不要使用默认的序列化形式

**SER08-J。** 在反序列化特权上下文之前，请最小化权限

**SER09-J。** 不要从 readObject() 方法中调用可重写的方法

**SER10-J。** 避免序列化期间的内存和资源泄漏

**SER11-J。** 防止覆盖 Externalizable 对象
使用同步保证线程安全。

**SER12-J。** 防止不信任数据反序列化

**SER13-J。** 反序列化方法不应执行潜在的危险操作

### 规则 15.平台安全（SEC）

**SEC00-J。** 不允许特权块跨信任边界泄漏敏感信息

**SEC01-J。** 不允许在特权块中使用外部变量
外部变量不安全。

**SEC02-J。** 不要基于不受信任的来源进行安全检查

**SEC03-J。** 在允许不受信任的代码加载任意类之后，不要加载受信任的类

**SEC04-J。** 通过安全管理器检查来保护敏感操作

**SEC05-J。** 不要使用反射来增加类，方法或字段的可访问性

**SEC06-J。** 不要依赖 URLClassLoader 和 java.util.jar 提供的默认自动签名验证

**SEC07-J。** 编写自定义类加载器时，调用超类的 getPermissions()方法

**SEC08-J。** 可信代码必须丢弃或清理不可信代码提供的任何参数
外部输入总是不安全的，要做检查。

**SEC09-J。** 绝不将标准 API 方法的结果从受信任的代码中泄漏到不受信任的代码

**SEC10-J。** 绝不允许不受信任的代码调用任何可能调用反射 API 的 API

### 规则 16.运行时环境（ENV）

**ENV00-J。** 即使是仅执行非特权操作的代码也需要小心授权

**ENV01-J。** 将所有对安全敏感的代码放在一个 JAR 中并签名并封装

**ENV02-J。** 不信任外部环境变量的值
因为可能改变。使用 JVM 的环境变量。

**ENV03-J。** 不要对危险的权限组合授权

**ENV04-J。** 不要禁用字节码验证

**ENV05-J。** 不要部署可以远程监控的应用程序

**ENV06-J。** 生产代码中不得包含调试入口点

### 规则 17. Java 本机接口（JNI）

**JNI00-J。** 围绕本机方法定义包装器

**JNI01-J。** 使用直接调用者的类加载器实例（loadLibrary）安全地调用标准 API

**JNI02-J。** 不要假设对象引用是恒定的或唯一的

**JNI03-J。** 不要在 JNI 代码中使用指向 Java 对象的直接指针

因为垃圾回收器可能会在直接指针仍然存在的情况下移动或删除 Java 对象。

**JNI04-J。** 不要假设 Java 字符串以 null 结尾

### 规则 49.其他（MSC）

**MSC00-J。** 使用 SSLSocket 而不是 Socket 进行安全的数据交换

**MSC01-J。** 不要使用空的无限循环

**MSC02-J。** 生成强随机数

```java
public static void main (String args[]) {
  SecureRandom number = new SecureRandom();
  // Generate 20 integers 0..20
  for (int i = 0; i < 20; i++) {
    System.out.println(number.nextInt(21));
  }
}
```

**MSC03-J。** 绝不硬编码敏感信息

**MSC04-J。** 不要泄漏内存

**MSC05-J。** 不要耗尽堆空间

**MSC06-J。** 进行迭代时请勿修改基础集合
使用迭代器来操作。

**MSC07-J。** 防止单例对象的多个实例化

**MSC08-J。** 不要将不可序列化的对象作为属性存储在 HTTP 会话中

**MSC09-J。** 对于 OAuth，请确保（a）[在最后一步中接收用户 ID 的依赖方]与（b）[被授予访问令牌的依赖方]相同。

**MSC10-J。** 请勿使用 OAuth 2.0 隐式授权（隐式许可）进行身份验证
第三方重定向到授权服务器，资源所有者授权之后，授权服务器给浏览器一个 token，然后浏览器将这个 token 传给第三方，第三方拿到 token 才能访问资源服务器。

问题：其他应用从浏览器拿到这个 token 后，也能访问资源服务器。

**MSC11-J。** 不要让会话信息在 Servlet 中泄漏

Java Servlet 通常必须存储与连接到它们的每个客户端关联的信息。使用中的成员字段 javax.servlet.http.HttpServlet 存储特定于各个客户端的信息是一种常见的简单做法。但是，由于以下原因，这样做是错误的：

在任何 Java servlet 容器（如 Apache Tomcat）中，HttpServlet 都是单例类（请参阅 MSC07-J。有关单例类的信息，请防止单例对象的多个实例化）。因此，即使成员变量未声明为静态，也只能有一个实例。

允许 servlet 容器从多个线程调用 servlet。因此，访问 servlet 中的字段可能导致数据争用。

如果两个客户端启动与 Servlet 的会话，则 Servlet 可能会将信息从一个客户端泄漏到另一个客户端。

Java servlet 提供了一个 HttpSession 用于存储特定于会话的数据的类，该类在每个 Web 请求中进行了编码。使用此类可防止跨会话信息泄漏和数据争用。

## 第三部分-建议

https://wiki.sei.cmu.edu/confluence/display/java/3+Recommendations

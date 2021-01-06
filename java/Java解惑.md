### Java 解惑

```
作者: 布洛赫 / 加夫特
```

### 1. 奇偶性

下面的方法意图确定它那唯一的参数是否是一个奇数。这个方法能够正确运转吗？

```java
public static boolean isOdd(int i){
    return i % 2 == 1;
}
```

Java 对取余操作符（%）的定义为对于所有的 int 数值 a 和所有的非零 int 数值 b，都满足下面的恒等式：

```java
(a / b) * b + (a % b) == a
```

当 i 是一个负奇数时，i % 2 等于-1 而不是 1， 因此 isOdd 方法将错误地返回 false。

这个问题很容易订正。只需将 i % 2 与 0 而不是与 1 比较，并且反转比较的含义即可：

```java
public static boolean isOdd(int i){
    return i % 2 != 0;
}
```

用位操作符 AND（&）来替代取余操作符会显得更好：

```java
public static boolean isOdd(int i){
    return (i & 1) != 0;
}
```

### 2. 找零时刻

它会打印出什么呢？

```java
public class Change{
    public static void main(String args[]){
        System.out.println(2.00 - 1.10);
    }
}
```

如果你运行该程序，你就会发现它打印的是 0.8999999999999999。问题在于 1.1 这个数字不能被精确表示成为一个 double，因此它被表示成为最接近它的 double 值。

更一般地说，问题在于并不是所有的小数都可以用二进制浮点数来精确表示的。

二进制浮点对于货币计算是非常不适合的，因为它不可能将 0.1——或者 10 的其它任何次负幂——精确表示为一个长度有限的二进制小数。

解决该问题的一种方式是使用某种整数类型，例如 int 或 long，并且以分为单位来执行计算。

解决该问题的另一种方式是使用执行精确小数运算的 BigDecimal。一定要用 BigDecimal(String)构造器，而千万不要用 BigDecimal(double)。

通过正确使用 BigDecimal，程序就可以打印出我们所期望的结果 0.90：

```java
import java.math.BigDecimal;
public class Change1{
    public static void main(String args[]){
        System.out.println(new BigDecimal("2.00").subtract(new BigDecimal("1.10")));
    }
}
```

### 3. 长整除

这个程序会打印出什么呢？

```java
public class LongDivision{
    public static void main(String args[]){
        final long MICROS_PER_DAY = 24 * 60 * 60 * 1000 * 1000;
        final long MILLIS_PER_DAY = 24 * 60 * 60 * 1000;
        System.out.println(MICROS_PER_DAY/MILLIS_PER_DAY);
    }
}
```

除数和被除数都是 long 类型的，long 类型大到了可以很容易地保存这两个乘积而不产生溢出。因此，看起来程序打印的必定是 1000。

遗憾的是，它打印的是 5。这里到底发生了什么呢？

问题在于常数 数 MICROS_PER_DAY 的计算“确实”溢出了。

尽管计算的结果适合放入 long 中，并且其空间还有富余，但是这个结果并不适合放入 int 中。这个计算完全是以 int 运算来执行的，并且只有在运算完成之后，其结果才被提升到 long，而此时已经太迟了：计算已经溢出了，它返回的是一个小了 200 倍的数值。

那么为什么计算会是以 int 运算来执行的呢？因为所有乘在一起的因子都是 int 数值。当你将两个 int 数值相乘时，你将得到另一个 int 数值。Java 不具有目标确定类型的特性，这是一种语言特性，其含义是指存储结果的变量的类型会影响到计算所使用的类型。

通过使用 long 常量来替代 int 常量作为每一个乘积的第一个因子，我们就可以很容易地订正这个程序。这样做可以强制表达式中所有的后续计算都用 long 运作来完成。尽管这么做只在 MICROS_PER_DAY 表达式中是必需的，但是在两个乘积中都这么做是一种很好的方式。相似地，使用 long 作为乘积的“第一个”数 值也并不总是必需的，但是这么做也是一种很好的形式。在两个计算中都以 long 数值开始可以很清楚地表明它们都不会溢出。下面的程序将打印出我们所期望的 1000：

```java
public class LongDivision{
    public static void main(String args[]){
        final long MICROS_PER_DAY = 24L * 60 * 60 * 1000 * 1000;
        final long MILLIS_PER_DAY = 24L * 60 * 60 * 1000;
        System.out.println(MICROS_PER_DAY/MILLIS_PER_DAY);
    }
}
```

这个教训很简单：当你在操作很大的数字时，千万要提防溢出——它可是一个缄默杀手。即使用来保存结果的变量已显得足够大，也并不意味着要产生结果的计算具有正确的类型。当你拿不准时，就使用 long 运算来执行整个计算。

### 4. 初级问题

下面的程序只涉及加法，它又会打印出什么呢？

```java
public class Elementary{
    public static void main(String[] args){
        System.out.println(12345+5432l);
    }
}
```

当你运行该程序时，它打印出的是 17777。

请仔细观察 + 操作符的两个操作数，我们是将一个 int 类型的 12345 加到了 long 类型的 5432l 上。请注意左操作数开头的数字 1 和右操作数结尾的小写字母 l 之间的细微差异。

这里确实有一个教训：在 long 型字面常量中，一定要用大写的 L，千万不要用小写的 l。这样就可以完全掐断这个谜题所产生的混乱的源头。

### 5. 循环

你的任务就是写一个变量声明，在将它作用于该循环之上时，使得该循环无限循环下去。

（1）考虑下面的 for 循环：

```java
for (int i = start; i <= start + 1; i++) {}
```

看起来它好像应该只迭代两次，但是通过利用在谜题 26 中所展示的溢出行为，可以使它无限循环下去。下面的的声明就采用了这项技巧：

```java
int start = Integer.MAX_VALUE - 1;
```

（2）什么样的声明能够让下面的循环变成一个无限循环？

```java
While (i == i + 1) {}
```

如果 i 在循环开始之前被初始化为无穷大，那么终止条件测试(i == i + 1)就会被计算为 true，从而使循环永远都不会终止。

你可以用任何被计算为无穷大的浮点算术表达式来初始化 i，例如：

```java
double i = 1.0 / 0.0;
```

不过，你最好是能够利用标准类库为你提供的常量：

```java
double i = Double.POSITIVE_INFINITY;
```

（3）请提供一个对 i 的声明，将下面的循环转变为一个无限循环：

```java
while (i != i) {
}
```

NaN 不等于任何浮点数值，包括它自身在内。因此，如果 i 在循环开始之前被初始化为 NaN，那么终止条件测试(i != i)的计算结果就是 true，循环就永远不会终止。很奇怪但却是事实。

你可以用任何计算结果为 NaN 的浮点算术表达式来初始化 i，例如：

```java
double i = 0.0 / 0.0;
```

同样，为了表达清晰，你可以使用标准类库提供的常量：

```java
double i = Double.NaN;
```

（4）请提供一个对 i 的声明，将下面的循环转变为一个无限循环：

```java
while (i != i + 0) {
}
```

唯一的 + 操作符有定义的非数值类型就是 String。+ 操作符被重载了：对于 String 类型，它执行的不是加法而是字符串连接。如果在连接中的某个操作数具有非 String 的类型，那么这个操作书就会在连接之前转换成字符串。

（5）请提供一个对 i 的声明，将下面的循环转变为一个无限循环：

```java
while (i != 0) {
    i >>>= 1;
}
```

回想一下，>>>=是对应于无符号右移操作符的赋值操作符。0 被从左移入到由移位操作而空出来的位上，即使被移位的负数也是如此。

假设你在循环的前面放置了下面的声明：

```java
short i = -1;
```

因为 i 的初始值（(short)0xffff）是非 0 的，所以循环体会被执行。在执行移位操作时，第一步是将 i 提升为 int 类型。所有算数操作都会对 short、byte 和 char 类型的操作数执行这样的提升。这种提升是一个拓宽原始类型转换，因此没有任何信息会丢失。这种提升执行的是符号扩展，因此所产生的 int 数值是
0xffffffff。然后，这个数值右移 1 位，但不使用符号扩展，因此产生了 int 数值 0x7fffffff。最后，这个数值被存回到 i 中。为了将 int 数值存入 short 变量，Java 执行的是可怕的窄化原始类型转换，它直接将高 16 位截掉。这样就 只剩下(short)oxffff 了，我们又回到了开始处。循环的第二次以及后续的迭代行为都是一样的，因此循环将永远不会终止。

如果你将 i 声明为一个 short 或 byte 变量，并且初始化为任何负数，那么这种行为也会发生。

如果你声明 i 为一个 char，那么你将无法得到无限循环，因为 char 是无符号的，所以发生在移位之前的拓宽原始类型转换不会执行符号扩展。

总之，不要在 short、byte 或 char 类型的变量之上使用复合赋值操作符。因为这样的表达式执行的是混合类型算术运算，它容易造成混乱。更糟的是，它们执行将隐式地执行会丢失信息的窄化转型，其结果是灾难性的。

（6）请提供一个对 i 的声明，将下面的循环转变为一个无限循环：

```java
while (i <= j && j <= i && i != j) {
}
```

因为 Java 的判等操作符（==和!=）在作用于对象引用时，执行的是引用 ID 的比较，而不是值的比较。让我们更具体一些，下面的声明赋予表达式(i <= j && j <= i && i != j)的值为 true，从而将这个循环变成了一个无限循环：

```java
Integer i = new Integer(0);
Integer j = new Integer(0);
```

（7）请提供一个对 i 的声明，将下面的循环转变为一个无限循环。这个循环不需要使用任何 5.0 版的特性：

```java
while (i != 0 && i == -i) {
}
```

一元减号操作符作用于 i，这意味着它的类型必须是数字型的：一元减号操作符作用于一个非数字
型操作数是非法的。因此，我们要寻找一个非 0 的数字型数值，它等于它自己的负值。NaN 不能满足这个属性，因为它不等于任何数值，因此，i 必须表示一个实际的数字。肯定没有任何数字满足这样的属性吗？

事实上，恰恰就有一个这样的 int 数值，它就是 Integer.MIN_VALUE，即-2^31。他的十六进制表示是 0x80000000。其符号位为 1，其余所有的位都是 0。如果我们对这个值取负值，那么我们将得到 0x7fffffff+1，也就是 0x80000000，即 Integer.MIN_VALUE！

换句话说，Integer.MIN_VALUE 是它自己的负值， Long.MIN_VALUE 也是一样。对这两个值取负值将会产生溢出，但是 Java 在整数计算中忽略了溢出。

### 6. 创建个数

下面将打印出已经创建的 Creature 实例的数量。那么，这个程序会打印出什么呢？

```java
public class Creator {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++)
            Creature creature = new Creature();
        System.out.println(Creature.numCreated());
    }
}
class Creature {
    private static long numCreated = 0;
    public Creature() {
        numCreated++;
    }
    public static long numCreated() {
        return numCreated;
    }
}
```

这是一个捉弄人的问题。该程序看起来似乎应该打印 100，但是它没有打印任何东西，因为它根本就不能编译。

Java 语言规范不允许一个本地变量声明语句作为一条语句在 for、while 或 do 循环中重复执行。一个本地变量声明作为一条语句只能直接出现在一个语句块中（一个语句块是由一对花括号以及包含在这对花括展中的语句和声明构成的）。

最显而易见的方式是将这个声明至于一个语句块中：

```java
for (int i = 0; i < 100; i++) {
    Creature creature = new Creature();
}
```

### 7. 大问题

我们来测试一下你对 BigInteger 的了解程度。下面这个程序将打印出什么呢？

```java
import java.math.BigInteger;
public class BigProblem {
    public static void main(String[ ] args) {
        BigInteger fiveThousand = new BigInteger("5000");
        BigInteger fiftyThousand = new BigInteger("50000");
        BigInteger fiveHundredThousand = new BigInteger("500000");
        BigInteger total = BigInteger.ZERO;
        total.add(fiveThousand);
        total.add(fiftyThousand);
        total.add(fiveHundredThousand);
        System.out.println(total);
    }
}
```

你可能会认为这个程序会打印出 555000。如果你运行该程序，你就会发现它打印的不是 555000，而是 0。很明显，所有这些加法对 total 没有产生任何影响。

对此有一个很好理由可以解释：BigInteger 实例是不可变的。

为了在一个包含对不可变对象引用的变量上执行计算，我们需要将计算的结果赋值给该变量。这样做就会产生下面的程序，它将打印出我们所期望的 555000：

```java
import java.math.BigInteger;
public class BigProblem {
    public static void main(String[] args) {
        BigInteger fiveThousand = new BigInteger("5000");
        BigInteger fiftyThousand = new BigInteger("50000");
        BigInteger fiveHundredThousand = new BigInteger("500000");
        BigInteger total = BigInteger.ZERO;
        total = total.add(fiveThousand);
        total = total.add(fiftyThousand);
        total = total.add(fiveHundredThousand);
        System.out.println(total);
    }
}
```

公正地说，Java 不可变类型的某些方法名促使我们走上了歧途。像 add、subtract 和 negate 之类的名字似乎是在暗示这些方法将修改它们所调用的实例。也许 plus、minus 和 negation 才是更好的名字。

对 API 设计来说，其教训是：在命名不可变类型的方法时，应该优选介词和名词，而不是动词。介词适用于带有参数的方法，而名词适用于不带参数的方法。

### 8. 什么是差

下面的程序在计算一个 int 数组中的元素两两之间的差，将这些差置于一个集合中，然后打印该集合的尺寸大小。那么，这个程序将打印出什么呢？

```java
import java.util.*;
public class Differences {
    public static void main(String[] args) {
        int vals[] = {234, 123, 012};
        Set diffs = new HashSet();
        for (int i = 0; i < vals.length; i++) {
            for (int j = i; j < vals.length; j++) {
                diffs.add(vals[i] - vals[j]);
            }
        }
        System.out.println(diffs.size());
        System.err.println(diffs);
        System.err.println(Arrays.toString(vals));
    }
}
```

如果你观察该数组的声明，不可能很清楚地发现原因所在：

```java
int vals[ ] = { 234, 123, 012 };
```

但是如果你打印数组的内容，你就会看见下面的内容：

```java
[234,123,10]
```

为什么数组中的最后一个元素是 10 而不是 12 呢？因为以 0 开头的整数类型字面常量将被解释成为八进制数值。这个隐晦的结构是从 C 编程语言那里遗留下来东西，C 语言产生于 1970 年代，那时八进制比现在要通用得多。

本谜题的教训很简单：千万不要在一个整型字面常量的前面加上一个 0；这会使它变成一个八进制字面常量。

### 9. 排序

```java
Comparator<Integer> cmp = new Comparator<Integer>() {
    public int compare(Integer i1, Integer i2) {
        return i2 - i1;
    }
};
```

本谜题也许应该称为“白痴一般的惯用法的案例”。这种惯用法的问题在于定长的整数没有大到可以保存任意两个同等长度的整数之差的程度。当你在做两个 int 或 long 数值的减法时，其结果可能会溢出，在这种情况下我们就会得到错误的符号。

例如，请考虑下面的程序：

```java
public class Overflow {
    public static void main(String[] args){
        int x = -2000000000;
        int z = 2000000000;
        System.out.println(x - z);
    }
}
```

很明显，x 比 z 小，但是程序打印的是 294967296，它是一个正数。既然这种比较的惯用法是有问题的，那么为什么它会被如此广泛地应用呢？因为它在大多数时间里可以正常工作的。它只在用来进行比较的两个数字的差大于 Integer.MAX_VALUE 的时候才会出问题。这意味着对于许多应用而言，在实际使用中是不会看到这种错误的。更糟的是，它们被观察到的次数少之又少，以至于这个 bug 永远都不会被发现和订正。

本谜题有数个教训，最具体的是：不要使用基于减法的比较器，除非你能够确保要比较的数值之间的差永远不会大于 Integer.MAX_VALUE 。

### 10. 名字重用的术语表

（1）override
一个实例方法可以覆写（override）在其超类中可访问到的具有相同签名的所有实例方法，从而使能了动态分派（dynamic dispatch）；换句话说，VM 将基于实例的运行期类型来选择要调用的覆写方法。覆写是面向对象编程技术的基础，并且是唯一没有被普遍劝阻的名字重用形式：

```java
class Base {
    public void f() { }
}
class Derived extends Base {
    public void f() { } // overrides Base.f()
}
```

（2）hide
一个域、静态方法或成员类型可以分别隐藏（hide）在其超类中可访问到的具有相同名字（对方法而言就是相同的方法签名）的所有域、静态方法或成员类型。隐藏一个成员将阻止其被继承：

```java
class Base {
    public static void f() { }
}
class Derived extends Base {
    private static void f() { } // hides Base.f()
}
```

（3）overload
在某个类中的方法可以重载（overload）另一个方法，只要它们具有相同的名字和不同的签
名。由调用所指定的重载方法是在编译期选定的：

```java
class CircuitBreaker {
    public void f(int i) { } // int overloading
    public void f(String s) { } // String overloading
}
```

（4）shadow
一个变量、方法或类型可以分别遮蔽（shadow）在一个闭合的文本范围内的具有相同名字的所有变量、方法或类型。如果一个实体被遮蔽了，那么你用它的简单名是无法引用到它的；根据实体的不同，有时你根本就无法引用到它：

```java
class WhoKnows {
    static String sentence = "I don't know.";
    public static void main(String[] args) {
        String sentence = "I know!"; // shadows static field
        System.out.println(sentence); // prints local variable
    }
}
```

尽管遮蔽通常是被劝阻的，但是有一种通用的惯用法确实涉及遮蔽。构造器经常将来自其所在类的某个域名重用为一个参数，以传递这个命名域的值。这种惯用法并不是没有风险，但是大多数 Java 程序员都认为这种风格带来的实惠要超过其风险：

```java
class Belt {
    private final int size;
    public Belt(int size) { // Parameter shadows Belt.size
        this.size = size;
    }
}
```

（5）obscure
一个变量可以遮掩具有相同名字的一个类型，只要它们都在同一个范围内：如果这个名字被用于变量与类型都被许可的范围，那么它将引用到变量上。相似地，一个变量或一个类型可以遮掩一个包。

遮掩是唯一一种两个名字位于不同的名字空间的名字重用形式，这些名字空间包括：变量、包、方法或类型。

如果一个类型或一个包被遮掩了，那么你不能通过其简单名引用到它，除非是在这样一个上下文环境中，即语法只允许在其名字空间中出现一种名字。

遵守命名习惯就可以极大地消除产生遮掩的可能性：

```java
public class Obscure {
    static String System; // Obscures type java.lang.System
    public static void main(String[] args) {
        // Next line won't compile: System refers to static field
        System.out.println("hello, obscure world!");
    }
}
```

### 11. 有毒的括号垃圾

你能否举出这样一个合法的 Java 表达式，只要对它的某个子表达式加上括号就可以使其成为不合法的表达式，而添加的括号只是为了注解未加括号时赋值的顺序？

插入一对用来注解现有赋值顺序的括号对程序的合法性似乎是应该没有任何影响的。事实上，绝大多数情况下确实是没有影响的。但是，在两种情况下，插入一对看上去没有影响的括号可能会令合法的 Java 程序变得不合法。这种奇怪的情况是由于数值的二进制补码的不对称性引起的。

符号-2147483648 构成了一个合法的 Java 表达式，它由一元负操作符加上一个 int 型字面常量 2147483648 组成。通过添加一对括号来注解（很不重要的）赋值顺序，即写成-（2147483648），就会破坏这条规则。

### 12. 紧张的关系

在数学中，等号（＝）定义了一种真实的数之间的等价关系（equivalence relation）。

操作符 == 不是自反的，因为表达式 ( Double.NaN == Double.NaN ）值为 false，表达式（ Float.NaN == Float.NaN ）也是如此。

但是操作符 == 是否还违反了对称性和传递性呢？事实上它并不违反对称性：对于所有 x 和 y 的值，（ x == y ）意味着（ y == x ）。

传递性则 完全是另一回事。当比较两个原始类型数值时，操作符 == 首先进行二进制数据类型提升（binary numeric promotion）。这会导致这两个数值中有一个会进行拓宽原始类型转换（widening primitive conversion）。大部分拓宽原始类型转换是不会有问题的，但有三个值得注意的异常情况：将 int 或 long 值转换成 float 值，或 long 值转换成 double 值时，均会导致精度丢失。这种精度丢失可以证明 == 操作符的不可传递性。

实现这种不可传递性的窍门就是利用上述三种数值比较中的两种去丢失精度，然后就可以得到与事实相反的结果。可以这样构造例子：选择两个较大的但不相同的 long 型数值赋给 x 和 z，将一个与前面两个 long 型数值相近的 double 型数值赋给 y。下面的程序就是其代码，它打印的结果是 true true false，这显然证明了操作符 == 作用于原始类型时具有不可传递性。

```java
public class Transitive {
    public static void main(String[] args) throws Exception {
        long x = Long.MAX_VALUE;
        double y = (double) Long.MAX_VALUE;
        long z = Long.MAX_VALUE - 1;
        System.out.print((x == y) + ""); // Imprecise!
        System.out.print((y == z) + ""); // Imprecise!
        System.out.println(x == z); // Precise!
    }
}
```

### 13. 最后的甜点

```java
public class ApplePie {
    public static void main(String[] args) {
        int count = 0;
        for (int i = 0; i < 100; i++);
        {
            count++;
        }
        System.out.println(count);
    }
}

```

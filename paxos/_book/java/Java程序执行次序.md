### 深入了解Java程序执行顺序
来自：https://www.cnblogs.com/greatfish/p/5771548.html

### 测试程序
```java
public class Text {

    public static int k = 0;
    public static Text t1 = new Text("t1");
    public static Text t2 = new Text("t2");
    public static int i = print("i");
    public static int n = 99;
    public int j = print("j");

    {
        print("构造块");
    }
    static {
        print("静态块");
    }

    public Text(String str) {

        System.out.println((++k) + ":" + str + "   i=" + i + "    n=" + n);
        ++i;
        ++n;
    }

    public static int print(String str) {

        System.out.println((++k) + ":" + str + "   i=" + i + "    n=" + n);
        ++n;
        return ++i;
    }

    public static void main(String args[]) {

        Text t = new Text("init");
    }
}
```

### 测试结果
建议每行打断点验证。
```
1:j   i=0    n=0
2:构造块   i=1    n=1
3:t1   i=2    n=2
4:j   i=3    n=3
5:构造块   i=4    n=4
6:t2   i=5    n=5
7:i   i=6    n=6
8:静态块   i=7    n=99
9:j   i=8    n=100
10:构造块   i=9    n=101
11:init   i=10    n=102

```

### 结论

+ 1-3：类加载过程，不涉及构造方法
+ 4-5: 实例化过程，涉及构造方法

#### 1. 类中所有属性的默认值（一举而成）

#### 2. 父类静态属性初始化，静态块，静态方法的声明（按出现顺序执行）

#### 3. 子类静态属性初始化，静态块，静态方法的声明 （按出现顺序执行）

#### 4. 调用父类的构造方法，首先父类的非静态成员初始化，构造块，普通方法的声明（按出现顺序执行），然后父类构造方法

#### 5. 调用子类的构造方法，首先子类的非静态成员初始化，构造块，普通方法的声明（按出现顺序执行），然后子类构造方法


注意：类加载过程中，可能调用了实例化过程（因为static可以修饰方法，属性，代码块，内部类），此时则会暂停类加载过程而先执行实例化过程（被打断），执行结束再进行类加载过程，上面阿里那道面试题就是典型的暂停类加载。

### 扩展阅读
#### 1. Java语言规范（The Java® Language Specification Java SE 8 Edition）
https://docs.oracle.com/javase/specs/jls/se8/html/

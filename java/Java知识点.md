### NoClassDefFoundError

NoClassDefFoundError 错误的发生，是因为 Java 虚拟机在编译时能找到合适的类，而在运行时不能找到合适的类导致的错误。与 ClassNotFoundException 的不同在于，这个错误发生只在运行时需要加载对应的类不成功，而不是编译时发生。

NoClassDefFoundError 发生在 JVM 在动态运行时，根据你提供的类名，在 classpath 中找到对应的类进行加载，但当它找不到这个类时，就发生了 java.lang.NoClassDefFoundError 的错误，而 ClassNotFoundException 是在编译的时候在 classpath 中找不到对应的类而发生的错误。

### finally

不要在 finally 中使用 return 语句。

finally 总是执行，除非程序或者线程被中断。

### 静态内部类

内部类如果不是 static，它本身对外面那个类有引用关系，这一点其实从构造阶段就能看出来，你可以写段代码试试；有强引用就是 strong reachable 状态

### 4 引用

不同的引用类型，主要体现的是对象不同的可达性（reachable）状态和对垃圾收集的影响

### LRU 与 LinkedHashMap

LinkedHashMap 在构造函数中可以指定链接方式：一种是按照插入的顺序组织成双链表；一种是根据访问的顺序组织成双链表，维持链表头 head 是最长时间没有访问的结点。很显然，实现 LRU 用到的应该是后一种方式

### 并发编程“串行的故事”

起源是一个硬件的核心矛盾：CPU 与内存、I/O 的速度差异，出现了 CPU 缓存，系统软件（操作系统、编译器）在解决这个核心矛盾的同时，引入了可见性、原子性和有序性问题；

简言之就是，导致可见性的原因是 CPU 缓存，导致原子性问题的原因是线程切换，导致有序性的原因是编译优化。

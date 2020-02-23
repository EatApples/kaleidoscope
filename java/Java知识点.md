### NoClassDefFoundError

NoClassDefFoundError 错误的发生，是因为 Java 虚拟机在编译时能找到合适的类，而在运行时不能找到合适的类导致的错误。与 ClassNotFoundException 的不同在于，这个错误发生只在运行时需要加载对应的类不成功，而不是编译时发生。

NoClassDefFoundError 发生在 JVM 在动态运行时，根据你提供的类名，在 classpath 中找到对应的类进行加载，但当它找不到这个类时，就发生了 java.lang.NoClassDefFoundError 的错误，而 ClassNotFoundException 是在编译的时候在 classpath 中找不到对应的类而发生的错误。

### finally

不要在 finally 中使用 return 语句。

finally 总是执行，除非程序或者线程被中断。

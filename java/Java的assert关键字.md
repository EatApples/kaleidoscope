### java 的 assert 关键字
#### Throwable
在 Java 语言规范中，所有异常都是 Throwable 类或其子类的实例。

Throwable 有两个重要的子类：Exception（异常）和 Error（错误）。而 Exception 又分为 RuntimeException 和其他类型。

Exception 和 Error 的区别是：Exception 涵盖程序可能需要捕获并处理的异常；Error 涵盖不应捕获的异常。

RuntimeException 与 Error 属于 Java 里的非检查异常，其他异常则属于检查异常。

简单理解：

（1）检查异常：程序编译时就知道了，不处理不行；

（2）RuntimeException：程序运行时抛出，有点意外，处不处理都可以（建议关键代码 catch Exception）；

（3）Error：鬼知道程序发生了什么？不知道怎么处理，也不应该处理。

注意：即使有这些区分，其实 catch 能捕获 Throwable！！！

#### assert

平时的java开发中几乎不会用到assert关键字，原因：

（1）assert需要显示的开启生效才有作用 VM参数 -ea（开启） -da（关闭，默认）

（2）assert断言失败将抛出 AssertionError（注意，这是Error，一般不会被捕获）

（3）即使处理了Error，如果断言的代码嵌入到业务流程中，一旦assert失效，也就改变了正常的业务逻辑(如果condition中加入了赋值等操作)

即便如此，JDK中还是存在不少使用assert的地方，比方说java.util.Locale类等（注意：JDK的事，雨女无瓜）
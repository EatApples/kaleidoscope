### UndeclaredThrowableException

为啥会有 UndeclaredThrowableException 异常

If a checked exception is thrown by this method that is not assignable to any of the exception types declared in the throws clause of the interface method, then an UndeclaredThrowableException containing the exception that was thrown by this method will be thrown by the method invocation on the proxy instance.

我们的异常处理类，实际是动态代理的一个实现。

如果一个异常是检查型异常并且没有在动态代理的接口处声明，那么它将会被包装成 UndeclaredThrowableException。
而我们定义的自定义异常，被定义成了检查型异常，导致被包装成了 UndeclaredThrowableException

解决方案：

知道原因就很简单了。要么抛 java.lang.RuntimeException or java.lang.Error 非检查性异常, 要么接口要声明异常。

这里选择修改 自定义异常为运行时异常即可

作者：有时右逝
链接：https://www.jianshu.com/p/7edab536e4b9
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

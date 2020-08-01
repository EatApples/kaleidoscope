### Java NIO 早期版本中的 Epoll 空转问题，以及 Netty 的解决方式？

https://blog.csdn.net/hchaoh/article/details/103907256

Java nio 空轮询 bug 也就是 Java nio 在 Linux 系统下的 epoll 空轮询问题

部分 Linux 内核中，在 poll 或 epoll 一个已连接的 socket，且请求事件掩码为 0 的情况下，如果连接被突然中断（连接出现了 RST），那么 poll/epoll 会被唤醒，相应事件标识为 POLLHUP（或 POLLERR）。继而 Selector 被唤醒，且 interest set 为 0，没有相应的 Channel，select() 返回值也是 0。进而导致 CPU 100%问题。

可能因为此问题的根源在于底层 Linux 内核行为的不一致，所以 Java 官方一开始将其抛给了操作系统实现方，导致该 bug 存在了很久。

该问题最早在 Java 6 发现，随后很多版本声称解决了该问题，但实际上只是降低了该 bug 的出现频率，目前从网上搜索到的资料显示，Java 8 还是存在该问题（当 Thrift 遇到 JDK Epoll Bug）。

最后一起来分析下，nio epoll bug 不是 linux epoll 的问题，而是 JDK 自己实现 epoll 时没有考虑这种情况，或者说因为其他系统不存在这个问题，Java 为了封装（比如 SelectionKey 中的 4 个事件类型）的统一而没去处理？

比如 netty 就很巧妙的规避了这个问题，它的处理机制就是如果发生了这种情况，并且发生次数超过了 SELECTOR_AUTO_REBUILD_THRESHOLD（默认 512），则调用 rebuildSelector()进行 Selector 重建，这样就不用管之前发生了异常情况的那个连接了。因为重建也是根据 SelectionKey 事件对应的连接来重新注册的。

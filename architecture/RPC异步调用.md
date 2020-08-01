# RPC 调用

### 1. 协议约定问题

通信协议：Thrift、HTTP、Dubbo、HTTP/2 等

序列化格式：JSON、XML、ProtoBuf 等

解决：

（1）如何规定远程调用的语法
（2）如何传递参数
（3）如何表示数据

### 2. 传输问题

通信框架：Netty、Socket 等

解决网络传输过程中的错误、重传、丢包、性能问题

### 3. 服务发现问题

RPC 客户端组件

（1）序列化组件
（2）连接池组件

以下是异步 RPC 必须：

（3）收发队列
（4）收发线程
（5）上下文管理器，存储<请求-响应-回调>等映射信息
（6）超时管理器，超时后直接回调，进入失败处理

### 举例：

Dubbo 通过注册中心 Zookeeper 解决服务发现问题，通过 Hessian2 序列化解决协议约定的问题，通过 Netty 解决网络传输的问题。

Spring Cloud 通过注册中心 Eureka 解决服务发现问题，通过 JSON 解决协议约定的问题，通过 HTTP 解决网络传输的问题。

### 资料来源

#### 1. 第 32 讲 | RPC 协议综述：远在天边，近在眼前

https://time.geekbang.org/column/article/12230

#### 2. 离不开的微服务架构，脱不开的 RPC 细节（值得收藏）！！！

https://mp.weixin.qq.com/s/CepNdL2A_QMcESgQVZBn2g

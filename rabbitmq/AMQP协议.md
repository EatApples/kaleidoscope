### AMQP 分层
AMQP，即 Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。因此AMQP模型描述了一套模块化的组件以及这些组件之间进行连接的标准规则。

AMQP协议是一种二进制协议，提供客户端应用与消息中间件之间异步、安全、高效地交互。从整体来看，AMQP协议可划分为三层：Model层，Session层，Transport层。

+ 模型层：定义了一套命令（按功能分类），客户端应用可以利用这些命令来实现它的业务功能。

+ 会话层：负责将命令从客户端应用传递给服务器，再将服务器的应答传递给客户端应用，会话层为这个传递过程提供可靠性、同步机制和错误处理。

+ 传输层：提供帧处理、信道复用、错误检测和数据表示。

实现者可以将传输层替换成任意传输协议，只要不改变 AMQP 协议中与客户端应用程序相关的功能。实现者还可以使用其他高层协议中的会话层。

这种分层架构类似于OSI网络协议，可替换各层实现而不影响与其它层的交互。AMQP定义了合适的服务器端域模型，用于规范服务器的行为(AMQP服务器端可称为broker)。在这里Model层决定这些基本域模型所产生的行为，这种行为在AMQP中用”command”表示，在后文中会着重来分析这些域模型。Session层定义客户端与broker之间的通信(通信双方都是一个peer，可互称做partner)，为command的可靠传输提供保障。Transport层专注于数据传送，并与Session保持交互，接受上层的数据，组装成二进制流，传送到receiver后再解析数据，交付给Session层。Session层需要Transport层完成网络异常情况的汇报，顺序传送command等工作。


### AMQP 术语定义
+ AMQP模型（AMQP Model）：一个由关键实体和语义表示的逻辑框架，遵从AMQP规范的服务器必须提供这些实体和语义。为了实现本规范中定义的语义，客户端可以发送命令来控制AMQP服务器。

+ 连接（Connection）：一个网络连接，比如TCP/IP套接字连接。

+ 会话（Session）：端点之间的命名对话。在一个会话上下文中，保证“恰好传递一次”。

+ 信道（Channel）：多路复用连接中的一条独立的双向数据流通道。为会话提供物理传输介质。

+ 客户端（Client）：AMQP连接或者会话的发起者。AMQP是非对称的，客户端生产和消费消息，服务器存储和路由这些消息。

+ 服务器（Server）：接受客户端连接，实现AMQP消息队列和路由功能的进程。也称为“消息代理”。

+ 端点（Peer）：AMQP对话的任意一方。一个AMQP连接包括两个端点（一个是客户端，一个是服务器）。

+ 搭档（Partner）：当描述两个端点之间的交互过程时，使用术语“搭档”来表示“另一个”端点的简记法。比如我们定义端点A和端点B，当它们进行通信时，端点B是端点A的搭档，端点A是端点B的搭档。

+ 片段集（Assembly）：段的有序集合，形成一个逻辑工作单元。

+ 段（Segment）：帧的有序集合，形成片段集中一个完整子单元。

+ 帧（Frame）：AMQP传输的一个原子单元。一个帧是一个段中的任意分片。

+ 控制（Control）：单向指令，AMQP规范假设这些指令的传输是不可靠的。

+ 命令（Command）：需要确认的指令，AMQP规范规定这些指令的传输是可靠的。

+ 异常（Exception）：在执行一个或者多个命令时可能发生的错误状态。

+ 类（Class）：一批用来描述某种特定功能的AMQP命令或者控制。

+ 消息头（Header）：描述消息数据属性的一种特殊段。

+ 消息体（Body）：包含应用程序数据的一种特殊段。消息体段对于服务器来说完全不透明——服务器不能查看或者修改消息体。

+ 消息内容（Content）：包含在消息体段中的的消息数据。

+ 交换器（Exchange）：服务器中的实体，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

+ 交换器类型（Exchange Type）：基于不同路由语义的交换器类。

+ 消息队列（Message Queue）：一个命名实体，用来保存消息直到发送给消费者。

+ 绑定器（Binding）：消息队列和交换器之间的关联。

+ 绑定器关键字（Binding Key）：绑定的名称。一些交换器类型可能使用这个名称作为定义绑定器路由行为的模式。

+ 路由关键字（Routing Key）：一个消息头，交换器可以用这个消息头决定如何路由某条消息。

+ 持久存储（Durable）：一种服务器资源，当服务器重启时，保存的消息数据不会丢失。

+ 临时存储（Transient）：一种服务器资源，当服务器重启时，保存的消息数据会丢失。

+ 持久化（Persistent）：服务器将消息保存在可靠磁盘存储中，当服务器重启时，消息不会丢失。

+ 非持久化（Non-Persistent）：服务器将消息保存在内存中，当服务器重启时，消息可能丢失。

+ 消费者（Consumer）：一个从消息队列中请求消息的客户端应用程序。

+ 生产者（Producer）：一个向交换器发布消息的客户端应用程序。

+ 虚拟主机（Virtual Host）：一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。客户端应用程序在登录到服务器之后，可以选择一个虚拟主机。

### AMQP 功能模块
在服务器中，三个主要功能模块连接成一个处理链完成预期的功能：

+ exchange：接收发布应用程序发送的消息，并根据一定的规则将这些消息路由到“消息队列”。

+ message queue：存储消息，直到这些消息被消费者安全处理完为止。

+ binding：定义了 exchange 和 message queue 之间的关联，提供路由规则。

使用这个模型我们可以很容易的模拟出存储转发队列和主题订阅这些典型的消息中间件概念。

一个 AMQP 服务器类似于邮件服务器，exchage 类似于消息传输代理（email 里的概念），message queue 类似于邮箱。Binding 定义了每一个传输代理中的消息路由表，发布者将消息发给特定的传输代理，然后传输代理将这些消息路由到邮箱中，消费者从这些邮箱中取出消息。

在以前的中间件系统的应用场景中，发布者直接将消息发送给邮箱或者邮件列表。

区别就在于用户可以控制 message queue 与 exchage 的连接规则，这可以做很多有趣的事情，比如定义一条规则：“将所有包含这样这样的消息头的消息都复制一份再发送到消息队列中”。

支持各种消息交换的体系结构：

+ 存储转发（多个消息发送者，单个消息接收者）。

+ 分布式事务（多个消息发送者，多个消息接收者）。

+ 发布订阅（多个消息发送者，多个消息接收者）。

+ 基于内容的路由（多个消息发送者，多个消息接收者）。

+ 文件传输队列（多个消息发送者，多个消息接收者）。

+ 点对点连接（单个消息发送者，单个消息接收者）

### 域模型
上面是对AMQP协议的大致说明。下面会以我们对消息服务的需求来理解AMQP所提供的域模型。消息中间件的主要功能是消息的路由(Routing)和缓存(Buffering)。在AMQP中提供类似功能的两种域模型：Exchange 和 Message queue。

Exchange 接收消息生产者(Message Producer)发送的消息根据不同的路由算法将消息发送往Message queue。Message queue 会在消息不能被正常消费时缓存这些消息，具体的缓存策略由实现者决定，当 message queue 与消息消费者(Message consumer)之间的连接通畅时，Message queue 有将消息转发到consumer的责任。

Message 是当前模型中所操纵的基本单位，它由 Producer 产生，经过 Broker 被 Consumer 所消费。它的基本结构有两部分: Header 和Body。Header 是由 Producer 添加上的各种属性的集合，这些属性有控制 Message 是否可被缓存，接收的 queue 是哪个，优先级是多少等。Body 是真正需要传送的数据，它是对 Broker 不可见的二进制数据流，在传输过程中不应该受到影响。

 一个 broker 中会存在多个Message queue，Exchange 怎样知道它要把消息发送到哪个Message queue中去呢? 这就是Binding的作用。Message queue 的创建是由 client application 控制的，在创建 Message queue 后需要确定它来接收并保存哪个 Exchange 路由的结果。Binding 是用来关联 Exchange 与 Message queue 的域模型。Client application 控制 Exchange 与某个特定 Message queue 关联，并将这个 queue 接受哪种消息的条件绑定到 Exchange，这个条件也叫 Binding key 或是 Criteria。

在与多个 Message queue 关联后，Exchange 中就会存在一个路由表，这个表中存储着每个 Message queue 所需要消息的限制条件。Exchange 就会检查它接受到的每个 Message的 Header 及 Body 信息，来决定将Message路由到哪个queue中去。Message的Header中应该有个属性叫 Routing Key，它由 Message 发送者产生，提供给 Exchange 路由这条Message的标准。Exchange根据不同路由算法有不同有Exchange Type。比如有Direct类似，需要Binding key等于Routing key；也有Binding key与Routing key符合一个模式关系；也有根据Message包含的某些属性来判断。一些基础的路由算法由AMQP所提供，client application也可以自定义各种自己的扩展路由算法。

一个Virtual Host可持有一些Exchange和Message queue。它是一个虚拟概念，一个Virtual Host可以是一台服务器，也可以是由多台服务器组成的集群。同步扩展下，Exchange与Message queue的部署也可以是一台或是多台服务器上。 Message的产生者和消费者可能是同一个应用。整个AMQP定义的就是Client application与Broker之间的交互。在粗略介绍完AMQP的域模型后，可以关注下Client是怎样与Broker建立起连接的。

在AMQP中，Client application想要与Broker沟通，就需要建立起与Broker的connection，这种connection其实是与Virtual Host相关联的，也就是说，connection是建立在client与Virtual Host之间。可以在一个connection上并发运行多个channel，每个channel执行与Broker的通信，我们前面提供的session就是依附于channel上的。

这里的Session可以有多种定义，既可以表示AMQP内部提供的command分发机制，也可以说是在宏观上区别与域模型的接口。正常理解就是我们平时所说的交互context，主要作用就是在网络上可靠地传递每一个command。在AMQP的设计中，应当是借鉴了TCP的各种设计，用于保证这种可靠性。

在Session层，为上层所需要交互的每个command分配一个惟一标识符(可以是一个UUID)，是为了在传输过程中可以对command做校验和重传。Command发送端也需要记录每个发送出去的command到Replay Buffer，以期得到接收方的回馈，保证这个command被接收方明确地接收或是已执行这个command。对于超时没有收到反馈的command，发送方再次重传。如果接收方已明确地回馈信息想要告知command发送方但这条信息在中途丢失或是其它问题发送方没有收到，那么发送方不断重传会对接收方产生影响，为了降低这种影响，command接收方设置一个过滤器Idempotency Barrier，来拦截那些已接收过的command。 关于这种重传及确认机制，可以参考下TCP的相关设计。

上面大致介绍了AMQP的域模型及连接机制中的确认及重传模型，不涉及AMQP的详细二进制规范。

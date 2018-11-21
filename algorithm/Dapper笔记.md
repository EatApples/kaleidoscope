### 概述
Dapper：Google生产环境下的分布式跟踪系统，满足低损耗、应用透明的、大范围部署这三个需求。

### 介绍
对 Dapper 我们只有两点要求：无所不在的部署，持续的监控。根据这两个明确的需求，我们可以直接推出三个具体的设计目标：

1.低消耗：跟踪系统对在线服务的影响应该做到足够小。在一些高度优化过的服务，即使一点点损耗也会很容易察觉到，而且有可能迫使在线服务的部署团队不得不将跟踪系统关停。

2.应用级的透明：对于应用的程序员来说，是不需要知道有跟踪系统这回事的。如果一个跟踪系统想生效，就必须需要依赖应用的开发者主动配合，那么这个跟踪系统也太脆弱了，往往由于跟踪系统在应用中植入代码的bug或疏忽导致应用出问题，这样才是无法满足对跟踪系统“无所不在的部署”这个需求。面对当下想Google这样的快节奏的开发环境来说，尤其重要。

3.延展性：Google至少在未来几年的服务和集群的规模，监控系统都应该能完全把控住。

做到真正的应用级别的透明，这应该是当下面临的最挑战性的设计目标，我们把核心跟踪代码做的很轻巧，然后把它植入到那些无所不在的公共组件种，比如线程调用、控制流以及RPC库。

虽然单独使用Dapper有时就足够让开发人员查明异常的来源，但是Dapper的初衷不是要取代所有其他监控的工具。我们发现，Dapper的数据往往侧重性能方面的调查，所以其他监控工具也有他们各自的用处。

### 文献的总结
分布式系统跟踪工具的设计空间已经被一些优秀文章探索过了，其中的Pinpoint[9]、Magpie[3]和X-Trace[12]和Dapper最为相近。

虽然Dapper在许多高阶的设计思想上吸取了Pinpoint和Magpie的研究成果，但在分布式跟踪这个领域中，Dapper的实现包含了许多新的贡献。例如，我们想实现低损耗的话，特别是在高度优化的而且趋于极端延迟敏感的Web服务中，采样率是很必要的。或许更令人惊讶的是，我们发现即便是1/1000的采样率，对于跟踪数据的通用使用层面上，也可以提供足够多的信息。

我们的系统的另一个重要的特征，就是我们能实现的应用级的透明。我们的组件对应用的侵入被先限制在足够低的水平上，即使想Google网页搜索这么大规模的分布式系统，也可以直接进行跟踪而无需加入额外的标注(Annotation)。虽然由于我们的部署系统有幸是一定程度的同质化的，所以更容易做到对应用层的透明这点，但是我们证明了这是实现这种程度的透明性的充分条件。

### Dapper的分布式跟踪
为了将所有记录条目与一个给定的发起者（例如，图1中的RequestX）关联上并记录所有信息，现在有两种解决方案，黑盒(black-box)和基于标注(annotation-based)的监控方案。黑盒方案[1，15，2]假定需要跟踪的除了上述信息之外没有额外的信息，这样使用统计回归技术来推断两者之间的关系。基于标注的方案[3，12，9，16]依赖于应用程序或中间件明确地标记一个全局ID，从而连接每一条记录和发起者的请求。虽然黑盒方案比标注方案更轻便，他们需要更多的数据，以获得足够的精度，因为他们依赖于统计推论。基于标注的方案最主要的缺点是，很明显，需要代码植入。在我们的生产环境中，因为所有的应用程序都使用相同的线程模型，控制流和RPC系统，我们发现，可以把代码植入限制在一个很小的通用组件库中，从而实现了监测系统的应用对开发人员是有效地透明。

### 跟踪树和span
在Dapper跟踪树结构中，树节点是整个架构的基本单元，而每一个节点又是对span的引用。节点之间的连线表示的span和它的父span直接的关系。

### 植入点
Dapper可以以对应用开发者近乎零浸入的成本对分布式控制路径进行跟踪，几乎完全依赖于基于少量通用组件库的改造。如下：

当一个线程在处理跟踪控制路径的过程中，Dapper把这次跟踪的上下文的在ThreadLocal中进行存储。追踪上下文是一个小而且容易复制的容器，其中承载了Scan的属性比如跟踪ID和span ID。

当计算过程是延迟调用的或是异步的，大多数Google开发者通过线程池或其他执行器，使用一个通用的控制流库来回调。Dapper确保所有这样的回调可以存储这次跟踪的上下文，而当回调函数被触发时，这次跟踪的上下文会与适当的线程关联上。在这种方式下，Dapper可以使用trace ID和span ID来辅助构建异步调用的路径。

几乎所有的Google的进程间通信是建立在一个用C++和Java开发的RPC框架上。我们把跟踪植入该框架来定义RPC中所有的span。span的ID和跟踪的ID会从客户端发送到服务端。像那样的基于RPC的系统被广泛使用在Google中，这是一个重要的植入点。当那些非RPC通信框架发展成熟并找到了自己的用户群之后，我们会计划对RPC通信框架进行植入。

### 采样率
除了把Dapper的收集工作对基本组件的性能损耗限制的尽可能小之外，我们还有进一步控制损耗的办法，那就是遇到大量请求时只记录其中的一小部分。

考虑到生产环境的安全，Dapper的跟踪也可以关闭。

我们在部署可变采样的过程中，参数化配置采样率时，不是使用一个统一的采样方案，而是使用一个采样期望率来标识单位时间内采样的追踪。这样一来，低流量低负载自动提高采样率，而在高流量高负载的情况下会降低采样率，使损耗一直保持在控制之下。实际使用的采样率会随着跟踪本身记录下来，这有利于从Dapper的跟踪数据中准确的分析。

### 经验
Google维护了一个从运行进程中不断收集并集中异常信息报告的服务。

Dapper能够验证对于搜索请求的关键路径。当一个系统不仅涉及数个子系统，而是几十个开发团队的涉及到的系统的情况下，端到端性能较差的根本原因到底在哪，这个问题即使是我们最好的和最有经验的工程师也无法正确回答。在这种情况下，Dapper可以提供急需的数据，而且可以对许多重要的性能问题得出结论。

Dapper核心组件与Dapper跟踪Annotation一并使用的情况下，“Service Dependencies”项目能够推算出任务各自之间的依赖，以及任务和其他软件组件之间的依赖。

虽然Dapper不是设计用来做链路级的监控的，但是我们发现，它是非常适合去做集群之间网络活动性的应用级任务的分析。

下面我们将介绍一些我们已知的最重要的Dapper的不足：

合并的影响：我们的模型隐含的前提是不同的子系统在处理的都是来自同一个被跟踪的请求。

跟踪批处理负载：Dapper的设计，主要是针对在线服务系统，最初的目标是了解一个用户请求产生的系统行为。

寻找根源：Dapper可以有效地确定系统中的哪一部分致使系统整个速度变慢，但并不总是能够找出问题的根源。

记录内核级的信息：一些内核可见的事件的详细信息有时对确定问题根源是很有用的。

### 总结
我们相信，Dapper比以前的基于Annotation的分布式跟踪达到更高的应用透明度，这一点已经通过只需要少量人工干预的工作量得以证明。虽然一定程度上得益于我们的系统的同质性，但它本身仍然是一个重大的挑战。最重要的是，我们的设计提出了一些实现应用级透明性的充分条件，对此我们希望能够对更错杂环境下的解决方案的开发有所帮助。

### References
[1] M. K. Aguilera, J. C. Mogul, J. L. Wiener, P. Reynolds, and A. Muthitacharoen. Performance Debugging for Distributed Systems of Black Boxes. In Proceedings of the 19th ACM Symposium on Operating Systems Principles, December 2003.

[2] P. Bahl, R. Chandra, A. Greenberg, S. Kandula, D. A. Maltz, and M. Zhang. Towards Highly Reliable Enterprise Network Services Via Inference of Multi-level Dependencies. In Proceedings of SIGCOMM, 2007.

[3] P. Barham, R. Isaacs, R. Mortier, and D. Narayanan. Magpie: online modelling and performance-aware systems. In Proceedings of USENIX HotOS IX, 2003.

[4] L. A. Barroso, J. Dean, and U. Holzle. Web Search for a Planet: The Google Cluster Architecture. IEEE Micro, 23(2):22–28, March/April 2003.

[5] T. O. G. Blog. Developers, start your engines. http://googleblog.blogspot.com/2008/04/developers-start-your-engines.html,2007.

[6] T. O. G. Blog. Universal search: The best answer is still the best answer. http://googleblog.blogspot.com/2007/05/universal-search-best-answer-is-still.html, 2007.

[7] M. Burrows. The Chubby lock service for loosely-coupled distributed systems. In Proceedings of the 7th USENIX Symposium on Operating Systems Design and Implementation, pages 335 – 350, 2006.

[8] F. Chang, J. Dean, S. Ghemawat, W. C. Hsieh, D. A. Wallach, M. Burrows, T. Chandra, A. Fikes, and R. E. Gruber. Bigtable: A Distributed Storage System for Structured Data. In Proceedings of the 7th USENIX Symposium on Operating Systems Design and Implementation (OSDI’06), November 2006.

[9] M. Y. Chen, E. Kiciman, E. Fratkin, A. fox, and E. Brewer. Pinpoint: Problem Determination in Large, Dynamic Internet Services. In Proceedings of ACM International Conference on Dependable Systems and Networks, 2002.

[10] J. Dean and S. Ghemawat. MapReduce: Simplified Data Processing on Large Clusters. In Proceedings of the 6th USENIX Symposium on Operating Systems Design and Implementation (OSDI’04), pages 137 – 150, December 2004.

[11] J. Dean, J. E. Hicks, C. A. Waldspurger, W. E. Weihl, and G. Chrysos. ProfileMe: Hardware Support for Instruction-Level Profiling on Out-of-Order Processors. In Proceedings of the IEEE/ACM International Symposium on Microarchitecture, 1997.

[12] R. Fonseca, G. Porter, R. H. Katz, S. Shenker, and I. Stoica. X-Trace: A Pervasive Network Tracing Framework. In Proceedings of USENIX NSDI, 2007.

[13] B. Lee and K. Bourrillion. The Guice Project Home Page. http://code.google.com/p/google-guice/, 2007.

[14] P. Reynolds, C. Killian, J. L. Wiener, J. C. Mogul, M. A. Shah, and A. Vahdat. Pip: Detecting the Unexpected in Distributed Systems. In Proceedings of USENIX NSDI, 2006.

[15] P. Reynolds, J. L. Wiener, J. C. Mogul, M. K. Aguilera, and A. Vahdat. WAP5: Black Box Performance Debugging for Wide-Area Systems. In Proceedings of the 15th International World Wide Web Conference, 2006.

[16] P. K. G. T. Gschwind, K. Eshghi and K. Wurster. WebMon: A Performance Profiler for Web Transactions. In E-Commerce Workshop, 2002.

### 参考资料
#### 1. Dapper, a Large-Scale Distributed Systems Tracing Infrastructure
http://systemsandpapers.s3.amazonaws.com/papers/dapper.pdf

#### 2. Dapper分布式跟踪系统 -----翻译
http://bigbully.github.io/Dapper-translation/

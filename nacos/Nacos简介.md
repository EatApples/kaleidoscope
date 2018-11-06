### 1. 什么是 Nacos /nɑ:kəʊs/?
Nacos 是阿里巴巴的新开源项目，其核心定位是 “一个更易于帮助构建云原生应用的动态服务发现、配置和服务管理平台”。

Dubbo + Nacos，专为Dubbo而生的注册中心与配置中心。

### 2. Nacos 有三大主要功能:

#### 2.1 服务发现与服务管理
在采用以“服务(Service)”为中心的诸如微服务及云原生方式的现代应用架构时，动态服务发现至关重要。 Nacos同时支持基于DNS和基于RPC（如Dubbo/gRPC）的服务发现，并为您提供服务的实时的健康检查以防止将请求发送给不健康的主机，基于Nacos您也可以更方便的实现服务断路器。Nacos提供的强大的服务的元数据管理，路由及流量管理策略也能够帮助您更好的构建更强壮的微服务平台。

#### 2.2 动态配置管理
动态配置服务允许您在所有环境中以集中和动态的方式管理所有应用程序或服务的配置。动态配置消除了配置更新时重新部署应用程序和服务的需要。可以更方便的帮助您实现无状态服务，更轻松地实现按需弹性扩展服务实例。

#### 2.3 动态DNS服务
支持权重路由的动态DNS服务使您可以更轻松地在数据中心内的生产环境中实施中间层负载平衡，灵活的路由策略，流量控制和简单的DNS解析服务，帮助您更容易的实现DNS-based服务发现。

### 3. Nacos 与 主流开源生态的关系
#### 3.1 Dubbo + Nacos， 专为Dubbo而生的注册中心与配置中心
在阿里巴巴生产环境上，Dubbo和Nacos天然就是长在一起的，因为Nacos的缺失，传统的注册中心解决方案让Dubbo在服务治理、流量治理、服务运营和管理等方面的威力被限制和削弱了，Nacos的开源和开放会在采用Dubbo的用户环境中释放这些威力

#### 3.2 Nacos 会完全兼容Spring Cloud
Nacos会无缝支持Spring Cloud，为Spring Cloud用户其提供更简便的配置中心和注册中心的解决方案，使用Nacos不用再仅仅为服务和配置就需要在生产上hold住 Eureka，Spring Cloud Config Server，Git，RabbitMQ/Kafka 起码四个开源产品。

#### 3.3 Nacos 支持Kubernetes DNS-based Service Discovery
阿里巴巴这么多年在VIPServer DNS-based Service Discovery上的实践证明，在云原生时代，应用会更关注与基础设施的解耦合、多语言乃至多云的诉求，服务发现的未来一定是基于标准的DNS协议做，而不是像Eureka或者像ZooKeeper这样的私有API或者协议做, 同时在云上，在服务发现场景中，注册中心更关注的是可用性而不是数据一致性，所以Nacos会首推DNS-based Servcie Discovery，并优先关注可用性，而这也正是Nacos可以无缝融合进Kubernetes服务发现体系的原因所在。


### 资料来源
#### 1. 支持Dubbo生态发展，阿里巴巴启动新的开源项目 Nacos
https://yq.aliyun.com/articles/604028

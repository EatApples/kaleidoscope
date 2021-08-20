### Kubernetes 的设计理念

http://docs.kubernetes.org.cn/249.html
https://jimmysong.io/kubernetes-handbook/concepts/concepts.html

### 1. API 对象

API 对象是 Kubernetes 集群中的管理操作单元。Kubernetes 集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的 API 对象，支持对该功能的管理操作。例如副本集 Replica Set 对应的 API 对象是 RS。

每个 API 对象都有 3 大类属性：元数据 metadata、规范 spec 和状态 status。

元数据是用来标识 API 对象的，每个对象都至少有 3 个元数据：namespace，name 和 uid；除此以外还有各种各样的标签 labels 用来标识和匹配不同的对象，例如用户可以用标签 env 来标识区分不同的服务部署环境，分别用 env=dev、env=testing、env=production 来标识开发、测试、生产的不同服务。

规范描述了用户期望 Kubernetes 集群中的分布式系统达到的理想状态（Desired State），例如用户可以通过复制控制器 Replication Controller 设置期望的 Pod 副本数为 3；

status 描述了系统实际当前达到的状态（Status），例如系统当前实际的 Pod 副本数为 2；那么复制控制器当前的程序逻辑就是自动启动新的 Pod，争取达到副本数为 3。

Kubernetes 中所有的配置都是通过 API 对象的 spec 去设置的，也就是用户通过配置系统的理想状态来改变系统，这是 Kubernetes 重要设计理念之一，即所有的操作都是声明式（Declarative）的而不是命令式（Imperative）的。声明式操作在分布式系统中的好处是稳定，不怕丢操作或运行多次，例如设置副本数为 3 的操作运行多次也还是一个结果，而给副本数加 1 的操作就不是声明式的，运行多次结果就错了。

### 2. POD

Pod 是 Kubernetes 集群中所有业务类型的基础，可以看作运行在 Kubernetes 集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。

目前 Kubernetes 中的业务主要可以分为长期伺服型（long-running）、批处理型（batch）、节点后台支撑型（node-daemon）和有状态应用型（stateful application）；分别对应的小机器人控制器为 Deployment、Job、DaemonSet 和 StatefulSet，本文后面会一一介绍。

### 3. 部署（Deployment）

部署表示用户对 Kubernetes 集群的一次更新操作。部署是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。

滚动升级一个服务，实际是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，将旧 RS 中的副本数减小到 0 的复合操作；这样一个复合操作用一个 RS 是不太好描述的，所以用一个更通用的 Deployment 来描述。

以 Kubernetes 的发展方向，未来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理。

### 4. 服务（Service）

RC、RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量，但是没有解决如何访问这些服务的问题。一个 Pod 只是一个运行服务的实例，随时可能在一个节点上停止，在另一个节点以一个新的 IP 启动一个新的 Pod，因此不能以确定的 IP 和端口号提供服务。要稳定地提供服务需要服务发现和负载均衡能力。

服务发现完成的工作，是针对客户端访问的服务，找到对应的的后端服务实例。在 K8 集群中，客户端需要访问的服务就是 Service 对象。

每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。在 Kubernetes 集群中微服务的负载均衡是由 Kube-proxy 实现的。

Kube-proxy 是 Kubernetes 集群内部的负载均衡器。它是一个分布式代理服务器，在 Kubernetes 的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 Kube-proxy 就越多，高可用节点也随之增多。与之相比，我们平时在服务器端做个反向代理做负载均衡，还要进一步解决反向代理的负载均衡和高可用问题。

### 5. 后台支撑服务集（DaemonSet）

长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod 运行；而后台支撑型服务的核心关注点在 Kubernetes 集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类 Pod 运行。节点可能是所有集群节点也可能是通过 nodeSelector 选定的一些特定节点。

典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 Kubernetes 集群运行的服务。

### 6. 有状态服务集（StatefulSet）

适合于 StatefulSet 的业务包括数据库服务 MySQL 和 PostgreSQL，集群化管理服务 ZooKeeper、etcd 等有状态服务。

StatefulSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。

传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用 StatefulSet，Pod 仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，StatefulSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。

### 7. 存储卷（Volume）

Kubernetes 集群中的存储卷跟 Docker 的存储卷有些类似，只不过 Docker 的存储卷作用范围为一个容器，而 Kubernetes 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。

### 8. 密钥对象（Secret）

Secret 是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用 Secret 的好处是可以避免把敏感信息明文写在配置文件里。在 Kubernetes 集群中配置和使用服务不可避免的要用到各种敏感信息实现登录、认证等功能，例如访问 AWS 存储的用户名密码。为了避免将类似的敏感信息明文写在所有需要使用的配置文件中，可以将这些信息存入一个 Secret 对象，而在配置文件中通过 Secret 对象引用这些敏感信息。这种方式的好处包括：意图明确，避免重复，减少暴漏机会。

### 9. 命名空间（Namespace）

命名空间为 Kubernetes 集群提供虚拟的隔离作用，Kubernetes 集群初始有两个命名空间，分别是默认命名空间 default 和系统命名空间 kube-system，除此以外，管理员可以可以创建新的命名空间满足需要。

### 10. 资源对象

Kubernetes 提供了多种资源对象，用户可以根据自己应用的特性加以选择。

以下列举的内容都是 kubernetes 中的 Object，这些对象都可以在 yaml 文件中作为一种 API 类型来配置。

- Pod
- Node
- Namespace
- Service
- Volume
- PersistentVolume
- Deployment
- Secret
- StatefulSet
- DaemonSet
- ServiceAccount
- ReplicationController
- ReplicaSet
- Job
- CronJob
- SecurityContext
- ResourceQuota
- LimitRange
- HorizontalPodAutoscaling
- Ingress
- ConfigMap
- Label
- CustomResourceDefinition
- Role
- ClusterRole

简单的分类为以下几种资源对象：

| 类别     | 名称                                                                                                               |
| -------- | ------------------------------------------------------------------------------------------------------------------ |
| 资源对象 | Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling |
| 配置对象 | Node、Namespace、Service、Ingress、Label、CustomResourceDefinition                                                 |
| 存储对象 | Volume、PersistentVolume、Secret、ConfigMap                                                                        |
| 策略对象 | SecurityContext、ResourceQuota、LimitRange                                                                         |
| 身份对象 | ServiceAccount、Role、ClusterRole                                                                                  |

### 11. 资源限制与配额

两层的资源限制与配置

- Pod 级别，最小的资源调度单位
- Namespace 级别，限制资源配额和每个 Pod 的资源使用区间

### 12. 必需字段

在想要创建的 Kubernetes 对象对应的 .yaml 文件中，需要配置如下的字段：

- apiVersion - 创建该对象所使用的 Kubernetes API 的版本
- kind - 想要创建的对象的类型
- metadata - 帮助识别对象唯一性的数据，包括一个 name 字符串、UID 和可选的 namespace

也需要提供对象的 spec 字段。对象 spec 的精确格式对每个 Kubernetes 对象来说是不同的，包含了特定于该对象的嵌套字段。

### 13. Node

一个 Pod 总是在一个（Node）节点上运行，Node 是 Kubernetes 中的工作节点，可以是虚拟机或物理机。每个 Node 由 Master 管理，Node 上可以有多个 pod，Kubernetes Master 会自动处理群集中 Node 的 pod 调度，同时 Master 的自动调度会考虑每个 Node 上的可用资源。

每个 Kubernetes Node 上至少运行着：

Kubelet，管理 Kubernetes Master 和 Node 之间的通信; 管理机器上运行的 Pods 和 containers 容器。

container runtime（如 Docker，rkt）。

### 14. Kubernetes Names

Kubernetes REST API 中的所有对象都用 Name 和 UID 来明确地标识。

Namespace 为名称提供了一个范围。资源的 Names 在 Namespace 中具有唯一性。

对于非唯一用户提供的属性，Kubernetes 提供 labels 和 annotations。

Name 在一个对象中同一时间只能拥有单个 Name，如果对象被删除，也可以使用相同 Name 创建新的对象，Name 用于在资源引用 URL 中的对象，例如/api/v1/pods/some-name。通常情况，Kubernetes 资源的 Name 能有最长到 253 个字符（包括数字字符、-和.），但某些资源可能有更具体的限制条件，具体情况可以参考：标识符设计文档。

UIDs 是由 Kubernetes 生成的，在 Kubernetes 集群的整个生命周期中创建的每个对象都有不同的 UID（即它们在空间和时间上是唯一的）。

### 15. 所有对象都在 Namespace 中?

大多数 Kubernetes 资源（例如 pod、services、replication controllers 或其他）都在某些 Namespace 中，但 Namespace 资源本身并不在 Namespace 中。

而低级别资源（如 Node 和 persistentVolumes）不在任何 Namespace 中。

Events 是一个例外：它们可能有也可能没有 Namespace，具体取决于 Events 的对象。

### 16. Kubernetes Labels

Labels 其实就一对 key/value ，被关联到对象上，标签的使用我们倾向于能够标示对象的特殊特点，并且对用户而言是有意义的（就是一眼就看出了这个 Pod 是尼玛数据库），但是标签对内核系统是没有直接意义的。标签可以用来划分特定组的对象（比如，所有女的），标签可以在创建一个对象的时候直接给与，也可以在后期随时修改，每一个对象可以拥有多个标签，但是，key 值必须是唯一的。

我们最终会索引并且反向索引（reverse-index）labels，以获得更高效的查询和监视，把他们用到 UI 或者 CLI 中用来排序或者分组等等。

我们不想用那些不具有指认效果的 label 来污染 label，特别是那些体积较大和结构型的的数据。不具有指认效果的信息应该使用 annotation 来记录。

### 17. Kubernetes Annotations

可以使用 Kubernetes Annotations 将任何非标识 metadata 附加到对象。客户端（如工具和库）可以检索此 metadata。

可以使用 Labels 或 Annotations 将元数据附加到 Kubernetes 对象。

标签可用于选择对象并查找满足某些条件的对象集合。

相比之下，Annotations 不用于标识和选择对象。Annotations 中的元数据可以是 small 或 large，structured 或 unstructured，并且可以包括标签不允许使用的字符。

Annotations 就如标签一样，也是由 key/value 组成

### 1. Prometheus 是什么

Prometheus（普罗米修斯）是一套开源的监控&报警&时间序列数据库的组合，由 SoundCloud 公司开发。

Prometheus 基本原理是通过 HTTP 协议周期性抓取被监控组件的状态，这样做的好处是任意组件只要提供 HTTP 接口就可以接入监控系统，不需要任何 SDK 或者其他的集成过程。这样做非常适合虚拟化环境比如 VM 或者 Docker 。

Prometheus 应该是为数不多的适合 Docker、Mesos、Kubernetes 环境的监控系统之一。近几年随着 k8s 的流行，prometheus 成为了一个越来越流行的监控工具。

### 2. Prometheus 可以做什么

在业务层用作埋点系统。Prometheus 支持各个主流开发语言（Go，java，python，ruby 官方提供客户端，其他语言有第三方开源客户端）。我们可以通过客户端方面的对核心业务进行埋点。如下单流程、添加购物车流程。

在应用层用作应用监控系统。一些主流应用可以通过官方或第三方的导出器，来对这些应用做核心指标的收集。如 redis，Mysql。

在系统层用作系统监控。除了常用软件， prometheus 也有相关系统层和网络层 exporter，用以监控服务器或网络。

集成其他的监控。prometheus 还可以通过各种 exporter，集成其他的监控系统，收集监控数据，如 AWS CloudWatch，JMX，Pingdom 等等。

### 3. 不要用 Prometheus 做什么

prometheus 也提供了 Grok exporter 等工具可以用来读取日志，但是 prometheus 是监控系统，不是日志系统。应用的日志还是应该走 ELK 等工具栈。

### 4. prometheus 的告警管理

prometheus 的告警管理分为两部分。通过在 prometheus 服务端设置告警规则， Prometheus 服务器端产生告警向 Alertmanager 发送告警。 然后，Alertmanager 管理这些告警，包括静默，抑制，聚合以及通过电子邮件，PagerDuty 和 HipChat 等方法发送通知。

设置警报和通知的主要步骤如下：

- 设置并配置 Alertmanager；
- 配置 Prometheus 对 Alertmanager 访问；
- 在普罗米修斯创建警报规则；

### 5. 告警管理模块 ALERTMANAGER

Alertmanager 处理客户端应用程序（如 Prometheus 服务器）发送的告警。 它负责对它们进行重复数据删除，分组和路由，以及正确的接收器集成，例如电子邮件，PagerDuty 或 OpsGenie。 它还负责警报的静默和抑制。

以下描述了 Alertmanager 实现的核心概念。 请参阅配置文档以了解如何更详细地使用它们。

#### （1）分组(Grouping)

分组将类似性质的告警分类为单个通知。 这在大型中断期间尤其有用，因为许多系统一次失败，并且可能同时发射数百到数千个警报。

示例：
发生网络分区时，群集中正在运行数十或数百个服务实例。 一半的服务实例无法再访问数据库。 Prometheus 中的告警规则配置为在每个服务实例无法与数据库通信时发送告警。 结果，数百个告警被发送到 Alertmanager。

作为用户，只能想要获得单个页面，同时仍能够确切地看到哪些服务实例受到影响。 因此，可以将 Alertmanager 配置为按群集和 alertname 对警报进行分组，以便发送单个紧凑通知。

这些通知的接收器通过配置文件中的路由树配置告警的分组，定时的进行分组通知。

#### （2）抑制(Inhibition)

如果某些特定的告警已经触发，则某些告警需要被抑制。

示例：
如果某个告警触发，通知无法访问整个集群。 Alertmanager 可以配置为在该特定告警触发时将与该集群有关的所有其他告警静音。 这可以防止通知数百或数千个与实际问题无关的告警触发。

#### （3）静默(SILENCES)

静默是在给定时间内简单地静音告警的方法。 基于匹配器配置静默，就像路由树一样。 检查告警是否匹配或者正则表达式匹配静默。 如果匹配，则不会发送该告警的通知。

在 Alertmanager 的 Web 界面中可以配置静默。

#### （4）客户端行为(Client behavior)

Alertmanager 对其客户的行为有特殊要求。 这些仅适用于不使用 Prometheus 发送警报的高级用例。

### 6. Prometheus 四种指标类型

（1）Counter （计算器）

counter 类型代表一种样本数据单调递增的指标，即只增不减，除非监控系统发生了重置。

（2）Gauge（仪表盘）

Gauge 类型代表一种样本数据可以任意变化的指标，即可增可减。

（3）Histogram（直方图）

Histogram 在一段时间范围内对数据进行采样（通常是持续时间或响应大小等，并将其计入可配置的存储桶中，后续可通过制定区间筛选样本，也可以统计样本总数，最后一般将数据展示为直方图，

样本的值分布在 bucket 中的数量，命名为 <basename>\_bucket{le="<上边界>"}。解释的更通俗易懂一点，这个值表示指标值小于等于上边界的所有样本数量。所有样本值的大小总和，命名为 <basename>\_sum。

样本总数，命名为 <basename>\_count。值和 <basename>\_bucket{le="+Inf"} 相同。

（4）Summary（摘要）

与 Histogram 类似类型，用于表示一段时间内的数据采样结果（通常是请求持续时间或响应大小等），但它直接存储了分位数（通过客户端计算，然后展示出来），而不是通过区间计算

样本值的分位数分布情况，命名为 <basename>{quantile="<φ>"}。所有样本值的大小总和，命名为 <basename>\_sum。

Histogram 与 Summary 的异同：

它们都包含了 <basename>\_sum 和 <basename>\_count 指标。
Histogram 需要通过 <basename>\_bucket 来计算分位数，而 Summary 则直接存储了分位数的值。

### 7. Prometheus 表达式语言数据类型

（1）瞬时向量（Instant vector） 一组时间序列，每个时间序列包含单个样本，它们共享相同的时间戳。也就是说，表达式的返回值中只会包含该时间序列中的最新的一个样本值。而相应的这样的表达式称之为瞬时向量表达式。

（2）区间向量（Range vector） 一组时间序列，每个时间序列包含一段时间范围内的样本数据。

（3）标量（Scalar） 一个浮点型的数据值。

（4）字符串（String） 一个简单的字符串值。

### 8. 函数及其适用指标

| 函数名称     | 向量类型 | 返回类型 | 适用指标   | 备注                                       |
| ------------ | -------- | -------- | ---------- | ------------------------------------------ |
| delta        | 区间     | 瞬时     | GAUGE      | 第一个元素和最后一个元素之间的差值         |
| idelta       | 区间     | 瞬时     | GAUGE      | 最新的 2 个样本值之间的差值                |
| increase     | 区间     | ?        | COUNTER    | 第一个和最后一个样本并返回其增长量         |
| rate         | 区间     | ?        | COUNTER    | 在时间窗口内平均增长速率                   |
| irate        | 区间     | ?        | COUNTER    | 最后两个样本数据增长速率                   |
| sum/max 等   | 瞬时     | ?        | ?          | 可以搭配 by 与 without 过滤标签            |
| XX_over_time | 区间     | 瞬时     | 同聚合函数 | 聚合每个时间序列的范围，并返回一个瞬时向量 |

### 资料来源

#### 1. Prometheus 中文文档

https://prometheus.fuckcloudnative.io/

#### 2. 官方文档

https://prometheus.io/docs/introduction/overview/

#### 3. Prometheus 四种指标类型以及表达式语言类型

https://www.cnblogs.com/gavin11/p/12636082.html

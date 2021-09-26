### 1. Observability 的未来趋势

[Google-Wide Profiling: A Continuous Profiling Infrastructure for Data Centers](https://research.google/pubs/pub36575/) 论文介绍了 Google 内部的 Profilling 系统，简称 GWP。GWP 能够持续地对跨数据中心的基础设施进行 profilling，获取包括了栈调用，硬件事件，堆 profile，内核事件等等信息，并进行后续的数据分析。

受这篇论文影响，Conprof 是一个对应用程序进行持续 profilling 的系统。虽然目前还不算成熟。Conprof 最早亮相于 KubeCon EU 2019 的 Keynote 演讲上。Conprof 的主要作者 Frederic 和 Grafana 的 VP Tom 分享了 Observability 的未来趋势，当时他们预测了三点。很有趣的是，2020 年底这个时间节点回过头来看，这三点基本已经成为现实：

（1）More correlation between the three pillars (monitoring, logging and tracing)
三大支柱（监控、日志记录和跟踪）之间的更多相关性

（2）New signals and new analysis
新信号和新分析

（3）Rise of index free log aggregation
无索引日志聚合的兴起

先来聊聊第三点。Index free log aggregation 的服务，在这里主要指的是 Grafana Loki。以往我们在日志的解决方案上通常采用 EFK 来收集、索引以及查询日志，但是这真的有必要吗？很多时候我们并不需要建立那么多的索引，我们只需要类似 grep 那样的功能来对日志进行查询。

再来说第一点，打通可观察性目前也是很多公司正在做的工作，包括了 Grafana，Chronosphere (M3DB 背后的公司)。在日前结束的 ObservabilityCon 上， Grafana 演示了一个 demo 来展示他们如何打通 metrics，logs，traces。首先在查询 Loki 的日志，对于使用了 tracing 库的 HTTP 请求，每个请求都会带上一个 traceID，并且这些 ID 会被打印在日志中。通过 traceID 可以定位到一个唯一的 trace， 从而跳转到 trace 系统的 UI 进行查询。在 metrics 与 trace 的结合上，主要是采用 exemplars 的机制在 metrics 中带上额外的信息。

Exemplar 最早被用在 Google 的 StackDriver 中，后面成为了 OpenMetrics 标准的一部分，在应用通过标准 /metrics 端口暴露 metrics 时，exemplar 信息也会被一起暴露。对于 Prometheus 来说，在写路径上，Prometheus 收集 metrics 的时候也会一并收集 exemplars 信息，并存储下来。在读路径上，会通过一个单独的 API 来暴露 exemplars 信息。

借用这种机制，我们可以把 trace ID 作为一个 label pair 加入 exemplar 中，从而可以从 Prometheus 中查询到 trace 的信息，从而将 metrics 和 trace 连接起来。

最后讲讲今天的主题，第二点中的 new signals，指的就是 profiles。

### 2. Continous Profilling

Conprof 本质上是和 Prometheus 一样的系统，它可以通过一个二进制文件直接运行。不过为了更好的可扩展性，它也支持微服务模式来更好的支持横向扩展。它将内部的组件分成了三个部分，组件之间通过 gRPC 进行通信。主要的组件包括：

（1）Sampler
Sampler 是数据的写入部分，它基于 Pull 模型，直接使用了 Prometheus 的服务发现模块来发现 targets，定期对他们进行 Profiling，并将结果的 protobuf 压缩后，通过 gRPC API 写入到 Store 中。

（2）Store
Store 就是一个 gRPC API 封装起来的 TSDB。Conprof 的 TSDB 是一个 fork 版本的 Prometheus TSDB。它与上游的 TSDB 基本一样，唯一的区别就是 Conprof 中存储的是 protobuf，sample 的值类型是 []byte, 而 Prometheus sample 的值是 float64。这导致了 Prometheus 中的数据可以使用 Gorilla paper 中的方式对样本进行 delta-xor 压缩，而 Conprof 不行，所以在内存和磁盘使用量上目前还是会高于 Prometheus。

（3）API
最后是 API 也就是 Query 组件。它对外暴露了类似于 Prometheus 的 API，包括 query, label_names, label_values, series 等等。

### 3. Conprof

#### 1. 玩玩 PolarSignals

https://yeya24.github.io/post/polar_signals/

#### 2. 聊聊 Conprof 和可观测性

https://yeya24.github.io/post/conprof/

#### 3. conprof 官网

https://github.com/conprof/conprof

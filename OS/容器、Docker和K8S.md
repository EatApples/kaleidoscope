### 容器

在容器技术之前，业界的网红是虚拟机。虚拟机技术的代表，是 VMWare 和 OpenStack。

虚拟机属于虚拟化技术。而 Docker 这样的容器技术，也是虚拟化技术，属于轻量级的虚拟化。

虚拟机虽然可以隔离出很多“子电脑”，但占用空间更大，启动更慢，虚拟机软件可能还要花钱（例如 VMWare）。

而容器技术恰好没有这些缺点。它不需要虚拟出整个操作系统，只需要虚拟一个小规模的环境（类似“沙箱”）。

容器和虚拟机的对比

| 特性       | 虚拟机           | 容器             |
| ---------- | ---------------- | ---------------- |
| 隔离级别   | 操作系统级别     | 进程级别         |
| 隔离策略   | Hypervisor       | CGroups          |
| 系统资源   | 5~15%            | 0~5%             |
| 启动时间   | 分钟级           | 秒级             |
| 镜像存储   | GB-TB            | KB-MB            |
| 集群规模   | 上百             | 上万             |
| 高可用策略 | 备份、容灾、迁移 | 弹性、负载、动态 |

### Docker

Docker 本身并不是容器，它是创建容器的工具，是应用容器引擎。

想要搞懂 Docker，其实看它的两句口号就行。

第一句，是“Build, Ship and Run”。

Docker 的第二句口号就是：“Build once，Run anywhere（搭建一次，到处能用）”。

Docker 技术的三大核心概念，分别是：

- 镜像（Image）

- 容器（Container）

- 仓库（Repository）

Docker 镜像，是一个特殊的文件系统。它除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（例如环境变量）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

负责对 Docker 镜像进行管理的，是 Docker Registry 服务（类似仓库管理员）。

### kubernetes

好了，说完了 Docker，我们再把目光转向 K8S。

就在 Docker 容器技术被炒得热火朝天之时，大家发现，如果想要将 Docker 应用于具体的业务实现，是存在困难的——编排、管理和调度等各个方面，都不容易。于是，人们迫切需要一套管理系统，对 Docker 及容器进行更高级更灵活的管理。

就在这个时候，K8S 出现了。

K8S，就是基于容器的集群管理平台，它的全称，是 kubernetes。

K8S 并不是一件全新的发明。它的前身，是 Google 自己捣鼓了十多年的 Borg 系统。

一个 K8S 系统，通常称为一个 K8S 集群（Cluster）。

这个集群主要包括两个部分：

- 一个 Master 节点（主节点）

- 一群 Node 节点（计算节点）

一看就明白：Master 节点主要还是负责管理和控制。Node 节点是工作负载节点，里面是具体的容器。

首先是 Master 节点。

Master 节点包括 API Server、Scheduler、Controller manager、etcd。

API Server 是整个系统的对外接口，供客户端和其它组件调用，相当于“营业厅”。

Scheduler 负责对集群内部的资源进行调度，相当于“调度室”。

Controller manager 负责管理控制器，相当于“大总管”。

然后是 Node 节点。

Node 节点包括 Docker、kubelet、kube-proxy、Fluentd、kube-dns（可选），还有就是 Pod。

Docker，不用说了，创建容器的。

Kubelet，主要负责监视指派到它所在 Node 上的 Pod，包括创建、修改、监控、删除等。

Kube-proxy，主要负责为 Pod 对象提供代理。

Fluentd，主要负责日志收集、存储与查询。

### 资料来源

#### 1. 干货满满！10 分钟看懂 Docker 和 K8S

https://my.oschina.net/jamesview/blog/2994112

# 服务网格介绍

## 服务网格定义

Service Mesh 定义 Service Mesh 一词最早由开发 Linkerd 的 Buoyant 公司提出，并于 2016 年 9 月29 日第一次公开使用了这一术语。William Morgan，Buoyant CEO，对 Service Mesh 这一概念定义如下：

> A service mesh is a dedicated infrastructure layer for handling service-to-service communication. It’s responsible for the reliable delivery of requests through the complex topology of services that comprise a modern, cloud native application. In practice, the service mesh is typically implemented as an array of lightweight network proxies that are deployed alongside application code, without the application needing to be aware.

翻译成中文如下：

> Service Mesh 是一个专门处理服务通讯的基础设施层。它的职责是在由云原生应用组成服务的复杂拓扑结构下进行可靠的请求传送。在实践中，它是一组和应用服务部署在一起的轻量级的网络代理，并且对应用服务透明。

以上这段话有四个关键点：

* 本质：基础设施层；
* 功能：请求分发；
* 部署形式：网络代理；
* 特点：透明；

## 传统结构的缺陷

面对上述暴露出的问题，并在传统微服务架构下，经过实践的不断冲击，面临了更多新的挑战，综上所述，产生这些问题的原因有以下这几点：

* **过于绑定特定技术栈** 当面对异构系统时，需要花费大量精力来进行代码的改造，不同异构系统可能面临不同的改造。
* **代码侵入度过高** 开发者往往需要花费大量的精力来考虑如何与框架或 SDK 结合，并在业务中更好的深度融合，对于大部分开发者而言都是一个高曲线的学习过程。
* **多语言支持受限** 微服务提倡不同组件可以使用最适合它的语言开发，但是传统微服务框架，如 Spring Cloud 则是 Java 的天下，多语言的支持难度很大。这也就导致在面对异构系统对接时的无奈，或选择退而求其次的方案了。
* **老旧系统维护难** 面对老旧系统，很难做到统一维护、治理、监控等，在过度时期往往需要多个团队分而管之，维护难度加大。

## 服务网格要解决的问题

* 消除了将特定语言的通信库编译到单个服务中的需求，以处理服务发现、路由和应用层（第 7 层）非功能通信要求。
* 外部化服务通信配置，包括外部服务的网络位置、安全凭证和服务质量目标。
* 提供对其他服务的被动和主动监测。
* 在整个分布式系统中分布式地执行策略。
* 提供可观察性的默认值，并使相关数据的收集标准化。
  * 启用请求记录
  * 配置分布式追踪
  * 收集指标

## 服务网格架构

服务网格模式主要侧重于处理传统上被称为 “东西向 “的基于远程过程调用（RPC）的流量：请求 / 响应类型的通信，源自数据中心内部，在服务之间传播。这与 API 网关或边缘代理相反，后者被设计为处理 “南北 “流量。来自外部的通信，进入数据中心内的一个终端或服务。

服务网格由两部分组成：数据平面和控制平面。Matt Klein，[Envoy Proxy](https://www.envoyproxy.io/) 的作者，写了一篇关于 “ [服务网格数据平面与控制平面 ](https://blog.envoyproxy.io/service-mesh-data-plane-vs-control-plane-2774e720f7fc)“的深入探讨。\
广义上讲，数据平面 “执行工作”，负责 “有条件地翻译、转发和观察流向和来自 \[网络终端] 的每个网络数据包”。在现代系统中，数据平面通常以代理的形式实现，（如 Envoy、[HAProxy](http://www.haproxy.org/) 或 [MOSN](https://github.com/mosn/mosn)），它作为 “sidecar” 与每个服务一起在进程外运行。Linkerd 使用了一种 [微型代理](https://linkerd.io/2020/12/03/why-linkerd-doesnt-use-envoy/)方法，该方法针对服务网格的使用情况进行了优化。

控制平面 “监督工作”，并将数据平面的所有单个实例 —— 一组孤立的无状态 sidecar 代理变成一个分布式系统。控制平面不接触系统中的任何数据包 / 请求，相反，它允许人类运维人员为网格中所有正在运行的数据平面提供策略和配置。控制平面还能够收集和集中数据平面的遥测数据，供运维人员使用。

控制平面和数据平面的结合提供了两方面的优势，即策略可以集中定义和管理，同时，同样的政策可以以分散的方式，在 Kubernetes 集群的每个 pod 中本地执行。这些策略可以与安全、路由、断路器或监控有关。

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* **Pilot**：负责 Istio 数据平面的 xDS 配置管理，具体包括服务发现、配置规则发现、xDS 配置下发。
* **Citadel**：负责安全证书的管理和发放，实现授权和认证等操作。
*   **Galley**：负责配置的验证、提取和处理等功能，将 Istio 和底层平台(如,Kubernetes)进行解耦。

    其中，Citadel、Galley 组件逐步在弱化，在 Istio 版本迭代中，已经基本看不见它们的踪迹了。（已经不断整合在其它组件中）

\


<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

* **抽象模型（Abstract model）**：为了实现对不同服务注册中心 （如，Kubernetes、Consul） 的支持，完成对不同输入来源数据的抽象，形成统一的存储格式。
* **平台适配器 （Platform adapters）**：借助平台适配器 Pilot 实现服务注册中心数据到抽象模型之间的数据转换。
* **xDS API**：是源于 Envoy 项目的标准数据平面 API， 将服务信息和流量规则下发到数据平面的 Sidecar。通过采用该标准 API， Istio 将控制平面和数据平面进行了解耦，为多种数据平面 Sidecar 实现提供了可能性，如：蚂蚁金服开源的 Golang 版本的 MOSN。
* **用户 API（User API）**：提供了面向业务的高层抽象，可以被运维人员理解和使用。

\


\


<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## 提供的功能

连接性

* 流量控制（路由，分流）
* 网关（入口、出口）
* 服务发现
* A/B 测试、金丝雀
* 服务超时、重试

可靠性

* 断路器
* 故障注入 / 混沌测试

安全性

* 服务间认证（mTLS）
* 证书管理
* 用户认证（JWT）
* 用户授权（RBAC）
* 加密

可观察性

* 监测
* 遥测、仪表、计量
* 分布式追踪
* 服务图表

## 服务网格的缺陷

1. 有人在生产使用 Istio 吗？
2. 为 pod 注入 sidecar 后带来的大量资源消耗，影响应用性能？
3. Istio 支持的协议有限，不易扩展？
4. Istio 太过复杂，老的服务迁移成本太高，业界经验太少，学习曲线陡峭？



Istio 是一款优秀的开源软件，具有极高的社区活跃度和强大的社区生态，也具备比较优秀的架构设计，这一点毋庸置疑。但是，有一个缺陷，也是比较致命的缺陷，Istio 不是从企业中大规模落地验证后开源出来的，Istio 是从出生就是一个开源软件，Istio 是一个理想型的开源软件（譬如强调流量劫持的无侵入性）。

软件架构没有银弹，往往是取舍。在设计之初，Istio 考虑的是普适性和功能的完备性，支持流量管理、安全、可观测性等多个维度的功能设计，并且随着项目的发展，这个功能不断增强，其实带来的损害就是性能，以及海量实例场景下 CPU 与 内存的巨量消耗。

### 流量劫持问题

我们可以看到，Istio 在很多实例规模比较小的公司或者业务团队，是可以逐步落地和推广的，但是一旦上了体量，问题就暴露出来了。早期 mixer 组件带来的性能问题尚且不谈，毕竟已经废弃了，但是 iptable 的流量劫持机制，在一定程度上来讲，就是在大规模公司落地的拦路虎。目前 Istio 使用 iptables 实现透明劫持，主要存在以下三个问题：

* 需要借助于 conntrack 模块实现连接跟踪，在连接数较多的情况下，会造成较大的 消耗，同时可能会造成 track 表满的情况，为了避免这个问题，业内有关闭 conntrack 的做法。
* iptables 属于常用模块，全局生效，不能显式的禁止相关联的修改， 可管控性比较差。
* iptables 重定向流量本质上是通过 loopback 交换数据，outbond 流量将两次穿越协议栈，在大并发场景下会损失转发 性能。

需要借助于 conntrack 模块实现连接跟踪，在连接数较多的情况下，会造成较大的 消耗，同时可能会造成 track 表满的情况，为了避免这个问题，业内有关闭 conntrack 的做法。

iptables 属于常用模块，全局生效，不能显式的禁止相关联的修改， 可管控性比较差。

iptables 重定向流量本质上是通过 loopback 交换数据，outbond 流量将两次穿越协议栈，在大并发场景下会损失转发 性能。

### 配置下发问题

再来看数据面 envoy 与控制面 istiod 的通信协议 xDS。xDS 包含多种协议的集合，比如：LDS 表示监听器，CDS 表示服务和版本，EDS 表示服务和版本有哪些实例，以及每个服务实例的特征，RDS 表示路由。可以简单的把 xDS 理解为，网格内的服务发现数据和治理规则的集合。xDS 数据量的大小和网格规模是正相关的。

Istio 下发 xDS 使用的是全量下发策略，也就是网格里的所有 sidecar，内存里都会有整个网格内所有的服务发现数据，理由是用户很难梳理清楚服务间依赖关系并且提供给 Istio。按照这种模式，每个 sidecar 内存都会随着网格规模增长而增长。根据社区有团队对其做的性能测试可以看出，如果网格规模超过 1 万个实例，单个 envoy 的内存超过了 250 兆，而整个网格的开销还要再乘以网格规模大小，即 2500 千兆，惊人的消耗！！！

当然，社区也提供了一些解决方案，比如通过手动配置 Sidecar 这个 CRD，可以显示地定义服务间的依赖关系，这要求用户需要手动配置梳理好每一条调用链关系，服务间才能发现彼此，这在大规模场景下也是有一定的局限性。也有社区同学开源出来了一些解决方案，但是都或多或少带来了一些其它问题，譬如单点问题、系统的复杂度提高、运维复杂性、峰值压力等。

\


## 参考文档

[服务网格终极指南第二版——下一代微服务开发](https://cloudnative.to/blog/service-mesh-ultimate-guide-e2/)

[全方位解读服务网格（Service Mesh）的背景和概念](https://developer.aliyun.com/article/867274)

[行至2022，我们该如何看待服务网格？ ](https://www.sohu.com/a/516755841\_355140)


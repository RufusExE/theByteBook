# 3.3.3 快速数据路径 XDP

2010 年，由 Intel 领导的 DPDK 实现了一个基于内核旁路（Kernel bypass）思想的高性能网络应用开发解决方案，并逐渐成为了独树一帜的成熟技术体系。但是 DPDK 也由于内核旁路这一前提，天然就无法与内核技术生态很好的结合。

2016 年，在 Linux Netdev 会议上，David S. Miller[^1] 喊出了 “DPDK is not Linux” 的口号。同年，伴随着 eBPF 技术的成熟，Linux 内核终于迎来了属于自己的高速公路 —— XDP（eXpress Data Path，快速数据路径），其具有足以媲美 DPDK 的性能以及背靠 Linux 内核的多种独特优势。

## 1. XDP 概述

XDP 本质上是 Linux 内核网络模块中的一个 BPF Hook，能够动态挂载 eBPF 程序逻辑，使得 Linux 内核能够在数据报文到达 L2（网卡驱动层）时就对其进行针对性高速处理，无需再「循规蹈矩」地进入到内核网络协议栈。

如图 3-13 所示的 XDP 工作原理，XDP 程序在内核提供的网卡驱动中直接取得网卡收到的数据帧，然后直接送到用户态应用程序。应用程序利用 AF_XDP[^2] 类型的 socket 接收数据。

<div  align="center">
	<img src="../assets/XDP.svg" width = "500"  align=center />
	<p>图 3-13 XDP 流程概念</p>
</div>

## 2. XDP 应用示例

前面讲过的 conntrack 实际上只是其 Netfilter 在 Linux 内核中的连接跟踪实现。换句话说，只要具备了 hook 能力，能拦截到进出主机的每个数据包，就完全可以摆脱 Netfilter，实现另外一套连接跟踪。

云原生网络方案 Cilium 在 1.7.4+ 版本就实现了这样一套独立的连接跟踪和 NAT 机制，其基本原理是：

- 基于 BPF hook 实现数据包的拦截功能（等价于 netfilter 的 hook 机制）。
- 在 BPF hook 的基础上，实现一套全新的 conntrack 和 NAT。

因此使用 Cilium 方案的 Kubernetes 网络模型，即便在 Node 节点卸载 Netfilter，也不会影响 Cilium 对 Kubernetes ClusterIP、NodePort、ExternalIPs 和 LoadBalancer 等功能的支持。

<div  align="center">
	<img src="../assets/cilium.svg" width = "500"  align=center />
	<p>图 3-14 conntrack Cilium 方案</p>
</div>

由于 Cilium 方案的连接跟踪机制独立于 Netfilter，因此它的 conntrack 和 NAT 信息也没有存储在内核中的 conntrack table 和 NAT table 中，常规的 conntrack/netstats/ss/lsof 等工具看不到 nat、conntrack 数据，所以需要另外使用 Cilium 的命令查询，例如：

```
$ cilium bpf nat list
$ cilium bpf ct list global
```

[^1]: Linux 内核开发者，自 2005 年开始，已经提交过 4989 个 patch，是 Linux 核心源代码第二大的贡献者。
[^2]: 类同 AF_INET 是用于 IPv4 类型地址的通讯，AF_XDP 则是一套基于 XDP 的通讯的实现。


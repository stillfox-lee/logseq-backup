title:: Linkerd｜服务网格2022回顾：项目采用率翻倍，对eBPF、Gateway API等的了解
author:: [[Linkerd]]
url:: https://mp.weixin.qq.com/s/KtE2RLuzxjpCPsiw86ux8g
tags:: #[[k8s]] #[[servicemesh]]

- Highlights first synced by [[Readwise]] [[2023-01-05]]
	- > 这种增长来自哪里，为什么是现在？根据全年与新采用者的交谈，我们的理论是这样的：服务网格在早期得到了不好的评价，这是由于极度的宣传，加上最受宣传的项目的相对不成熟和复杂性。早期采用者决定推迟到尘埃落定。现在他们回来了，看到了第一次发生的事情，他们正在寻找一个不会让他们背负众所周知的运营复杂性包袱的选项。 ([View Highlight](https://read.readwise.io/read/01gnztp6adn4y8wtvwrcwp69d4))
		- 也许是看到了Linkerd 从CNCF毕业之后有了信心？
	- > 他们自然会来到 Linkerd，因为它的简单性在服务网格领域是独一无二的。Linkerd 的大部分优势，归结于它的数据平面。Linkerd 是唯一一个避开 Envoy 而专注于专用边车“微代理”的服务网格。2018 年，这是一个有争议的决定；2022 年，这种方法继续得到回报。当其他项目，花费时间，为其数据平面的复杂性和资源消耗，构建解决方案时，Linkerd 反而专注于提供强大的功能，如多集群故障转移[1]和基于 Gateway API 的完整 L7 授权策略[2]。 ([View Highlight](https://read.readwise.io/read/01gnztw5g4w3n624g9gt6wvpdh))
		- 简单性体现在哪里？多集群的故障转移怎么做的？
	- > 今年早些时候，当围绕 eBPF 的关于服务网格的讨论达到高潮时，我们决定进行更深入的研究。我们的发现，没有我们希望的那么令人信服。虽然 eBPF 可以简化一些基本的服务网格任务，例如转发原始 TCP 连接，但是如果没有用户空间组件，它根本无法处理 HTTP/2、mTLS 或其他 L7 任务，这意味着它不会提供根本性的改变——即使有了 eBPF，服务网格仍然需要集群上的 L7 代理。 ([View Highlight](https://read.readwise.io/read/01gnzw0pzqmkzkq43pxeamthv0))
		- **Tags**: #[[ebpf]] #[[servicemesh]]
	- > Istio 的无边车“环境网格（ambient mesh）”模式很快加入了无边车 eBPF 服务网格，该模式使用了每主机和每服务代理的组合。采用这种方法，对我们来说是另一种学习经历，我们很高兴地看到，至少在这里，安全方面变得更好了：例如，用于单独身份的 TLS 密钥材料在单独的进程中维护。
	  
	  > 然而，取消边车的代价是巨大的：需要大量的新机器，其结果是不可忽视的限制[9]，以及对性能的重大影响。
	  
	  > 总的来说，在 Linkerd 的案例中，生命周期管理和资源消耗的改进并没有增加。我们的感觉是，环境网格更像是用来解决大规模运行 Envoy 问题的解决方案。 ([View Highlight](https://read.readwise.io/read/01gnzw4ncznkyt9rgkc9bgba2q))
		- **Tags**: #[[servicemesh]]
	- > 多集群故障转移: *https://linkerd.io/2022/03/09/announcing-automated-multi-cluster-failover-for-kubernetes/* ([View Highlight](https://read.readwise.io/read/01gnzw9mg6gjqb06h4mtaady2y))
		- **Tags**: #[[servicemesh]]
	- > 启动时到底发生了什么：Linkerd、init 容器、CNI 等等: *https://linkerd.io/2022/12/01/what-really-happens-at-startup-linkerd-init-containers-the-cni-and-more/* ([View Highlight](https://read.readwise.io/read/01gnzw7bckt8862ja5t37b8adb))
		- TODO: 值得阅读
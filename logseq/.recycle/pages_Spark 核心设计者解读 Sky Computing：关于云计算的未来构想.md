title:: Spark 核心设计者解读 Sky Computing：关于云计算的未来构想
author:: [[infoq.cn]]
url:: https://www.infoq.cn/article/WIm2YhkS1I3z6Zds8tcq
tags:: #[[skycomputing]] #[[SkyComputing]]

- Highlights first synced by [[Readwise]] [[2023-01-05]]
	- > 尽管云计算和互联网在许多方面存在差异，但是互联网的公用设施化为公用计算提供了有用的历史经验。有三个关键的设计决策使互联网能够为大量异构技术（从以太网到 ATM 再到无线）和竞争公司提供统一的接口。第一个是掩盖技术差异的“兼容性”层。 第二种是域间路由，它将 互联网粘合在一起，使其对最终用户来说是一个网络。 第三个是一组经济协议，称之为“对等”层，允许相互竞争的网络合作创建一个统一的网络。 ([View Highlight](https://read.readwise.io/read/01gmceh7wh4a0fted2d03ghd73))
	- > Sky Computing 需要一个兼容层来掩盖低级技术差异，一个云间层将作业路由到正确的云，以及一个对等层，允许云之间就如何交换服务达成协议。
	  
	  
	  
	  ![](https://static001.geekbang.org/infoq/b5/b5accf9aed397e02a23d12acaaf85ee6.png)
	  
	  表 1 Sky Computing 架构与互联网之间的类比
	  
	  表 1 给出了 Sky Computing 架构与互联网之间的类比性：路由器可类比于服务器，AS 类比于可用区，而 ISP 类比于云供应商。 就像在互联网中，IP 不关心在同一 ISP 或跨 ISP 的路由器之间的路由数据包，构建在云间层上的应用程序不需要在意它所运行的云平台。 ([View Highlight](https://read.readwise.io/read/01gmcem0c2ts4h1he4y5bdx469))
	- > 兼容层可以从这些 OSS 解决方案中构建出来。例如 Cloud Foundry，是一种开源多云应用平台，支持主要的云供应商以及内部部署集群。 以及 RedHat 的 OpenShift，是一个基于 Kubernetes 的多云和内部部署平台。因此，从纯粹的技术角度来看，实现一个能够广泛使用的兼容层是很容易的。问题是市场是否会支持这一发展。因为虽然兼容层对用户有明显的好处，但它自然会使云平台商品化，这可能不符合供应商的利益。 ([View Highlight](https://read.readwise.io/read/01gmcewfx663v50bvhd52n2vyy))
		- 云厂商就被打到下层去了，一个是无法深度绑定，还有一个是价值贬低。可以从快递行业来看：菜鸟网络做平台，快递公司做辛苦的物流。快递公司都是赚辛苦费，菜鸟赚的是服务费。
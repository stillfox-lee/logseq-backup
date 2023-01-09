title:: 想彻底改变云行业！Spark发源地UC伯克利分校再推开源项目应对云成本飙升：平均降至三分之一
author:: [[微信公众平台]]
url:: https://mp.weixin.qq.com/s/hKfj_z2TuNag5xPGP4Uq3g
tags:: #[[云计算]]

- Highlights first synced by [[Readwise]] [[2023-01-05]]
	- > 开源框架 SkyPilot，这套框架能够在任何云环境上无缝、且经济高效地运行机器学习与数据科学批量作业，适用于多云和单云用户。SkyPilot 的目标是大大降低云使用门槛、控制运行成本，而且全程无需任何云基础设施专业知识。 ([View Highlight](https://read.readwise.io/read/01gmcaxgm8804wpbqmpsadfcs9))
	- > 以 GPU 为例，截至 11 月份时，Azure 的英伟达 A100 GPU 实例价格最低，GCP 和 AWS 分别要比其高出 8% 和 20%。CPU 同样存在价格差异：对于最新的通用实例（配备同样的 vCPU 和内存），不同云服务商的定价差异可能超过 50%。因此，为某项任务选择最合适的云厂商和相应硬件，无疑能显著降低成本、提高性能。 ([View Highlight](https://read.readwise.io/read/01gmcck240t3tkq6vnr3pftzr2))
		- 如果加入价格识别和成本优化模型，这就为乙方带来了很大的便利性。
	- > Sky Computing 构想的底层是云兼容层，通过抽象出云计算服务，使在该层之上开发的应用程序无需更改即可在不同的云上运行。兼容层可以从当前很多 OSS 解决方案中构建出来，如操作系统 Linux，集群资源管理器 Kubernetes、Mesos，数据库 MySQL、Postgres，⼤数据执⾏引擎 Spark、Hadoop，机器学习库 PyTorch 、Ten sorflow，通⽤分布式框架 Ray、Erlang 等等。
	  
	  云兼容层之上云间层，用户可以指定有关其作业应在何处运行的策略。云间层之上是对等层，旨在让云间可以通过建立高速连接互相传递数据，使数据传输又快又便宜，实现更大的工作流动自由。 ([View Highlight](https://read.readwise.io/read/01gmcdw6xfh114zaw7h8k9b3ww))
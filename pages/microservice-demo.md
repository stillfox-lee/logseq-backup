tags:: [[SideProject]]
state:: DOING

-
- 这是一个个人的微服务的项目软件工程展示项目，想要展示我现阶段的技能栈。以及 Learning by doing
- 主要应该包括
	- DevOps
		- GitOps
	- observability
		- 参考 https://mp.weixin.qq.com/s/F_Uoq4ebrslrF7VWQsQRiQ
		-
	- servicemesh
		- 除了业务服务之外，其他的中间件如何治理？
- 可以通过几个阶段来演进
	- ### 第一阶段
		- 主要是 go-kit 的介绍
			- 设计思想和哲学
			- 如何加入服务治理
		- 加入 BDD 测试，服务治理的测试
		- 只使用 docker-compose 来部署，可实现`服务发现`
	- ### 第二阶段
		- k8s 部署
		- GitOps
		- 发布策略
			- 灰度发布
	- ### 第三阶段
		- observability
			- 分布式追踪
			- APM
			- 日志分析
			- 告警
		- auto scale
			- 通过 k8s 实现自动拓容
		- 性能测试
			- 性能测试的 metrics
			- 参考 https://mp.weixin.qq.com/s/uX9TlbjJTr0z_YeWnannYQ
			-
			-
- ### 一些需求整理
	- 配置管理
		- 配置分类
			- 业务的配置。会动态调整。应该通过统一的 Redis 来获取数据
			- 软件工程的配置。静态配置。可以在 CI 解决，应该跟着版本走
			- 服务治理的配置。例如 QPS 限制等等。
			  #+BEGIN_CAUTION
			  这个比较复杂，如果是 SDK 的方式，如何动态修改？
			  #+END_CAUTION
			-
	- 架构
		- 1 台测试
		- 3 台线上(一台小流量用来做预发布)
	- 使用 go-kit的 example
		- gateway-api
	- 框架层面的服务治理
		- 服务发现 --> 通过 docker-compose.yaml
		- 配置管理 --> 基于 Redis 制作的配置管理，利用 PubSub 来更新推送。
		- 服务治理
			- 限流测试
				- TODO 怎么配置限流的大小？
					- 通过 Redis 统一配置一个数值，
					- 需要配置默认值，
					- push 即时更新内存值
					- push 调小的时候怎么处理？
				- 自适应限流
				- 测试怎么进行？
				- 测试监控数据？
	- 监控 使用elastic APM
		- 遇到了什么坑？能解决什么问题？
			- TODO 自定义 metrics
				- 限流的相关指标
					- bucket 大小
					- bucket丢弃请求数量
					- 现在处理的速度
		- 告警怎么做
			- 触发限流阈值
			- 服务进程挂了 考虑`CD的场景`
		- 监控的数据
			- latency
		- 监控的东西
	- devops
		- 数据库变更怎么做？
			- 通过 migration 进行版本管理控制
			- [参考](https://semaphoreci.medium.com/database-management-with-ci-cd-d8b74e9febf2)
		- CI
			- unit test
			- docker build
			- push image
		- 使用 GitOps
		- CD
			- TODO 怎么用 docker-compose 来平滑更新、拓容
			- 线上预发布
				- 在 gateway 做流量标记，路由到特定的环境中
			- 升级策略：启动新容器-->导一部分流量-->通过监控采样确认没问题-->再逐步升级容器
			- 回滚策略：
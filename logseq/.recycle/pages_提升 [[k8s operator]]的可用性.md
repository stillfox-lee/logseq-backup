-
- url:: https://itnext.io/kubernetes-operator-development-guidelines-for-improved-usability-222390b00dc4
  tags:: #Reading #[[k8s operator]] 
  category:: article
- https://miro.medium.com/max/1400/1*HhofnJCiAOlma0eeJLCXSw.jpeg
- 设计准则
	- 使用声明式设计，而不是命令式。
	- 优先使用k8s 自身的机制
		- 使用 CRD、Aggregated API Server、Custom Sub-resource 来构建
	- 确定 CR 的 metrics 策略
		- 制定自定义资源对应的 metrics 指标，可以提高 controller 的可观测性。一个方案是：将 controller 的 metric 暴露给类似于 Prometheus。如果不想在 controller 的代码中处理 metric 的话，也可以通过 k8s 的审计日志来完成。
	- 把 CRD 注册为 [[Helm]] chart 的一部分
		- 有些controller的实现中，是将 CRD 在代码库中有一份，然后 chart 另外维护一份 CRD。可以通过统一使用 chart 管理 CRD。
	- 如果依赖于[[etcd]]
		- 如果 Operator 依赖使用 etcd 作为存储。那么在 [[Helm]]的 chart 中把 etcd 作为可配置项。使用者可以自定义配置如何使用 etcd：使用集群中已有的实例，为Operator独立部署实例。
- #apisix #k8s #[[k8s programming]] #[[gateway api]]
- init Informer & Lister
- run Informer & Lister
- GatewayClass
	- init `GatewayClass Controller`
		- 注册`EventHandler`。所有的 EventHandler 都是将 Event 添加到`workqueue`即可。
		- 认领 GatewayClass
			- 通过`GatewayClass.Spec.ControllerName`匹配 GatewayClass 资源
			- 对`GatewayClass.Status.conditions.Status=True`记录更新`Reason` `Message`。[官方说明](https://gateway-api.sigs.k8s.io/api-types/gatewayclass/#gatewayclass-status)
			- Controller 内存标记这个 GatewayClass 资源的 name
- Gateway
	- translate 逻辑
		- getAllowedKinds
			- get RouteGroupKind from Listener
			- validate Listener
		- return Listeners
	- add Listeners
		-
- 关于 Informer的使用
	- 按照资源粒度创建 SharedInformer
	- 注册 EventHandler
	- 调用informer.run()
-
- 疑问整理
	- Route 资源的管理
		- 为什么每次都要创建新的 ID？
		- 比如给 Route 添加 Upstream 的时候，都是创建新的 Upstream 对象，生成新的 UpstreamID。为什么不是先查询 APISIX 是否有这个 Upstream，复用 Upstream？
		- *因为这时 Controller 无法从 APISIX 中去查询已有的 Upstream，所以无法复用。*
	- IngressController 生命周期里面，初始化错误时为什么要去调用`ctx.Done()`然后 return？
		- ```go
		  	c.namespaceProvider, err = namespace.NewWatchingProvider(ctx, c.kubeClient, c.cfg)
		  	if err != nil {
		  		ctx.Done()
		  		return
		  	}
		  ```
	- delete Event的处理逻辑
		- `Event.Tomstone`的作用是什么？
		-
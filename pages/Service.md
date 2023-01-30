- ## Headless Service
	- Headless Service 是一种特殊的模式，它不会有 ClusterIP，也就意味着它没有LB和 proxy的功能。
	- 在 Headless Service 中，如果带有`Selector`的话，会创建对应的`EndpointSlice`。如果没有`Selector`的话，则不会创建`EndpointSlice`。
		- DONE 有 Selector 和无 Selector 的 DNS 记录有什么不同？
		  collapsed:: true
		  :LOGBOOK:
		  CLOCK: [2023-01-17 Tue 11:07:15]--[2023-01-17 Tue 11:07:16] =>  00:00:01
		  CLOCK: [2023-01-17 Tue 11:07:17]--[2023-01-17 Tue 11:07:18] =>  00:00:01
		  :END:
			- 无 Selector 的 Headless Service 的 DNS 解析没有结果：
			  ```yaml
			  / # nslookup my-headless-service
			  Server:		10.96.0.10
			  Address:	10.96.0.10:53
			  
			  ** server can't find my-headless-service.default.svc.cluster.local: NXDOMAIN
			  
			  ** server can't find my-headless-service.svc.cluster.local: NXDOMAIN
			  
			  ** server can't find my-headless-service.cluster.local: NXDOMAIN
			  
			  ** server can't find my-headless-service.default.svc.cluster.local: NXDOMAIN
			  
			  ** server can't find my-headless-service.svc.cluster.local: NXDOMAIN
			  
			  ** server can't find my-headless-service.cluster.local: NXDOMAIN
			  ```
			- 有 Selector 的 Headless Service DNS 记录会返回所有的关联 Pod 的 IP：
			  ```yaml
			  / # nslookup headless-service-selector
			  Server:		10.96.0.10
			  Address:	10.96.0.10:53
			  
			  ** server can't find headless-service-selector.cluster.local: NXDOMAIN
			  
			  ** server can't find headless-service-selector.svc.cluster.local: NXDOMAIN
			  
			  Name:	headless-service-selector.default.svc.cluster.local
			  Address: 10.244.1.7
			  Name:	headless-service-selector.default.svc.cluster.local
			  Address: 10.244.1.6
			  
			  
			  ** server can't find headless-service-selector.cluster.local: NXDOMAIN
			  
			  ** server can't find headless-service-selector.svc.cluster.local: NXDOMAIN
			  
			  ```
	- Headless Service 与 DNS
		- k8s 也会为 Headless Service 创建 [[DNS]] 记录，但是这个 A 记录会返回所有对应的 Pod 的 IP。
		- *这就需要客户端来决定如何处理 IP 集合了*
	- Headless Service 典型的使用场景就是 stateful。StatefulSet 后端的 Pod 都有独自的 DNS 名称，客户端可以通过特定的 DNS 名称来访问特定的节点。**例如数据库的读写分离场景。**
	-
- ## ServiceType
	- ClusterIP
	- NodePort
	- LoadBalancer：通过 vendor 提供的负载均衡向外暴露Service。
	- ExternalName
- ## Service 和 DNS
	- ref:
		- https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/
		- https://github.com/kubernetes/dns/blob/master/docs/specification.md
	- Headless 的 DNS 记录
-
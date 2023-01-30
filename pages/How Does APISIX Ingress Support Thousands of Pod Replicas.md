url:: https://api7.ai/blog/apisix-ingress-support-thousands-pod-replicas
tags:: #Reading #k8s
category:: article

-
- k8s 的 Endpoint 在大规模拓容场景中的局限性
- 在 1.21 版本引入了 EndpointSlice，为scale 和 extension 带来了好处。
- [[Endpoint]]资源的缺点
	- 一个 [[Service]]只能对应一个 Endpoint。一个 Endpoint 就需要存储 Service 下的所有 Pod 信息(IP、port)。
	- Endpoint 资源是需要存储在 [[etcd]]中的，etcd 单个数据存储默认限制是1.5MB。在特定情况下，一个 Endpoint 下最多只能存储 5000 个 Pod 信息。
	- 假设一个具有 5000 个 Pod 的 Service，3000 个节点的场景，对应的 Endpoint 可能有 1.5MB。假设只有一个 Pod更新，那么这个 1.5MB 的数据需要推送到 3000 个节点，那就是 4.5GB 数据。如果 5000 个 Pod都更新了，那就是 22TB数据。
		- 吐槽一下：要是通信协议的设计能只推送增量更新的信息，这个问题不就不存在了吗。
	-
- [[Endpointslices]]如何解决问题
	- 通过 [[sharding]] ，将原本一个 Endpoint 的资源，分配到多个小的`Endpointslices`来存储。
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20221025153434.png)
	-
- [[Endpointslices]]带来的新特性
	- ipv4/ipv6双栈
	- 拓扑感知
	-
	-
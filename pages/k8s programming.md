- #Reading #k8s
- Controller和 Operator 的不同
	- Controller 可以控制 k8s 的核心资源，对其进行操作。
	- Operator 也是一种 Controller，但是它包含着一些运维逻辑。比如应用程序的生命周期管理。
- Controller的控制循环
	- 1. 通过 Watch 获取到资源更新的事件
	  2. 执行更新操作
	  3. 通过 API Server 更新资源的状态
	  4. 返回第一步
	- ![control loop](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20220711225136.png)
	- k8s 中的 Controller 控制逻辑采用`边沿触发`(事件驱动)，同时增加了定时同步的逻辑。详细的细节可以参考:[level-triggering-and-reconciliaction-in-k8s](https://hackernoon.com/level-triggering-and-reconciliation-in-kubernetes-1f17fe30333d)
- MVCC
	- 在分布式环境中，并发控制是不可避免的。k8s 继承了 Omega 的 MVCC 机制。通过 etcd 提供的 version 对资源进行版本控制。
- 关于开发组件的版本
	- 查看 k8s 的版本
		- ```
		  ❯ kubectl version -ojson
		  {
		    "clientVersion": {
		      "major": "1",
		      "minor": "24",
		      "gitVersion": "v1.24.0",
		      "gitCommit": "4ce5a8954017644c5420bae81d72b09b735c21f0",
		      "gitTreeState": "clean",
		      "buildDate": "2022-05-03T13:46:05Z",
		      "goVersion": "go1.18.1",
		      "compiler": "gc",
		      "platform": "darwin/arm64"
		    },
		    "kustomizeVersion": "v4.5.4",
		    "serverVersion": {
		      "major": "1",
		      "minor": "22",
		      "gitVersion": "v1.22.3",
		      "gitCommit": "c92036820499fedefec0f847e2054d824aea6cd1",
		      "gitTreeState": "clean",
		      "buildDate": "2021-10-27T18:35:25Z",
		      "goVersion": "go1.16.9",
		      "compiler": "gc",
		      "platform": "linux/arm64"
		    }
		  }
		  ```
	- k8s 与 client-go 之间的版本依赖关系
- ## Client-go
	- ![architecture](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20220713001120.png)
	- Informer
		- Informer 是基于 Watch 功能的封装，提供了缓存以及更加高级的编程接口。
		- Informer 会处理 Watch 的网络连接问题，比如错误重连等。
		- Informer 可以配置一个定时器，用于处理业务逻辑与内存缓存之间的差异：*通过调用已注册的事件处理函数，把完整的对象列表通知过去*。
		- 使用共享 Informer Factory 来创建 Informer，它可以减少对 APIServer 的负担。*对于一个 GroupVersionResource 只会生成一个 Informer*。
		- EventHandler
			- EventHandler是 Informer 的消费者
	- Lister
		- Lister 获取的对象是内存中的对象，如果要修改它的话，需要执行一次 DeepCopy。
	- WorkerQueue
		- ```go
		  type Interface interface {
		  	Add(item interface{})
		      Len() int
		      Get() (item interface{}, shutdown bool)
		      Done(item interface{})
		      ShutDown()
		      ShuttingDown() bool
		  }
		  ```
- API Machinery
	- 实现了 Kubernetes 的基础类型系统
	- Kind
		- API Machinery 中的 GroupVersionKind
	- Resource
		- API Machinery 中的 GroupVersionResource
		- 每种 GVR 对应一个 HTTP 路径
		- GVR 用于标识 KubernetesAPI 的 REST Endpoint
	- ![transfer](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20220712193706.png){:height 415, :width 250}
- CRD
	- CRD的 schema 定义编写规则(yaml)
	- 验证 CRD 的合法性
		- APIServer 会根据 OpenAPI v3 的Schema (JSON) 格式进行合法性验证
			- OpenAPI 的 Schema可以通过`Kubebuilder`来生成
		- 可以通过 Webhook 进行复杂验证
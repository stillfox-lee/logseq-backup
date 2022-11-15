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
- apimachinery、k8s api、client-go 的关系：
	- apimachinery 的资源
		- 实现了 Go 中基础的对象。例如`runtime.Object`,`runtime.ObjectKind`等。
		- 提供了 Watch 的功能
	- k8s api 的资源
		- 基于 apimachinery 实现了k8s 的内置资源对象。例如`pod`，`service`等。
	- client-go 里的资源
		- 基于 Watch 封装了 Informer 和 Lister。
- ## [[client-go]]架构
	- ![architecture](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20220713001120.png)
	- ## Client
		- RESTClient
		- Clientset
		- DynamicClient
			- 对 GVK 无感知。不使用`Scheme`和`RESTMapper`。开发者需要手动提供一个 GVR来获取资源。多用于处理不确定对象类型的通用 Controller，例如垃圾回收时，删除那些父对象已经不存在的对象。
		- DiscoveryClient
		- 强类型 Client
			- Client 代码中就带上了类型。
			- 由于很多重复代码，所以又有了生成代码的方案：client-gen。
			- 代码生成的工作：
				- 生成k8s 对象对应的 Go struct。
				- 将这些生成的资源再注册到 Scheme 中。
				- 再生成的 Clientset 就是带有强类型绑定的了。
		- controller-runtime
			- 可以对任意的资源进行查询，但是需要提供一个该资源已经注册过的 Scheme
	- Reflector
		- 实现了 `List` 和 `Watch` 的功能
	- Informer
		- Informer 的模型：
			- 从 APIServer 获得 Event
			- 提供一个 Lister 接口给客户端查询
			- 为 Add、Update、Delete 事件注册 handler 函数
			- 通过内部存储实现缓存
		- 特性
			- Informer 会处理 Watch 的网络连接问题，比如错误重连等。
		- 使用：
			- Informer 可以配置一个定时器，用于处理业务逻辑与内存缓存之间的差异：*通过调用已注册的事件处理函数，把完整的对象列表通知过去*。
			- 使用共享 Informer Factory 来创建 Informer，它可以减少对 APIServer 的负担。*对于一个 GroupVersionResource 只会生成一个 Informer，底层使用一个 Watch 连接*。
			- EventHandler
				- EventHandler是 Informer 的消费者
				- 通常只将修改过的对象放到`workqueue`中
		- Index
		- Lister
			- Lister 获取的对象是内存中的对象，如果要修改它的话，需要执行一次 DeepCopy。
	- WorkerQueue
		- ```go
		  type Interface interface {
		  	Add(item interface{})
		      Len() int
		      Get() (item interface{}, shutdown bool)  // 返回优先级最高的元素，会 block
		      Done(item interface{})	// 所有 Get 返回的元素，都需要调用 Done重新放回队列。
		      ShutDown()
		      ShuttingDown() bool
		  }
		  ```
		- WorkerQueue的数据类型转换原理。
			- WorkerQueue 的流转：
				- EventHandler 获取到 interface{} 的 object
				- EventHandler 将数据发送到 WorkerQueue 中
				- Worker 通过 Lister 获取到实际的 Resource。
- ## API Machinery
	- ### Kubernetes 的基础类型系统
		- Kind
			- API Machinery 中的 GroupVersionKind，是一个 object，对应着 Go 语言的一个类型。
			- 从 `GVR` 对应的 HTTP endpoint中获取到的数据，经过`REST MAPPING`之后，就是 `GVK`。
			- 对应的是 etcd 中存储的数据
		- Resource
			- 因为 Resource 是属于特定的Group、Version的，所以在API Machinery 中用 `GroupVersionResource` 表示。
			- 每种 GVR 对应一个 HTTP 路径。例如
			  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20221009172548.png)
			- GVR 用于标识 KubernetesAPI 的 REST Endpoint
		- RestMapper
			- 设想一下我们开发的一个场景：需要从 APIServer 中获取某个 GVK。那就需要知道这个 GVK 对应的 HTTP endpoint，发起一个 HTTP request，再将获取到的数据反序列化为 GVK。完成这个工作的就是 RestMapper。
			- RestMapper 是一个 interface，它有多种实现。最有用的就是`DeferredDiscoveryRESTMapper`，它可以带有`discovery information`，能够 **通过 APIServer 提供的查询接口**。例如我们执行`kubectl get pods`，并没有指定 Group 和 Version，它却可以自动发现，并且转换为 GVR。
		- Scheme
			- Scheme用于把 GVK 与 Golang 类型关联起来，通过反射机制来获得一个 Golang 对象。
				- Golang 类型需要通过如下方法来注册 Scheme
				  `scheme.AddKonwTypes(schema.GroupVersionKind{"", "v1", "Pod"}, &Pod{})`
				-
		- ![transfer](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20220712193706.png){:height 415, :width 250}
	- 提供 Watch 功能
		- ```go
		  // watch.Interface
		  type Interface interface {
		  	Stop()
		      ResultChan() <- chan Event
		  }
		  
		  // watch 返回的 Event
		  type EventType string
		  
		  const (
		      Added   EventType = "ADDED"
		      Modified EventType = "MODIFIED"
		      Deleted EventType = "DELETED"
		      Error EventType = "ERROR"
		  )
		  
		  type Event struct {
		      Type EventType
		      Object runtime.Object
		  }
		  ```
- ## CRD
	- CRD的 schema 定义编写规则(yaml)
	- 验证 CRD 的合法性
		- APIServer 会根据 OpenAPI v3 的Schema (JSON) 格式进行合法性验证
			- OpenAPI 的 Schema可以通过`Kubebuilder`来生成
		- 可以通过 Webhook 进行复杂验证
		- 整体流程
		  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20221009232738.png)
		-
	- 子资源
		- Status
		- Scale
- ## 自定义 API 服务器
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20221009232012.png)
	- 为什么需要自定义 API server？CRD 有什么问题解决不了？
	- 怎么实现自定义 API server？
	- kube-aggregator
	- apiextensions-apiserver —— 为 CRD 提供服务
	-
- ## CI
	- 单元测试
	- 集成测试
	- 构建 image
	- 完备性测试
	- 冒烟测试
	- 性能测试。给出资源占用的数据
	- 浸泡测试，发现长期使用的场景下是否有资源泄漏。
- ## 可观测性
	- 日志
	- metrics
	- k8s audit log
- ## 注意事项
	- 使用 `Deployment`或者`DaemonSet`来部署
	- 考虑主、从冷备模型，**如何做好状态管理**。
	- 访问控制
	- 监控和日志
	- checklist
		- 合适的准入控制 RBAC
		- 管辖范围。*RC 是一个 namespace 还是多个 namespace*
		-
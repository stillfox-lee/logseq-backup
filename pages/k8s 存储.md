## 基础概念
	- PV
	- PVC
	- StorageClass
		- 通过 StorageClass 将 PV 与 PVC 绑定，同时 StorageClass 可以指定一个存储插件（Provisioner）
	- Provisioner（存储插件）
		- in-tree out-of-tree的区别：out-of-tree 就是非 k8s 内置的插件。
		- 支持 Dynamic Provisioning 或者是 Static Provisioning
	- 生命周期相关
		- 1: Provision
		- 2: Bind
			- 当cp上的控制循环注意到PVC时，绑定就发生了，PVC包含一个存储量、访问请求和可选的StorageClass。观察者找到一个匹配的PV或等待StorageClass供应者创建一个。该PV必须至少与请求的存储量相匹配，但可以提供更多。
		- 3: Use
			- Volume 绑定完成之后，就会 mount 到Pod 中使用
		- 4: Release
			-
		- 5: Reclaim
- ### Volume
	- 分类
		- EmptyDir
		- HostPath
		- Projected
- ### Projected Volume
	- 分类
		- Secret
		- ConfigMap
		- Downward API      —— 让 Pod 容器能够直接获取到 Pod API 本身的信息
		- ServiceAccountToken
- ### Secret
	- > Secret 实际上是存储在 Etcd 里面的数据，默认情况下是 base64 编码后存储，也可以通过`Key Management Service`来加密存储。
	- #### 创建 Secret
		- Secret 也是一种 k8s 的资源，可以通过 kubectl 创建：
		  ```bash
		  $ kubectl create secret generic user --from-file=./username.txt
		  $ kubectl create secret generic pwd --from-file=./password.txt
		  ```
		  *（这种方式的一个好处是：命令行里面不会显示出具体的值）*
		- 也可以通过 YAML 来创建：
		  ```yaml
		  apiVersion: v1
		  kind: Secret
		  metadata:
		    name: mysecret
		  type: Opaque
		  data:
		    user: YWRtaW4=             # 需要 base64 编码
		    pass: MWYyZDFlMmU2N2Rm
		  ```
		- 为 Secret 加密
			- > Secret 默认是使用 base64 编码的，有时候我们希望为 Secret 加密之后存储。k8s 支持通过 KMS 为 Secret 加密，再存储至 etcd，待使用时，再通过 KMS 解密。
			- 以 Google Cloud KMS 插件为例，创建一个加密的 Secret：
			- ```bash
			  kubectl create secret generic my-secret \
			  --from-literal=password=mypassword \
			  --kms-provider=gcp-kms \
			  --kms-key=projects/my-project/locations/global/keyRings/my-key-ring/cryptoKeys/my-key
			  
			  ```
	- #### 使用Secret
		- 通过 Volume
			- ```yaml
			  apiVersion: v1
			  kind: Pod
			  ...
			  spec:
			    containers:
			    ...
			    valumeMounts:
			    - name: mysql-credi
			    	mountPath: "/projected-volume"
			      readOnly: true
			    volumes:
			    - name: mysql-credi
			    	projected:
			        source:
			        - secret:			# k8s 的 secret API 资源
			        	  name: user
			        - secret:
			        	  name: pwd
			  ```
		- 通过环境变量
			- ```yaml
			  spec:
			    template:
			      spec:
			        containers:
			        - name: my-container
			          env:
			          - name: MY_SECRET_VAR
			            valueFrom:
			              secretKeyRef:
			                name: my-secret
			                key: password
			  
			  ```
			- > 设置一个 ilike 的环境变量，值是从 Secret `my-secret`中，key 为 password 中获取的。
- ReclaimPolicy 控制 PV、PVC 的行为
	- Retain： 在 PVC 删除的时候，Volume不被删除
	- Delete (by default)：在 PVC 删除的时候，Volume 会被删除
	- Recyle：在 PVC 删除的时候，Volume 会被回收，可以重用，但是数据会清除。
- 使用 ResourceQuota 限制 PVC
- PV AccessMode
	- ReadWriteOnce —— 只能被一个 node 挂载，读写
	- ReadOnlyMany —— 多个 node 挂载，只读
	- ReadWriteMany —— 多个 node 挂载，读写
	- ReadWriteOncePod —— 只能被一个 Pod 挂载，读写。*粒度不一样了*
- ## PV、PVC 、StorageClass
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202405071234984.png)
	-
	- PV 和 PVC 通过 `storageClassName` 关联。
	- 工作流程：
		- Attach
		- Provision
- ## CSI
	-
- [凤凰架构](http://icyfenix.cn/immutable-infrastructure/storage/storage-evolution.html)
	- docker 的存储演进过程
		- 最初只有`Bind`的概念。docker 只是为宿主机的磁盘在内部做了一个映射而已。没有管理、隔离。
		- 简单的映射不实用——如果需要跨主机的共享存储，需要在宿主机先挂载，再`Bind`。很不方便。
		- 通过抽象出`Volume`和`Volume Driver`的概念。将通过抽象出卷和驱动，一来满足了自身的存储管理需求，二来丰富了生态。
	- k8s 的存储设计 #k8s
		- `Volume` 与 pod 生命周期相同。pod 内的 container 可以共享。
		- `PersistentVolume` 持久化的存储。
		- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20220708171818.png)
		- `Local PersistentVolume`。磁盘 IO 性能需求高的情况下，需要运用本地的磁盘。所以抽象了这个资源。但是，这个会影响到**调度**逻辑。在 k8s 调度的时候只能把 pod 调度到具有本地磁盘的 node 中。
		- `PersistentVolumeClaim`。PV 是设计给运维的，PVC 是设计给开发者的。
			- PV —— 对于存储的具体描述。容量、访问模式、存储位置等
			- PVC —— 声明需要的存储能力。容量、访问方式等。
		- 动态存储分配
- [CSI 驱动开发指南](https://mp.weixin.qq.com/s/jUlTHAKhHZD1dkNudPlS9w)
-
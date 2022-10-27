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
	- Secret
		- Secret的使用方式
		  ```yaml
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
			- Secret 也是一种 k8s 的资源，可以通过 kubectl 创建：
			  ```bash
			  $ kubectl create secret generic user --from-file=./username.txt
			  $ kubectl create secret generic pwd --from-file=./password.txt
			  ```
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
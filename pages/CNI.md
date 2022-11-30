- ## CNI spec
	- 名词
		- runtime —— 负责执行 CNI plugin 的程序
		- plugin —— 负责应用 `Network configuration`生效的程序
		- protocol —— runtime 向 plugin发送请求的协议
		- Pod [CIRD](https://zh.wikipedia.org/wiki/%E6%97%A0%E7%B1%BB%E5%88%AB%E5%9F%9F%E9%97%B4%E8%B7%AF%E7%94%B1) —— 基于可变长度子网掩码，指定一个范围的的IP 地址。如 `192.168.2.0/24`
	- ## Summary
		- A format for administrators to define network configuration.
		- *提供给网络管理员的一个 network configuration，让管理员可以定义网络。*
		- A protocol for container runtimes to make requests to network plugins.
		- *一个容器 runtime 向 network plugin 发送请求的协议。*
		- A procedure for executing plugins based on a supplied configuration.
		- *一个按照配置执行的 plugin 程序。*
		- A procedure for plugins to delegate functionality to other plugins.
		- *一个可以将部分功能delegate 给其他 plugin 的程序。*
		- Data types for plugins to return their results to the runtime.
		- *plugin 返回给 runtime 的结果的数据类型*
	- ## Section 1: Network configuration format
		- TODO 配置文件各个部分的意义？
		- 让管理员可以对网络进行定义。其中包含了多种提供给 runtime 和 plugin 的`directives`。`runtime`负责解析并格式化后传递给`plugin`。 **(通过 protocol 传递)**
		- 通常情况下，这个是一个静态配置。
		- ### Configuration format
			- 配置文件是一个 JSON 文件，主要有以下的 key：
				- `cniVersion` (string) ：指定版本信息，当下最新的是“1.0.0”
				- `name` (string)：Network name，在网络环境中是唯一的。命名有特定的格式要求。
				- `disableCheck` (bool)：如果为 true，则runtime 不会调用`CHECK`命令检查这个配置列表。可用在一个已知会返回虚假错误的的 plugin。
					- #+BEGIN_CAUTION
					  不太理解这个使用场景，可能需要结合 CHECK 来看。
					  #+END_CAUTION
				- `plugins` (list)：一份 CNI 插件的列表和对应的配置信息，同时也被称为`plugin configuration objects`
			- #### Plugin configuration objects
				- Plugin configuration objects 可能会有这里未定义的字段。runtime 必须不做修改地将这些字段传递给 plugin。
				- **Required keys:**
					- `type`  (string): CNI 插件的名字(安装在机器上的 plugin二进制程序的名字)。runtime 会按照这个名字去匹配对应的 plugin 程序。
				- **Optional keys, used by the protocol:**
					- `capabilities`  (dictionary): 参考： ((63772bb6-c284-4910-a80e-54c8e048b452))
				- **Reserved keys, used by the protocol:**  这些 key 是由 runtime 在运行时生成的，所以不应该在 configuration 里面被使用。
					- `runtimeConfig`
					- `args`
					- Any keys starting with  `cni.dev/`
				- **Optional keys, well-known:** These keys are not used by the protocol, but have a standard meaning to plugins. Plugins that consume any of these configuration keys should respect their intended semantics. 这些 key 不会被 `protocol` 使用，但是对于 plugin 是有意义的。任何使用这些 key 的 plugin 都需要遵循配置原本的语义。
					- TODO 这里是字面翻译，不太理解实际的意思是什么。
						- 既然要通过 protocol 来传递，为什么*are not used by the protocol*？
						- 需要进一步理解之后，再来翻译一遍
					- `ipMasq`  (boolean): If supported by the plugin, sets up an IP masquerade on the host for this network. This is necessary if the host will act as a gateway to subnets that are not able to route to the IP assigned to the container. 如果插件支持，在主机上为这个网络设置一个 *IP masquerade*。
						- 主机作为容器的网关，需要使用一个虚拟 IP 来替换数据包中的容器 IP。具体的工作原理可以参考[ip-masq-agent](https://kubernetes.io/docs/tasks/administer-cluster/ip-masq-agent/)
					- `ipam`  (dictionary): IPAM 配置 (IP Address Management) 。配置用于管理容器的 IP 地址。:
						- `type`  (string):  指定 IPAM 插件。如：dhcp、host-local etc
					- `dns`  (dictionary, optional): Dictionary with DNS specific values:
						- `nameservers`  (list of strings, optional): list of a priority-ordered list of DNS nameservers that this network is aware of. Each entry in the list is a string containing either an IPv4 or an IPv6 address.
						- `domain`  (string, optional): the local domain used for short hostname lookups.
						- `search`  (list of strings, optional): list of priority ordered search domains for short hostname lookups. Will be preferred over  `domain`  by most resolvers.
						- `options`  (list of strings, optional): list of options that can be passed to the resolver
			- #### 一个配置例子
				- ```
				  {
				    "cniVersion": "1.0.0",
				    "name": "dbnet",
				    "plugins": [
				      {
				        "type": "bridge",
				        // plugin specific parameters
				        "bridge": "cni0",
				        "keyA": ["some more", "plugin specific", "configuration"],
				        
				        "ipam": {
				          "type": "host-local",
				          // ipam specific
				          "subnet": "10.1.0.0/16",
				          "gateway": "10.1.0.1",
				          "routes": [
				              {"dst": "0.0.0.0/0"}
				          ]
				        },
				        "dns": {
				          "nameservers": [ "10.1.0.1" ]
				        }
				      },
				      {
				        "type": "tuning",
				        "capabilities": {
				          "mac": true
				        },
				        "sysctl": {
				          "net.core.somaxconn": "500"
				        }
				      },
				      {
				          "type": "portmap",
				          "capabilities": {"portMappings": true}
				      }
				    ]
				  }
				  ```
	- ## Section2: Execute Protocol
		- ### Overview
			- CNI 规范定义的一个 protocol，描述了`plugin`二进制程序和`runtime`之间如何交互。
			- `plugin`需要负责配置容器的网络接口，plugin 主要可以分类为：
				- `Interface` 插件，在容器内创建网络接口设备，并且保证它是可连通的。
				- `Chained` 插件，根据配置调整*已创建*的网络接口设备。
			- `runtime`通过环境变量和配置文件将网络 parameters 传递给`plugin`。`plugin`通过 stdin 接收配置。`plugin`执行成功返回`result`到 stdout，如果失败则将错误信息返回到 stderr。
		- ### Parameters
			- 协议参数通过 OS 的环境变量传递：
				- `CNI_COMMAND` : 指明操作：`ADD`、`DEL`、`CHECK`、`VERSION`
				- `CNI_CONTAINERID`： runtime 分配的容器唯一 ID 值。不可为空。有格式要求。
				- `CNI_NETNS`：指向容器的**isolation domain**。*如果是使用`namespace`隔离的话，那就是`/run/netns/[nsname]`*。
				- `CNI_IFNAME`：在容器内部创建的网络接口的名称；如果plugin 无法使用这个网络接口的话，需要返回**error**。
				- `CNI_ARGS`：在**调用期**可以另外携带的参数。格式类似于：`FOO=BAR;ABC=123`。
				- `CNI_PATH`：CNI plugin 可执行文件的路径列表。
		- CNI定义了 4 种操作，它们都是通过环境变量`CNI_COMMAND`来传递的
			- `ADD`
				- 将容器加入网络，或者修改配置。如果CNI plugin 收到 ADD 命令，需要做其中一件事：
					- 在`CNI_NETNS`中，创建一个由`CNI_IFNAME`定义的网络接口；
					- 在`CNI_NETNS`中，根据配置调整`CNI_IFNAME`接口
				- 如果执行成功，则 plugin 需要返回`result`到标准输出。
			- `DEL`
			- `CHECK`
			- `VERSION`
			-
	- ## Section 3: Execution of Network Configurations
		- 本节描述runntime如何解析网络配置（如section 1中定义的）并相应地执行plugin。runtime可能希望ADD、DEL或CHECK容器中的网络配置，同时会相应地调用 plugin 执行 ADD、DELETE 或 CHECK 。
		- 本节还定义了网络配置是如何被转换并提供给插件的。
		- 容器上的网络配置的操作被称为`attachment`。一个attachment可以由（CNI_CONTAINERID, CNI_IFNAME）元组唯一地识别。
		-
		- ### Lifecycle & Ordering
			- runtime 负责在调用任何 plugin 之前，为容器创建 Network namespace
			- runtime 对同一个容器的操作不能并发执行，但是不同容器可以
			- plugin 必须能够响应不同容器之间的并发调用。如果需要的话，可以为共享资源加锁 *(例如 IPAM 数据库)*
			- runtime 必须保证`Add`之后，最终会有`Delete`调用。
			- runtime 的`Delete`调用可能会有多次。
			- **Network configuration** 在`Add`和`Delete`之间不能被修改
			- **Network configuration** 在*Attachments*之间不能被修改
		- ### Attachment Parameters
			- **Container ID:**  由runtime创建的一个唯一 ID，通过`CNI_CONTAINERID` 参数来设置。
			- **Namespace**:  指定 domain 隔离策略的位置。如果使用network namespace，那么就是network namespace 在宿主机上的文件路径（例如：`/run/netns/[nsname]` ）。通过`CNI_NETNS`参数传递的 plugin。
			- **Container interface name**:  容器内部创建的网络接口的名字。通过 `CNI_IFNAME` 参数设置。
			- **Generic Arguments**: 额外参数，格式是key-value字符串对。通过参数  `CNI_ARGS`  设置.
			- **Capability Arguments**: These are also key-value pairs. The key is a string, whereas the value is any JSON-serializable type. The keys and values are defined by [convention](https://www.cni.dev/docs/conventions/) .
			- 同时，runtime 还需要提供一个 list，用于搜索 CNI plugins。需要在调用 CNI plugin 时通过`CNI_PATH`环境变量传递。
		- ### Adding an attachment
		- ### Deleting an attachment
		- ### Checking an attachment
		- ### Deriving execution configuration from plugin configuration
			- #### Deriving    `runtimeConfig`
			  id:: 63772bb6-c284-4910-a80e-54c8e048b452
	- ## Section 4: Plugin Delegation
		- 有些功能，在**chainde plugin**没有实现，就需要通过代理其他的 plugin 来完成。最常见的就是 IP 地址管理。
		- SPEC 为此定义了第三种插件：IP Address Management Plugin (IPAM plugin)。
		- IPAM 插件必须决定网络接口的 *IP/subnet*, *Gateway*, *Routes*，并且将这些信息返回给**Main**插件。
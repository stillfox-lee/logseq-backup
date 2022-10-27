- ## CNI spec
	- 名词
		- runtime —— 负责执行 CNI plugin 的程序
		- plugin —— 负责应用 `Network configuration`生效的程序
	- ### Network configuration
		- 让管理员可以对网络进行定义。`runtime`负责解析并格式化后传递给`plugin`。
		- 通常情况下，这个是一个静态配置。
		- 一个配置例子
			- {
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
	- ### Execute Protocol
		- CNI 规范定义的，`plugin`二进制程序和`runtime`之间如何交互。
		- `plugin`主要职责
			- `Interface` 插件，在容器内创建网络接口设备，并且保证它是可连通的。
			- `Chained` 插件，根据配置调整*已创建*的网络接口设备。
		- `runtime`通过环境变量和配置文件将网络参数传递给`plugin`。`plugin`通过 stdin 接收配置。`plugin`执行成功返回`result`到 stdout，如果失败则将错误信息返回到 stderr。
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
					-
			- `DEL`
			- `CHECK`
			- `VERSION`
			-
	- ## Execution of Network Configurations
		- 本节描述runntime如何解释网络配置（如第 1 节中定义的）并相应地执行plugin。runtime可能希望ADD、DEL或CHECK容器中的网络配置。这将导致一系列相应的插件 ADD、DELETE 或 CHECK 的执行。本节还定义了网络配置是如何被转换并提供给插件的。
		- 容器上的网络配置的操作被称为`attachment`。一个attachment可以由（CNI_CONTAINERID, CNI_IFNAME）元组唯一地识别。
		-
		-
		-
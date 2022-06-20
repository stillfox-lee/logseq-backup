- # Primary/Backup Replication
  
  核心是关于容错 #fault-tolerance 。通过 #replication ，无论是server 或者是网络的错误发生，都能够保障系统的**可用性**。
- replication 可能应对的错误有
	- 单个镜像服务器的各种故障宕机：
		- 机器风扇故障
		- CPU 超载
		- 掉电
		- 硬盘写满
	- 非软、硬件的错误
		- 级联故障（所有 replicas 都失效）
		- 地震
		  
		  > 需要考虑的一个问题是：是否值得花这么高的成本来维持大量的 replicas？
- 两种主要的replication 方式：
	- **state transfer**
	- primary 对外提供服务
	- primary 负责将状态发送给 backup
- **Replicated state machine**
	- client 的请求发送给 primary
	- primary 将请求操作按序发送给 backup
	  
	  > 确定性状态机：如果所有的 replicas 初始 state 相同，它们都按照同样的顺序执行client请求，都执行了所有的client请求。那么，所有的 replicas 可以保持一样的状态。
	  
	  *由于 Replicated state machine 需要指令按序执行，如果在多核系统中，这点可能无法得到保证。*
	  
	  可见，Replicated state machine 要能够工作，是有一系列的前提条件的。
- **两者对比**
	- State transfer 比较简单。但是，需要通过网络发送的 state 数据量可能是很大的。
	- Replicated state machine 的网络传输较小，每次都是发送执行的指令数据。
	- Replicated state machine 比较难实现
- VM-FT 使用 Replicated state machine 实现。
  Replicated state machine 实现 Fault-Tolerant的相关问题：
- 什么 state 应该被复制，同步的状态应该在控制什么粒度？
	- 应用基本：如MySQL 只同步 binlog。
	- 机器级别：VMware 在 CPU 指令、客户端网络请求等级别同步。
- primary 需要同步等待 backup 吗？
- 怎么实现 primary backup 之间的替换？
- 如何快速替换新的 backup上线？
## VM-FT 的实现
### 概述

利用 Replicated state machine 来实现 Primary 和 Backup 的一致性，同时通过设计一个 failure over 机制，为 VMware 虚拟机提供了Fault-Tolerant特性。

通过 VMM 拦截所有对 Primary 虚拟机operation（网络请求、CPU 中断等），并且将 operation 封装为一个 Log entry 发送给 Backup 虚拟机。Backup 虚拟机通过重放（replay）以达到和 Primary 一致的状态。



![image-20220422153310429](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/image-20220422153310429.png)
### VM-FT 如何实现 Replicated state machine

Replicated state machine 的要求：
- 节点初始状态一致
- 节点按照一致的顺序执行 operation
- 节点执行一样的 operation
  
  
  
  > 所以，在 VM-FT 中重点需要处理的就是保证 operation 能够在 Primary 和 Backup 之间按序、一致得到执行。
  
  
  
  在 VMware 虚拟机中，会遇到哪些 operation？
  
  Primary 节点什么时候应该发送信息给 Backup？
- 当遇到任何会引起 Primary 和 Backup不一致的时候
- 执行任何具有不确定性的指令的时候
  
  有哪些会引起不一致的 operation需要FT 协议处理？
- Backup 节点
- 不执行任何的指令
- 只执行 Logging Channel 发送的指令
### 确定性和不确定性事件
#### 不确定性operation

Primary 是主要对client 提供服务的，它会接受 client 的各种请求，这些 operation 包含`确定性 operation`和`不确定性 operation`。

**确定性 operation**

类似于纯函数，在 Primary 和 Backup 执行都是一样的。可以直接在 Backup replay。

**不确定性 operation**
- 随机数
- 获取当前时间戳
- 生成 UUID 等
  
  这些指令，需要在 Primary 产生了结果之后，将结果写入 log entry 再发送给 Backup。而 Backup 不需要执行 operation，直接用 log entry 的结果即可。
  
  > 还有一种不确定性 operation，就是多核并发程序，VMware-FT 不兼容多核并发。VMM 只单核运行。
#### VMM 

VMM（Virtual Machine Monitor），VMM 介于硬件和虚拟机之间。实现了很多特性以达到虚拟机高可用。

**VMM 处理 operation**

在 VMware 的场景下，因为是在机器硬件级别提供的软件服务。所以primary 会接受到很多具有不确定性的 operation。例如：定时器中断、生成随机数等。所以，VMware 设计了一个VMM，VMM 处于硬件和虚拟机操作系统之间，通过 VMM 可以拦截所有的指令，在硬件级别可控制。将具有**不确定性**的 operation 转化为**确定性**operation。

**VMM处理虚拟机存储**

FT 通过一个局域网内的 Shared disk server 为虚拟机提供存储服务。VMM 提供了一个抽象的本地存储接口，虚拟机的本地存储实际上是通过 VMM 由网络发送到 Shared disk server 实现的。
#### 处理 operation
##### 对于CPU 的定时器中断

目标：Primary 和 Backup 应该在指令流的同一时刻看到中断指令

Primary 的执行流程：

1. VMM 获取到定时器中断
2. VMM 从 CPU 中读取指令

![image-20220423232612692](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/image-20220423232612692.png)

这也意味着，当 Primary 的 Log entry 发送过来之前，Backup 是无法执行任何操作的。
#### Log entry
- 指令序号——Primary 执行的指令的序号
- 类型——网络指令、非确定性指令等
- 数据——网络指令则是网络数据；如果是非确定性指令则是 Primary 执行的结果值
  
  对于确定性的指令和不确定性的指令。Backup 接收到确定性指令的话，VMM 会在 Backup 中重放执行。如果是非确定性指令，VMM 则不会让 Backup 运行。而是直接将 结果作为 Backup 执行的结果值。
### 输出规则

因为在Replicated state machine 中可能会出现各种问题导致无法同步状态。所以，设计了一个 VM-FT 的对外（client）输出规则。满足这个规则的情况下，可以避免因为 fail over 导致整个 VM-FT 系统对外的不一致状态。
- #### Failure over
	- 探活：
- Primary 和 Backup 之间 heartbeat
- Logging Channel 之间的流量监控
  
  如果发现 Primary失效了，就将 backup 切换为 Primary。
  
  Backup 启用之前，需要将 buffer 中的 Log entry 全部执行完。
  
  由于网络工作在 TCP 层，如果在 failure over 期间发生了 Backup 发送了重复的网络请求，TCP 会当做重复包丢弃。
- ### VMware-FT 如何解决脑裂问题 #split-brain
	- 通过使用一个Test-And-Set disk server，来提供一个原子性的保证。用于确定一个唯一的 Primary。
	- TODO Test-And-Set disk server是如何工作的？
-
- ### 关于 VMware-FT 的疑问 #TODO 
  
  **Primary 与 Backup 之间的operation 发送是否需要同步？**
  
  从不确定operation的场景出发：需要等待 Primary 执行完之后，再将产生的数据写入 Log entry 发送给 Backup。那么这个情况下是需要阻塞等待 Primary 执行结果在发送 Log entry 的。
  
  确定性operation的场景：即使Backup 和 Primary 执行 operation 的效果是一样的。但是，为了要保证 Primary 和 Backup 都一致；Log entry 需要确定了 Primary 已经执行完 operation 之后，再发送给 Backup 执行。如果 Primary 执行失败的话，这个 operation 就不应该发送给 Backup 了。
  
  所以，Primary 的 operation 执行与 log entry 的发送应该是同步阻塞的。
- 怎么做同步？
	- 什么操作需要同步？
- 怎么做 failure over？
	- 如何确认Primary故障？
	- Primary 与 Backup 的心跳
	- Primary 会拦截 CPU 的定时器，发送中断指令给 Backup。如果长时间没有定时中断可认为 Primary 故障。
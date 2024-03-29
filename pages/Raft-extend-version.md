- [中文翻译版论文](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)
- # Raft 论文摘要 #paper #[[Distributed System]]
	- > Raft 是一种为了管理 Replicated log 的一种 #共识算法 Raft 主要包含了一下几个模块：**领导人选举**、**日志复制**和**安全性**。它通过一个更强的一致性来减少需要考虑的状态数量。
	- ## 复制状态机 #[[State Machine Replication]]
		- ![image.png](../assets/image_1655709613987_0.png)
		- 通过一个**共识模块**，将 client 的请求存储为一些列的日志。将这些日志按照同样的顺序，在集群的其他节点重新执行，这样就能实现集群中的节点都有一致的状态了。
	- ## Raft 的共识算法介绍
		- > Raft 通过选举一个强力的 leader，然后赋予它权利来管理 Replicated log 来实现一致性。leader 从客户端接受请求，转化为 `log entries`，再复制到其他的服务器上。同时，它可以控制`log entries`的顺序，什么时候其他服务器可以执行重放。这个强力 leader 如果发生了故障的话，集群会重新选举出一个新的 leader。
		- ### leader 选举
		- ### 日志复制
			- 流程
			  1. 成为 Leader，开始响应客户端请求；
			  2. 记录 `log entries`，并且发送 `AppendEntries`；
			  3. 等到`log entries`被**安全复制**，Leader 将这个日志应用到自己的 [[State Machine Replication]]中；
			  4. 返回请求响应给客户端
			- ![log entries](https://github.com/maemual/raft-zh_cn/raw/master/images/raft-%E5%9B%BE6.png)
			- > 方框中的数字是 Term。
			- `log entries`的构成：Term 和操作日志
			- `log entries`的状态：
				- committed
		- ### 安全性
- # 个人整理
	- Raft 中和时间有关的部分
		- election timeout
			- follower 超过这个时间没有收到 heartbeat 就会变为 candidate，发起 election。
			- 论文限定范围是 `150ms~300ms`
			- > 主要是为了多个解决 Follower 都同一时间过期，防止它们瓜分选票。一直无法完成选举。
		- heartbeat 间隔时间
			- leader 周期性向其他节点发送心跳。
			- > 主要目的是：同步日志信息；保持自己的 leader 地位。
	-
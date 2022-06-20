- # Redis集群高可用的学习笔记
- Redis 的集群有两种类型：
	- Sentinel 的集群
	- Cluster 的集群
- ## Sentinel 集群
- 关于 Sentinel 集群的知识：
	- 最少部署 3 个节点
	- quorum——用于对 Master 做故障检测
		- 一旦Sentinel 检测到 Master 故障，则会向整个 Sentinel 集群对 Master 的故障进行检测。一旦满足 quorum 数量的 Sentinel 都认为 Master 故障。那么Sentinel 集群则认为 Master 故障了，开始执行 failover。
	- Sentinel Majority vote
	  
	  quorum
	  
	  用于对 Master 进行下线检测，只有达到了这个值，才能把 Master 标记为下线。
	  
	  Majority
	  
	  用于failover。在 Master 标记为下线之后，Sentinel 需要通过 Majority vote，选出一个 leader。由 leader 进行 failover 的处理。
- [ ] Sentinel 集群的故障节点容忍数是多少？
- [ ] 为什么需要至少 3 个节点？
  
  
  
  基数个数节点主要是为了对抗**网络分区**的问题。
## cluster 的实现
- slot 的分配算法
- slot 路由表的维护
- 节点的维护（上线、下线）、迁移过程
- hash tag
## 哨兵集群高可用
- Master 与 Slave 之间的异步复制
- Sentinel 集群负责**监控**、**通知**、**故障转移**
  
  
  
  Sentinel集群部署注意事项：
- Sentinel集群个数不能少于 3 个*（投票机制的约束条件）*
- Sentinel 节点最好是分布在相互隔离的环境中
  
  
  
  集群部署流程：
  
  1. 启动 Master 节点
  2. 启动多个 Slave 节点，向 Master 注册异步复制
  3. 启动多个 Sentinel 节点
  4. Sentinel 向 Master 建立网络连接
  5. Sentinel 向 Master 获取INFO 数据，建立网络拓扑结构
  6. Sentinel 与 Slave 建立连接
  7. Sentinel 之间建立网络连接
#### 集群的故障转移

Sentinel 的监控机制：Sentinel 会周期性与其他的 Master、Slave、Sentinel 实例发送 PING，以检测实例是否在线。



Master 的下线检测
- Sentinel PING指令未收到回复，且时长超过配置的阈值的话，Sentinel 实例会认为 Master `主观下线`；
- Sentinel 实例与集群内其他节点同步信息，以确定 Master 真实下线；
- 如果集群中认为 Master 下线的 Sentinel 节点数，达到了配置的阈值（quorum），则 Master 变为`客观下线`；
- 开启故障转移流程；
- 集群内Sentinel实例开始选 Leader；*如果 Sentinel 获得半数以上投票，则成为 leader* （Majority vote）
- Leader Sentinel 开始在集群中所有 Slave 中选主；
- Leader Sentinel 通知其他 Slave 新 Master
- Slave 与新 Master 开启主从
## 脑裂问题

什么是脑裂：

集群中出现了两个 Master 。两个节点都能接收写请求，客户端会向两个 Master 都写入数据。



什么情况下会发生：

这个情况有可能会在集群做主从切换的时候发生。旧 Master 在 sentinel 判定为下线，且未完成重新选主切换的时候，又再次恢复了。这样就会短暂的出现两个 Master 同时工作。



如何解决脑裂问题：

Redis 提供两个配置项：
- Min-slaves-to-write：Master 至少要有配置数量的 slave 才能进行数据同步；
- Min-slaves-max-lag：主从同步时，slave 给 master 发送 ACK 消息的最大延迟。
  
  将两个配置项配合一起使用，就可以适当解决脑裂问题。当 Redis 的 Master 不满足这两个条件的话，这个 Master 就无法对外提供服务。
### HashTag

在集群中可以使用 HashTag 方式给 key 加上一个 HashTag。例如：`user:profile:2122`加上HashTag 之后变成：`user:profile:{2122}`。这样在 Redis 计算 Slot 的时候，只会用HashTag 里的值计算。这样，不同的 key 也可以被分配到同一个 Slot 中。
#### 使用场景

主要使用在事务操作和范围查询。

事务支持：RedisCluster 不支持跨实例的事务。所以，通过 HashTag 可以把数据放在同一个实例中，这样就可以使用事务的特性了。
#### HashTag 的问题

容易造成数据倾斜。
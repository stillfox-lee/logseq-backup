- From [wikipedia](https://en.wikipedia.org/wiki/Quorum_(distributed_computing)):
	- > A **quorum** is the minimum number of votes that a distributed transaction has to obtain in order to be allowed to perform an operation in a [distributed system](https://en.wikipedia.org/wiki/Distributed_system). A **quorum-based** technique is implemented to enforce consistent operation in a distributed system.
	- WAITING Quorum-based voting in commit protocols
	- WAITING Quorum-based voting for replica control
	-
- From [martinfowler](https://martinfowler.com/articles/patterns-of-distributed-systems/quorum.html):
	- > Avoid two groups of servers making independent decisions, by requiring majority for taking every decision.
	- ### 问题引入：  
	  在分布式系统中，为了做到高可用，通常会引入 replicas 集群。但是，这会带来一个问题：在集群中一个server 收到了 update 请求的时候，它需要和其他的 replica server 同步这个 update 操作。那么我们需要多少个 replica server 确认这个 update，集群才能达成 ^^共识^^ ？如果数量太多，这个同步操作会造成很大的 delay。如果这个数量不够，这个 update 的结果可能就会因为 replica 故障而丢失。这导致我们需要在系统性能和系统可用性之间做出 trade-off。
	- ### 解决方案：
		- 我们把集群能够对update 达成^^共识^^的 replica 数据称为`quorum`。假设我们具有 5 个 节点的集群，那么`quorum`的数量则是 3(`quorum=n/2+1`).
		- 集群的`quorum`的数量要求，同时也指明了集群的*故障容忍度*。一个 5 节点的集群，可以容忍 2 个节点的故障。通常来说，如果我们的集群系统需要能容忍`f`个节点的故障，那么集群的数量需要是：`2f+1`。 #fault-tolerance
		- #### 如何确定集群中服务器的数量
			- 集群只有在大多数节点都正常工作的时候才能对外提供服务。在集群系统内部，replica 之间在处理数据复制，这点主要需要考虑两个问题：
				- **写入操作的吞吐**  
				  每次向集群写入数据，都需要复制到多个节点。每次增加新的节点，都会对这个写入同步带来负担。
				- **集群能够容忍的故障数量**  
				  集群节点的数量决定了它可以容忍的故障数量。但是，为集群增加新的节点并不一定会增加故障的容忍量：为 3 节点的集群增加一个节点并不会增长故障容忍度。
			- 考虑到上面两个原因，多数的**quorum-based**系统一般都持有 3 或者 5 个节点。一个 5 节点集群能够容忍 2 个节点的故障，同时能够支撑每秒上千的 QPS 请求。
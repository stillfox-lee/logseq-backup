- #resilient
- ref: https://aws.amazon.com/cn/blogs/architecture/exponential-backoff-and-jitter/
-
- ## 什么是 OCC
	- 乐观并发控制（Optimistic concurrency control，简称OCC）是一种经典的方法，用于多个写入者安全地修改单个对象，而不会丢失。 OCC具有三个好处：只要底层存储可用它就会被处理，易于理解，易于实现。
- ## OCC在并发场景下的问题
	- 但是，在多个链接高并发的场景下，它的耗时是比较严重的。例如：多个 client 同时更新数据库中的一行数据，由于同时只有一个 client 被正确处理，所以总体的处理时间会随着 client 的数量线性增长。
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031003027.png)
	- 由于每一次数据库只能响应一个 client。所以，如果有 N 个 client 并发，第一轮中就会有 N 个 client 竞争；第二轮就会有 N-1 个 client竞争。这是一个**指数级**的关系。
	- 还可以换一个视角：由于 OCC 需要做 Retry 来完成写入，所以，我们的系统会收到大量的调用：
	  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031102494.png)
	-
- ## 引入Backoff优化
	- 每一轮都有很多的 client 在竞争是很浪费的，所以我们需要引入一个方法来减少竞争。那就是——Backoff。
	- 为 client 引入一个 sleep 时间，计算公式为`sleep = min(cap, base * 2 ** attempt)`。其中 cap 是最长的等待时间，base 是基础等待时间，attempt 是尝试的次数。
	- ### backoff 的问题
		- 引入了`指数级 Backoff`之后，调用数量减少了，但是效果不是特别明显：
		  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031017203.png)
		- 从时间维度来看会更加明显：虽然是加入了 backoff，但是client 的并发还是聚集在一起，而且中间有一些时间是空闲的。
		  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031043773.png)
- ## 引入 jitter
	- 为了优化 backoff 的问题，我们希望：中间的空闲时间可以利用起来，同时减少同一时刻的 client 竞争数量。让总体看起来平滑一些。
	- 为此可以引入一个抖动（FullJitter）`sleep = random_between(0, min(cap, base * 2 **attempt)`
	- 效果如下：除了开始阶段竞争数量较大，减少了空闲时段。整体平滑了一些。
	  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031107499.png)
	- 在 100 个 client 的竞争下，整体的调用数量也减少了很多：
	  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031109558.png)
	- 从完成时间来看，效果也不错：
	  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303031109544.png)
	- 除此以外，还有`Equal Jitter`，算法如下
	- ```
	  temp = min(cap, base * 2 ** attempt)
	  sleep = temp / 2 + random_between(0, temp / 2 )
	  ```
	- 还有`Decorrelated Jitter`：`sleep = min(cap, random_between(base, sleep * 3)`
	-
title:: https://t.co/gODWxXs0sU...
author:: [[@haricheung on Twitter]]
url:: https://twitter.com/haricheung/status/1619719361828372484

- Highlights first synced by [[Readwise]] [[2023-01-30]]
	- > https://t.co/gODWxXs0sU
	  
	  该论文《Segcache: a memory-efficient and scalable in-memory key-value cache for small objects》是 2021 年 NSDI best paper. 
	  
	  论文从缓存系统三个特性(内存使用效率、吞吐、扩展性)入手, 分析现有系统常见设计及其问题, 最后引出 Segcache 的解决方案和效果. ([View Tweet](https://twitter.com/haricheung/status/1619719361828372484))
	- > 在缓存系统中同时满足内存高效使用、高吞吐、高扩展性是一件相当有挑战性的事情, 大部分系统都只能做到部分, 但是本篇论文设计实现的 Segcache 做到了这三点. Segcache 用 TTL 做索引(把同样 TTL 的对象放在一组再按照对象创建时间排序)、动态分区、基于 segment 结构的缓存. ([View Tweet](https://twitter.com/haricheung/status/1619719421089710080))
	- > Segcache 采用了 segment-structured 设计, 该设计将数据存储在固定大小的 segments 中, 同时具备如下三个特性: ([View Tweet](https://twitter.com/haricheung/status/1619719718797201410))
	- > (1) 该数据结构基于相似的创建时间和过期时间把对象分组, 目的是高效率实现对象过期和驱逐, (2) 该数据结构通过近似估计, 将大部分对象的元数据抽取到共享的 segment header 和共享的 hash table slot 中, 目的是见效对象的元数据大小, 同时 ([View Tweet](https://twitter.com/haricheung/status/1619719772710772736))
	- > (3) 该数据结构使用极小的 critical sections 在 segment-level 上对批量对象进行过期和驱逐处理, 从而实现高扩展性(扩展性随着线程数增加而增大). 生产环境评估显示, Segcache 相比目前业界顶尖的缓存设计少使用了 22-60% 的内存, 而且适用于多种类型的工作负载. ([View Tweet](https://twitter.com/haricheung/status/1619719850808733698))
	- > Segcache 同时提供了更高的吞吐, 单线程层面相比 Memcached 提升了最多 40% 吞吐量. 同时, 该设计提供了接近线性的扩展性, 在 24 线程上相比 Memcached 增加 8 倍的速度. ([View Tweet](https://twitter.com/haricheung/status/1619719867908902921))
	- > 这个图是整个系统设计的精华: 
	  
	  ![](https://pbs.twimg.com/media/FnpnUw-akAEm-qL.jpg) ([View Tweet](https://twitter.com/haricheung/status/1619721077256753152))
	- > TTL bucket 数组+segment 用于写场景. Segcache 将 TTL 分成若干 range, 然后用每个 range 起始值作为所属 range 的 approximate TTL, 这样可以保证所属 range 全部对象绝对不会在超出超时时间后才被删除. ([View Tweet](https://twitter.com/haricheung/status/1619721178792460288))
	- > hash table 用于读场景. 不同于常见的 object-chained 设计, 它是 bulk chaining 的. 前者问题在于每个指针浪费 8 字节空间, 同时不连续空间导致耗时的随机访问; 后者通过分配连续的 64 字节(cpu cache line 大小)空间作为 8 个 slot, 一举解决了前述两个问题, 既减少了元数据大小又减少了随机访问. ([View Tweet](https://twitter.com/haricheung/status/1619721413228892161))
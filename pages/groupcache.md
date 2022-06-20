# Groupcache的介绍

特点：

- 同时是server和client
- 附属于进程（In Code Distributed Cache），不需要独立部署
- 可解决**数据热点**以及**缓存击穿**问题
- 一致性哈希



#### 如何解决数据热点问题：
通过一致性哈希，将单个key分布到多个节点中。



#### 在现实场景中，GroupCache是如何工作的：

1. Application向GroupCache请求数据
2. GroupCache在本地内存中查找，有则返回；无则继续
3. GroupCache通过一致性哈希计算出数据所属的节点
4. GroupCache通过网络向该节点请求数据
5. GroupCache在本地内存查找数据，无数据则调用Application获取数据，并缓存
6. GroupCache向请求数据的节点返回数据



#### 如何解决缓存击穿：
确保只能有一个**缓存未命中**的数据请求执行数据缓存，然后在多个peer中复用这个数据，以应付后续的数据查询请求。
在以上的步骤中，第5步是一个**同步操作**，同时只有一个节点在获取数据。其他的节点都需要等待这个节点返回数据。



## GroupCache 实现



组件：

- Groups
- Group
- Peer

Groups 是一个groupcache内部的所有内存空间。

Group是Groups里以name唯一的内存空间。

Peer是一个逻辑概念，它指的是一个groupcache集群中，同一个**命名空间**的多个Group集群。



### Groups

![image-20220102101756752](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/image-20220102101756752.png)



每个groupcache进程内部维护一个groups对象`map[string]*Group`。

同时通过一个`sync.RWMutex`限制一些groups的操作：

- GetGroup
- newGroup



### Group

```go
type Group struct {
	name       string  // 唯一名字，类似于缓存的namespace
	getter     Getter
	peersOnce  sync.Once
	peers      PeerPicker
	cacheBytes int64  // mainCache+hotCache的大小限制
	mainCache cache		// 本地的cache
	hotCache cache		// 全局热点数据的cache
  loadGroup filghtGroup  // 保证即便在并发下，key的数据只会获取一次（本地或者是网络获取）
	_ int32 // force Stats to be 8-byte aligned on 32-bit platforms
	Stats Stats		// 数据统计
}
```

mainCache——用于存储本该归属于该节点（一致性哈希）的数据的缓存。

hotCache——包含key/value的数据。由于系统中可能会存在一些**热点数据**，按照groupcache的基础设计中：这些数据都需要通过一致性哈希到固定的节点中去获取，这样就会使得网络成为一个瓶颈。为了应对这个问题，在本地使用了hotCache存储这些热点数据，进而避免大量的网络请求。



#### 查询逻辑





### PeerPicker

PeerPicker用于处理一致性哈希的逻辑：计算特定的key应该归属于哪个节点。



###  Cache

```go
type cache struct {
	mu         sync.RWMutex
	nbytes     int64 // of all keys and values
	lru        *lru.Cache
	nhit, nget int64
	nevict     int64 // number of evictions
}
```





`mainCache ` 



`hotCache`



#### 底层数据

ByteView



# 我的疑问



## group.load 方法的涉及的Go调度器问题：

> Check the cache again because singleflight can only dedup calls that overlap concurrently.  It's possible for 2 concurrent requests to miss the cache, resulting in 2 load() calls.  An unfortunate goroutine scheduling would result in this callback being run twice, serially.  If we don't check the cache again, cache.nbytes would be incremented below even though there will be only one entry for this key.
>
> Consider the following serialized event ordering for two goroutines in which this callback gets called twice for the same key:
>
> ​		// 1: Get("key")
> ​		// 2: Get("key")
> ​		// 1: lookupCache("key")
> ​		// 2: lookupCache("key")
> ​		// 1: load("key")
> ​		// 2: load("key")
> ​		// 1: loadGroup.Do("key", fn)
> ​		// 1: fn()
> ​		// 2: loadGroup.Do("key", fn)
> ​		// 2: fn()





# 我可以定制的部分

- 加入日志，让客户端可以定制日志：格式、level，输出stat信息。
- 参考uwsgi的stat服务，做一个性能状态查看的工具。
- 考虑加入冷启动的场景如何处理
- 性能分析
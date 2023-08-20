title:: 100 Go Mistakes and How to Avoid Them
author:: [[Teiva Harsanyi]]
url:: https://readwise.io/reader/document_raw_content/8740249

- > Let’s take concurrency, for example. In 2019, a study focusing on concurrency
  bugs was published: “Understanding Real-World Concurrency Bugs in Go.”1 This
  study was the first systematic analysis of concurrency bugs. It focused on multiple pop-
  ular Go repositories such as Docker, gRPC, and Kubernetes. One of the most import-
  ant takeaways from this study is that most of the blocking bugs are caused by
  inaccurate use of the message-passing paradigm via channels, despite the belief that
  message passing is easier to handle and less error-prone than sharing memory. ([View Highlight](https://read.readwise.io/read/01gnw1xcnmqy3aqqyxtwynjhpd))
	- 《Understanding Real-World Concurrency Bugs in Go》这本书可以另外找来看看。
- > In Go, a variable
  name declared in a block can be redeclared in an inner block. This principle,
  called variable shadowing, is prone to common mistakes. ([View Highlight](https://read.readwise.io/read/01gny2hzraxd9pz0r90xt7rb9c))
-
- ## shadow variable
- ## init function
	- 执行顺序
		- main中`import`的 package 的`init`先于 main 的`init`
		- 多个package 的 `init`，按照`alphabetical`顺序执行
	- 使用 init 函数需要考虑的问题
		- init 函数无法返回 error。如果是在作为第三方pakcage 创建资源的时候，最好是不要使用 init 函数。caller 在使用的时候可能还需要执行`retry`和`fallback`的逻辑，所以需要错误处理的场景下不能使用 init 函数。
		- 测试。init 函数会优先于测试代码开始执行。在 unit test 中，我们需要将一些外部资源 mock，这种情况下如果使用 init 函数初始化则不便于写测试代码。
		-
- ## 不要过度使用interface
	- > Do not design with interfaces, discovery them.   --Rob Pike
	- 不用过早开始抽象和设计。在你观察到复杂性的时候再通过接口来重构代码。
	- 在通过`interface`调用方法时，需要在 hashtable 中查找接口指向的具体类型，也会有性能开销。
	-
- ## 在生产者实现接口
	- 在经典的**生产-消费**场景下，其实接口应该是在消费者方定义的，由生产者方负责实现。 #CleanCode
		- 首先是编码方式：我们应该先写*消费者* 代码，在消费者则 *discover* 需要的接口，*生产者*再来实现它。
		- 设计层面：*生产者* 可以有多种的实现，而且在 *消费者* 定义接口，在多个生产者实现接口。
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202305220921328.png){:height 589, :width 329}
	- 因为 golang 的接口使用的是**隐式实现**而非**显式实现**，这里不会出现循环依赖的问题。*producer*包不需要导入*consumer* 包中的 interface
	- 一种在消费者定义接口的情况
		- 假设有一个接口：
		  ```go
		  package store
		  
		  type CustomerStorage interface {
		    StoreCustomer(c Customer) error
		    GetCustomer(id string) (Customer, error)
		    GetAllCustomers() ([]Customer, error)
		  }
		  ```
		- 有一个消费者只关心`StoreCustomer`方法，而另一个消费者只需要`GetCustomer`和`GetAllCustomers`。如果另外的方法暴露对应消费者都违背了**接口隔离**原则。那么就可以在消费者侧定义接口。
		  ```go
		  package client
		  
		  type customerGetter interface {		// 内部使用的接口，不对外暴露
		    GetCustomer(id string) (Customer, error)
		    GetAllCustomers() ([]sotre.Customer, error)
		  }
		  ```
		- 同时，由于 golang 的 interface 采用的是隐式实现，所以在生产者可以实现所有的方法，在消费者定义接口，也不会有**循环依赖**的问题。
		  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202306020857742.png){:height 334, :width 488}
	- ### 返回 interface
		- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202306020930230.png){:height 261, :width 617}
		- 在 Go 的很多项目中，会有这种返回一个 interface 的函数。随之而来的问题是：client 无法调用这个`NewInMemoryStore`的方法了。
		- 为此我们应该遵循一个原则：
			- 返回 struct 而不是 interface
			- 尽可能接收 interface 参数
		- 返回一个 interface，会导致调用方被限制于一个固定的抽象。可能会不够灵活。
- ## 泛型
	- golang[[泛型]]**类型参数**的使用
		- 函数中使用泛型参数：`func Foo[T comparable, V any] (m map[K]V) []K {}`
			- 可以分为两个部分来看：
				- 函数的输入输出 signature：`func Foo (m map[K]V) []K`
				- 函数的类型参数：`[]`里面是类型参数
		- 数据类型中使用泛型
			- ```go
			  // Linked List
			  type Node[T any] struct {
			    Val T
			    next *Node[T]
			  }
			  
			  func (n *Node[T]) Add(next *Node[T]) {
			    n.next = next
			  }
			  ```
		- **类型参数**的使用限制
			- 类型参数只能用在：`method receiver`、`function argument`
			- 不能在 `method argument`中使用类型参数：
			  ```go
			  type Foo struct {}
			  
			  func (Foo) bar[T any](t T) {}
			  
			  // ./main.go:29:15: methods cannot have type parameters
			  ```
			-
		- **constraint**
			- 一系列行为的约束（methods）
			- 类型的约束
				- ```go
				  type customConstraint interface {
				    ~int | ~string	// 底层类型是 int 或者 string
				  }
				  
				  func getKeys[K customConstraint, v any] (m map[K]V) []K {
				    
				  }
				  ```
- ## Slice 相关
	- 初始化的小技巧
		- ```go
		  var s int[]
		  s = append(s, 1)
		  
		  // 等价于
		  s := append([]int(nil), 1)
		  
		  // []int(nil) 相当于是声明了一个 nil slice
		  ```
	- ### empty & nil slice
		- 对于 json 序列化中的不同表现
			- ```go
			  type customer {
			    ID string
			    Operations []float32
			  }
			  
			  var s1 []float32	// nil slice
			  customer1 := customer{ID: "foo", Operations: s1}
			  s2 := make([]float32, 0)	// empty slice
			  customer2 := customer{ID: "bar", Operations: s2}
			  
			  b1, _ := json.Marshal(customer1)
			  fmt.Println(string(b1))
			  b2, _ := json.Marshal(customer2)
			  fmt.Println(string(b2))
			  
			  // result
			  // b1: {"ID": "foo", "Operations": null}
			  // b2: {"ID": "bar", "Operations": []}
			  ```
		- 同样的，在标准库中`reflect.DeepEqual()`函数也是一样的。
	- ### Copy 函数
		- `copy(dst, src)`
		- 首先是参数次序与直觉是相反的
		- copy 函数实际拷贝值的数量=min(len(dst), len(src))
		-
	- ### full slice expression
		- 一个切片表达式：`[low:high:max]`。限制了 slice 的容量只能是 max。
		- 使用场景：在 append
	- ### slice memory leaks
		- #### leaking capacity
			- ```go
			  func consumeMessages() {
			  	for {
			      	msg := receiveMessage()
			          // Do something with msg
			          sotreMessageType(getMessageType(msg))
			      }
			  }
			  
			  func getMessageType(msg []byte) []byte {
			    returm msg[:5]	// get message type from first 5 bytes
			  }
			  ```
			- `getMessageType`函数可能会带来**内存泄漏**
			- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202307030907346.png)
			- `msg[:5]`会持有与 msg 一样的容量*（共享底层数组）*。`msg[:5]`虽然只截取了一小部分，但是它如果没有被 GC 回收的话。这里也会造成大量的内存泄漏。
			- 考虑一个情况：
			  ```go
			  func storeMessageType(msgType []byte) {
			      // 将msgType保存到某个全局变量中
			      globalCache[msgType] = time.Now()
			  }
			  ```
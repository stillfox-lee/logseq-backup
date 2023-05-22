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
		-
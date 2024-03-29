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
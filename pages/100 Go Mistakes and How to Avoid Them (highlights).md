title:: 100 Go Mistakes and How to Avoid Them (highlights)
author:: [[Teiva Harsanyi]]
full-title:: "100 Go Mistakes and How to Avoid Them"
category:: #articles #golang #Reading 
url:: https://readwise.io/reader/document_raw_content/8740249

- Highlights first synced by [[Readwise]] [[2023-01-04]]
	- Let’s take concurrency, for example. In 2019, a study focusing on concurrency
	  bugs was published: “Understanding Real-World Concurrency Bugs in Go.”1 This
	  study was the first systematic analysis of concurrency bugs. It focused on multiple pop-
	  ular Go repositories such as Docker, gRPC, and Kubernetes. One of the most import-
	  ant takeaways from this study is that most of the blocking bugs are caused by
	  inaccurate use of the message-passing paradigm via channels, despite the belief that
	  message passing is easier to handle and less error-prone than sharing memory. ([View Highlight](https://read.readwise.io/read/01gnw1xcnmqy3aqqyxtwynjhpd))
		- > **Note**: 《Understanding Real-World Concurrency Bugs in Go》这本书可以另外找来看看。
- New highlights added [[2023-01-04]] at 5:43 PM
	- In Go, a variable
	  name declared in a block can be redeclared in an inner block. This principle,
	  called variable shadowing, is prone to common mistakes. ([View Highlight](https://read.readwise.io/read/01gny2hzraxd9pz0r90xt7rb9c))
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/20230104174610.png)
	- > 这里 client 被重新声明，外层的 client 就是 nil。
	-
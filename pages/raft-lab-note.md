# 项目实现建议
	- ## 关于结构的设计
	- ref: http://nil.csail.mit.edu/6.824/2021/labs/raft-structure.txt
	- 每个 Raft 节点都需要处理一些外部触发的事件：`Start()`的函数调用，`AppendEntries` 和`RequestVote`的 RPC 请求；以及两个周期性任务：Election 和 Heartbeat。以下是一些关于如何实现它们的建议。
	  
	  Raft 节点都会有很多的状态：log、current index，&c，它们都必须在并发的事件中更新。对于这些非并发安全的更新操作，最好是使用 Lock 来保证数据安全。
	  
	  每个 Raft 节点都会有两个时间驱动的活动：leader 必须定期发送 Heartbeat；其他节点在 Heartbeat 超时的时候需要发起新的 election。这两个行为最好是用两个长时间运行的 goroutine 来运行；而不是在一个 goroutine 中执行多个。
	  
	  election超时的管理是一个令人头痛的事情。也许存在着一种简单的处理方式：在 Raft 的数据结构中持有一个变量，用来记录上一次收到 leader 的 Heartbeat 的时间；然后让负责检测的 goroutine 来定期检测这个时间，看是否已经超时。实现定时检测最简单的方法是：使用`time.Sleep()`和一个 interval 常量来定期检测。不要使用 `time.Ticker`和`tie.Timer`，它们的使用存在着太多的 tricky。
	  
	  你需要使用一个独立的、长时间运行的 goroutine 来**按序**向`applyCh`发送`commited log`。因为这个发送行为可能会造成阻塞，所以它必须在一个独立的 goroutine 中执行。同时，因为**按序发送**的需求，它也必须是在单个 goroutine 中发送。执行`commitIndex`逻辑的代码需要激活`apply`goroutine，最简单的一种方式就是使用条件变量`sync.Cond`。
	  
	  每个 RPC 的请求发送和响应处理最好都是在同一个 goroutine中处理，这样设计是出于两个原因：1、某个网络故障的节点不会造成其他大多数节点的响应延迟；2、Heartbeat 和 election 的计时器可以在一直延续下去。要做到这些，最简单的方式就是在同一个 goroutine 中发送RPC 请求和处理 RPC 响应，而不是通过 Channel来传递。
	  
	  记住：网络会对 RPC 的通信造成延迟，如果你并行发送 RPC 请求的话，网络可能会让请求发生乱序。论文中的**Figure2**很好地指出了关于 RPC 处理方面需要注意的情况，例如 RPC 的 handler 需要忽略旧的 Term 请求。**Figure2**并没有把所有的 RPC 响应该如何处理指出。leader 在处理 RPC 响应的时候需要特别小心，它必须检查在发送了 RPC 之后Term 是否发生了变化，并且必须考虑到从同一个 Follower 并行发送的 RPC 响应是否会改变 leader 的状态（并发安全），例如 `nextIndex`的值。
	- ## 关于 Raft 的锁的建议
		- ref: http://nil.csail.mit.edu/6.824/2021/labs/raft-locking.txt
		- 规则 1：只要有数据会涉及多个 goroutine 修改的情况，需要加锁以避免数据竞争。
		- 规则 2：如果有一系列的状态变更操作，而其他的 goroutine 可能会在*中途*来查询其中的某个状态值。那就需要加锁。例如：
		  
		  ```go
		  rf.mu.Lock()
		  rf.currentTerm += 1
		  rf.state = Candidate
		  rf.mu.Unlock()
		  ```
		  
		  > 临界区需要加锁
		- 规则 3：如果代码需要对共享变量进行一系列读操作（或者读和写），如果期间有其他的 goroutine 修改了共享变量的话，也会造成问题。对于这种情况，也需要使用锁。例如：
		  
		  ```go
		  rf.mu.Lock()
		  if args.Term > rf.currentTerm {
		   rf.currentTerm = args.Term
		  }
		  rf.mu.Unlock()
		  ```
		  这里如果不加锁的话，最终有可能会变成 rf.currentTerm 会变小的情况，而这是 Raft 协议不允许发生的事情。
		  实际上，在实现 Raft 的代码中可能会存在比示例中更长的临界区。比如在整个 RPC 的 handler 中都加锁处理。
		- 规则 4：在任何需要等待的情况下加锁是一个不好的实现。例如：从 Channel 中读取数据、向 Channel 发送数据、等待定时器、调用`time.Sleep()`或者收发 RPC。因为你可能会希望在等待的同时使用其他 goroutine来提高执行速度，而且这种使用方式很可能会造成死锁发生。设想两个节点同时发送 RPC 请求并且都加锁，二者都需要等待对方的响应来释放锁，这样就触发了死锁。
		  会引起阻塞的代码，需要先释放锁。如果这个约束条件会增加代码实现的难度的话，可以考虑使用其他的 goroutine 来执行会引发阻塞的代码。
		- 规则 5：谨慎使用多段锁。这种锁可能会在避免阻塞等待的时候使用。例如：
		  ```go
		  rf.mu.Lock()
		  rf.currentTerm += 1
		  for <each peer> {
		    go func() {
		      rf.mu.Lock()
		      args.Term = rf.currentTerm		// rf.currentTerm may changed
		      rf.mu.Unlock()
		      Call("Raft.RequestVote", &args, ...)
		      // handle the reply...
		    } ()
		    rf.mu.Unlock()
		  }
		  ```
		  这里为了减少发送 RPC 的阻塞，使用了独立的 goroutine 处理发送请求，而且每个 goroutine 都使用了锁。但是，`rf.currentTerm`还是有可能与初始时不一样了。
		  可以采取的一个方案是：使用闭包特性，将 Term 参数传递到 goroutine 中，这样就保证了发送的 Term 是一致的。同时，在收到 reply 的时候，校验这个 Term 是否一致。
		- 要做到上面的要求是困难的，一个实际的方案是：
			- 首先考虑对应的场景中是否有并发的情况，如果没有并发，则不需要锁。
			- 对于存在并发的地方：在 RPC 的并发请求，`Make()`函数中创建的后台 goroutine 等。可以通过在 goroutine 的初始就 Lock，在确定 goroutine 结束返回的时候 Unlock。这样可以保障代码逻辑的执行不会出现并发的情况，并发存在阻塞的 IO 中。
			- 有了上面的保障之后，规则 1、2、3、5 应该都没有问题了。对于**规则 4**，你需要仔细找出这些处理 IO 的并发场景，确保前后的一致性。
- # Lab 2A leader election
  
  TODO:
- 实现 Raft 的 leader 选举
- 实现 Raft 的 heartbeat 机制
	- ## 提示
		- 按照 paper 的 Figure 2，处理发送和接收`RequestVote` RPC，与选举有关的规则，以及与领导选举有关的状态。
		- 在 `raft.go`的 `Raft`中，添加用于维护选举状态的数据。同样的，你需要定义用于维护 log的结构体。
		- 实现`RequestVoteArgs`和`RequestVoteReply`结构。修改`Make()`以创建一个后台goroutine，当它有一段时间没有收到另一个peer的消息时，它将通过发送`RequestVote` RPC来定期启动领导者选举。这样，如果已经有了一个领导者，peer将了解谁是领导者，或者自己成为领导者。实现`RequestVote()` RPC处理程序，这样服务器就可以互相投票了。
		- 为了实现心跳，定义一个`AppendEntries` RPC结构，并让领导者定期发送它们。编写一个`AppendEntries` RPC处理方法，重设选举超时，这样当一个人已经当选时，其他服务器就不会站出来当领导者。
		- 确保在不同的 peer 中，选举超时不会同时发生，不然每个 peer 都只会选择自己做领导。
		- 测试用例要求领导发送 heartbeat 不能超过每秒 10 次。
		- 测试用例要求选主在 5 秒钟内完成（如果大多数 peer 还能通信的情况下）。选举可能会产生很多轮，你必须选择足够短的选举超时时间（以及心跳间隔时间），以在 5s 内完成选举。
		- 论文的第5.2节提到选举超时的范围是150到300毫秒。只有当领导者发送心跳的频率大大超过每150毫秒一次时，这样的范围才有意义。因为测试用例把你限制在每秒10次心跳，所以你必须使用比150到300毫秒更大的选举超时，但不能太大，因为那样你可能无法在5秒内选出一个领导者。
		- 可以使用Go 的 rand 模块。
		- 你需要实现周期性轮询，最简单的方式就是在goroutine 中循环调用 `time.Sleep()`。不要使用`time.Timer`和`time.Ticker`，它们太难使用了。
		- Read this advice about [locking](http://nil.csail.mit.edu/6.824/2020/labs/raft-locking.txt)  and [structure](http://nil.csail.mit.edu/6.824/2020/labs/raft-structure.txt).
		- 不要忘记实现`GetState()`
		- 测试用例会调用`rf.Kill()`来永久关闭一个实例。你可以通过`rf.killed()`来查看是否调用过`Kill()`。你可能会需要在循环中处理它，以避免关闭的实例打印出干扰的信息。
		  
		  
		  测试的执行：
		  ```shell
		  go test -run 2A
		  ```
	- ## Lab 2A 实现
		- 关于 election timeout 和 heartbeat interval 如何确定？
			- sleep ticket 设置为 10 millisecond
				- 可能导致超时之后，过了一个 ticket 周期才触发超时检测。
			- 需求
				- election 完成需要少于 5 秒。(测试用例限定)
					- ==需要考虑多轮选举的情况，多轮选举也需要在 5 秒内完成。==
				- heartbeat 每秒不能超过 10 次。(测试用例限定)
		- #+BEGIN_NOTE
		  时间相关参数设置
		  - ticket：10milliseconds
		  - election timeout：700milliseconds to 1000 milliseconds
		  - heartbeat：10/s
		  #+END_NOTE
		- #+BEGIN_IMPORTANT
		  如果 Candidate 的 RequestVote 收到了大于自己 Term 的reply 怎么处理？
		  是等待
		  #+END_IMPORTANT
		-
		- server init：
			- term = 0
			- wait for election timeout
			- election timeout —— Follower wait until become Candidate
		- ### 选举阶段的 RPCs：
			- #### RequestVote
				- Sender:
					- currentTerm+=1
					- Votedfor = me
					- Role = candidate
					- reset Timer
					- 不关心通信是否成功。通信失败由 election timeout 来触发再次选举。
				- Handler（Follower）:
				- Handler（Candidate）：
			- #### Election timeout
				- Follower 的Heartbeat timeout。所有 Follower 等待随机的 Heartbeat超时。
				- Candidate 发起 RequestVote 之后，成为 Leader 之前超时，这个成为 Leader 的超时时间也是一个随机的超时时间。
			- #### 关于RPC 的调用
			  RPC 的返回值是布尔值，如果返回 false，可能是网络超时。这个时候是否需要考虑**重试**策略？
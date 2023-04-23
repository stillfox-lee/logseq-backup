title:: MySQL · 源码分析 · 事务锁调度分析
author:: [[mysql.taobao.org]]
url:: http://mysql.taobao.org/monthly/2021/09/01/
tags:: #[[database]] #[[Readwise]]

- > 所以在事务申请 lock 的过程中, 需要判断是否与其他事务持有的 lock 冲突, 对于冲突情况需要进入 waiting 队列, 而在持有 lock 的事务提交或者回滚之后, 都会释放持有的事务锁, 从而选择等待队列里的事务进行 grant lock. ([View Highlight](https://read.readwise.io/read/01gybkg2zf0jys3xbxtx7e6yga))
- > First Come First Served (FCFS)
  
  在 8.0.3 之前的 MySQL 版本, 采用的是 FCFS 的调度算法, 原理也相对简单. 在事务执行阶段向对应的 record 进行加锁行为, 通过 lock_sys 记录的 record lock 来判断是否存在冲突, 因为两阶段加锁的限制, 对于冲突的 lock 我们将其放入等待队列, 当持有的事务提交或者回滚时, 逐一释放其持有的 lock时, 会检查相应的等待队列，并按 FCFS 顺序检查是否可以将锁授予等待事务. ([View Highlight](https://read.readwise.io/read/01gybkgb07ntsjehtmawffej72))
	- 这样就有可能出现MDL导致后续的read请求被排队，进而整个表不可读。
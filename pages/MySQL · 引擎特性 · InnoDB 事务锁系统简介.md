title:: MySQL · 引擎特性 · InnoDB 事务锁系统简介
author:: [[mysql.taobao.org]]
url:: http://mysql.taobao.org/monthly/2016/01/01/
tags:: #[[database]] #[[Readwise]]

- > **LOCK_ORDINARY(Next-Key Lock)**
  
  也就是所谓的 NEXT-KEY 锁，包含记录本身及记录之前的GAP。当前 MySQL 默认情况下使用RR的隔离级别，而NEXT-KEY LOCK正是为了解决RR隔离级别下的幻读问题。所谓幻读就是一个事务内执行相同的查询，会看到不同的行记录。在RR隔离级别下这是不允许的。
  
  假设索引上有记录1, 4, 5, 8，12 我们执行类似语句：SELECT… WHERE col > 10 FOR UPDATE。如果我们不在(8, 12)之间加上Gap锁，另外一个 Session 就可能向其中插入一条记录，例如9，再执行一次相同的SELECT FOR UPDATE，就会看到新插入的记录。
  
  这也是为什么插入一条记录时，需要判断下一条记录上是否加锁了。 ([View Highlight](https://read.readwise.io/read/01gybj1demq6ct5ggf3jp4qtc2))
	- FOR UPDATE 语句是当前读
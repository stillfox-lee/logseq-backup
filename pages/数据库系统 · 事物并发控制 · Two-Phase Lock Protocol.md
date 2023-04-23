title:: 数据库系统 · 事物并发控制 · Two-Phase Lock Protocol
author:: [[mysql.taobao.org]]
url:: http://mysql.taobao.org/monthly/2021/10/02/
tags:: #[[database]] #[[Readwise]]

- > **事务并发控制如何保证正确性？**  
  事务并发调度需要保证调度事务并发执行的结果与某一种事务串行执行的结果相同，也就是事务并发调度如果是可串行化的，那么认为该调度是正确的。”冲突可串行化”是验证事务并发调度是否是可串行化的常用手段，通过交换事务间不冲突操作的执行顺序并保证冲突操作执行顺序的偏序关系，如果最终事务操作执行序列与某一种事务串行执行的操作执行序列相同，那么认为该事务并发调度是”冲突可串行化的”。
  
  两个操作冲突需要满足以下两个条件： 1）操作的是相同的数据； 2）两个操作来自不同的事务，并且至少一个是write操作。 因此冲突操作可以分为读写冲突、写读冲突、写写冲突。 ([View Highlight](https://read.readwise.io/read/01gybjyeztr7rehb4fd47rc0dv))
- > **两阶段锁协议内容如下：**  
  **阶段一(Growing)：事务向锁管理器申请所需要的锁，锁管理器分配或拒绝对应的锁。**  
  **阶段二(Shrinking)：事务释放在Growing阶段申请的锁，并不能再申请新的锁。**
  
  ![two-phase-protocal](http://mysql.taobao.org/monthly/pic/202110/202110/two-phase-protocal.jpg) ([View Highlight](https://read.readwise.io/read/01gybk4amq0grryde67pfdc5dy))
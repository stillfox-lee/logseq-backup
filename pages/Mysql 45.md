- #Mysql
- ## 自增主键
	- 为什么需要自增主键：因为单调递增的主键可以避免`页分裂`，索引更加紧凑。
	- auto increment 出现 **空洞**
		- MySQL5.7 之前的版本，自增值保存在内存中。8.0 之后的版本才被持久化到 `redo log`中。所以，自增值是会丢失的。
		- 自增值无法 rollback。如果有事务消耗了一个自增值，但是 rollback 了。这个自增值也会出现空洞。
		- 为了优化批量插入数据的场景，MySQL 在分配自增 ID 的时候会按倍数去分配。这样也会导致一些 ID 没有被使用。
- MySQL 的 [[WAL]]
	- 优化 IO，将随机 IO（写数据页） 改为顺序 IO（写日志）。先写入 `redo-log`，再通过后台线程异步写入数据页。其实也类似于 Linux 的 [[writeback]]的思路。
- [[redo-log]]
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303141700575.png){:height 375, :width 551}
	- redo-log 设计为可循环写入
	- 通过`innodb_log_file_size`可以指定文件的大小
	- 通过`innodb_log_files_in_group`可以指定文件的数量
	- **write pos** —— 当前写入的位置
	- **check point** —— 已经落盘的问题
	- redo-log 的内容是物理日志。记录的是物理上的操作，例如：*将 X 页的Y 数据更新为 W*。
	- redo log buffer
	  id:: 64104716-7754-4c18-af42-c6f8e4659801
		- 事务开始执行的时候，会将数据先写入到 redo log buffer，等特定时刻再写入到磁盘。具体逻辑如下：
		- buffer 的配置参数 [MySQLManual](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_flush_log_at_trx_commit)
		  id:: 641143ca-4028-48da-9f07-7749ca45338a
			- innodb_flush_log_at_trx_commit = 0. 每秒都从 buffer write 并且flush 一次到磁盘。
			- innodb_flush_log_at_trx_commit = 1. 事务提交每次都 从 buffer write 并且 flush 到磁盘
			- innodb_flush_log_at_trx_commit = 2. 事务提交每次都从 buffer write，每秒 flush 到磁盘一次
		- write 和 flush 指的都是都是 syscall。write 只是将 buffer 的内容写入到了文件系统中的 Page Cache，并没有实际写入到硬盘。而 flush就是`fsync`这个系统调佣，它要求内核将 Page Cache 中的内容写入到硬盘中，是一个同步操作。
		- **每秒 flush**这个操作是由 MySQL 的后台线程负责执行的。而redo log buffer 中也可能有**未提交**的事务，所以一些未提交的事务也会被写入到磁盘。
- ## binlog
	- 逻辑日志，记录的是执行的 sql 语句的逻辑，例如：*为 ID=2 的一行字段加 1*
	- binlog 有几种模式
		- statement —— 记录 SQL
		- row —— 记录行的内容
		- mixed —— 如果有风险的 sql 就记录 row，否则记录 statement
	- 与 redo-log 不同，bin-log 是追加写。
	- binlog 的 cache
		- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303280726978.png)
		- `write`和`fsync`操作可以理解为对应的系统调用。`write`只写入到了文件系统的**PageCache**中，并未落盘。
		- binlog 的`write`和`fsync`的机制，由参数`sync_bilog`控制：
			- sync_binlog = 0，每次提交事务只调用`write`，不强制`fsync`
			- sync_binlog = 1，每次提交事务都会`fsync`
			- sync_binlog = N，每次提交事务都会`write`，累积 N 个事务之后再`fsync`
		- 如果 binlog cache 占用空间太大（超过`binlog_cache_size`），就需要将它写入到磁盘中。
	- **一些难点问题**
		- binlog_format与事务隔离级别 [月报](http://mysql.taobao.org/monthly/2018/08/04/)
			- 在`Read-Commited`和`Read-Uncommited`隔离级别下，MySQL 会禁止 statement格式的日志写入。
			- 因为在这两个隔离级别中，无法解决事务的*不可重复*和*幻读*问题。例如，slave 根据 statement 重放时，由于幻读，执行结果会与 master 不一致。
			  id:: 643df0d7-9618-4c7c-bc4d-7ed12823a7c7
			-
- ## group commit
	- group commit。指的是将一组多个事务的`fsync`合并为一次调用，将多个事务一次写入，减少实际的 IO 次数。
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303280750207.png)
	- 在 MySQL 的 [[2PC]]中，也使用了 group commit 来优化磁盘的 IO。
	  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303280755434.png)
	  将两个 log 的`fsync`**延迟**，从而更好地利用 group commit来提升 IO。
- ## 基于 redolog 和 binlog 的 2PC
	- MySQL 利用 redolog 和 binlog 实现了 [[2PC]]。核心是先写 redolog，redolog prepare 之后，在写 binlog。完成之后就 commit redo log。
	-
	- 如果数据库发生了异常，可以通过二者结合恢复数据。
	- 具体流程
	  https://static001.geekbang.org/resource/image/2e/be/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png?wh=1142*1522
	- 也可以从分布式事务的角度来看：binlog 和 redolog 是两个分布式服务。
	-
-
- 一些概念
	- group commit
	- buffer_pool
		- undo page
	- [[undo-log]]
	- redo log
	- redo log buffer
		- ((64104716-7754-4c18-af42-c6f8e4659801))
		-
	- binlog cache
	- change buffer page
	- doublewrite buffer
	-
- ## innodb 的线程模型
	- Master thread
		- 负责将缓冲区的数据 flush
	- IO thread
		- write thread
		- read thread
		- insert buffer thread
		- log IO thread
	- Purge Thread
		- 在执行了`DELETE`语句之后，数据没有被实际删除，而是添加了一个 flag 标记。等异步的线程来完成实际的删除操作。
		- [[purge]] 用于最终完成`UPDATE`和`DELETE`的操作。
	- Page Cleaner Thread
	-
- ## Buffer Pool
	- 淘宝月报
		- [buffer pool 浅析](http://mysql.taobao.org/monthly/2020/02/02/)
		- [各个版本中的 buffer pool](http://mysql.taobao.org/monthly/2019/07/03/)
		- [2017 年的 buffer pool 介绍](http://mysql.taobao.org/monthly/2017/05/01/)
		- [2020 buffer pool 生命周期](http://mysql.taobao.org/monthly/2020/08/04/)
		-
	- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303151023691.png)
	- innodb buffer pool
		- data page
		- index page
		- undo page
		- ### change buffer
			- [月报](http://mysql.taobao.org/monthly/2015/07/01/)
			- 介绍
				- Change buffer的主要目的是将对二级索引的数据操作缓存下来，以此减少二级索引的随机IO，并达到**操作合并**的效果（将多个写入变为一个写入）。
				- 在MySQL5.5之前的版本中，由于只支持缓存insert操作，所以最初叫做insert buffer，只是后来的版本中支持了更多的操作类型缓存，才改叫change buffer。
				- 不需要磁盘 IO 就能完成更新的场景：**对非唯一索引的数据进行更新**。因为数据库不需要考虑唯一性约束，可以直接写入 change buffer 就完成。如果是唯一索引，必须将该数据读入内存中进行唯一性校验；这种场景 change buffer 就没有效果了。
				- **change buffer 最高效的场景**：写多读少。如果写完马上就要读取的话，就会涉及到磁盘 IO 和 Merge。单纯的写就只需要操作 change buffer即可。后台线程去完成异步的 Merge，这样change buffer 就能起到特别好的效果。
			-
			- 如何工作
				- **写入**：对于不需要磁盘 IO 的更新，就只写在 change buffer 中即可。等后续需要读这个数据页的时候，再做 merge。
				- **merge**：将 change buffer 中的操作，应用到实际的page 中。例如，执行了二级索引的数据查询，需要将索引 page 读入内存；这个时候就要将 change buffer 中的内容应用到内存中的 page 了。
				- [[purge]]
			- change buffer 的数据结构
				- 在物理上，change buffer 就是一个`btree`，存储在 ibdata 系统表空间中。
				- 这就是一个 change buffer 的一个 entity 的结构：
				  ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202304250905447.png)
				-
			- 所以，change buffer 和普通索引，在数据量大的表的更新场景下能起到的优化效果是比较好的。
			  id:: 6444873d-2989-49c0-b911-9ea586045f79
			- 在事务中，SQL 对应的操作不仅在change buffer 会记录，在 [[redo-log]] 中也会记录。所以，即使掉电 change buffer未持久化，也可以通过 redo log 来完成恢复。
		- adaptive hash index
		- lock info
		- data dictionary
		- redo log buffer
			- 落盘
				- Master Thread 每秒一次的刷新
				- free size 小于 1/2 的时候
				- ((641143ca-4028-48da-9f07-7749ca45338a))
		- 实际上，数据库会存在多个buffer pool 的实例，每个页会哈希到不同的实例中。通过`show variables like 'innodb_buffer_pool_instances'`能够得到实例的数量。
	- buffer pool 的管理链表
		- 这些链表都是逻辑链表，内部有指向`数据页`的指针。设计这些链表主要是为了方便对 buffer pool 进行管理。
		- [[LRU]] List
			- 用于缓存淘汰的管理。Free List 没有可用节点的话，就会淘汰 LRU List 的末尾节点。
		- Free List
			- 其上的节点都是未被使用的节点，如果需要从数据库中分配新的数据页，直接从上获取即可。InnoDB需要保证Free List有足够的节点，提供给用户线程用，否则需要从FLU List或者LRU List淘汰一定的节点。
		- Flush List
			- 这个链表中的所有节点都是脏页，也就是说这些数据页都被修改过，但是还没来得及被刷新到磁盘上。在FLU List上的页面一定在LRU List上，但是反之则不成立。一个数据页可能会在不同的时刻被修改多次，在数据页上记录了最老(也就是第一次)的一次修改的lsn，即oldest_modification。不同数据页有不同的oldest_modification，FLU List中的节点按照oldest_modification排序，链表尾是最小的，也就是最早被修改的数据页，当需要从FLU List中淘汰页面时候，从链表尾部开始淘汰。加入FLU List，需要使用flush_list_mutex保护，所以能保证FLU List中节点的顺序。
	- 淘汰策略
		- midpoint-LRU
			- 引入一个 midpoint的概念。新读取的页，放到 midpoint 的位置，而不是队首。通过 `show variables like 'innodb_old_blocks_pct';`可以看到这个值。
			- **背景原因**：一个大范围数据查询，但数据不是热点数据。这种情况如果放队首的话，会导致热点数据被淘汰。改良了传统的 [[LRU]]算法在数据库场景下的不足。
	- buffer pool 和 B+数的关系是什么？
		- 读取数据的时候，先从B+数遍历索引，找到对应的数据页。得到数据页的ID 之后，就可以去buffer pool 查找了。
		- 读取数据的时候，会先找 buffer，不存在就读磁盘，再放入 buffer
- ### Insert Buffer
	- Buffer Pool中只有 Insert Buffer 的部分信息，真正的 Insert Buffer 其实也是一个物理页的数据页。
	- 现在已经升级为 change buffer
-
- ## 索引
	- 回表
		- 二级索引的叶子节点存储的是主键 ID。所以通过二级索引查询数据的话，需要`回表`。
	- 覆盖索引
		- 为了优化回表的情况，最好是使得二级索引的叶子节点能够持有`select`的数据，这样就不用回表了。
	- 左前缀匹配
		- 左前缀匹配。指的是索引查找的时候，可以用`联合索引`的最左 N 个字段，或者字符串索引的最左 M 个字符来匹配。
	- 索引下推
		- 5.6 引入的新功能，需要覆盖索引。
		- 在联合索引的查询中，（如果 where 条件的字段能够覆盖索引）可以先基于联合索引的值进行匹配。不符合条件的就不用`回表`了。
	- 重建索引
		- 由于数据的删除也`页分裂`，可能会导致数据空洞。索引重建之后可以使得数据更加紧凑。
		- 对于二级索引，可以执行`alter table T drop index *` `alter table T add index *`完成
		- 对于主键，直接执行这个语句会整个表重建（包括二级索引）。可以使用`alter table T engine=InnoDB`来完成。
		- DONE 重建的底层是什么？删除为什么会有空洞？
		  :LOGBOOK:
		  CLOCK: [2023-05-05 Fri 08:59:50]--[2023-05-05 Fri 08:59:51] =>  00:00:01
		  :END:
			- 删除数据有空洞是因为：引擎在执行删除数据的时候只是加了一个删除标记，并不是直接物理删除。物理空间上就会产生空洞。
	- 自适应哈希索引
		- innodb 会监控查询，为一些高频的查询建立哈希索引。使得复杂度减低为O(1)。
	- ### 优化
		- 优化器
			- 优化器主要目的是为了减少 CPU 的资源消耗来确定选择什么方式查表（选索引）。主要考量的方面有：**扫描行数**、**是否使用临时表**、**是否排序**等
			- 区分度
				- 使用 `show index from T;`可以查看索引的情况
				  https://static001.geekbang.org/resource/image/16/d4/16dbf8124ad529fec0066950446079d4.png?wh=1850*209
				- `cardinality` 就是区分度，这个值越大越好
			- 优化器对于索引的区分度数组，采用的是采样统计。结果可能会不够准确，可以使用`analyze table T`来重新统计索引信息。
		- > SQL 如何选择索引执行查询，主要就是看优化器如何评估**查询计划**的代价。优化器会选择它认为代价小的计划去执行，但是它评估的结果并不一定是准确的。这里就产生了很多问题。
		- **创建索引的技巧**
			- **字符串索引截取部分建索引**
				- 对于字符串字段，可以使用前缀索引的优势，做到节省空间的同时为查询加速。
				- 实操：例如 email 字段建索引。可以使用
				  ```sql
				  mysql> select 
				    count(distinct left(email,4)）as L4,
				    count(distinct left(email,5)）as L5,
				    count(distinct left(email,6)）as L6,
				    count(distinct left(email,7)）as L7,
				  from SUser;
				  ```
				  查看取不同长度各自的`区分度`如何，进而确定选取哪个长度来建立索引。
				- **注意**：这个前缀索引会影响到覆盖索引的优化！
			- **字符串倒序存储建索引**
				- 对于像身份证类的字段，前面大部分的相同的，可以利用后面倒序部分来建索引。这样前缀的区分度就比原本正序的高很多。
				- 由于倒序存储，所以在业务代码上需要额外处理这个倒序的逻辑。
				- 但是，由于没有了原来的顺序，那么范围查询就必须要全表扫描了。
			- **使用 hash 压缩字符串长度建索引**
				- 使用类似于 crc32 的算法处理字符串再建立索引。
				- 需要考虑哈希算法的冲突问题，即使索引匹配，实际值可能是不同的。
- ## 锁
	- 相关月报列表
		- [Innodb 事务锁系统](http://mysql.taobao.org/monthly/2016/01/01/)
		- [Innodb 锁子系统浅析](http://mysql.taobao.org/monthly/2017/12/02/)
		- [row-lock 和 range-lock](http://mysql.taobao.org/monthly/2015/04/03/)
		- [事务并发控制 two-phase lock Protocol](http://mysql.taobao.org/monthly/2021/10/02/)
		- [事务锁调度分析](http://mysql.taobao.org/monthly/2021/09/01/)
	- 全局锁
	- ### 表锁
		- 显式执行 `lock tables Table1 read, Table2 write`
		- MDL（metadata lock）
			- 为了保障读写的正确性，设计的一个锁。在访问表的时候会自动添加。
			- 设想一个场景：读数据，同时删除表中的一个字段。
			- MDL 的逻辑：
				- 执行 DML 的时候，会加上 MDL 的 read lock
				- 执行 DDL 的时候，会加上 MDL 的 write lock
				- MDL 的 read lock 可以共享，read lock 与 write lock 互斥
			- MDL 的死锁情况
			  https://static001.geekbang.org/resource/image/7c/ce/7cf6a3bf90d72d1f0fc156ececdfb0ce.jpg?wh=1142*856
			- >  MySQL 对于锁的调度默认是 FCFS（First Come First Served）。所以，SessionD 必须等待 SessionC 申请、释放锁之后，才能获得锁。这样就会导致整个表都不可读了。
			- 注意给 DDL 语句加上一个超时时间，避免长时间的等待导致死锁问题。
	- ### 行锁
		- [[2PL]] (两阶段锁)
			- 分为 shard lock 和 exclusive lock。两阶段指的是：lock 阶段和 unlock 阶段。
			- lock阶段：读操作的时候加上 shared lock，写操作的时候加上 exclusive lock。
			- unlock 阶段：事务 commit 或者 rollback
		- 死锁问题
		  https://static001.geekbang.org/resource/image/4d/52/4d0eeec7b136371b79248a0aed005a52.jpg?wh=1142*856
	- next lock
	- #### 锁的优化
		- > MySQL 有死锁的检测机制，但它本身会耗费资源。所以大量并发的时候，可能会因为死锁检测导致 CPU 很高。
		- 避免长事务持有锁 *把有竞争的 SQL 放在事务后面执行*
		- 降低同一行的更新并发度。考虑请求排队或者数据层面的行切分。
	- ### 幻读
		- 由于事务`当前读`的特性，所以在同一个事务中进行多次同样的`当前读`的SQL，会不一样的结果。*设想在两次 SQL 之间，有其他的事务执行了`insert`*
		- 幻读带来的问题
			- 语义失效
				- 使得`select * from T where *** for update`的语义失效。
				- 在事务中执行了`for update`之后，只锁了 3 行数据；此时再插入了一条新的符合`where`的数据并不会被锁上。
			- 数据一致性问题
				- ((643df0d7-9618-4c7c-bc4d-7ed12823a7c7))
				-
		- 幻读与`当前读`在 RR 隔离级别下的场景
			- 如果当前读命中**主键**，则通过`写锁`即可解决幻读问题
			- 如果当前读未命中**主键**，则会添加**next-key lock**
			  ![死锁](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202303251558031.png)
			- 如果当前读是走**非唯一索引**，则会添加**next-key lock**
			- 如果没有走索引，则全表加 **gap lock**
	- ### gap lock
		- 为了解决幻读的问题，能够在行之间的 gap 加入一个锁。使得这个 gap 之间无法进行更新的操作。
		- 特性
			- gap lock 之间是可共存的
			- 只在 RR 隔离级别下生效
			- gap lock 锁的是 write 行为
	- ### next-key lock
		- 行锁和 gap lock 合称 next-key lock，是前开后闭区间 `( ]`。因为 gap lock 是开区间，并没有锁上记录行，所以通过 next-key lock，就可以完整覆盖。
	- ### RR 级别下加锁的规则
		- [45 讲](https://time.geekbang.org/column/article/75659)
		- 原则 1：加锁的基本单位是 next-key lock。
		- 原则 2：查找过程中访问到的对象才会加锁。*limit 语言会影响扫描的行*
		- 优化 1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。
		- 优化 2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化gap lock。
		- 一个 bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止。
	- ### 加锁的一些场景
		- 锁是加在索引上的。如果有覆盖索引，那么只会锁住二级索引，而不会锁主键。
		- 如果在二级索引上使用了`for update`这些 write lock，那么会在二级索引以及主键索引同时加锁。
		- `select * from t where id>9 and id<12 order by id desc for update;`这类语句，在查找的时候，首先是**等值查询**找到 id=12 的值；再进行**范围查询**。所以，这里的加锁规则就要分两种情况进行了。
		-
- ## 事务
	- 月报
		- [事务系统](http://mysql.taobao.org/monthly/2017/12/01/)
	- 隔离级别
		- Read-uncommitted
			- 没有 *consistent snapshot*
		- Read-commited
			- 每个 SQL 执行时，创建一个 *consistent snapshot*
		- Repeatable-read
			- 事务启动时就创建一个 *consistent snapshot*
		- Serializable
	- consistent snapshot
		- 事务的 `RR` 隔离是依靠事务启动时，创建的一个 consistent snapshot 实现的。可以用语句`start transaction with consistent snapshot`来主动触发*立即*创建一个快照。
		- **注意**：这个快照并不是在事务启动的时候*立即*创建的，而是在执行第一个 SQL 语句的时候创建的。
		- 具体是实现是通过 [[undo-log]] 来完成的，每次有 DML 的时候，就会记录下对应的 undo-log。如果想要得到某个版本的数据，就可以通过 undo-log 回滚得到。这样也就实现了 [[mvcc]]
	- [[mvcc]]
		- 内部记录一个全局活跃的读写事务数组，通过它来判断事务的可见性。
		- consistent read view —— 开始一个事务的时候，会构建一个当前事务的`readview`。后续事务中对数据的查询，根据不同的可见性，会按照 readview 来获取不同版本的数据。
		  id:: 641280eb-9ae7-4665-9f02-c42d039748c6
		- 查询出来的每一行记录，都会用readview来判断一下当前这行是否可以被当前事务看到，如果可以，则输出，否则就利用undolog来构建历史版本，再进行判断，直到记录构建到最老的版本或者可见性条件满足。
		- 相关数据结构
			- ```c
			  struct trx_t {
			    
			  };
			  ```
			- ```c
			  struct trx_sys_t {
			    int max_trx_id;  // 当前未分配的最小 id，系统创建新事务就用这个 id，递增
			    []int descriptors;	// 当前所有活跃(未提交)的读写事务 id
			  }
			  ```
			- ```c
			  struct read_view_t {
			    int low_limit_id;		// 所有大于等于此值的记录都不应该被此 readview 见到，high water mark
			    int up_limit_id;		// 所有小于此值的记录都应该被此 readview 见到，low water mark
			    []int descriptors;	// readview 创建时的所有读写事务 id
			  };
			  ```
		- 实现逻辑
			- 每一行数据都有一个`trx_id`，它代表着最后更新这行记录的事务 ID。
			- 当事务需要一个`readview`的时候，会从`trx_sys_t.descriptors`复制一份当前所有**读写事务**，当做一个快照。
			- `read_view_t.up_limit_id`就是`read_view_t.descriptors`中的最小值。同时也是当前事务创建时，所有的`读写事务`中的最小事务 ID。作为**低水位标记**。
			- `read_view_t.low_limit_id`是创建 readview 时`trx_sys_t.max_trx_id`值。同时也是创建事务时，系统中最大的事务 ID。作为**高水位标记**。
			- 每个事务的可见性逻辑，就是基于这三个元素*（trx_id、低水位、高水位）*计算得到的。
			- https://static001.geekbang.org/resource/image/88/5e/882114aaf55861832b4270d44507695e.png?wh=1142*856{:height 284, :width 530}
		- 可见性判断规则
			- 如果记录上的`trx_id`小于`低水位`，则说明这条记录的最后修改在readview创建之前，因此这条记录可以被看见。
			- 如果记录上的`trx_id`大于等于`高水位`，则说明这条记录的最后修改在readview创建之后，因此这条记录肯定不可以被看见。
			- 如果记录上的`trx_id`在`低水位`和`高水位`之间，且`trx_id`在`read_view_t.descriptors`之中，则表示这条记录的最后修改是在readview创建之时，被另外一个活跃事务所修改，所以这条记录也不可以被看见。如果`trx_id`不在`read_view_t.descriptors`之中，则表示这条记录的最后修改在readview创建之前，所以可以看到。
			- 基于上述判断，如果记录不可见，则尝试使用undo去构建老的版本(`row_vers_build_for_consistent_read`)，直到找到可以被看见的记录或者解析完所有的undo。
			-
	- 触发**current read** (当前读)的情况
		- update 语句。所有更新数据都是**先读后写**，读的时候会读取最新的**当前值**。
		- 加锁语句
	- XA 事务
- ## 分布式相关
	- 双 M 结构
	- 主备延迟
		- slave 库消费 **relay log** 的速度比 master 生产 binlog 的速度要慢。可能是磁盘 IOPS 的能力问题
		- 避免大事务的执行，大事务本身耗时比较高
		- slave 的并行复制能力
	- 主备切换
		- 可靠性优先策略
			- master 设置为 read only，待 slave 完全同步完成，再切换。牺牲了 A 而获得了 C。
		- 可用性优先策略
			- 直接将 slave 设置为可读写。获得了 A 但是牺牲了 C。
			- 遇到一致性问题的时候，可考虑通过 binlog 来实现恢复。
		- 一些相关工具
			- semi-sync
			- async
			- AliSQL
			- WAITING 业内成熟的 HA方案是什么
	- ### 主备并行复制策略的演进
		- 背景：原本的 MySQL 中，slave 只有一个`sql_thread`负责从 relay log 中读取 SQL 并执行。当 master 的 TPS 高的时候，slave 就会产生大延迟。需要使用多线程提高并发度。
		- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202304021512663.png)
		- `coordinator`的角色就是分发 SQL 给`worker`执行。执行 SQL 的粒度自然是以*事务*为单位。那么，为了保障正确性，`coordinator`分发事务的时候，会有两条规则：
			- **保证事务原子性**。同一个事务，必须在同一个`worker`中执行。
			- **不能造成更新覆盖**。更新同一 row 的两个事务，必须在同一个`worker`中。
		- #### 按表分发策略
			- ![](https://raw.githubusercontent.com/stillfox-lee/image/main/picgo/202304021525113.png)
			- 每个`worker`都记录着自己正在更新的table，当有一个新事务需要分配的时候，需要将事务涉及的表与`worker`中正在执行的 table 对比，根据冲突情况来决定如何执行。对应的策略是：
				- 如果跟所有 worker 都不冲突，coordinator 线程就会把这个事务分配给最空闲的 woker;
				- 如果跟多于一个 worker 冲突，coordinator 线程就进入等待状态，直到和这个事务存在冲突关系的 worker 只剩下 1 个；
				- 如果只跟一个 worker 冲突，coordinator 线程就会把这个事务分配给这个存在冲突关系的 worker。
			- **缺点**：如果存在*热点表*的话，那么这个并发就相当于是单线程的并发度了。
		- #### 按行分发策略
			-
- ## 性能相关
	- 可能影响性能的几种场景
		- redo log 写满，flush 脏页
		- buffer pool 的脏页比例高
			- 合理设置`innodb_io_capacity`
			- 关注一下脏页比例
	- `query_rewrite`功能可以将 SQL 语句改写，从而提升效率。
	- [检查所有 SQL 语句的返回结果](https://www.percona.com/doc/percona-toolkit/3.0/pt-query-digest.html)
	- > 一般情况下，把生产库改成“非双 1”配置，是设置 innodb_flush_logs_at_trx_commit=2、sync_binlog=1000。
	- WAITING Explain 语句如何使用
	- ((6444873d-2989-49c0-b911-9ea586045f79))
	-
- SQL 语句相关
	- order by
		- 实现：每个线程会有一个 `sort_buffer`，将获取到的数据放入排序。如果需要排序的内容大于`sort_buffer`的大小的话，那就需要使用磁盘作为临时文件辅助排序。外部排序一般是使用*归并排序*算法。
		- 排序过程
			- 按照`where`条件，选出所有的数据，放入 sort_buffer中，再sort，再 limit，返回。
		- 全字段排序
			- 将需要返回的数据 *（select的字段）*，全部放入 sort_buffer中。
			- `max_length_for_sort_data`配置规定了单行数据在 sort_buffer中的值，超过的话，就需要使用**rowid**排序。
		- rowid 排序
			- 只将需要排序的 column 和主键 ID 放入 sort_buffer
			- 最后返回数据需要回表去捞数据
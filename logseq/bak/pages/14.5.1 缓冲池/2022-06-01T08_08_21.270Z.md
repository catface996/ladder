- 缓冲区是用于缓存被 InnoDB 访问的表或者索引，位于主存中的一块内存区域。缓冲池允许直接从内存访问频繁使用的数据，这加快了处理速度。在专用服务器上，高达80% 的物理内存通常分配给缓冲池。
- 为了提高大容量读取操作的效率，将缓冲池划分为可以容纳多行的页。为了提高缓存管理的效率，缓冲池被实现为一个页的链接列表; 很少使用的数据使用最近最少使用的(LRU)算法的变体在缓存之外老化。
- 了解如何利用缓冲池将频繁访问的数据保存在内存中是 MySQL 调优的一个重要方面。
- ## 缓冲池 LRU 算法
	- 使用 LRU 算法的变体将缓冲池作为一个列表管理。当需要 room 向缓冲池添加新页面时，将删除最近使用次数最少的页面，并将新页面添加到列表中间。这种中点插入策略将列表视为两个子列表:
		- 在开头，是最近访问的新(“年轻”)页面的子列表
		- 在尾部，是最近访问过的旧页面的子列表
	- ![image.png](../assets/image_1652426156208_0.png)
	- 该算法保留了新子列表中常用的页面。旧的子列表包含较少使用的页面; 这些页面可能会被逐出。
	- 默认情况下，该算法的操作方式如下:
		- 缓冲池的3/8用于旧的子列表。
		- 列表的中点是新子列表的尾部与旧子列表的头部相交的边界。
		- 当 InnoDB 将一个页面读入缓冲池时，它首先将其插入到中点(旧子列表的头部)。可以读取页面，因为它是用户发起的操作(如 SQL 查询)所必需的，或者是 InnoDB 自动执行的预读操作的一部分。
		- 访问旧子列表中的页面会使其“年轻”，将其移动到新子列表的头部。如果由于用户发起的操作需要而读取页面，则立即进行第一次访问，使页面变得年轻。如果由于预读操作而读取了页面，则第一次访问不会立即发生，并且可能根本不会在驱逐页面之前发生。
			- 插入Midpoint 的是用户要读取的数据页。
			- 插入Midpoint 是 InnoDB 预读取的数据页。
		- 随着数据库的运行，缓冲池中，那些从来不会被“年龄”访问页面，向列表的尾部移动。新旧子列表中的页面都会随着其他页面的更新而老化。旧子列表中的页面也会随着页面插入到中点而变老。最终，一个未使用的页面到达旧子列表的尾部并被驱逐。
	- 默认情况下，查询读取的页面会立即移动到新的子列表中，这意味着它们在缓冲池中停留的时间更长。**例如，对 mysqldump 操作或没有 WHERE 子句的 SELECT 语句执行表扫描，可以将大量数据带入缓冲池，并将等量的旧数据驱逐出去，即使新数据再也不会被使用。类似地，由预读后台线程加载并且只访问一次的页面被移动到新列表的头部。**这些情况可能会将频繁使用的页面推到旧的子列表中，从而使其遭到驱逐。有关优化此行为的信息，请参阅第14.8.3.3节“设置缓冲池扫描抗性”和第14.8.3.4节“配置 InnoDB 缓冲池预取(Read-Ahead)”。
	- **问题：如果是链表，每次在缓冲区寻找指定的 Page，复杂度岂不是 O(n)??**
- ## 缓冲池配置
	- 您可以配置缓冲池的各个方面，以提高性能。
	- 理想情况下，您可以将缓冲池的大小设置为尽可能大的值，从而为服务器上的其他进程留下足够的内存，以便在不进行过多分页的情况下运行。缓冲池越大，InnoDB 就越像一个内存数据库，只从磁盘读取一次数据，然后在接下来的读取过程中从内存访问数据。请参阅第14.8.3.1节“配置 InnoDB 缓冲池大小”。
	- 在具有足够内存的64位系统上，可以将缓冲池拆分为多个部分，以最小化并发操作之间对内存结构的争用。有关详细信息，请参阅第[[14.8.3.2 配置多个缓冲池实例]]。
		- 阿里云 8 核 64G 的 MySQL 实例，innodb_buffer_pool_instances=8
	- 您可以将频繁访问的数据保存在内存中，而不用担心操作中会突然出现活动峰值，从而将大量不经常访问的数据带入缓冲池。有关详细信息，请参阅第[[14.8.3.3 使缓冲池具有抗扫描性]]。
	- 您可以控制如何以及何时执行预读请求，以便在预期即将需要页面的情况下异步地将页面预取到缓冲池中。有关详细信息，请参阅第[[14.8.3.4 配置 InnoDB 缓冲池预取(Read-Ahead)]]。
	- 您可以控制何时发生后台刷新，以及是否根据工作负载动态调整刷新速率。有关详细信息，请参阅第[[14.8.3.5 配置缓冲池冲洗]]。
	- 您可以配置 InnoDB 如何保持当前缓冲池状态，以避免服务器重新启动后的长时间预热。有关详细信息，请参阅第[[14.8.3.6 保存和还原缓冲池状态]]。
- ## 使用 InnoDB 标准监视器监视缓冲池
	- InnoDB 标准监视器的输出可以通过 SHOW ENGINE InnoDB STATUS 访问，它提供了关于缓冲池操作的指标。缓冲池指标位于 InnoDB 标准监视器输出的 BUFFER POOL AND MEMORY 部分:
	  collapsed:: true
		- ```shell
		  SHOW ENGINE InnoDB STATUS
		  ```
		- ![image.png](../assets/image_1652431282194_0.png)
		- ```shell
		  
		  =====================================
		  2022-05-12 15:44:04 0x7f9f5004c700 INNODB MONITOR OUTPUT
		  =====================================
		  Per second averages calculated from the last 41 seconds
		  -----------------
		  BACKGROUND THREAD
		  -----------------
		  srv_master_thread loops: 16 srv_active, 0 srv_shutdown, 20902 srv_idle
		  srv_master_thread log flush and writes: 20918
		  ----------
		  SEMAPHORES
		  ----------
		  OS WAIT ARRAY INFO: reservation count 59
		  OS WAIT ARRAY INFO: signal count 32
		  RW-shared spins 0, rounds 116, OS waits 56
		  RW-excl spins 0, rounds 0, OS waits 0
		  RW-sx spins 0, rounds 0, OS waits 0
		  Spin rounds per wait: 116.00 RW-shared, 0.00 RW-excl, 0.00 RW-sx
		  ------------
		  TRANSACTIONS
		  ------------
		  Trx id counter 2498
		  Purge done for trx's n:o < 2496 undo n:o < 0 state: running but idle
		  History list length 0
		  LIST OF TRANSACTIONS FOR EACH SESSION:
		  ---TRANSACTION 421797471399536, not started
		  0 lock struct(s), heap size 1136, 0 row lock(s)
		  ---TRANSACTION 421797471398624, not started
		  0 lock struct(s), heap size 1136, 0 row lock(s)
		  ---TRANSACTION 421797471397712, not started
		  0 lock struct(s), heap size 1136, 0 row lock(s)
		  --------
		  FILE I/O
		  --------
		  I/O thread 0 state: waiting for completed aio requests (insert buffer thread)
		  I/O thread 1 state: waiting for completed aio requests (log thread)
		  I/O thread 2 state: waiting for completed aio requests (read thread)
		  I/O thread 3 state: waiting for completed aio requests (read thread)
		  I/O thread 4 state: waiting for completed aio requests (read thread)
		  I/O thread 5 state: waiting for completed aio requests (read thread)
		  I/O thread 6 state: waiting for completed aio requests (write thread)
		  I/O thread 7 state: waiting for completed aio requests (write thread)
		  I/O thread 8 state: waiting for completed aio requests (write thread)
		  I/O thread 9 state: waiting for completed aio requests (write thread)
		  Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
		   ibuf aio reads:, log i/o's:, sync i/o's:
		  Pending flushes (fsync) log: 0; buffer pool: 0
		  275 OS file reads, 688 OS file writes, 213 OS fsyncs
		  0.00 reads/s, 0 avg bytes/read, 0.00 writes/s, 0.00 fsyncs/s
		  -------------------------------------
		  INSERT BUFFER AND ADAPTIVE HASH INDEX
		  -------------------------------------
		  Ibuf: size 1, free list len 0, seg size 2, 0 merges
		  merged operations:
		   insert 0, delete mark 0, delete 0
		  discarded operations:
		   insert 0, delete mark 0, delete 0
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  Hash table size 34673, node heap has 0 buffer(s)
		  0.00 hash searches/s, 0.00 non-hash searches/s
		  ---
		  LOG
		  ---
		  Log sequence number 2893899
		  Log flushed up to   2893899
		  Pages flushed up to 2893899
		  Last checkpoint at  2893890
		  0 pending log flushes, 0 pending chkp writes
		  143 log i/o's done, 0.00 log i/o's/second
		  ----------------------
		  BUFFER POOL AND MEMORY
		  ----------------------
		  Total large memory allocated 137428992
		  Dictionary memory allocated 153263
		  Buffer pool size   8191
		  Free buffers       7786
		  Database pages     405
		  Old database pages 0
		  Modified db pages  0
		  Pending reads      0
		  Pending writes: LRU 0, flush list 0, single page 0
		  Pages made young 0, not young 0
		  0.00 youngs/s, 0.00 non-youngs/s
		  Pages read 241, created 164, written 492
		  0.00 reads/s, 0.00 creates/s, 0.00 writes/s
		  No buffer pool page gets since the last printout
		  Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
		  LRU len: 405, unzip_LRU len: 0
		  I/O sum[0]:cur[0], unzip sum[0]:cur[0]
		  --------------
		  ROW OPERATIONS
		  --------------
		  0 queries inside InnoDB, 0 queries in queue
		  0 read views open inside InnoDB
		  Process ID=1750, Main thread ID=140322126415616, state: sleeping
		  Number of rows inserted 11611, updated 0, deleted 0, read 21063
		  0.00 inserts/s, 0.00 updates/s, 0.00 deletes/s, 0.00 reads/s
		  ----------------------------
		  END OF INNODB MONITOR OUTPUT
		  ============================
		  
		  ```
	- 指标
		- ![image.png](../assets/image_1652431393724_0.png)
		- ![image.png](../assets/image_1652431428368_0.png)
	- 备注
		- 年轻的度量标准只适用于旧的页面。它基于页面访问次数。一个给定的页面可以有多个访问，所有访问都被计算在内。如果在没有发生大的扫描时，您看到非常低的 youngs/s 值，那么考虑减少延迟时间或增加用于旧子列表的缓冲池的百分比。增加百分比会使旧的子列表变大，从而使该子列表中的页面移动到尾部所需的时间更长，这增加了再次访问这些页面并使其变年轻的可能性。请参阅第14.8.3.3节“使缓冲池具有抗扫描性”。
		- 非年轻的度量只适用于旧页面。它基于页面访问次数。一个给定的页面可以有多个访问，所有访问都被计算在内。如果在执行大型表扫描时没有看到更高的非杨/s 值(以及更高的杨/s 值) ，则增加延迟值。请参阅第[[14.8.3.3 使缓冲池具有抗扫描性]]。
		- 年轻创建速率包含所有缓冲池页面访问，而不仅仅是旧的子列表中的页面访问。Young-making 速率和 not 速率通常不会合计为总的缓冲池命中率。在旧的子列表中的页面点击会导致页面移动到新的子列表中，但是在新的子列表中的页面点击会导致页面移动到列表的头部，只有当页面与头部有一定距离时才会这样。
		- Not (young-making rate)是页面访问没有导致页面变年轻的平均命中率，这是由于 innodb _ old _ blocks _ time 没有得到满足所定义的延迟，或者由于新子列表中的页面命中没有导致页面移动到头部。这个速率用于访问所有缓冲池页面，而不仅仅是访问旧的子列表中的页面。
	- 缓冲池服务器状态变量和 INNODB _ Buffer _ pool _ stats 表提供了很多与 INNODB 标准监视器输出中相同的缓冲池指标。有关更多信息，请参见示例[[14.10 查询 INNODB _ buffer _ pool _ stats 表]]。
-
-
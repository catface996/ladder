- 什么是可扩展性
	- 容量和可扩展性并不依赖性能。以告诉公路上的汽车来类比的话：
		- 性能是汽车的时速
		- 容量是车道数乘以最大安全时速
		- 可扩展性是在不减慢交通的情况下，能增加更多车和车道的程度
	- 数据量
	- 用户量
	- 用户活跃度
	- 相关数据集的大小
	- [[USL]]
		- USL说的线性扩展的偏差可以通过两个因素来建立模型
			- 无法并发执行的一部分工作
			- 需要交互的另外一部分工作
		- 为第一个因素建模就有了著名的Amdahl定律，它会导致吞吐量趋于平缓。如果部分任务无法分治，那么不管你如何分而治之，该任务至少需要串行部分的时间。
		- 增加第二个因素——内部节点间或者进程间的通信——到Amdahl定律就得出了USL。
		- ![image.png](../assets/image_1657531015533_0.png)
- 扩展MySQL
	- 垂直扩展
	  collapsed:: true
		- 也叫向上扩展，就是买更强悍的服务器
	- 水平扩展
	  collapsed:: true
		- 任务分配到多台服务器上
	- 规划可扩展性
	  collapsed:: true
		- 难点是估算需要承担的负载到底有多少
	- 为扩展赢得时间
	  collapsed:: true
		- 优化性能
		- 购买性能更强的硬件
	- 向上扩展
	  collapsed:: true
		- 费钱
		- 向上扩展最终会有无法突破的瓶颈
	- 向外扩展
	  collapsed:: true
		- 水平扩展
		  collapsed:: true
			- 复制
			- 拆分
			- 数据分片
		- 按功能拆分
		  collapsed:: true
			- 或者说按职责拆分
		- 数据分片
		- 选择分区键
		  collapsed:: true
			- 确定分区键一个比较好的办法是用实体关系图
			- ![image.png](../assets/image_1657532509502_0.png)
		- 多个分区键
		- 跨分片查询
		- 分配数据、分片和节点
		  collapsed:: true
			- 分片大小要适度，方便运维，可以在5到10分钟完成日常维护
		- 在节点上部署分片
		- 固定分配
		  collapsed:: true
			- 缺点
				- 如果分片很大且数量不多，就很难平衡不同分片间的负载。
				- 固定分片的方式无法自定义数据放在哪个分片上，这一点对于那些在分片间负载不均衡的应用来说尤其重要。
				- 修改分片策略通常比较困难，因为需要重新分配已有的数据。
		- 动态分配
		- 混合动态分配和固定分配
		- 显示分配
		- 重新均衡分片数据
		- 生成全局唯一ID
		- 分片工具
		  collapsed:: true
			- 主要完成以下任务
				- 连接到正确的分片并执行查询
				- 分布式一致性校验
				- 跨分片结果集聚合
				- 跨分片关联操作
				- 锁和事务管理
				- 创建新的数据分片
	- 通过多实例扩展
	- 通过集群扩展
	- 向内部扩展
	  collapsed:: true
		- 归档和清理策略
		- 对应用的影响
		- 要归档的行
		- 维护数据一致性
			- 当数据间存在联系时，会导致归档和清理工作更加复杂。
		- 避免数据丢失
		- 解除归档
		- 保持活跃数据独立
		- 将表划分为几个部分
			- 例如，active_user 和 inactive_user
		- 基于时间的数据分区
			- 一般新的数据总是比旧的数据活跃
- 负载均衡
	- 负载均衡的基本思路很简单：在一个服务器集群中尽可能地平均 负载量。
	- 可扩展性
		- 例如读写分离时，从备库读取数据
	- 高效性
		- 更有效的使用资源，因为它能够控制请求被路由到何处
	- 可用性
		- 一个灵活的负载均衡解决方案能够使用时刻保持可用的服务器
	- 透明性
		- 客户端无需知道是否存在负载均衡设置，也无需关心在负载均衡器的背后有多少机器
	- 一致性
		- 如果应用是有状态的（数据库事务，网站会话），那么负载均衡器就应该将相关的查询指向同一个服务器，以防止状态丢失。应用无须去跟踪到底连接的时哪个服务器
	- 直接连接
	  collapsed:: true
		- 复制上的读/写分离
			- 基于查询分离
			- 基于脏数据分离
			- 基于会话分离
			- 基于版本分离
			- 基于全局版本/会话分离
		- 修改应用的配置
		- 修改DNS名
		- 转移IP地址
		-
	- 引入中间件
		- 负载均衡器
			- 除非负载均衡器知道MySQL的真实负载，否则在分发请求时可能无法做到很好的负载均衡。
			- 许多负载均衡器知道如何检查一个HTTP请求并把会话固定到一个服务器上以保护在Web服务器上的会话状态。MySQL连接也是有状态的，但负载均衡器可能并不知道如何把所有从单个HTTP会话发送的连接请求“固定”到一个MySQL服务器上。
			- 连接池和长链接可能会阻碍负载均衡器分发连接请求。
			- 许多多用途负载均衡器只会针对HTTP服务器做健康检查和负载检查。
		- 负载均衡算法
		  collapsed:: true
			- 随机
			- 轮询
			- 最少连接数
			- 最快响应
			- 哈希
			- 权重
		- 在服务器池中增加/移除服务器
	- 一主多备间的负载均衡
		- 功能分区
		- 过滤和数据分区
		- 将部分写操作转移到备库
		- 保证备库跟上主库
		- 同步写操作
- 总结
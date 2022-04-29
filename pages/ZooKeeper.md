- Zookeeper在 [[CAP]]中选择了C和P。
- 特点
	- 分布式协调
	- 可靠性
		- 有集群不可用的时候，即leader下线，follower在选举的期间，集群不可用。
	- 一致性
		- 最终一致性
		- 一致性协议 [[ZAB]]
	- 时序
	- 扩展
	  collapsed:: true
		- 架构
			- 角色
				- leader
					- 支持写，支持读
				- follower
					- 支持读，参与选举
					- 10个人选举和10亿人选举，自然是10人选举的速度快，所以follower的数量保证选举即可，无需过多的数量。
				- observer
					- 支持读，扩展查询的性能。
			- 角色在zk配置文件中的体现 zoo.cfg
				- id:: 62694a99-55ec-4935-b71b-b68738b4ff45
				  ~~~properties
				  server.1=node-001:2888:3888
				  server.2=node-001:2888:3888
				  server.3=node-001:2888:3888
				  server.4=node-001:2888:3888:observer
				  server.5=node-001:2888:3888:observer
				  server.6=node-001:2888:3888:observer
				  server.7=node-001:2888:3888:observer
				  ~~~
	- 快速
	  collapsed:: true
		- 正常情况下的读写速度快
		- 当leader掉线，选举新的leader速度快，集群恢复速度快。
- 数据存储
  collapsed:: true
	- 在内存中存数据的状态。状态数据必定是在日志数据写入成功之后，才有可能生成。
	- 在磁盘中写日志。当然，日志也是先写到内存，再到磁盘，内存中的状态数据和日志数据是两类数据。
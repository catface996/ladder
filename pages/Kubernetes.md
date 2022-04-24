- 概念
	- [概述](https://kubernetes.io/zh/docs/concepts/overview/)
		- [Kubernetes是什么? ](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)
		  id:: 62638d0b-8cf5-412e-b7e3-eb6a3d29b788
		  collapsed:: true
			- 时光回溯
			  collapsed:: true
			  ![image.png](../assets/image_1650621232230_0.png)
				- 传统部署时代
					- 例如：安装Oracle
						- 针对操作系统准备安装包(下载或者U盘拷贝)
						- 安装Oracle需要的基础类库
						- 配置操作系统的相应配置
						- 执行安装检查，安装命令
						- 安装失败之后，要完全卸载
				- 虚拟化部署时代
					- 只是做了物理资源隔离，安装一个Oracle仍旧是个复杂的过程。
				- 容器部署时代
					- 敏捷应用程序的创建和部署
						- 关注点在如何构建应用程序的镜像以及启动镜像时的参数配置
					- 持续开发、集成和部署
					- 关注开发和运维的分离
					- 可观察性
					- 跨开发、测试和生产环境的一致性
						- 可以使用同样的构建产物，在不同的环境上运行，只是启动参数的差异。
					- 跨云和操作系统发型版本的可移植性
						- 容器运行环境解决了不同的云和操作系统之间的差异
					- 以应用程序为中心的管理
					- 松耦合、分布式、弹性、解放的微服务
						- 应用跟硬件，操作系统，网络都没有关系
						- 弹性：副本数可伸缩
					- 资源隔离
						- 每个应用使用的资管互不影响
					- 资源利用
						- 一个应用的资源利用不是固定的，例如启动一个应用的最低配置和最大需要的配置等。
			- 为什么需要Kubernetes,它能做什么?
			  collapsed:: true
				- 服务发现和负载均衡
					- 微服务的两大基石
						- RPC
						- 消息
				- 存储编排
				- 自动部署和回滚
				- 自动完成装箱计算
					- 背包问题
				- 自我修复
					- 传统部署方案需要写定时任务来做应用健康检查和自我修复
				- 秘钥与配置管理
			- Kubernetes不是什么
			  collapsed:: true
				- 不限制支持的应用类型。
				- 不部署源代码,也不构建你的应用程序。
				-
				-
		- [Kubernetes组件](https://kubernetes.io/zh/docs/concepts/overview/components/)
		  id:: 62638d0b-2509-439b-a1e1-df37b3aa61e2
		  ![image.png](../assets/image_1650596136346_0.png)
			- 控制平面组件(Control Plane Components)
				- kube-apiserver
					- 该组件公开了 Kubernetes API
					- 支持水平扩展
						- x轴扩展 增加副本数 ✔️
						- y轴扩展 功能、业务拆分成不同的微服务
						- z轴扩展 数据分片，租户隔离
				- [[etcd]]
					- etcd 是兼具一致性和高可用性的键值数据库
						- ((6263b38c-6891-4342-9190-7b68c9b803ee)) 一致性
						- ((6263b398-d03f-403d-a306-39fb4491a41b)) 高可用
						- ((6263b3ab-4843-43de-97ba-a431f053b10a)) 分区容错
				- kube-scheduler
					- 负责监视新创建的、未指定运行节点的 Pods，选择节点让 Pod 在上面运行。
					- 调度决策考虑的因素包括
						- 单个 Pod 和 Pod 集合的资源需求
						- 硬件/软件/策略约束
						- 亲和性和反亲和性规范
						- 数据位置
						- 工作负载间的干扰和最后时限
				- kube-controller-manager
				  collapsed:: true
					- 节点控制器（Node Controller）: 负责在节点出现故障时进行通知和响应
					- 任务控制器（Job controller）: 监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
					- 端点控制器（Endpoints Controller）: 填充端点(Endpoints)对象(即加入 Service 与 Pod)
					- 服务帐户和令牌控制器（Service Account & Token Controllers）: 为新的命名空间创建默认帐户和 API 访问令牌
				- cloud-controller-manager
				  collapsed:: true
					- 节点控制器（Node Controller）: 用于在节点终止响应后检查云提供商以确定节点是否已被删除
					- 路由控制器（Route Controller）: 用于在底层云基础架构中设置路由
					- 服务控制器（Service Controller）: 用于创建、更新和删除云提供商负载均衡器
			- Node组件
				- kubelet
				  collapsed:: true
					- 一个在集群中每个节点上运行的代理。
					- 它保证容器（containers）都 运行在 Pod 中。
					- kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。
					- kubelet 不会管理不是由 Kubernetes 创建的容器。
				- kube-proxy
				  collapsed:: true
					- 是集群中每个节点上运行的网络代理
					- 维护节点上的网络规则
					- 如果操作系统提供了数据包过滤层并可用的话，kube-proxy 会通过它(例如[[iptables]])来实现网络规则。否则， kube-proxy 仅转发流量本身。
				- 容器运行时(Container Runtime)
				  collapsed:: true
					- [[Docker]]
					- [[containerd]]
					- [[CRI-O]]
			- 插件(Addon
				- DNS
				  collapsed:: true
					- 集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。
					- Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。
				- Web界面(仪表盘)
				- 容器资源监控
				- 集群层面日志
		- [KubernetesAPI](https://kubernetes.io/zh/docs/concepts/overview/kubernetes-api/)
		  collapsed:: true
			- OpenAPI规范
				- OpenAPI V2
				- OpenAPI V3
		- [使用Kubernetes对象](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/)
		  id:: 62638d0b-9f49-4406-a924-d4ea728d6efb
			- [理解Kubernetes对象](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/kubernetes-objects/)
			  id:: 62638d0b-62aa-4e9f-9892-128653b42c87
				- 总结
					- Kubernetes 对象是持久化的实体
					- Kubernetes 使用这些实体去表示整个集群的状态
					- 达成的效果：
						- 哪些容器化应用在运行（以及在哪些节点上）
						- 可以被应用使用的资源
						- 关于应用运行时表现的策略，比如重启策略、升级策略，以及容错策略
					- Kubernetes 对象是 “目标性记录” —— 一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。 通过创建对象，本质上是在告知 Kubernetes 系统，所需要的集群工作负载看起来是什么样子的， 这就是 Kubernetes 集群的 期望状态（Desired State）。
					- 所有对Kubernetes对象的操作，最终都是通过使用 Kubernetes API
				- 对象规约（Spec）与状态（Status）
					- Spec： 对象期望状态（Desired State）。
					- Status：对象当前状态（Current State）
				- 描述 Kubernetes 对象
					- 多数情况下是使用yaml文件描述。
				- 必需字段
					- apiVersion - 创建该对象所使用的 Kubernetes API 的版本
						- 例如：apps/v1
					- kind - 想要创建的对象的类别
						- 例如：Pod，Deployment，Service
					- metadata - 帮助唯一性标识对象的一些数据，包括一个 name 字符串、UID 和可选的 namespac
					- spec - 你所期望的该对象的状态，对象 spec 的精确格式对每个 Kubernetes 对象来说是不同的。
			- [Kubernetes对象管理](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/object-management/)
			  id:: 62638d0b-2d2a-4742-b172-68791f409b92
				- 管理技巧
					- ![image.png](../assets/image_1650726685838_0.png)
				- [指令式命令](https://kubernetes.io/zh/docs/tasks/manage-kubernetes-objects/imperative-command/)
					- 例子
					  kubectl create deployment nginx --image nginx
					- 权衡
						- 与对象配置相比的有点：
							- 命令简单，易学且易于记忆。
							- 命令仅需一步即可对集群进行更改。
						- 与对象配置相比的缺点：
							- 命令不便于与变更审查流程的集成。
							- 命令不提供与更改关联的审核跟踪。
							- 除了实时内容外，命令不提供记录原。
							- 明星不提供用于创建新对象的模板。
				- [指令式对象配置](https://kubernetes.io/zh/docs/tasks/manage-kubernetes-objects/imperative-config/)
					- 例子
						- kubectl create -f nginx.yaml
						- kubectl delete -f nginx.yaml -f redis.yaml
						- kubectl replace -f nginx.yaml
					- 权衡
						- 与执行令式命令相比
							- 优点：
								- 对象配置可以存储在源控制系统中，比如Git。
								- 对象配置可以与流程集成，例如在推送和审计之前检查更新。
								- 对象配置提供了用于创建新对象的模板。
							- 缺点：
								- 对象配置需要对对象架构有基本的了解。(不算缺点)
								- 对象配置需要额外的步骤来编写yaml文件。
						- 与声明式对象配置相比：
							- 优点：
								- 指令式对象配置行为更加简单易懂。
								- 从 Kubernetes 1.5 版本开始，指令对象配置更加成熟。
							- 缺点：
								- 指令式对象配置更适合文件，而非目录。
								- 对活动对象的更新，必须反应在配置文件中，否则会在下一次替换时丢失。(比如副本数，文件中为3，手动调整为6后，重新发布，又变回了3)
				- [声明式对象配置](https://kubernetes.io/zh/docs/tasks/manage-kubernetes-objects/declarative-config/)
					- 例子
						- kubectl diff -f configs/
						  kubectl apply -f configs/
						- kubectl diff -R -f configs/
						  kubectl apply -R -f configs/
					- 权衡
						- 与指令式对象配置相比
							- 优点：
								- 对活动对象所做的更改即使未合并到配置文件中，也会被保留下来。
								- 声明性对象配置更好地支持对目录进行操作并自动检测每个文件的操作类型(创建，修补，删除)。
							- 缺点：
								- 声明式对象配置难于调试并且出现异常时结果难以解释。
								- 使用diff产生的部分更新会创建复杂的合并和布丁操作。
				- 分析工作场景，选择合适的方案
					- 运维中要解决的问题
						- 1.审计，谁在什么时间做了什么操作，达到了什么效果
						- 2.可重复执行，相同的操作重复执行，结果不变
						- 3.可回滚
							- 便捷且安全的回滚方案，支持单个应用和多个相关应用的回滚。
							- 命令如何回滚，配置如何回滚？
						- 4.版本控制，版本差异的比对
						- 5.CI/CD
						-
			- [对象名称和IDs](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/names/)
			  id:: 62638d0b-e442-4e7d-bf15-8874f56955cb
			  collapsed:: true
				- 名称 集群中的每一个对象都有一个名称 来标识在同类资源中的唯一性。
					- DNS子域名
						- 不能超过253个字符
						- 只能包含小写字母、数字，以及'-' 和 '.'
						- 须以字母数字开头
						- 须以字母数字结尾
					- RFC 1123
						- 最多 63 个字符
						- 只能包含小写字母、数字，以及 '-'
						- 须以字母数字开头
						- 须以字母数字结尾
					- RFC 1035
					- 路径分段名称
				- UIDs 每个 Kubernetes 对象也有一个UID 来标识在整个集群中的唯一性。
			- [命名空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/)
			  id:: 62638d0b-7c8b-424a-be77-3df5bd50c77f
			- [标签和选择算符](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels/)
			- [注释](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/annotations/)
			- [Finalizers](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/finalizers/)
			- [字段选择器](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/field-selectors/)
			- [属主与附属](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/owners-dependents/)
			- [推荐时间用的标签](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/common-labels/)
	- [服务、负载均衡和联网](https://kubernetes.io/zh/docs/concepts/services-networking/)
	  collapsed:: true
		- Ingress
			- [Ingress nginx](https://kubernetes.github.io/ingress-nginx/deploy/)
				- 部署命令：
				  ~~~shell
				  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
				  ~~~
				- 使用到的镜像
					- k8s.gcr.io/ingress-nginx/controller:v1.2.0
					- k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
					- registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.2.0
					- registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.1.1
					-
		- Ingress控制器
			-
	-
	-
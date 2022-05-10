- 已经在阿里云创建好 k8s 集群，例如：ACK 专有集群
	- PS：集群创建完成后，可提前配置域名解析
- 通过 yaml 一次性部署 bookinfo
	- 进入 k8s 集群管理页面，工作负载->无状态，点击“使用 yaml 创建资源”
	  collapsed:: true
		- ![image.png](../assets/image_1652084967958_0.png)
		- ![image.png](../assets/image_1652084983310_0.png)
	- 将 ladder 中的 samples 目录下的 bookinfo.yaml 中的内容复制到模板中，点击创建
	  collapsed:: true
		- ![image.png](../assets/image_1652085079274_0.png)
	- 查看所有的 Deployment
	  collapsed:: true
		- ![image.png](../assets/image_1652085167977_0.png)
	- 查看所有的 Service
	  collapsed:: true
		- ![image.png](../assets/image_1652085196549_0.png)
		-
- 配置路由 所有请求均路由到 product 服务
	- 进入 k8s 集群管理页面，网络->路由，点击“创建”
	  collapsed:: true
		- ![image.png](../assets/image_1652085576562_0.png)
	- 填写路由信息
	  collapsed:: true
		- 填写名称，例如：product-route
		- 填写域名，例如：k8s.catface996.com
		- 选择服务，例如：product
		  collapsed:: true
			- ![image.png](../assets/image_1652085641035_0.png)
	- 点击“创建”，生效路由配置
	  collapsed:: true
		- ![image.png](../assets/image_1652085924197_0.png)
- 配置域名解析  k8s.catface996.com 解析到 slb 的公网 IP
	- 前提：已经有购买域名，且对域名做了备案。
	- 找到 k8s 集群公网暴露的 IP 地址
	  collapsed:: true
		- 负载均衡->传统型负载均衡->实例管理，找到公网 IP，例如：47.110.63.59
		- ![image.png](../assets/image_1652078842816_0.png)
	- 域名服务->域名列表->点击相应域名的“解析”，例如：catface996.com
	  collapsed:: true
		- ![image.png](../assets/image_1652079069897_0.png)
	- 进入解析页面，点击“添加记录”
	  collapsed:: true
		- 选择记录类型，例如：A
		- 填写主机记录，例如：api
		- 选择解析线路，例如：默认
		- 填写记录值，例如：47.110.63.59
		- 填写 TTL，例如：10 分钟
		- ![image.png](../assets/image_1652079267732_0.png){:height 354, :width 685}
		- 点击确认，返回解析列表页，等待解析生效，等待生效的过程中，可以打开浏览器去访问该域名
	- **PS：视频中的配置是有问题的，主机记录写错了，应该是 k8s**
	  collapsed:: true
		- 错误的示范
		- ![image.png](../assets/image_1652089806208_0.png)
	- 浏览器访问配置的域名解析 http://k8s.catface996.com
	  collapsed:: true
		- 浏览器有返回，即表示解析生效
		- ![image.png](../assets/image_1652079477519_0.png)
		-
- 使用 HTTP通访问 bookinfo
	- 请求地址： http://k8s.catface996.com/productpage
	  collapsed:: true
		- ![image.png](../assets/image_1652079698893_0.png)
- 使用 HTTPS 访问 bookinfo
	- PS：有多种方案，可以在 slb 上，也可以在集群的 Ingress 上终结 SSL/TLS，此次是选择在 Ingress 终结 TLS
	- 申请证书并下载
	  collapsed:: true
		- 前提，已经购买证书，免费版可以提供 20 个免费证书。
		- 点击“创建证书”，会在证书列表生成一条新的证书记录
			- ![image.png](../assets/image_1652078099857_0.png)
		- 点击“证书申请”，进入证书申请页面
			- ![image.png](../assets/image_1652078200832_0.png)
		- 填写申请证书信息
		  collapsed:: true
			- 填写证书绑定的域名，例如：api.catface996.com
			- 选择验证方式，例如：自动DNS 验证
			- 选择联系人
			- 选择所在地
			- 选择秘钥算法，例如：RSA
			- 选择 CSR 生成方式，例如：系统生成
			- ![image.png](../assets/image_1652078293342_0.png){:height 353, :width 685}
			- 点击“下一步”，进入验证页面
				- 点击“验证”
					- ![image.png](../assets/image_1652078565258_0.png)
					- ![image.png](../assets/image_1652078599642_0.png)
				- 验证通过后，点击“提交审核”
					- ![image.png](../assets/image_1652078646471_0.png)
					-
				- 查看申请的证书
					- ![image.png](../assets/image_1652078692535_0.png)
					-
	- 部署证书到 k8s 集群的
	  collapsed:: true
		- 下载证书，选择 nginx类型
			- ![image.png](../assets/image_1652084365250_0.png)
			-
		- 在 k8s 集群中，增加保密字典
			- 配置管理->保密字典->创建
				- ![image.png](../assets/image_1652083868035_0.png)
			- 填写名称，例如：k8s.catface996.com
			- 选择类型，例如：TLS 证书
			- 填写 Cert，对应下载文件  xxx.pem 中的内容
			- 填写 Key，对应下载文件 xxx.key 中的内容
			- 点击“确定”
				- ![image.png](../assets/image_1652083928780_0.png)
				-
			-
		- 在路由中启用 TLS
			- k8s 集群详情->网络->路由->变更相应的路由
				- ![image.png](../assets/image_1652084546536_0.png)
			- 启用TLS，选择创建的证书
				- ![image.png](../assets/image_1652084587635_0.png)
	- 浏览器访问 https://k8s.catface996.com  or https://k8s.catface996.com/productpage
	-
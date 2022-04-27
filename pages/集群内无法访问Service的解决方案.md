- 集群内部访问cat-svc
	- 期望在容器内部: curl http://cat-svc:80/sayHello
	- 解决pod中访问service的过程如下
		- 根据Kubernetes官网,查看pod的dns配置
		  collapsed:: true
			- 官方文档地址:
			  https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
			  
			  截图:
			  ![20220408000100](https://picgo.catface996.com/picgo20220408000100.png)
			  
			  pod的dns策略
			  ![20220408103013](https://picgo.catface996.com/picgo20220408103013.png)
			  
			  pod的dns配置
			  ![20220408103128](https://picgo.catface996.com/picgo20220408103128.png)
			  
			  ~~~shell
			  # 查看pod的dns配置的命令
			  cat /etc/resolv.conf
			  
			  [root@cat-dp-58cf7df488-9xxfh app]# cat /etc/resolv.conf
			  nameserver 10.10.0.10
			  search default.svc.cluster.local svc.cluster.local cluster.local
			  options ndots:5
			  ~~~
		- 排查过程
		- 检查cat-dp的pod是否存在且能访问
			- ~~~shell
			  # 进入pod的容器后
			  curl http://127.0.0.1:9001/sayHello
			  ~~~
		- 检查cat-svc的service是否存在且能访问
			- 查看service的clusterIP
			  
			  ~~~shell
			  # 进入pod的容器后
			  telnet service的clusterIP 80
			  
			  curl http://service的clusterIP:80/sayHello
			  ~~~
		- 查看pod的dns配置
			- ~~~shell
			  # 进入pod的容器后
			  
			  nslookup cat-svc
			  # 执行结果,没有响应
			  
			  cat /etc/resolv.conf
			  # 执行结果
			  [root@cat-dp-58cf7df488-9xxfh app]# cat /etc/resolv.conf
			  nameserver 10.10.0.10
			  search default.svc.cluster.local svc.cluster.local cluster.local
			  options ndots:5
			  ~~~
		- 检查kube-dns是否存在且能访问
		- 检查coredns是否存在且能访问
		- 搭建一个相同的集群,网络插件使用calico
		- 解决方案
			- 在宿主机上配置iptables
				- ~~~shell
				  iptables -P FORWARD ACCEPT
				  iptables -P INPUT ACCEPT
				  iptables -P OUTPUT ACCEPT
				  iptables -F
				  ~~~
			- 重启docker服务
				- ~~~shell
				  systemctl restart docker.service
				  ~~~
		- 思考
			- 我们这样解决安全吗?
			- 是否有其他更好的解决方案?
			- 背后的原理是什么?
			- 引出的知识盲区
			- iptables和netfilter
			- flannel的工作机制是怎样的?
			- calico的工作机制是怎样的?
		-
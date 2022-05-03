- 希望达成的目标
	- 业务网关做 prod、gray 决策
	- 动态修改用户所属 prod、gray
- 流量拓扑图
	- ![image.png](../assets/image_1651554979453_0.png)
	-
- SpringCloudGateway
	- 参考: https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-Hoxton-Release-Notes#hoxtonsr9
- istio-demo-app
	- 用于部署 cat、dog、monkey 服务的源代码
	- github 地址：https://github.com/catface996/istio-demo-app
	- gitee地址： https://gitee.com/catface996/istio-demo-app
- spring-cloud-gateway-istio 工程源码
	- github 地址：https://github.com/catface996/spring-cloud-gateway-istio
	- gitee 地址：https://gitee.com/catface996/spring-cloud-gateway-istio
- spring-cloud-gateway-istio 代码讲解
	- token转 userId
	- 根据 userId 决策 prod、gray
	- env=prod/gray 写入 request header 并传递
- 实施步骤
	- ((62638d0b-abac-4fa7-bb9a-8d08efa0a4f6))
	- 打开jaejer 面板
		- ```shell
		  istioctl dashboard jaeger --address 192.168.162.22
		  ```
	- 部署Deployment
		- ```shell
		  ## 进入 ladder/code/istio/spring-cloud-gateway 后执行以下命令
		  
		  kubectl apply -f deployment.yaml
		  ```
		- ![image.png](../assets/image_1651556611047_0.png)
		- ![image.png](../assets/image_1651556631421_0.png)
	- 部署Service
		- ```shell
		  ## 进入 ladder/code/istio/spring-cloud-gateway 后执行以下命令
		  
		  kubectl apply -f service.yaml
		  ```
		- ![image.png](../assets/image_1651557021369_0.png)
	- 绑定 spring-cloud-gateway 的 Virtual Service 和 istio-ingressgateway
		- ```shell
		  ## 进入 ladder/code/istio/spring-cloud-gateway 后执行以下命令
		  
		  kubectl apply -f biz-gateway-vs-bind-gateway.yaml
		  ```
	- 配置本地 hosts，通过 curl 访问 http://gateway.catface996.com:31606/cat/sayHello
		- 为什么要访问 /cat/sayHello   /cat 是用来在 spring cloud gateway 中做服务路由的前缀。
		- ![image.png](../assets/image_1651556731059_0.png)
		- ![image.png](../assets/image_1651556764657_0.png)
		- ```shell
		  curl http://gateway.catface996.com:31606/cat/sayHello
		  ```
	- 部署 Destination Rule
	- 部署 Virtual Service
	- 针对不同的用户做灰度验证
	- 修改用户所属环境再验证
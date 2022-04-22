- 任务
	- 交通管理
		- 请求路由
			- 应用目标规则
			- ~~~shell
			  # 应用目标规则
			  kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
			  
			  # 查看目标规则
			  kubectl get destinationrules -o yaml
			  
			  # 删除目标规则
			  kubectl delete -f samples/bookinfo/networking/destination-rule-all.yaml
			  ~~~
			- 应用虚拟服务之前
				- ![image.png](../assets/image_1650644198341_0.png)
				- ![image.png](../assets/image_1650644215695_0.png)
				- ![image.png](../assets/image_1650644254232_0.png)
			- 应用虚拟服务
				- ~~~shell
				  # 应用虚拟服务
				  kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
				  
				  # 实际执行结果
				  [root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
				  virtualservice.networking.istio.io/productpage created
				  virtualservice.networking.istio.io/reviews created
				  virtualservice.networking.istio.io/ratings created
				  virtualservice.networking.istio.io/details created
				  
				  # 查看虚拟服务
				  kubectl get virtualservices -o yaml
				  
				  # 删除虚拟服务
				  kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
				  ~~~
				- 只出现v1版本
				  ![image.png](../assets/image_1650644356611_0.png){:height 177, :width 716}
				- 如果只应用了虚拟服务，没有应用目标规则
					- ![image.png](../assets/image_1650645107656_0.png)
			- 测试新的路由
				- 查找istio-ingressgateway的NodePort端口：
				  ~~~shell
				  [root@k8s-master-22 ~]# kubectl get svc -n istio-system
				  NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
				  grafana                ClusterIP      10.10.194.35    <none>        3000/TCP                                                                     11d
				  istio-egressgateway    ClusterIP      10.10.175.208   <none>        80/TCP,443/TCP                                                               11d
				  istio-ingressgateway   LoadBalancer   10.10.29.102    <pending>     15021:31622/TCP,80:31606/TCP,443:32649/TCP,31400:31288/TCP,15443:32075/TCP   11d
				  istiod                 ClusterIP      10.10.204.13    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        11d
				  jaeger-collector       ClusterIP      10.10.111.47    <none>        14268/TCP,14250/TCP,9411/TCP                                                 11d
				  kiali                  ClusterIP      10.10.28.105    <none>        20001/TCP,9090/TCP                                                           11d
				  prometheus             ClusterIP      10.10.189.192   <none>        9090/TCP                                                                     11d
				  tracing                ClusterIP      10.10.208.37    <none>        80/TCP,16685/TCP                                                             11d
				  zipkin                 ClusterIP      10.10.9.194     <none>        9411/TCP                                                                     11d
				  ~~~
				- 通过浏览器请求 /productpage   完整地址： http://192.168.162.22:31606/productpage
				- 验证结果，无论请求多少次，始终是v1版本
					- ![image.png](../assets/image_1650644841159_0.png)
					-
			- 基于用户身份的路由
				- 启用基于用户的路由
				- ~~~shell
				  kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
				  
				  # 实际执行结果
				  [root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
				  virtualservice.networking.istio.io/reviews configured
				  ~~~
				- 未登录
					- ![image.png](../assets/image_1650644921434_0.png)
				- 使用jason登录
					- ![image.png](../assets/image_1650644964614_0.png)
				- 使用其他任何用户登录，例如：catface996
					- ![image.png](../assets/image_1650645014127_0.png)
			- 了解发生了什么
			- 清理
				- ~~~shell
				  kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
				  ~~~
- 例子
	- [图书应用](https://istio.io/latest/docs/examples/bookinfo/)
		- 没有istio的版本
			- ![image.png](../assets/image_1650644733819_0.png)
			-
		- 部署了istio的版本
			- ![image.png](../assets/image_1650644695876_0.png)
			-
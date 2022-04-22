- 任务
	- 交通管理
		- 请求路由
			- 应用虚拟服务
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
				- 通过浏览器请求 /productpage
				-
			- 基于用户身份的路由
			- 了解发生了什么
			- 清理
-
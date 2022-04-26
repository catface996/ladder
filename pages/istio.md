- 任务
	- 流量管理
		- [官方文档](https://istio.io/latest/docs/tasks/traffic-management/)
		- 请求路由
		  collapsed:: true
		  id:: 62638d0b-1d62-4986-bc05-43098be840e2
			- [官方文档](https://istio.io/latest/docs/tasks/traffic-management/request-routing/)
			- 前置准备
				- 访问bookinfo
				  id:: 62638d0b-8892-4c54-b53c-439a6660ef62
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
				- 打开kiali面板
				  id:: 62638d0b-abac-4fa7-bb9a-8d08efa0a4f6
					- ~~~shell
					  istioctl dashboard kiali --address 192.168.162.22
					  
					  # 实际执行结果
					  [root@k8s-master-22 ~]# istioctl dashboard kiali --address 192.168.162.22
					  http://192.168.162.22:20001/kiali
					  Failed to open browser; open http://192.168.162.22:20001/kiali in your browser.
					  ~~~
			- 应用目标规则
				- id:: 62638d0b-25b9-4026-8323-d21e1daf5501
				  ~~~shell
				  # 应用目标规则
				  kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
				  
				  # 查看目标规则
				  kubectl get destinationrules -o yaml
				  
				  # 删除目标规则
				  kubectl delete -f samples/bookinfo/networking/destination-rule-all.yaml
				  ~~~
				- ~~~yml
				  apiVersion: networking.istio.io/v1alpha3
				  kind: DestinationRule
				  metadata:
				    name: productpage
				  spec:
				    host: productpage
				    subsets:
				    - name: v1
				      labels:
				        version: v1
				  ---
				  apiVersion: networking.istio.io/v1alpha3
				  kind: DestinationRule
				  metadata:
				    name: reviews
				  spec:
				    host: reviews
				    subsets:
				    - name: v1
				      labels:
				        version: v1
				    - name: v2
				      labels:
				        version: v2
				    - name: v3
				      labels:
				        version: v3
				  ---
				  apiVersion: networking.istio.io/v1alpha3
				  kind: DestinationRule
				  metadata:
				    name: ratings
				  spec:
				    host: ratings
				    subsets:
				    - name: v1
				      labels:
				        version: v1
				    - name: v2
				      labels:
				        version: v2
				    - name: v2-mysql
				      labels:
				        version: v2-mysql
				    - name: v2-mysql-vm
				      labels:
				        version: v2-mysql-vm
				  ---
				  apiVersion: networking.istio.io/v1alpha3
				  kind: DestinationRule
				  metadata:
				    name: details
				  spec:
				    host: details
				    subsets:
				    - name: v1
				      labels:
				        version: v1
				    - name: v2
				      labels:
				        version: v2
				  ---
				  
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
				- ~~~yaml
				  apiVersion: networking.istio.io/v1alpha3
				  kind: VirtualService
				  metadata:
				    name: productpage
				  spec:
				    hosts:
				    - productpage
				    http:
				    - route:
				      - destination:
				          host: productpage
				          subset: v1
				  ---
				  apiVersion: networking.istio.io/v1alpha3
				  kind: VirtualService
				  metadata:
				    name: reviews
				  spec:
				    hosts:
				    - reviews
				    http:
				    - route:
				      - destination:
				          host: reviews
				          subset: v1
				  ---
				  apiVersion: networking.istio.io/v1alpha3
				  kind: VirtualService
				  metadata:
				    name: ratings
				  spec:
				    hosts:
				    - ratings
				    http:
				    - route:
				      - destination:
				          host: ratings
				          subset: v1
				  ---
				  apiVersion: networking.istio.io/v1alpha3
				  kind: VirtualService
				  metadata:
				    name: details
				  spec:
				    hosts:
				    - details
				    http:
				    - route:
				      - destination:
				          host: details
				          subset: v1
				  ---
				  
				  ~~~
				- 只出现v1版本
					- ![image.png](../assets/image_1650644356611_0.png){:height 177, :width 716}
				- 如果只应用了虚拟服务，没有应用目标规则
					- ![image.png](../assets/image_1650645107656_0.png)
			- 应用虚拟服务之后，测试新的路由
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
					- ~~~yaml
					  apiVersion: networking.istio.io/v1alpha3
					  kind: VirtualService
					  metadata:
					    name: reviews
					  spec:
					    hosts:
					      - reviews
					    http:
					    - match:
					      - headers:
					          end-user:
					            exact: jason
					      route:
					      - destination:
					          host: reviews
					          subset: v2
					    - route:
					      - destination:
					          host: reviews
					          subset: v1
					  
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
		- 故障注入
		  id:: 6264e673-0dc2-4025-9229-96f2cb81461a
		  collapsed:: true
			- [官方文档](https://istio.io/latest/zh/docs/tasks/traffic-management/fault-injection/)
			- 开始之前
				- ✔️已经装好istio。
				- ✔️已经部署Bookinfo，并应用了默认的目标规则。
				  collapsed:: true
					- ((62638d0b-25b9-4026-8323-d21e1daf5501))
				- TODO  在流量管理概念文档中查看有关 ((6264e77a-3345-4ce0-a63f-2bae8b1149fb)) 的讨论。
				- TODO 通过自行配置请求路由任务或运行一下命令来初始化应用程序版本路由。
				  collapsed:: true
				  :LOGBOOK:
				  CLOCK: [2022-04-24 Sun 16:20:06]--[2022-04-24 Sun 16:20:07] =>  00:00:01
				  :END:
					- ~~~shell
					  kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
					  kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
					  ~~~
				- 经过上面的配置，下面是请求的流程：
					- productpage → reviews:v2 → ratings (针对 jason 用户)
						- TODO 浏览器访问，kiali展示
					- productpage → reviews:v1 (其他用户)
						- TODO 浏览器访问，kiali展示
				- ((62638d0b-8892-4c54-b53c-439a6660ef62)) 和 ((62638d0b-abac-4fa7-bb9a-8d08efa0a4f6))
			- 注入HTTP延迟故障
				- 创建故障注入规则以延迟来自测试用户jason的流量
				  collapsed:: true
					- ~~~shell
					  # 查看 virtual-service-ratings-test-delay.yaml
					  [root@k8s-master-22 istio]# cat samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
					  apiVersion: networking.istio.io/v1alpha3
					  kind: VirtualService
					  metadata:
					    name: ratings
					  spec:
					    hosts:
					    - ratings
					    http:
					    - match:
					      - headers:
					          end-user:
					            exact: jason
					      fault:
					        delay:
					          percentage:
					            value: 100.0
					          fixedDelay: 7s
					      route:
					      - destination:
					          host: ratings
					          subset: v1
					    - route:
					      - destination:
					          host: ratings
					          subset: v1
					          
					  # 应用延迟
					  kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
					  ~~~
				- 确认规则已经创建
				  collapsed:: true
					- ~~~shell
					  kubectl get virtualservice ratings -o yaml
					  
					  # 确认结果
					  [root@k8s-master-22 istio]# kubectl get virtualservice ratings -o yaml
					  apiVersion: networking.istio.io/v1beta1
					  kind: VirtualService
					  metadata:
					    annotations:
					      kubectl.kubernetes.io/last-applied-configuration: |
					        {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"hosts":["ratings"],"http":[{"fault":{"delay":{"fixedDelay":"7s","percentage":{"value":100}}},"match":[{"headers":{"end-user":{"exact":"jason"}}}],"route":[{"destination":{"host":"ratings","subset":"v1"}}]},{"route":[{"destination":{"host":"ratings","subset":"v1"}}]}]}}
					    creationTimestamp: "2022-04-24T04:46:10Z"
					    generation: 3
					    name: ratings
					    namespace: default
					    resourceVersion: "102703"
					    uid: 5df5c140-d41c-4d31-ab5f-3b4885155f28
					  spec:
					    hosts:
					    - ratings
					    http:
					    - fault:
					        delay:
					          fixedDelay: 7s
					          percentage:
					            value: 100
					      match:
					      - headers:
					          end-user:
					            exact: jason
					      route:
					      - destination:
					          host: ratings
					          subset: v1
					    - route:
					      - destination:
					          host: ratings
					          subset: v1
					  ~~~
			- 测试延迟配置
				- 通过浏览器打开Bookinfo应用。
				- 使用用户jason登录到、productpage页面。
				- 查看页面的响应时间。
				- 结果：
					- ![image.png](../assets/image_1650784019533_0.png)
					- ![image.png](../assets/image_1650783924834_0.png)
					-
			- 理解原理
				- productpage -- 3s * 2 --> reviews -- 10s --> ratings
			- 错误修复
				- 增加productpage与reviews服务之间的超时或降低reviews与ratings的超时。
				- 终止并重启修复后端额微服务。
				- 确认/productpage页面正常响且没有任何错误。
			- 注入HTTP abort故障
				- 为用户jason创建一个发送HTTP abort的故障注入规则
				  ~~~shell
				  # 查看待应用的abort配置
				  [root@k8s-master-22 istio]# cat  samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
				  apiVersion: networking.istio.io/v1alpha3
				  kind: VirtualService
				  metadata:
				    name: ratings
				  spec:
				    hosts:
				    - ratings
				    http:
				    - match:
				      - headers:
				          end-user:
				            exact: jason
				      fault:
				        abort:
				          percentage:
				            value: 100.0
				          httpStatus: 500
				      route:
				      - destination:
				          host: ratings
				          subset: v1
				    - route:
				      - destination:
				          host: ratings
				          subset: v1
				          
				  # 应用配置        
				  kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
				  ~~~
				- 确认规则已经创建
				  ~~~shell
				  kubectl get virtualservice ratings -o yaml
				  
				  ## 查看结果
				  [root@k8s-master-22 istio]# kubectl get virtualservice ratings -o yaml
				  apiVersion: networking.istio.io/v1beta1
				  kind: VirtualService
				  metadata:
				    annotations:
				      kubectl.kubernetes.io/last-applied-configuration: |
				        {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"hosts":["ratings"],"http":[{"fault":{"abort":{"httpStatus":500,"percentage":{"value":100}}},"match":[{"headers":{"end-user":{"exact":"jason"}}}],"route":[{"destination":{"host":"ratings","subset":"v1"}}]},{"route":[{"destination":{"host":"ratings","subset":"v1"}}]}]}}
				    creationTimestamp: "2022-04-24T04:46:10Z"
				    generation: 4
				    name: ratings
				    namespace: default
				    resourceVersion: "104859"
				    uid: 5df5c140-d41c-4d31-ab5f-3b4885155f28
				  spec:
				    hosts:
				    - ratings
				    http:
				    - fault:
				        abort:
				          httpStatus: 500
				          percentage:
				            value: 100
				      match:
				      - headers:
				          end-user:
				            exact: jason
				      route:
				      - destination:
				          host: ratings
				          subset: v1
				    - route:
				      - destination:
				          host: ratings
				          subset: v1
				  ~~~
			- 测试终止配置
				- 用浏览器打开Bookinfo应用。
				- 使用用户jason登录到/productpage页面。
					- ![image.png](../assets/image_1650785710826_0.png)
					- ![image.png](../assets/image_1650785757677_0.png)
				- 使用匿名用户打开Bookinfo。
					- ![image.png](../assets/image_1650785829741_0.png)
					- ![image.png](../assets/image_1650785954965_0.png)
			- 清理
				- ~~~shell
				  kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
				  ~~~
		- 流量转移
		  id:: 626600dd-9313-4c88-85f4-59f1c2f17f5d
		  collapsed:: true
			- [官方文档](https://istio.io/latest/zh/docs/tasks/traffic-management/traffic-shifting/)
			- 开始之前
			  collapsed:: true
				- 已经安装好istio。
				- 已经部署好Bookinfo示例应用程序。
				- 已经了解流量管理概念。
			- 应用基于权重的路由
				- 已经配置了默认的目标规则
					- ~~~shell
					  kubectl get dr
					  
					  ## 实际执行结果
					  [root@k8s-master-22 istio]# kubectl get dr
					  NAME          HOST          AGE
					  details       details       2d8h
					  productpage   productpage   2d8h
					  ratings       ratings       2d8h
					  reviews       reviews       2d8h
					  ~~~
				- 将所有流量路由到v1版本
					- ~~~shell
					  kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
					  
					  ## 实际执行结果
					  [root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
					  virtualservice.networking.istio.io/productpage created
					  virtualservice.networking.istio.io/reviews created
					  virtualservice.networking.istio.io/ratings created
					  virtualservice.networking.istio.io/details created
					  ~~~
				- 在浏览器中打开/productpage，验证是否都路由到v1
					- ((62638d0b-8892-4c54-b53c-439a6660ef62))
					- ((62638d0b-abac-4fa7-bb9a-8d08efa0a4f6))
					- ![image.png](../assets/image_1650855080300_0.png)
					- ![image.png](../assets/image_1650855104116_0.png)
				- 使用以下命令把50%的流量从reviews:v1转移到reviews:v3
					- ~~~shell
					  ## 查看50%流量转移到v3的配置
					  cat samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
					  
					  ## 实际执行结果
					  [root@k8s-master-22 istio]# cat samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
					  apiVersion: networking.istio.io/v1alpha3
					  kind: VirtualService
					  metadata:
					    name: reviews
					  spec:
					    hosts:
					      - reviews
					    http:
					    - route:
					      - destination:
					          host: reviews
					          subset: v1
					        weight: 50
					      - destination:
					          host: reviews
					          subset: v3
					        weight: 50
					  
					  ## 应用50%流量转移到v3的配置
					  kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
					  
					  ## 实际执行结果
					  [root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
					  virtualservice.networking.istio.io/reviews configured
					  ~~~
				- 确认规则已经被替换
					- ~~~shell
					  ## 确认规则已经被替换
					  kubectl get virtualservice reviews -o yaml
					  
					  ## 实际执行结果
					  [root@k8s-master-22 istio]# kubectl get virtualservice reviews -o yaml
					  apiVersion: networking.istio.io/v1beta1
					  kind: VirtualService
					  metadata:
					    annotations:
					      kubectl.kubernetes.io/last-applied-configuration: |
					        {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":50},{"destination":{"host":"reviews","subset":"v3"},"weight":50}]}]}}
					    creationTimestamp: "2022-04-25T02:31:32Z"
					    generation: 2
					    name: reviews
					    namespace: default
					    resourceVersion: "139815"
					    uid: 32ac7e0c-444d-4835-b8f7-56c911405c2f
					  spec:
					    hosts:
					    - reviews
					    http:
					    - route:
					      - destination:
					          host: reviews
					          subset: v1
					        weight: 50
					      - destination:
					          host: reviews
					          subset: v3
					        weight: 50
					  ~~~
				- 刷新浏览器中的/productpage页面，约有50%的几率看到红色星级的评价内容。
					- ![image.png](../assets/image_1650854994067_0.png)
					- ![image.png](../assets/image_1650855011308_0.png)
					- ![image.png](../assets/image_1650854510872_0.png)
				- 流量100%路由到reviews:v3
					- ~~~shell
					  ## 查看待应用的配置
					  cat samples/bookinfo/networking/virtual-service-reviews-v3.yaml
					  
					  ## 待应用的配置
					  [root@k8s-master-22 istio]# cat samples/bookinfo/networking/virtual-service-reviews-v3.yaml
					  apiVersion: networking.istio.io/v1alpha3
					  kind: VirtualService
					  metadata:
					    name: reviews
					  spec:
					    hosts:
					      - reviews
					    http:
					    - route:
					      - destination:
					          host: reviews
					          subset: v3
					  
					  ## 应用配置，流量100%路由到v3
					  kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
					  
					  ## 应用结果
					  [root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
					  virtualservice.networking.istio.io/reviews configured
					  ~~~
					- ![image.png](../assets/image_1650854733359_0.png)
					- ![image.png](../assets/image_1650854770835_0.png)
			- 理解原理
				- 如果想了解支持自动伸缩的版本路由，请查看[使用Istio进行金丝雀部署](https://istio.io/latest/zh/blog/2017/0.1-canary/)
			- 清理
			  collapsed:: true
				- 删除应用的路由规则
				  ~~~shell
				  kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
				  ~~~
				-
		- Egress
		  collapsed:: true
			- [官方文档](https://istio.io/latest/zh/docs/tasks/traffic-management/egress/)
			- 访问外部服务
			  id:: 62678f97-a882-4d74-a432-bc4ccaef29bd
				- 官方文档[https://istio.io/latest/zh/docs/tasks/traffic-management/egress/egress-control/]
				- 开始之前
				- Envoy转发流量到外部服务
				- 控制对外部服务的访问
					- 更改默认的封锁策略
					- 访问一个外部的http服务
					- 访问外部的https服务
					- 管理到外部服务的流量
					- 清理对外服务的受控访问
				- 直接访问外部服务
					- 确定平台内部的IP范围
					- 配置代理绕行
					- 访问外部服务
					- 清除对外部服务的直接访问
				- 理解原理
				- 安全说明
				- 清理
	- 可观察性
	  collapsed:: true
		- [官方文档](https://istio.io/latest/zh/docs/tasks/observability/)
		- 指标度量
			- [官方文档](https://istio.io/latest/zh/docs/tasks/observability/metrics/)
			- 通过Prometheus查询度量指标
			  id:: 6266ab50-b550-4424-b3b4-0a17d07be1a5
				- [官方文档](https://istio.io/latest/zh/docs/tasks/observability/metrics/querying-metrics/)
				- 开始之前
				  collapsed:: true
					- DONE 已经在k8s集群中安装了istio。
					- DONE 安装 Prometheus Addon
						- 安装istio的时候，已经安装了Kiali和其他组件
						  ~~~shell
						  kubectl apply -f samples/addons
						  ~~~
					- DONE 部署Bookinfo应用。
				- 查询Istio度量指标
					- 验证集群中运行的Prometheus服务
					  collapsed:: true
						- ~~~shell
						  kubectl -n istio-system get svc prometheus
						  
						  ## 实际执行结果
						  [root@k8s-master-22 ~]# kubectl -n istio-system get svc prometheus
						  NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
						  prometheus   ClusterIP   10.10.189.192   <none>        9090/TCP   14d
						  ~~~
					- 向网格发送流量
					  collapsed:: true
						- ((62638d0b-8892-4c54-b53c-439a6660ef62))
					- 打开Prometheus UI
					  collapsed:: true
						- ~~~shell
						  ## 启动Prometheus代理
						  istioctl dashboard prometheus --address 192.168.162.22
						  
						  ## 执行结果
						  [root@k8s-master-22 ~]# istioctl dashboard prometheus --address 192.168.162.22
						  http://192.168.162.22:9090
						  Failed to open browser; open http://192.168.162.22:9090 in your browser.
						  ~~~
						- ![image.png](../assets/image_1650897954527_0.png)
					- 执行一个Prometheus查询
						- ~~~shell
						  istio_requests_total
						  ~~~
						- 执行结果
							- ![image.png](../assets/image_1650898331546_0.png)
						- 切换成Graph视图
							- ![image.png](../assets/image_1650898372619_0.png)
					- 其他尝试
						- 请求productpage服务的总次数
							- ~~~shell
							  istio_requests_total{destination_service="productpage.default.svc.cluster.local"}
							  ~~~
							- ![image.png](../assets/image_1650956801234_0.png)
						- 请求reviews服务v3版本的总次数
							- ~~~shell
							  istio_requests_total{destination_service="reviews.default.svc.cluster.local", destination_version="v3"}
							  ~~~
							- ![image.png](../assets/image_1650956784682_0.png)
						- 过去5分钟productpage服务所有实例的请求频次
							- ~~~shell
							  rate(istio_requests_total{destination_service=~"productpage.*", response_code="200"}[5m])
							  ~~~
							- ![image.png](../assets/image_1650956896033_0.png)
				-
			- 使用Grafana可视化指标
			  id:: 62678da1-e553-4b20-ab42-d054b66bf6a9
				- [官方文档](https://istio.io/latest/zh/docs/tasks/observability/metrics/using-istio-dashboard/)
				- 开始之前
					- 在集群中安装Istio，并启用Grafana组件。
					- 部署Bookinfo应用。
				- 查看Istio Dashboard
					- 验证prometheus服务正在集群中运行。
					- 验证Grafana服务正在集群中运行。
					- 通过Grafana UI打开Istio Dashboard。
					- 发送流量到网格。
					- 可视化服务仪表盘。
					- 可视化工作负载仪表盘。
				- 关于Grafana插件
		-
	-
- 概念
  collapsed:: true
	- [流量管理](https://istio.io/latest/zh/docs/concepts/traffic-management)
		- [故障注入](https://istio.io/latest/zh/docs/concepts/traffic-management/#fault-injection)
		  id:: 6264e77a-3345-4ce0-a63f-2bae8b1149fb
			- 描述
				- 可以使用 Istio 的故障注入机制来为整个应用程序测试故障恢复能力。
				- Istio 允许在应用层注入错误。这使您可以注入更多相关的故障，例如 HTTP 错误码，以获得更多相关的结果。
			- 故障分类
				- 延迟：延迟是时间故障。它们模拟增加的网络延迟或一个超载的上游服务。
				- 终止：终止是崩溃失败。他们模仿上游服务的失败。终止通常以 HTTP 错误码或 TCP 连接失败的形式出现。
			- 注意
				- Istio 故障恢复功能对应用程序来说是完全透明的。在返回响应之前，应用程序不知道 Envoy sidecar 代理是否正在处理被调用服务的故障。这意味着，如果在应用程序代码中设置了故障恢复策略，那么您需要记住这**两个策略都是独立工作**的，否则会发生冲突。例如，假设您设置了两个超时，一个在虚拟服务中配置，另一个在应用程序中配置。应用程序为服务的 API 调用设置了 2 秒超时。而您在虚拟服务中配置了一个 3 秒超时和重试。在这种情况下，应用程序的超时会先生效，因此 Envoy 的超时和重试尝试会失效。
			-
- 例子
  collapsed:: true
	- [图书应用](https://istio.io/latest/docs/examples/bookinfo/)
		- 没有istio的版本
			- ![image.png](../assets/image_1650644733819_0.png)
			-
		- 部署了istio的版本
			- ![image.png](../assets/image_1650644695876_0.png)
			-
-
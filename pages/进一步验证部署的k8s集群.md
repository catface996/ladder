- 镜像准备
	- catface996/spring-cloud-istio-demo:latest
- 部署工作负载
	- Kubernetes官方参考文档
	  collapsed:: true
		- https://kubernetes.io/docs/concepts/workloads/
		- ![image-20220406144704955](/Users/catface/Library/Application%20Support/typora-user-images/image-20220406144704955.png)
		- Kubernetes官方参考文档 之 Deployment:
		- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
		- ![image-20220406144930814](https://tva1.sinaimg.cn/large/e6c9d24ely1h1004s4o4rj21n50u0468.jpg)
		- ![image-20220406145105184](https://tva1.sinaimg.cn/large/e6c9d24ely1h1006ez5w7j21n40u0q8r.jpg){:height 371, :width 716}
	- 在服务器上创建cat-dp.yml文件
	  collapsed:: true
		- ~~~yaml
		  apiVersion: apps/v1
		  kind: Deployment
		  metadata:
		  name: cat-dp
		  spec:
		  selector:
		  matchLabels:
		  app: cat
		  env: prod
		  replicas: 1
		  template:
		  metadata:
		  labels:
		    app: cat
		    env: prod
		  spec:
		  containers:
		    - name: spring-cloud-istio-demo
		      image: catface996/spring-cloud-istio-demo:latest
		      ports:
		        - containerPort: 9001
		          protocol: TCP
		  ~~~
	- 创建工作负载(Deployment)
	  collapsed:: true
		- ~~~shell
		  ## 待执行的命令
		  kubectl apply -f cat-dp.yml
		  
		  ## 实际执行过程
		  [root@k8s-master-51 deployment]# kubectl apply -f cat-dp.yml
		  deployment.apps/cat-dp created
		  [root@k8s-master-51 deployment]#
		  ~~~
		- 查看创建的工作负载
		- ~~~shell
		  ## 待执行的命令  可以通过-n 指定命名空间
		  kubectl get deployment
		  kubectl get deployment -n default
		  
		  ## 实际执行过程
		  [root@k8s-master-51 deployment]# kubectl get deployment
		  NAME     READY   UP-TO-DATE   AVAILABLE   AGE
		  cat-dp   1/1     1            1           87s
		  [root@k8s-master-51 deployment]#
		  
		  ## 查看pod
		  [root@k8s-master-51 deployment]# kubectl get pod
		  NAME                      READY   STATUS    RESTARTS   AGE
		  cat-dp-548fddf68d-qsnmp   1/1     Running   0          2m3s
		  [root@k8s-master-51 deployment]#
		  ~~~
	- 查看pod的日志
	  collapsed:: true
		- pod中仅有一个container时,可以不指定container
		- ~~~shell
		  ## 待执行的命令
		  kubectl logs $podName
		  
		  ## 实际执行过程
		  [root@k8s-master-51 deployment]# kubectl logs cat-dp-548fddf68d-qsnmp
		  
		  .   ____          _            __ _ _
		   /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
		  ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
		   \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
		  '  |____| .__|_| |_|_| |_\__, | / / / /
		   =========|_|==============|___/=/_/_/_/
		   :: Spring Boot ::                (v2.6.3)
		  
		  2022-04-06 18:42:43.806  INFO [cat,,] 1 --- [           main] com.example.istio.IstioApplication       : Starting IstioApplication v0.0.1-SNAPSHOT using Java 1.8.0_271 on cat-dp-548fddf68d-qsnmp with PID 1 (/root/app/example-app.jar started by root in /app)
		  2022-04-06 18:42:43.808  INFO [cat,,] 1 --- [           main] com.example.istio.IstioApplication       : The following profiles are active: local
		  2022-04-06 18:42:44.666  INFO [cat,,] 1 --- [           main] o.s.cloud.context.scope.GenericScope     : BeanFactory id=d9b67be5-41df-3276-b98a-5b7495db7968
		  2022-04-06 18:42:45.453  INFO [cat,,] 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 9001 (http)
		  2022-04-06 18:42:45.463  INFO [cat,,] 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
		  2022-04-06 18:42:45.463  INFO [cat,,] 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.56]
		  2022-04-06 18:42:45.514  INFO [cat,,] 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
		  2022-04-06 18:42:45.514  INFO [cat,,] 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1662 ms
		  2022-04-06 18:42:46.803  INFO [cat,,] 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 9001 (http) with context path ''
		  2022-04-06 18:42:46.817  INFO [cat,,] 1 --- [           main] com.example.istio.IstioApplication       : Started IstioApplication in 3.55 seconds (JVM running for 4.047)
		  [root@k8s-master-51 deployment]#
		  ~~~
- 通过NodePort暴露工作负载
  collapsed:: true
	- 快速参考的Kubernetes官方文档
	  collapsed:: true
		- https://kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/
		- ![image-20220406144112429](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zzw7jfddj21n60u0gr2.jpg)
		- Kubernetes Service 详细官方文档:
		- https://kubernetes.io/docs/concepts/services-networking/
		- https://kubernetes.io/docs/concepts/services-networking/service/
		- ![image-20220406151013898](https://tva1.sinaimg.cn/large/e6c9d24ely1h100qcfgg2j21my0u0gtr.jpg)
		- ![image-20220406151212462](https://tva1.sinaimg.cn/large/e6c9d24ely1h100sjhb7oj21nf0u0ah5.jpg)
		- ![image-20220406152849439](https://tva1.sinaimg.cn/large/e6c9d24ely1h1019q15cyj21n90u0ajh.jpg)
		- ![image-20220406153039674](https://tva1.sinaimg.cn/large/e6c9d24ely1h101bmehwej21nk0u045t.jpg)
	- 部署Service
		- ~~~shell
		  ## 待执行的命令
		  kubectl apply -f cat-svc-node-port.yaml
		  
		  ## 实际执行过程
		  [root@k8s-master-51 service]# kubectl apply -f cat-svc-node-port.yaml
		  service/cat-svc created
		  [root@k8s-master-51 service]#
		  ~~~
	- 查看Serivce
		- ~~~shell
		  ## 待执行的命令
		  kubectl get service
		  
		  ## 实际执行过程
		  [root@k8s-master-51 service]# kubectl get service
		  NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
		  cat-svc      NodePort    10.10.192.135   <none>        30901:31901/TCP   81s
		  kubernetes   ClusterIP   10.10.0.1       <none>        443/TCP           47h
		  [root@k8s-master-51 service]#
		  ~~~
	- 通过NodePort访问sayHello接口
		- http://192.168.162.61:31901/sayHello
		- ![image-20220406185518043](https://tva1.sinaimg.cn/large/e6c9d24ely1h1078jkbt6j21ic0hi40a.jpg)
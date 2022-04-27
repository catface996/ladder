- istio 安装部署(参考官方文档)
	- 官网地址: https://istio.io/
	- ![20220410233725](https://picgo.catface996.com/picgo20220410233725.png)
	- Download and install istio
		- 选择哪个版本?  amd? arm? 1.12.4 ? 最新版本?
		- 阅读下载脚本: https://istio.io/downloadIstio  -->  https://raw.githubusercontent.com/istio/istio/master/release/downloadIstioCandidate.sh
		- 目前阿里云支持的版本:
		- ![20220410233649](https://picgo.catface996.com/picgo20220410233649.png)
		- 如何下载?
		- 从github下载,需要科学上网.
		- https://github.com/istio/istio/releases/
		- ![20220411101032](https://picgo.catface996.com/picgo20220411101032.png)
		- ![20220411101308](https://picgo.catface996.com/picgo20220411101308.png)
		- 从gitee克隆.
		- ![20220411101456](https://picgo.catface996.com/picgo20220411101456.png)
		- gitee: https://gitee.com/catface996/istio-download
		- ~~~shell
		   git clone https://gitee.com/catface996/istio-download.git
		   ~~~
		- 解压
		- ~~~shell
		   tar -xvzf istio-1.12.4-linux-amd64.tar.gz
		   mv istio-1.12.4 /usr/local/istio-1.12.4
		   cd /usr/local/
		   ln -s istio-1.12.4/ istio
		   ~~~
		- 加入环境变量
		- ~~~shell
		   cd /usr/local/istio
		  ~~~
# 临时加入环境变量
export PATH=$PWD/bin:$PATH
## 永久加入环境变量,编辑/etc/profile,在最后添加export,保存后,source /etc/profile
vim /etc/profile
export PATH=/usr/local/istio/bin:$PATH
source /etc/profile
~~~

* 执行安装

* 提前准备所需镜像

~~~shell
# istio相关镜像(master节点上拉取)
istio/proxyv2                                                     1.12.4              69574f8a643d        7 weeks ago         260MB
istio/pilot                                                       1.12.4              a11fe32e6ad7        7 weeks ago         192MB

docker pull istio/proxyv2:1.12.4
docker pull istio/pilot:1.12.4
# bookinfo sample镜像(worker节点上拉取)
istio/examples-bookinfo-reviews-v3                                1.16.2              83e6a8464b84        21 months ago       694MB
istio/examples-bookinfo-reviews-v2                                1.16.2              39cff5d782e1        21 months ago       694MB
istio/examples-bookinfo-reviews-v1                                1.16.2              181be23dc1af        21 months ago       694MB
istio/examples-bookinfo-ratings-v1                                1.16.2              99ce598b98cf        21 months ago       161MB
istio/examples-bookinfo-details-v1                                1.16.2              edf6b9bea3db        21 months ago       149MB
istio/examples-bookinfo-productpage-v1                            1.16.2              7f1e097aad6d        21 months ago       207MB

docker pull istio/examples-bookinfo-reviews-v3:1.16.2
docker pull istio/examples-bookinfo-reviews-v2:1.16.2
docker pull istio/examples-bookinfo-reviews-v1:1.16.2
docker pull istio/examples-bookinfo-ratings-v1:1.16.2
docker pull istio/examples-bookinfo-details-v1:1.16.2
docker pull istio/examples-bookinfo-productpage-v1:1.16.2
# dashboard相关镜像(worker节点上拉取)
quay.io/kiali/kiali                                               v1.42               f1dca328da23        5 months ago        203MB
grafana/grafana                                                   8.1.2               b9cdc06f46e6        7 months ago        213MB
jaegertracing/all-in-one                                          1.23                2c4eb3e70f5b        10 months ago       52.9MB
prom/prometheus                                                   v2.26.0             6d6859d1a42a        12 months ago       169MB
jimmidyson/configmap-reload                                       v0.5.0              d771cc9785a1        14 months ago       9.99MB

docker pull quay.io/kiali/kiali:v1.42
docker pull grafana/grafana:8.1.2
docker pull jaegertracing/all-in-one:1.23
docker pull prom/prometheus:v2.26.0
docker pull jimmidyson/configmap-reload:v0.5.0
~~~

* 查看安装配置-profile

地址: https://istio.io/latest/docs/setup/additional-setup/config-profiles/

![20220411104034](https://picgo.catface996.com/picgo20220411104034.png)

![20220411104118](https://picgo.catface996.com/picgo20220411104118.png)

* 运行安装命令

~~~shell
istioctl install --set profile=demo -y
# 执行结果
✔ Istio core installed
✔ Istiod installed
✔ Egress gateways installed
✔ Ingress gateways installed
✔ Installation complete
~~~

* 对default命名空间开启istio注入

~~~shell
kubectl label namespace default istio-injection=enabled
namespace/default labeled
# 验证注入是否生效,通过部署一个pod来验证,我们来部署cat-dp.yaml
# 可以通过比对的方式,创建一个没有开启istio注入的命名空间,同样部署一个cat-dp

kubectl apply -f dog-dp.yaml  

kubectl describe pod dog-dpxxx

kubectl replace --force -f cat-dp.yaml
# 或者
kubectl scale --replicas=2 deployment/cat-dp

kubectl describe pod $podName
## 查看pod中的container


~~~

2. Deploy the sample application

~~~shell
[root@k8s-master-22 istio]# pwd
/usr/local/istio
# 部署bookinfo的样例
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
# 实际执行结果
[root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
# 查看deployment 和 pod
[root@k8s-master-22 istio]# kubectl get deploy
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
cat-dp           2/2     2            2           39m
details-v1       1/1     1            1           3m9s
dog-dp           1/1     1            1           34m
productpage-v1   1/1     1            1           3m8s
ratings-v1       1/1     1            1           3m9s
reviews-v1       1/1     1            1           3m9s
reviews-v2       1/1     1            1           3m9s
reviews-v3       1/1     1            1           3m8s
[root@k8s-master-22 istio]#

[root@k8s-master-22 istio]# kubectl get pod
NAME                              READY   STATUS    RESTARTS   AGE
cat-dp-57585f9c77-bpdsn           2/2     Running   0          42m
cat-dp-57585f9c77-gwgkz           2/2     Running   0          27m
details-v1-79f774bdb9-4wgfw       2/2     Running   0          6m13s
dog-dp-b6c8757d4-46nlh            2/2     Running   0          37m
productpage-v1-6b746f74dc-sp4rk   2/2     Running   0          6m12s
ratings-v1-b6994bb9-5xz2f         2/2     Running   0          6m13s
reviews-v1-545db77b95-jn5cr       2/2     Running   0          6m12s
reviews-v2-7bf8c9648f-nkp4q       2/2     Running   0          6m12s
reviews-v3-84779c7bbc-glqbp       2/2     Running   0          6m12s
[root@k8s-master-22 istio]#
# 验证pod是否能够提供http服务
kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
# 翻译一下: 进入 ratings 容器,执行 curl http://productpage:9080/productpage  看返回结果中是否有 <title>.*</title>
[root@k8s-master-22 istio]# kubectl exec -ti ratings-v1-b6994bb9-5xz2f curl http://productpage:9080/productpage
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
<!DOCTYPE html>
<html>
<head>
<title>Simple Bookstore App</title>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">     

<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="static/bootstrap/css/bootstrap.min.css">
...
~~~


3. Open the application to outside traffic

* 将pod和istio-gateway关联
~~~shell
# 待执行的命令
cd /usr/local/istio/
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
# 实际的执行结果
[root@k8s-master-22 istio]# kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
~~~

*  确保配置没有问题

~~~shell
[root@k8s-master-22 istio]# istioctl analyze

✔ No validation issues found when analyzing namespace: default.
[root@k8s-master-22 istio]#
- # 查看VirtualService
- ~~~shell
  [root@k8s-master-22 istio]# kubectl get vs
  NAME       GATEWAYS               HOSTS   AGE
  bookinfo   ["bookinfo-gateway"]   ["*"]   9m48s
  [root@k8s-master-22 istio]# kubectl describe vs bookinfo
  Name:         bookinfo
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  networking.istio.io/v1beta1
  Kind:         VirtualService
  Metadata:
  Creation Timestamp:  2022-04-11T08:37:51Z
  Generation:          1
  Managed Fields:
  API Version:  networking.istio.io/v1alpha3
  Fields Type:  FieldsV1
  fieldsV1:
    f:metadata:
      f:annotations:
        .:
        f:kubectl.kubernetes.io/last-applied-configuration:
    f:spec:
      .:
      f:gateways:
      f:hosts:
      f:http:
  Manager:         kubectl-client-side-apply
  Operation:       Update
  Time:            2022-04-11T08:37:51Z
  Resource Version:  14625
  UID:               620bc446-aed0-48c2-a7cb-9026144b5e0d
  Spec:
  Gateways:
  bookinfo-gateway
  Hosts:
  *
  Http:
  Match:
    Uri:
      Exact:  /productpage
    Uri:
      Prefix:  /static
    Uri:
      Exact:  /login
    Uri:
      Exact:  /logout
    Uri:
      Prefix:  /api/v1/products
  Route:
    Destination:
      Host:  productpage
      Port:
        Number:  9080
  Events:            <none>
  ~~~
- 查看istio的gateway
- ~~~shell
  [root@k8s-master-22 istio]# kubectl get gateway
  NAME               AGE
  bookinfo-gateway   10m
  [root@k8s-master-22 istio]# kubectl describe gateway bookinfo-gateway
  Name:         bookinfo-gateway
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>
  API Version:  networking.istio.io/v1beta1
  Kind:         Gateway
  Metadata:
  Creation Timestamp:  2022-04-11T08:37:51Z
  Generation:          1
  Managed Fields:
  API Version:  networking.istio.io/v1alpha3
  Fields Type:  FieldsV1
  fieldsV1:
    f:metadata:
      f:annotations:
        .:
        f:kubectl.kubernetes.io/last-applied-configuration:
    f:spec:
      .:
      f:selector:
        .:
        f:istio:
      f:servers:
  Manager:         kubectl-client-side-apply
  Operation:       Update
  Time:            2022-04-11T08:37:51Z
  Resource Version:  14624
  UID:               df23873c-18d9-4e9b-94da-2724471e74ac
  Spec:
  Selector:
  Istio:  ingressgateway
  Servers:
  Hosts:
    *
  Port:
    Name:      http
    Number:    80
    Protocol:  HTTP
  Events:          <none>
  ~~~
- 访问bookinfo服务
- 通过Ingress访问
- 没有安装,需要在k8s集群中安装Ingress方能支持.
- 通过NodePort访问
- ~~~shell
  [root@k8s-master-22 istio]# kubectl get svc istio-ingressgateway -n istio-system
  NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                      AGE
  istio-ingressgateway   LoadBalancer   10.10.210.169   <pending>     15021:30523/TCP,80:32177/TCP,443:32262/TCP,31400:30612/TCP,15443:31112/TCP   128m
  ~~~~
# 可以看到 80:32177/TCP 中的 32177 即为NodePort,可以通过describe命令进一步确认
[root@k8s-master-22 istio]# kubectl describe service  -n istio-system istio-ingressgateway
Name:                     istio-ingressgateway
Namespace:                istio-system
Labels:                   app=istio-ingressgateway
                        install.operator.istio.io/owning-resource=unknown
                        install.operator.istio.io/owning-resource-namespace=istio-system
                        istio=ingressgateway
                        istio.io/rev=default
                        operator.istio.io/component=IngressGateways
                        operator.istio.io/managed=Reconcile
                        operator.istio.io/version=1.12.4
                        release=istio
Annotations:              <none>
Selector:                 app=istio-ingressgateway,istio=ingressgateway
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.10.210.169
IPs:                      10.10.210.169
Port:                     status-port  15021/TCP
TargetPort:               15021/TCP
NodePort:                 status-port  30523/TCP
Endpoints:                10.122.158.3:15021
Port:                     http2  80/TCP
TargetPort:               8080/TCP
NodePort:                 http2  32177/TCP
Endpoints:                10.122.158.3:8080
Port:                     https  443/TCP
TargetPort:               8443/TCP
NodePort:                 https  32262/TCP
Endpoints:                10.122.158.3:8443
Port:                     tcp  31400/TCP
TargetPort:               31400/TCP
NodePort:                 tcp  30612/TCP
Endpoints:                10.122.158.3:31400
Port:                     tls  15443/TCP
TargetPort:               15443/TCP
NodePort:                 tls  31112/TCP
Endpoints:                10.122.158.3:15443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
[root@k8s-master-22 istio]#
~~~

浏览器访问

通过master节点的nodePort访问
![20220411173430](https://picgo.catface996.com/picgo20220411173430.png)

通过worker节点的nodePort访问
![20220411173529](https://picgo.catface996.com/picgo20220411173529.png)


4. View the dashboard

* 安装Kiali 和其他插件并等待它们被部署

~~~shell
# 待执行的部署命令
kubectl apply -f samples/addons
# 实际执行结果
[root@k8s-master-22 istio]# kubectl apply -f samples/addons
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
[root@k8s-master-22 istio]#
# 查看部署后的pod
[root@k8s-master-22 istio]# kubectl get pod -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
grafana-6ccd56f4b6-6tj65                1/1     Running   0          32s
istio-egressgateway-56f4569d45-5jnq5    1/1     Running   0          3h7m
istio-ingressgateway-786c6bddb7-ntldl   1/1     Running   0          3h7m
istiod-84f87dcc49-tmqbf                 1/1     Running   0          3h7m
jaeger-5d44bc5c5d-s9whj                 1/1     Running   0          32s
kiali-79b86ff5bc-c72k6                  1/1     Running   0          32s
prometheus-64fd8ccd65-qhv7g             2/2     Running   0          32s
~~~


* 访问 Kiali 仪表板

注意:需要绑定服务暴露的ip地址 --address 192.168.162.22
~~~shell
# 查看 istioctl dashboard 帮助文档
[root@k8s-master-22 istio]# istioctl dashboard --help

Flags:
--address string   Address to listen on. Only accepts IP address or localhost as a value. When localhost is supplied, istioctl will try to bind on both 127.0.0.1 and ::1 and will fail if neither of these address are available to bind. (default "localhost")
# 需要执行的命令 
istioctl dashboard kiali --address 192.168.162.22

[root@k8s-master-22 istio]# istioctl dashboard kiali --address 192.168.162.22
http://192.168.162.22:20001/kiali
Failed to open browser; open http://192.168.162.22:20001/kiali in your browser.
~~~

![20220411180254](https://picgo.catface996.com/picgo20220411180254.png)

通过浏览器多访问几次 http://192.168.162.221:32177/productpage 后

![20220411180423](https://picgo.catface996.com/picgo20220411180423.png)
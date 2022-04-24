# Istio 实施灰度
## 1.主机规划

| 主机用途   | 主机名        | 主机IP         | 主机软件环境        |
| ---------- | ------------- | -------------- | ------------------- |
| rancher    | rancher-10    | 192.168.162.10 | Centos 7.x ; Docker |
| k8s-master | k8s-master-21 | 192.168.162.21 | Centos 7.x ; Docker |
| k8s-master | k8s-master-22 | 192.168.162.22 | Centos 7.x ; Docker |
| k8s-master | k8s-master-23 | 192.168.162.23 | Centos 7.x ; Docker |
| k8s-node   | K8s-node-31   | 192.168.162.31 | Centos 7.x ; Docker |
| k8s-node   | K8s-node-32   | 192.168.162.32 | Centos 7.x ; Docker |
## 2.修改ip和hostname

~~~shell
## 修改主机的ip,网关,dns等
vim /etc/sysconfig/network-scripts/ifcfg-ens33
## 修改主机hostname
vim /etc/hostname
~~~
## 3.安装docker

PS: Docker Server Version: 18.09.9,建议使用阿里云的docker镜像加速,[传送门](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
## 4.安装Rancher

* Rancher中文文档: https://docs.rancher.cn/

![image-20220321151354357](https://tva1.sinaimg.cn/large/e6c9d24ely1h0hixaa9l7j21na0u043n.jpg)

* 选择最新的稳定版本

![image-20220321151456076](https://tva1.sinaimg.cn/large/e6c9d24ely1h0hiyat80pj21nj0u0tcz.jpg)

* Docker官网查看rancher镜像

​		官网地址; https://www.docker.com/

​		DockerHub: https://hub.docker.com/

​				![image-20220321161844809](https://tva1.sinaimg.cn/large/e6c9d24ely1h0hksp3313j21n50u0q74.jpg)

​			拉取rancher镜像命令:

~~~shell
docker pull rancher/rancher:v2.5.12
~~~

* Rancher安装命令:

![image-20220321152255547](https://tva1.sinaimg.cn/large/e6c9d24ely1h0hj6mykowj21nk0u0jxu.jpg)



![image-20220321152402977](https://tva1.sinaimg.cn/large/e6c9d24ely1h0hj7t9syfj21na0u07a2.jpg)

![image-20220321152454709](https://tva1.sinaimg.cn/large/e6c9d24ely1h0hj8pa1r5j21n80u0jx5.jpg)





~~~shell
docker run -d --privileged --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  rancher/rancher:v2.5.12
~~~
## 5.搭建k8s集群

前置准备工作,可以提前把镜像下载到服务器,避免搭建过程中因为镜像拉取出现异常.

~~~shell
## k8s搭建相关
docker pull rancher/hyperkube:v1.20.15-rancher1
docker pull rancher/rancher-agent:v2.5.12
docker pull rancher/nginx-ingress-controller:nginx-1.1.0-rancher1
docker pull rancher/mirrored-coreos-flannel:v0.15.1
docker pull rancher/mirrored-ingress-nginx-kube-webhook-certgen:v1.1.1
docker pull rancher/rke-tools:v0.1.78
docker pull rancher/mirrored-metrics-server:v0.5.0
docker pull rancher/fleet-agent:v0.3.5
docker pull rancher/mirrored-coreos-etcd:v3.4.15-rancher1
docker pull rancher/mirrored-calico-node:v3.17.2
docker pull rancher/mirrored-calico-pod2daemon-flexvol:v3.17.2
docker pull rancher/mirrored-calico-cni:v3.17.2
docker pull rancher/mirrored-calico-kube-controllers:v3.17.2
docker pull rancher/mirrored-coredns-coredns:1.8.0
docker pull rancher/mirrored-cluster-proportional-autoscaler:1.8.1
docker pull rancher/kube-api-auth:v0.1.4
docker pull rancher/mirrored-pause:3.2

## istio相关
docker pull rancher/istio-1.5-migration:0.1.1
docker pull rancher/istio-citadel:1.5.9
docker pull rancher/istio-coredns-plugin:0.2-istio-1.1
docker pull rancher/istio-galley:1.5.9
docker pull rancher/istio-kubectl:1.4.6
docker pull rancher/istio-kubectl:1.5.10
docker pull rancher/istio-kubectl:1.5.9
docker pull rancher/istio-mixer:1.5.9
docker pull rancher/istio-node-agent-k8s:1.5.9
docker pull rancher/istio-pilot:1.5.9
docker pull rancher/istio-proxyv2:1.5.9
docker pull rancher/istio-sidecar_injector:1.5.9
docker pull rancher/jaegertracing-all-in-one:1.14
docker pull rancher/jetstack-cert-manager-controller:v0.8.1
docker pull rancher/jimmidyson-configmap-reload:v0.3.0
docker pull rancher/kiali-kiali:v1.17
~~~

* master节点上运行 etcd和control

![image-20220322120612558](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ij4bklhdj21nd0u0q7m.jpg)

~~~shell
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run  rancher/rancher-agent:v2.5.12 --server https://192.168.162.10 --token v8jsjdsk7rrmsgfkr9zmdmzwvjnlnlgsdql6qdw8brfkmml6m9lnhb --ca-checksum fc2f07be48c63e818d78a8a3d7ddbd9cb16590a441cef95b94e290e3355dfa52 --etcd --controlplane
~~~

注意:需要复制自己搭建的集群的命令到主机上执行,本文提供的只是样例.



* node节点上运行worker

~~~shell
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run  rancher/rancher-agent:v2.5.12 --server https://192.168.162.10 --token v8jsjdsk7rrmsgfkr9zmdmzwvjnlnlgsdql6qdw8brfkmml6m9lnhb --ca-checksum fc2f07be48c63e818d78a8a3d7ddbd9cb16590a441cef95b94e290e3355dfa52 --worker
~~~

![image-20220322120734470](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ij5nv9l6j21n40u0q79.jpg)



常用命令:

~~~shell
# 停止所有容器
docker stop $(docker ps -a -q)

# 删除所有未运行的容器
docker rm $(docker ps -a -q)
~~~

* 集群部署成功

![image-20220322123242210](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijvtdenyj21ng0u0tdb.jpg)
## 6.部署工作负载验证集群



* 需要部署的镜像: 

~~~properties
 catface996/spring-cloud-istio-demo:latest
~~~

该镜像打包了一个spring cloud应用,提供一个http接口,/sayHello.



实现sayHello接口的代码:

![image-20220322122103978](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijjp3vgxj21hd0u0wkx.jpg)



推送到DockerHub中的镜像:

![image-20220322121929500](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijl6ner1j21na0u0aem.jpg)

![image-20220322122321821](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijm462qfj21nj0u0n0m.jpg)

* 部署镜像时,指定的参数:

~~~properties
--spring.application.name=cat
--server.port=9001
--next=false
--env=prod
--service.url=http://localhost:9002
--spring.sleuth.baggage.remote-fields=env
~~~

* 验证是否部署成功

* 查看工作负载的状态

  工作负载状态变为Active:

  ![image-20220322122713547](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijq9naiwj21nd0u0gof.jpg)

  

  查看工作负载中container的日志:

  ![image-20220322122834676](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijrixkjrj21na0u0tdf.jpg)

  ![image-20220322123036505](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijtmuzdqj21ns0u0dmp.jpg)

* 访问sayHello接口

  ![image-20220322123132295](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijum0n68j21nf0u0wgw.jpg)

  ![image-20220322122626016](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ijp9u8p0j21bs0ii400.jpg)
## 7.安装并启用Istio
### k8s集群中安装Istio

在集群管理页面,进入"资源->Istio"页面

![image-20220322150909011](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ioelu3hkj21mq0u00v8.jpg)

![image-20220322151039115](https://tva1.sinaimg.cn/large/e6c9d24ely1h0iog5ywl1j21n50u0wg4.jpg)

可以看到,istio是尚未激活的,在此处,点击"这里",进入安装istio页面,rancher提供的istio版本是1.5.9

![image-20220322151204736](https://tva1.sinaimg.cn/large/e6c9d24egy1h0kxbxibibj21n60u0dj5.jpg)

如果要启用istio-ingressgateway话,需要勾选启用ingress.

![image-20220323155335048](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jvb5qognj21n30u0whl.jpg)

剩余配置可以保持默认,本期视频暂不介绍.最后点击"启用",等待istio安装完成.

![image-20220322151542285](https://tva1.sinaimg.cn/large/e6c9d24ely1h0iolf3b63j21ne0u0mzt.jpg)

![image-20220322151703407](https://tva1.sinaimg.cn/large/e6c9d24ely1h0iomtvzgwj21n00u0jui.jpg)

安装完成后,可以进入istio-gray集群的System空间,查看istio的安装情况

![image-20220322152718506](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ioxhshsoj21nc0u043e.jpg)



![image-20220322152833281](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ioysyiwej21na0u0dll.jpg)

![image-20220322152911258](https://tva1.sinaimg.cn/large/e6c9d24ely1h0iozf1kuoj21ne0u0q8a.jpg)

可以看到,istio-system命名空间下的所有工作负载均正常激活.
### 对命名空间启用istio

进入istio-gray集群的Default空间,选择"命名空间"菜单,进入命名空间列表,勾选"default",开启istio注入,操作如下图.

![image-20220322153232021](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ip2xfmdtj21na0u00vi.jpg)

启用注入成功后,命名空间后面会出现一个istio的logo.

![image-20220322153618102](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ip6uuvovj21mz0u0tav.jpg)
### 部署工作负载

如果在启用istio之前,命名空间中已经部署了工作负载,对已有的pod不会做注入,需要更新一下工作负载,才会使istio注入对工作负载生效.如下图,已经部署的pod中,只有cat-dp容器.

![image-20220322153954848](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ipalnmuvj21ng0u078c.jpg)

对工作负载进行升级,无需做任何配置改动

![image-20220322154124144](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ipc6myh5j21ne0u0777.jpg)

升级结束后,再观察工作负载,可以看到,pod中多了两个容器,分别是istio-init和istio-proxy,并且istio-init是终结状态.到此,k8s安装并启用istio注入就完成了.

![image-20220322154334993](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ipeewz39j21mb0u0ae7.jpg)
## 8.网格入口流量灰度
### 8.1 创建工作负载

我们需要创建同一个应用的两个不同版本的工作负载,分别是 cat-dp-gray 和 cat-dp-prod.如下图:

![image-20220323162501966](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jw7vq3tcj22eb0u0431.jpg)

cat-pd-gray:

![image-20220323162608424](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jw92dwxrj21n70u00vr.jpg)

![image-20220323162649004](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jw9pvzz5j21n70u0gpe.jpg)

cat-dp-prod:

![image-20220323163147534](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jwg1vrmxj21mx0u077a.jpg)

![image-20220323163340288](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jwgvxafaj21ne0u0n16.jpg)
### 8.2 配置k8s Service

创建工作负载时,rancher会默认创建与工作负载同名的Service,我们需要创建一个自定义的Service.

![image-20220323163528059](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jwirhnhdj21n00u0gog.jpg)

![image-20220323163656585](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jwk9cvbaj21na0u0q5y.jpg)
### 8.3 通过K8s Ingress 对外暴露服务(用于比对istio的流量转发方案)

通过创建负载均衡,对外暴露Service

![image-20220323163753490](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jwl83h4mj21ne0u0tb7.jpg)
### 8.4 验证对外暴露的K8s Service

本地配置hosts做域名映射![image-20220323163942890](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jwnchlvzj218g0qodho.jpg)

PS: 修改hosts的软件 SwitchHosts.

通过域名访问sayHello接口,负载均衡的默认策略是轮询,所以会看到返回结果中,prod和gray交替出现.

![image-20220323164206013](https://tva1.sinaimg.cn/large/e6c9d24egy1h0kxedsqu2j22200u0n04.jpg)

![image-20220323164236495](https://tva1.sinaimg.cn/large/e6c9d24egy1h0kxehe9blj21ti0u040s.jpg)
### 8.5 配置Istio网关(Istio ingressgateway)

istio-gateway用于对网格入口流量的分发,k8s集群的入口一般是k8s Ingress,可以将k8s集群的入口流量直接转发到istio-gateway,完成K8s集群入口流量和istio网格入口流量的对接.

![image-20220323165903984](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jx7arzy7j21nh0u0ack.jpg)

![image-20220323170021039](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jx8mx6awj21ni0u0418.jpg)
### 8.6 配置 Istio 目标规则(Destination Rule)

目标规则用于管理同应用的不同版本,提供给Istio Virtual做流量配置.注意: 在添加Subset的时候,点击添加Subset按钮可能没有发反应,可以通过编辑yaml文件来添加.

![image-20220323170327524](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jxc27q6ij21no0u0aci.jpg)

![image-20220323170622835](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jxezzwvyj21na0u0zme.jpg)

![image-20220323170822625](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jxgynw2qj21ne0u0jtx.jpg)

![image-20220323170902082](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jxho4ux7j21mu0u0q5z.jpg)
### 8.7 配置Istio 虚拟服务(VirtualService)

虚拟服务是用来声明网格中流量规则的组件.在配置虚拟服务时,hosts需要通过编辑yaml文件进行修改,istio网关的域名支持通配符,VirutalService配置的Hosts需符合istio网关的通配规则,为了方便起见,我们使其保持一致.

![image-20220323170952547](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jxio0fkhj21mz0u040s.jpg)

![image-20220323180157684](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jz0pxsduj21qk0u0dis.jpg)

![image-20220323180232072](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jz1bfv4uj21ni0u0q6d.jpg)
### 8.8 通过K8s Ingress 暴露Istio-ingressgateway

配置负载均衡,对外暴露istio-gateway,因为istio-gateway是在System->sitio-system命名空间下,所以负载均衡也要在System中配置.

![image-20220323180535496](https://tva1.sinaimg.cn/large/e6c9d24egy1h0kxckz8bzj21nh0u0wha.jpg)

![image-20220323180609662](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jz532ngrj21ms0u0adu.jpg)
### 验证灰度参数及流量转发

以上配置均完成后,配置本地hosts:

![image-20220323180755748](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jz73baezj218g0qowgh.jpg)



使用postman访问sayHello接口,并分别设置headers env=prod, env=gray和env=daily进行请求

* Header env=prod

![image-20220323180917364](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jz8d5w51j21w50u041d.jpg)

* header env=gray

![image-20220323181148593](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jzazh8ndj21yy0u0ju4.jpg)

* header env=daily

当流量路由规则均未匹配时,会使用默认路由规则,我们配置的默认路由规则是prod.

![image-20220323181226146](https://tva1.sinaimg.cn/large/e6c9d24ely1h0jzbmbo80j21x00u0acw.jpg)
## 9.网格内部流量灰度

我们希望已经部署的cat-dp访问新部署的mouse-dp,形成istio网格内部的微服务调用.
### 9.1 部署mouse工作负载

需要部署 mouse-dp-gray和mouse-dp-prod

![image-20220324150232825](https://tva1.sinaimg.cn/large/e6c9d24ely1h0kzgc5be9j21nb0u0djf.jpg)

![image-20220324150322968](https://tva1.sinaimg.cn/large/e6c9d24ely1h0kzhchkrfj21n90u00vq.jpg)

![image-20220324150412853](https://tva1.sinaimg.cn/large/e6c9d24ely1h0kzi20v8pj21n80u0n12.jpg)
### 9.2 配置mouse服务发现

需要配置自定义的服务发现,与cat-svc类似,通过标签选择的方式来配置mouse-svc,注意,通过页面操作可能会报错,提示分配集群ip异常,可以通过导入yaml文件的形式来创建.

![image-20220324150805409](https://tva1.sinaimg.cn/large/e6c9d24ely1h0kzm46bofj21nd0u0781.jpg)

![image-20220324151010450](https://tva1.sinaimg.cn/large/e6c9d24ely1h0kzo97q76j21n50u0wgo.jpg)
### 9.3 在cat-dp容器中执行cul对mouse-svc进行验证

在cat-dp详情页,进入执行命令界面,执行curl,操作步骤如下图:

![image-20220324151409667](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l48gcc4rj21na0u00xg.jpg)

![image-20220324151617607](https://tva1.sinaimg.cn/large/e6c9d24ely1h0kzusxr2aj21nd0u0gmu.jpg)

执行curl命令,可以看到,返回结果是prod和gray交替出现,mouse-svc的默认策略是轮询.

~~~shell
curl http://mouse-svc:9001/sayHello
~~~

![image-20220324152137087](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l007lmq0j21n40u0tek.jpg)

即使我们在请求mouse的sayHello接口时,携带了headers env=gray,流量灰度也不会生效,如下图:

![image-20220324153520112](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l0efshj3j21n60u0qc6.jpg)
### 9.4 通过postman访问cat-svc->mouse-svc(不经过istio-gateway)

requst 1:

![image-20220324152638862](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l05evsvuj21nf0u0mzs.jpg)

request 2:

![image-20220324152720612](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l06541qwj21r70u0777.jpg)

request 3:

![image-20220324152757445](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l06sh2z3j21vo0u0ad0.jpg)
### 9.5 通过postman访问cat-svc->mouse-svc(经过istio-gateway)

![image-20220324153019852](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l098zrwsj21r60u0n09.jpg)

![image-20220324153104068](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l0a0kjt8j21v90u0n0f.jpg)
### 9.6 部署mouse的目标规则mouse-dr

![image-20220324155934963](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l13t1nvnj21n50u0416.jpg)

![image-20220324160015322](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l14h3iuuj21n60u0mz0.jpg)
### 9.7 部署mouse的虚拟服务mouse-vs

![image-20220324160059740](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l158wm5kj21na0u076w.jpg)

![image-20220324160145745](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l15ym9knj21na0u0tbf.jpg)



![image-20220324160213405](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l16gd5x5j21nk0u0jur.jpg)
### 9.8 在cat-dp容器中执行curl对mouse-vs进行验证

分别执行以下三条命令,预期时,不携带header参数,固定返回prod,携带header参数,返回相应的环境

~~~shell
curl http://mouse-svc:9001/sayHello

curl -H "env:gray" http://mouse-svc:9001/sayHello

curl -H "env:prod" http://mouse-svc:9001/sayHello
~~~

不携带header参数:

![image-20220324161007330](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l1etks2zj21nh0u010v.jpg)

携带header参数 env=gray

![image-20220324161443076](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l1jgkuqjj21nc0u0dq4.jpg)

携带header参数 env=prod

![image-20220324162433886](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l1tode1rj21ne0u0wnk.jpg)
### 9.9 通过postman访问cat-svc-->mouse-svc(进过istio-gateway)

不携带header参数:

![image-20220324162830013](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l1xt1s7gj21tx0u0whl.jpg)

携带header参数 env=prod

![image-20220324162644209](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l1w3scurj21qq0u077c.jpg)

携带header参数 env=gray

![image-20220324162918049](https://tva1.sinaimg.cn/large/e6c9d24ely1h0l1ym8yjaj21ug0u0dj0.jpg)
### 9.10 注意

网格中的服务,在发起服务调用时,需要能在调用链路中传播灰度路由参数,本方案采用的是spring-cloud-sleuth中的参数传播机制,通过启用以下配置保证header中的env参数在调用链路中的传播.

~~~properties
spring.sleuth.baggage.remote-fields=env
~~~

选用的SpringCoud版本和SpringBoot版本

```xml
<spring-cloud.version>2021.0.1</spring-cloud.version>
<srping-boot.version>2.6.3</spring-boot.version>
```
## 10.后续视频规划

* ServiceEntry和Istio-egressgateway在网格灰度方案中使用较少,外部服务一般只提供一个线上环境,会在后续的视频中介绍.
* 基于原生搭建的k8s,原生安装istio,实施灰度方案.
* 基于阿里云提供的k8s集群和网格服务,实施灰度方案.
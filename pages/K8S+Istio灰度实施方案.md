- ## 1.快速体验K8S
	- 参考 [[K8S官网快速体验K8S]]
- ## 2.安装docker
	- 参考 [[Docker官方安装文档介绍和实战]]
  
## 3. 通过kubeadm安装k8s集群
参考：[[使用Kubeadm安装简单集群实战]]

### 3.2 安装kubeadm
参考：[[使用Kubeadm安装简单集群实战]]

### 3.3 故障排查
参考：[[使用Kubeadm安装简单集群实战]]

### 3.4 使用kubeadm创建集群
参考：[[使用Kubeadm安装简单集群实战]]

### 3.5 进一步验证部署的k8s集群
参考：[[进一步验证部署的k8s集群]]

### 3.6 Kubeadm部署集群的常见故障模拟
参考；[[安装Kubernetes集群常见故障模拟]]


### 3.7 kubeadm引导的集群导入rancher

针对三个版本的Kubernetes集群做导入验证,版本分别为:
* 1.18.20
~~~shell
# 安装kubeadm , kubectl , kubelet
sudo yum install -y --nogpgcheck kubelet-1.18.20 kubeadm-1.18.20 kubectl-1.18.20 --disableexcludes=kubernetes
yum list installed | grep kube
# 拉取镜像
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers  --kubernetes-version=1.18.20
docker images | grep registry.aliyuncs.com
# 引导集群
kubeadm init --kubernetes-version=1.18.20  \
--apiserver-advertise-address=192.168.162.18  \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16  \
--pod-network-cidr=10.122.0.0/16 

kubectl get node
kubectl get pod -A

~~~

* 1.20.15
~~~shell
# 安装kubeadm , kubectl , kubelet
sudo yum install -y --nogpgcheck kubelet-1.20.15 kubeadm-1.20.15 kubectl-1.20.15 --disableexcludes=kubernetes
yum list installed | grep kube
# 拉取镜像
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers  --kubernetes-version=1.20.15
docker images | grep registry.aliyuncs.com
# 引导集群
kubeadm init --kubernetes-version=1.20.15  \
--apiserver-advertise-address=192.168.162.20  \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16  \
--pod-network-cidr=10.122.0.0/16 

kubectl get node
kubectl get pod -A

~~~


* 1.22.3
~~~shell
# 安装kubeadm , kubectl , kubelet
sudo yum install -y --nogpgcheck kubelet-1.22.3 kubeadm-1.22.3 kubectl-1.22.3 --disableexcludes=kubernetes
yum list installed | grep kube
# 拉取镜像
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers  --kubernetes-version=1.22.3
docker images | grep registry.aliyuncs.com
# 引导集群
kubeadm init --kubernetes-version=1.22.3  \
--apiserver-advertise-address=192.168.162.22  \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16  \
--pod-network-cidr=10.122.0.0/16 

kubectl get node
kubectl get pod -A

kubeadm join 192.168.162.22:6443 --token xerdgt.jnte2f7rbs0bse5j \
	--discovery-token-ca-cert-hash sha256:4bfa979d801b011ca35984bcfbd8484b1d53106161e8e57290bc048879e0f9a0
~~~

rancher-2.5.12支持的Kubernetes版本如下:
![20220410180539](https://picgo.catface996.com/picgo20220410180539.png)
![20220410180558](https://picgo.catface996.com/picgo20220410180558.png)
![20220410180615](https://picgo.catface996.com/picgo20220410180615.png)
![20220410180634](https://picgo.catface996.com/picgo20220410180634.png)


rancher 中创建三个导入集群
* 导入之前
![20220410182525](https://picgo.catface996.com/picgo20220410182525.png)

* 导入之后
![20220410193700](https://picgo.catface996.com/picgo20220410193700.png)

* 解决 kube-controller-manager和kube-scheduler不健康的问题
* 注释掉 /etc/kubernetes/manifests/ 下的 kube-scheduler.yaml 和 kube-controller-manager.yaml中的 - --port=0
* 重启kubelet


### 3.8 部署和访问Kubernetes仪表盘

Kubernetes官方参考文档地址:

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

![image-20220406144249486](https://tva1.sinaimg.cn/large/e6c9d24ely1h0zzxtgnbhj21my0u0jz0.jpg)
#### 部署dashboard



* 可以先下载yaml文件到本地,然后再部署

~~~shell
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
~~~

* 需要修改 dashboard deployment的nodeSelector
* 对master节点打标签
~~~shell
kubectl label nodes k8s-master-101 node-role=master
~~~

* 修改dashbaord deploymnet的nodeSelector

~~~yaml
nodeSelector:
##"kubernetes.io/os": linux
"node-role": master
~~~

* 部署dashboard

~~~shell
kubectl apply -f recommended.yaml
~~~
#### 暴露dashboard

![image-20220406175600091](https://tva1.sinaimg.cn/large/e6c9d24ely1h105itn4dqj21e10u0tfb.jpg)

~~~shell
## 命令一
kubectl proxy
## 命令二
kubectl proxy --address='192.168.162.51'
## 命令三
kubectl proxy --address='192.168.162.51' --accept-hosts='^*$'
~~~
#### 生成Bearer Token

* 部署ServiceAccount

~~~yaml
apiVersion: v1
kind: ServiceAccount
metadata:
name: admin-user
namespace: kubernetes-dashboard
~~~

* 部署ClusterRoleBinding

~~~yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
name: admin-user
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: ClusterRole
name: cluster-admin
subjects:
	- kind: ServiceAccount
 name: admin-user
 namespace: kubernetes-dashboard
 ~~~

 * 获取BearerToken

 ~~~shell
 kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
 ~~~



 踩坑记录

 * 启动代理

 需要注意配置的参数 --address  --accept-hosts='^*$'

 * 访问dashboard提示 io/timeout

 需要修改 dashboard 的Deployment中的nodeSelector

 原有的标签选择器

 ~~~yaml
 nodeSelector:
 ##"kubernetes.io/os": linux
 "node-role": master
 ~~~


## 挖坑记录

containerd是什么鬼?

github:  https://github.com/containerd/containerd

官网: https://containerd.io/



CRI-O 这又是个什么鬼?

github:  https://github.com/cri-o/cri-o


netfilter 和 iptables

netfilter官网: https://www.netfilter.org/

什么是netfilter?
![20220408103936](https://picgo.catface996.com/picgo20220408103936.png)

什么是iptables?
![20220408104106](https://picgo.catface996.com/picgo20220408104106.png)

参考文档:

https://www.zsythink.net/archives/1199


继续挖坑,等待后续视频填坑...
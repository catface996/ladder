## SSH 慢解决方案

#### 现象

在本地虚拟机安装了CentOS，内网传输数据和ssh登陆都非常的慢

#### 解决办法

经过排查发现是服务器内ssh的配置文件设置导致。
vim /etc/ssh/sshd_config
配置文件中的UseDNS设置为yes，修改为no。
然后重启sshd服务即可。

------

> 解释下UseDNS,当客户端试图登录SSH服务器时，服务器端先根据客户端的IP地址进行DNS PTR反向查询出客户端的主机名，然后根据查询出的客户端主机名进行DNS正向A记录查询，验证与其原始IP地址是否一致，这是防止客户端欺骗的一种措施，但大部分IP都没有配置PTR记录，如果没有特别的情况，一般建议关闭。



## 安装Docker

使用之前请确保已经安装wget，如未安装请执行下面一条命令来安装

~~~bash
yum install -y wget 
~~~

备份当前yum源（可选）

~~~bash
cd /etc/yum.repos.d/ cp /CentOS-Base.repo /CentOS-Base-repo.bak 
~~~

下载阿里云镜像源

~~~bash
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
~~~

清理默认缓存包

~~~bash
yum clean all 
~~~

生成阿里云yum源缓存并更新yum源

~~~shell
yum makecache 
yum update
~~~

安装常用工具包

~~~bash
yum install vim bash-completion net-tools gcc -y
~~~

关闭swapp分区

~~~bash
[root@master01 ~]# swapoff -a

## 注释掉最后一行
[root@master01 ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Mar 31 22:44:34 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=5fecb240-379b-4331-ba04-f41338e81a6e /boot                   ext4    defaults        1 2
/dev/mapper/cl-home     /home                   xfs     defaults        0 0
#/dev/mapper/cl-swap     swap                    swap    defaults        0 0


[root@cdh-1 sbin]# sysctl -w vm.swappiness=0 
vm.swappiness = 0
[root@cdh-1 sbin]# echo "vm.swappiness = 0">> /etc/sysctl.conf  
[root@cdh-1 sbin]# swapoff -a 
## 永久关闭
echo vm.swappiness=0 >> /etc/sysctl.conf
reboot
## 查看是否关闭
free -m
~~~

关闭防火墙

~~~bash
systemctl stop firewalld && systemctl disable firewalld

## 查看防火墙状态
## firewall-cmd --state
[root@catface ~]# firewall-cmd --state
not running
~~~

关闭SELinux

~~~bash
## 关闭 SELinux 然后重启,再通过 sestatus 查看 SElinux的状态.
[root@catface ~]# sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config && setenforce 0
[root@catface ~]# reboot
[root@catface ~]# sestatus
SELinux status:                 disabled
~~~

配置同步时间

~~~bash
#安装chrony：
yum install -y chrony
#注释默认ntp服务器
sed -i 's/^server/#&/' /etc/chrony.conf
#指定上游公共 ntp 服务器，并允许其他节点同步时间
cat >> /etc/chrony.conf << EOF
server 0.asia.pool.ntp.org iburst
server 1.asia.pool.ntp.org iburst
server 2.asia.pool.ntp.org iburst
server 3.asia.pool.ntp.org iburst
allow all
EOF
#重启chronyd服务并设为开机启动：
systemctl enable chronyd && systemctl restart chronyd
#开启网络时间同步功能
timedatectl set-ntp true
~~~

配置内核参数，将桥接的IPv4流量传递到iptables的链

~~~bash
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
~~~



配置Docker-ce的yum源,安装docker,指定版本: docker-ce-18.09.9-3.el7

~~~bash
# step 1: 安装必要的一些系统工具 
[root@catface ~]# yum install -y yum-utils device-mapper-persistent-data lvm2

# Step 2: 添加软件源信息 
[root@catface ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# Step 3: 更新并安装Docker-CE 
[root@catface ~]# yum makecache fast

# Step 4: 查找Docker-CE的版本: 
[root@catface ~]# yum list docker-ce.x86_64 --showduplicates | sort -r

## Step 4: 安装Docker-CE的版本（18.09.9-3.el7）
[root@catface ~]# yum install docker-ce-18.09.9-3.el7

## 启动docker 
[root@catface ~]# service docker start 

## 检查安装的docker版本
[root@catface ~]# docker version
Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.39
 Go version:        go1.13.15
 Git commit:        f0df350
 Built:             Wed Jun  2 11:58:10 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          18.09.9
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.11.13
  Git commit:       039a7df
  Built:            Wed Sep  4 16:22:32 2019
  OS/Arch:          linux/amd64
  Experimental:     false
[root@catface ~]#
~~~

Docker命令参考:

[docker官网下载](https://www.docker.com/)

> docker启动    
>
> - systemctl start docker
>
> 重启docker服务
>
> - systemctl restart  docker
>
> 关闭docker   
>
> - systemctl stop docker
>
> 查看是否启动成功
>
> - docker ps -a

查看安装完成的docker信息

~~~bash
docker info
~~~

配置为开机启动

~~~bash
systemctl enable docker
~~~

复制虚拟机

~~~properties
#BOOTPROTO="dbcp"
BOOTPROTO="static"         # 使用静态IP地址，默认为dhcp
IPADDR="192.168.168.11"   # 设置的静态IP地址
NETMASK="255.255.255.0"    # 子网掩码
GATEWAY="192.168.168.1"    # 网关地址
DNS1="114.114.114.114"       # DNS服务器
~~~

修改hostname

~~~bash
hostnamectl set-hostname hanma-002
~~~

修改hosts

~~~properties
192.168.168.11 hanma-001
192.168.168.12 hanma-002
192.168.168.13 hanma-003
192.168.168.14 hanma-004
192.168.168.15 hanma-005
192.168.168.16 hanma-006
192.168.168.17 hanma-007
192.168.168.18 hanma-008
~~~

Docker 镜像加速

https://help.aliyun.com/document_detail/60750.html

https://www.runoob.com/docker/docker-mirror-acceleration.html

拉取Docker镜像

~~~bash
docker pull hello-world
~~~

运行Docker镜像

~~~bash
docker run hello-world
~~~

docker 停止所有的容器

~~~bash
docker stop $(docker ps -a -q)
~~~

添加阿里kubernetes源

~~~bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
~~~

master节点安装 



~~~bash

[root@master01 ~]# yum -y install kubeadm-1.18.1 kubectl-1.18.1 kubelet-1.18.1
[root@master01 ~]# systemctl enable kubelet

systemctl enable kubelet && systemctl start kubelet
~~~

初始化k8s集群

~~~bash
kubeadm init --kubernetes-version=1.18.0  \
--apiserver-advertise-address=192.168.168.11   \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
~~~

根据提示创建kubectl

~~~bash
[root@master01 ~]#  mkdir -p $HOME/.kube
[root@master01 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master01 ~]#   sudo chown $(id -u):$(id -g) $HOME/.kube/config
~~~

执行下面命令，使kubectl可以自动补充

~~~bash
[root@master01 ~]# source <(kubectl completion bash)
~~~

查看节点，pod

~~~bash
kubectl get node

kubectl get pod --all-namespaces

## node节点为NotReady，因为corednspod没有启动，缺少网络pod
~~~

安装calico网络

~~~bash
[root@master01 ~]# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
~~~

安装kubernetes-dashboard

~~~bash
[root@master01 ~]# wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml
[root@master01 ~]# vim recommended.yaml
[root@master01 ~]# kubectl create -f recommended.yaml
~~~

查看pod，service

~~~bash
NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-dc6947fbf-869kf   1/1     Running   0          37s
kubernetes-dashboard-5d4dc8b976-sdxxt       1/1     Running   0          37s
[root@master01 ~]# kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.10.58.93    <none>        8000/TCP        44s
kubernetes-dashboard        NodePort    10.10.132.66   <none>        443:30000/TCP   44s
[root@master01 ~]#
~~~

进入docker 容器

~~~bash
 docker exec -it <容器ID> /bin/bash  
 docker attach <容器ID>
~~~

Rancher中文社区

https://docs.rancher.cn/

~~~bash
#启动命令
sudo docker run --privileged -d   --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher

~~~



Docker常用命令:

~~~bash
#停止所有容器
docker stop $(docker ps -q)

#删除所有容器
docker rm $(docker ps -aq)

# 暂停并删除所有容器
docker stop $(docker ps -q) & docker rm $(docker ps -aq)

~~~

## 设置时区

-e "TZ=Asia/Shanghai"






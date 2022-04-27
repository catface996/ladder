- 仅未关闭swap
	- ~~~shell
	  ## 待执行的命令
	  kubeadm init --kubernetes-version=1.22.3  \
	  --apiserver-advertise-address=192.168.162.81   \
	  --image-repository registry.aliyuncs.com/google_containers  \
	  --service-cidr=10.10.0.0/16  \
	  --pod-network-cidr=10.122.0.0/16
	  ## 实际的执行结果
	  [root@k8s-master-81 ~]# kubeadm init --kubernetes-version=1.22.3  \
	  > --apiserver-advertise-address=192.168.162.81   \
	  > --image-repository registry.aliyuncs.com/google_containers  \
	  > --service-cidr=10.10.0.0/16  \
	  > --pod-network-cidr=10.122.0.0/16
	  [init] Using Kubernetes version: v1.22.3
	  [preflight] Running pre-flight checks
	  error execution phase preflight: [preflight] Some fatal errors occurred:
	  	[ERROR Swap]: running with swap on is not supported. Please disable swap
	  [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
	  To see the stack trace of this error execute with --v=5 or higher
	  [root@k8s-master-81 ~]#
	  ~~~
- 仅未设置vm.swapiness=0
	- ~~~shell
	  ## 待执行的命令
	  kubeadm init --kubernetes-version=1.22.3  \
	  --apiserver-advertise-address=192.168.162.81   \
	  --image-repository registry.aliyuncs.com/google_containers  \
	  --service-cidr=10.10.0.0/16  \
	  --pod-network-cidr=10.122.0.0/16
	  ## 实际执行结果
	  [root@k8s-master-81 ~]# kubeadm init --kubernetes-version=1.22.3  \
	  > --apiserver-advertise-address=192.168.162.81   \
	  > --image-repository registry.aliyuncs.com/google_containers  \
	  > --service-cidr=10.10.0.0/16  \
	  > --pod-network-cidr=10.122.0.0/16
	  [init] Using Kubernetes version: v1.22.3
	  [preflight] Running pre-flight checks
	  [preflight] Pulling images required for setting up a Kubernetes cluster
	  [preflight] This might take a minute or two, depending on the speed of your internet connection
	  [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
	  [certs] Using certificateDir folder "/etc/kubernetes/pki"
	  [certs] Generating "ca" certificate and key
	  [certs] Generating "apiserver" certificate and key
	  [certs] apiserver serving cert is signed for DNS names [k8s-master-81 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.10.0.1 192.168.162.81]
	  [certs] Generating "apiserver-kubelet-client" certificate and key
	  [certs] Generating "front-proxy-ca" certificate and key
	  [certs] Generating "front-proxy-client" certificate and key
	  [certs] Generating "etcd/ca" certificate and key
	  [certs] Generating "etcd/server" certificate and key
	  [certs] etcd/server serving cert is signed for DNS names [k8s-master-81 localhost] and IPs [192.168.162.81 127.0.0.1 ::1]
	  [certs] Generating "etcd/peer" certificate and key
	  [certs] etcd/peer serving cert is signed for DNS names [k8s-master-81 localhost] and IPs [192.168.162.81 127.0.0.1 ::1]
	  [certs] Generating "etcd/healthcheck-client" certificate and key
	  [certs] Generating "apiserver-etcd-client" certificate and key
	  [certs] Generating "sa" key and public key
	  [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
	  [kubeconfig] Writing "admin.conf" kubeconfig file
	  [kubeconfig] Writing "kubelet.conf" kubeconfig file
	  [kubeconfig] Writing "controller-manager.conf" kubeconfig file
	  [kubeconfig] Writing "scheduler.conf" kubeconfig file
	  [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
	  [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
	  [kubelet-start] Starting the kubelet
	  [control-plane] Using manifest folder "/etc/kubernetes/manifests"
	  [control-plane] Creating static Pod manifest for "kube-apiserver"
	  [control-plane] Creating static Pod manifest for "kube-controller-manager"
	  [control-plane] Creating static Pod manifest for "kube-scheduler"
	  [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
	  [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
	  [apiclient] All control plane components are healthy after 7.003061 seconds
	  [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
	  [kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
	  [upload-certs] Skipping phase. Please see --upload-certs
	  [mark-control-plane] Marking the node k8s-master-81 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
	  [mark-control-plane] Marking the node k8s-master-81 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
	  [bootstrap-token] Using token: 1oun4f.awoxh841b4wid0st
	  [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
	  [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
	  [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
	  [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
	  [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
	  [bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
	  [kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
	  [addons] Applied essential addon: CoreDNS
	  [addons] Applied essential addon: kube-proxy
	  
	  Your Kubernetes control-plane has initialized successfully!
	  
	  To start using your cluster, you need to run the following as a regular user:
	  
	  mkdir -p $HOME/.kube
	  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	  sudo chown $(id -u):$(id -g) $HOME/.kube/config
	  
	  Alternatively, if you are the root user, you can run:
	  
	  export KUBECONFIG=/etc/kubernetes/admin.conf
	  
	  You should now deploy a pod network to the cluster.
	  Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
	  https://kubernetes.io/docs/concepts/cluster-administration/addons/
	  
	  Then you can join any number of worker nodes by running the following on each as root:
	  
	  kubeadm join 192.168.162.81:6443 --token 1oun4f.awoxh841b4wid0st \
	  	--discovery-token-ca-cert-hash sha256:ce5a9861c06459c88f4833651e0c531862bafdc1522e9664977fafa26070b08c
	  [root@k8s-master-81 ~]#
	  ~~~
- 仅未设置SELinux为Permissive
	- ~~~shell
	  ## 待执行的命令
	  kubeadm init --kubernetes-version=1.22.3  \
	  --apiserver-advertise-address=192.168.162.81   \
	  --image-repository registry.aliyuncs.com/google_containers  \
	  --service-cidr=10.10.0.0/16  \
	  --pod-network-cidr=10.122.0.0/16
	  ## 实际执行结果
	  [root@k8s-master-81 ~]# kubeadm init --kubernetes-version=1.22.3  \
	  > --apiserver-advertise-address=192.168.162.81   \
	  > --image-repository registry.aliyuncs.com/google_containers  \
	  > --service-cidr=10.10.0.0/16  \
	  > --pod-network-cidr=10.122.0.0/16
	  [init] Using Kubernetes version: v1.22.3
	  [preflight] Running pre-flight checks
	  [preflight] Pulling images required for setting up a Kubernetes cluster
	  [preflight] This might take a minute or two, depending on the speed of your internet connection
	  [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
	  [certs] Using certificateDir folder "/etc/kubernetes/pki"
	  [certs] Generating "ca" certificate and key
	  [certs] Generating "apiserver" certificate and key
	  [certs] apiserver serving cert is signed for DNS names [k8s-master-81 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.10.0.1 192.168.162.81]
	  [certs] Generating "apiserver-kubelet-client" certificate and key
	  [certs] Generating "front-proxy-ca" certificate and key
	  [certs] Generating "front-proxy-client" certificate and key
	  [certs] Generating "etcd/ca" certificate and key
	  [certs] Generating "etcd/server" certificate and key
	  [certs] etcd/server serving cert is signed for DNS names [k8s-master-81 localhost] and IPs [192.168.162.81 127.0.0.1 ::1]
	  [certs] Generating "etcd/peer" certificate and key
	  [certs] etcd/peer serving cert is signed for DNS names [k8s-master-81 localhost] and IPs [192.168.162.81 127.0.0.1 ::1]
	  [certs] Generating "etcd/healthcheck-client" certificate and key
	  [certs] Generating "apiserver-etcd-client" certificate and key
	  [certs] Generating "sa" key and public key
	  [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
	  [kubeconfig] Writing "admin.conf" kubeconfig file
	  [kubeconfig] Writing "kubelet.conf" kubeconfig file
	  [kubeconfig] Writing "controller-manager.conf" kubeconfig file
	  [kubeconfig] Writing "scheduler.conf" kubeconfig file
	  [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
	  [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
	  [kubelet-start] Starting the kubelet
	  [control-plane] Using manifest folder "/etc/kubernetes/manifests"
	  [control-plane] Creating static Pod manifest for "kube-apiserver"
	  [control-plane] Creating static Pod manifest for "kube-controller-manager"
	  [control-plane] Creating static Pod manifest for "kube-scheduler"
	  [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
	  [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
	  [apiclient] All control plane components are healthy after 7.004292 seconds
	  [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
	  [kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
	  [upload-certs] Skipping phase. Please see --upload-certs
	  [mark-control-plane] Marking the node k8s-master-81 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
	  [mark-control-plane] Marking the node k8s-master-81 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
	  [bootstrap-token] Using token: v224pv.xmg7fph2x9agb86b
	  [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
	  [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
	  [bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
	  [bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
	  [bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
	  [bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
	  [kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
	  [addons] Applied essential addon: CoreDNS
	  [addons] Applied essential addon: kube-proxy
	  
	  Your Kubernetes control-plane has initialized successfully!
	  
	  To start using your cluster, you need to run the following as a regular user:
	  
	  mkdir -p $HOME/.kube
	  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	  sudo chown $(id -u):$(id -g) $HOME/.kube/config
	  
	  Alternatively, if you are the root user, you can run:
	  
	  export KUBECONFIG=/etc/kubernetes/admin.conf
	  
	  You should now deploy a pod network to the cluster.
	  Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
	  https://kubernetes.io/docs/concepts/cluster-administration/addons/
	  
	  Then you can join any number of worker nodes by running the following on each as root:
	  
	  kubeadm join 192.168.162.81:6443 --token v224pv.xmg7fph2x9agb86b \
	  	--discovery-token-ca-cert-hash sha256:f01aa3a80c4321ab51372ebb1489cc5cf117df91a791627f6d9c6aa6be1c4470
	  [root@k8s-master-81 ~]# getenforce
	  Enforcing
	  [root@k8s-master-81 ~]#
	  ~~~
	- 回顾一下安装 kubeadm , kubectl , kubelet
	- ![20220407192627](https://picgo.catface996.com/picgo20220407192627.png)
	- 未关闭SELinux影响的是安装阶段,不是初始化集群阶段
	- 再次执行安装 kubeadm , kubectl , kubelet
	- ~~~shell
	  sudo yum install -y --nogpgcheck kubelet-1.22.3 kubeadm-1.22.3 kubectl-1.22.3 --disableexcludes=kubernetes
	  ~~~
	- 实际结果,仍然可安装成功
- 仅未加载br_netfilter
	- ~~~shell
	  ## 临时卸载br_netfilter命令
	  sudo modprobe -r br_netfilter
	  ## 实际执行结果
	  [root@k8s-master-81 ~]# lsmod | grep br_netfilter
	  br_netfilter           22256  0
	  bridge                151336  1 br_netfilter
	  [root@k8s-master-81 ~]#
	  [root@k8s-master-81 ~]# sudo modprobe -r br_netfilter
	  [root@k8s-master-81 ~]# lsmod | grep br_netfilter
	  [root@k8s-master-81 ~]#
	  ## 待执行的初始化集群命令
	  kubeadm init --kubernetes-version=1.22.3  \
	  --apiserver-advertise-address=192.168.162.81   \
	  --image-repository registry.aliyuncs.com/google_containers  \
	  --service-cidr=10.10.0.0/16  \
	  --pod-network-cidr=10.122.0.0/16
	  ## 实际执行结果
	  [root@k8s-master-81 ~]# kubeadm init --kubernetes-version=1.22.3  \
	  > --apiserver-advertise-address=192.168.162.81   \
	  > --image-repository registry.aliyuncs.com/google_containers  \
	  > --service-cidr=10.10.0.0/16  \
	  > --pod-network-cidr=10.122.0.0/16
	  [init] Using Kubernetes version: v1.22.3
	  [preflight] Running pre-flight checks
	  error execution phase preflight: [preflight] Some fatal errors occurred:
	  	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
	  [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
	  To see the stack trace of this error execute with --v=5 or higher
	  [root@k8s-master-81 ~]#
	  ~~~
- 未配置iptables
	- ~~~shell
	  ## 待执行的命令
	  kubeadm init --kubernetes-version=1.22.3  \
	  --apiserver-advertise-address=192.168.162.81   \
	  --image-repository registry.aliyuncs.com/google_containers  \
	  --service-cidr=10.10.0.0/16  \
	  --pod-network-cidr=10.122.0.0/16 
	  
	  ## 实际执行结果
	  [root@k8s-master-81 ~]# kubeadm init --kubernetes-version=1.22.3  \
	  > --apiserver-advertise-address=192.168.162.81   \
	  > --image-repository registry.aliyuncs.com/google_containers  \
	  > --service-cidr=10.10.0.0/16  \
	  > --pod-network-cidr=10.122.0.0/16
	  [init] Using Kubernetes version: v1.22.3
	  [preflight] Running pre-flight checks
	  error execution phase preflight: [preflight] Some fatal errors occurred:
	  	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
	  [preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
	  To see the stack trace of this error execute with --v=5 or higher
	  [root@k8s-master-81 ~]#
	  ~~~
#### scheduler unhealthy

~~~shell
[root@k8s-master-101 manifests]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused
controller-manager   Healthy     ok
etcd-0               Healthy     {"health":"true","reason":""}
~~~

解决方法:
修改 /etc/kubernetes/manifests/ 目录下的 kube-scheduler.yaml 和 kube-controller-manager.yaml,注释掉port=0

~~~shell
[root@k8s-master-101 manifests]# pwd
/etc/kubernetes/manifests
[root@k8s-master-101 manifests]# ll
总用量 16
drwxr-xr-x. 2 root root   79 4月   7 20:20 backup
-rw-------. 1 root root 2272 4月   7 16:59 etcd.yaml
-rw-------. 1 root root 3382 4月   7 16:59 kube-apiserver.yaml
-rw-------. 1 root root 2894 4月   7 20:02 kube-controller-manager.yaml
-rw-------. 1 root root 1480 4月   7 20:03 kube-scheduler.yaml
[root@k8s-master-101 manifests]#
~~~

![20220407202550](https://picgo.catface996.com/picgo20220407202550.png)

然后重启kubelet即可

~~~shell
## 重启kubelet
systemctl restart kubelet

[root@k8s-master-101 manifests]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-0               Healthy   {"health":"true","reason":""}
~~~

**切记** 如果要备份文件,备份的文件不能在相同的目录下.
#### docker的cgroup driver和kubelet的cgroup driver不一致

~~~shell
# 执行集群初始化
kubeadm init --kubernetes-version=1.22.3  \
--apiserver-advertise-address=192.168.162.81  \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16  \
--pod-network-cidr=10.122.0.0/16 
~~~

init 日志:
~~~shell
[root@k8s-master-81 ~]# kubeadm init --kubernetes-version=1.22.3  \
> --apiserver-advertise-address=192.168.162.81  \
> --image-repository registry.aliyuncs.com/google_containers  \
> --service-cidr=10.10.0.0/16  \
> --pod-network-cidr=10.122.0.0/16
[init] Using Kubernetes version: v1.22.3
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master-81 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.10.0.1 192.168.162.81]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master-81 localhost] and IPs [192.168.162.81 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master-81 localhost] and IPs [192.168.162.81 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
[kubelet-check] It seems like the kubelet isn't running or healthy.
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

	Unfortunately, an error has occurred:
timed out waiting for the condition

	This error is likely caused by:
- The kubelet is not running
- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

	If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
- 'systemctl status kubelet'
- 'journalctl -xeu kubelet'

	Additionally, a control plane component may have crashed or exited when started by the container runtime.
	To troubleshoot, list all containers using your preferred container runtimes CLI.

	Here is one example how you may list all Kubernetes containers running in docker:
- 'docker ps -a | grep kube | grep -v pause'
Once you have found the failing container, you can inspect its logs with:
- 'docker logs CONTAINERID'

error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher
[root@k8s-master-81 ~]#
~~~


kubelet日志:
~~~shell
4月 09 10:27:10 k8s-master-81 kubelet[2350]: I0409 10:27:10.861147    2350 docker_service.go:264] "Docker Info" dockerInfo=&{ID:5NWG:XESZ:TE5Z:ND5P:MKBL:FDHX:CAVB:OSTQ:RLEU:D47J:N4MB:JB25 Containers:0 ContainersRunning:0 ContainersPaused:0 ContainersStopped:0 Images:9 Driver:overlay2 DriverStatus:[[Backing Filesystem xfs] [Supports d_type true] [Native Overlay Diff true]] SystemStatus:[] Plugins:{Volume:[local] Network:[bridge host ipvlan macvlan null overlay] Authorization:[] Log:[awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog]} MemoryLimit:true SwapLimit:true KernelMemory:true KernelMemoryTCP:true CPUCfsPeriod:true CPUCfsQuota:true CPUShares:true CPUSet:true PidsLimit:true IPv4Forwarding:true BridgeNfIptables:true BridgeNfIP6tables:true Debug:false NFd:22 OomKillDisable:true NGoroutines:35 SystemTime:2022-04-09T10:27:10.85210973+08:00 LoggingDriver:json-file CgroupDriver:cgroupfs CgroupVersion: NEventsListener:0 KernelVersion:3.10.0-1160.el7.x86_64 OperatingSystem:CentOS Linux 7 (Core) OSVersion: OSType:linux Architecture:x86_64 IndexServerAddress:https://index.docker.io/v1/ RegistryConfig:0xc000b42310 NCPU:2 MemTotal:3953459200 GenericResources:[] DockerRootDir:/var/lib/docker HTTPProxy: HTTPSProxy: NoProxy: Name:k8s-master-81 Labels:[] ExperimentalBuild:false ServerVersion:19.03.15 ClusterStore: ClusterAdvertise: Runtimes:map[runc:{Path:runc Args:[] Shim:<nil>}] DefaultRuntime:runc Swarm:{NodeID: NodeAddr: LocalNodeState:inactive ControlAvailable:false Error: RemoteManagers:[] Nodes:0 Managers:0 Cluster:<nil> Warnings:[]} LiveRestoreEnabled:false Isolation: InitBinary:docker-init ContainerdCommit:{ID:3df54a852345ae127d1fa3092b95168e4a88e2f8 Expected:3df54a852345ae127d1fa3092b95168e4a88e2f8} RuncCommit:{ID:v1.0.3-0-gf46b6ba Expected:v1.0.3-0-gf46b6ba} InitCommit:{ID:fec3683 Expected:fec3683} SecurityOptions:[name=seccomp,profile=default] ProductLicense: DefaultAddressPools:[] Warnings:[]}
4月 09 10:27:10 k8s-master-81 kubelet[2350]: E0409 10:27:10.861177    2350 server.go:294] "Failed to run kubelet" err="failed to run Kubelet: misconfiguration: kubelet cgroup driver: \"systemd\" is different from docker cgroup driver: \"cgroupfs\""
4月 09 10:27:10 k8s-master-81 systemd[1]: kubelet.service: main process exited, code=exited, status=1/FAILURE
4月 09 10:27:10 k8s-master-81 systemd[1]: Unit kubelet.service entered failed state.
4月 09 10:27:10 k8s-master-81 systemd[1]: kubelet.service failed.
~~~
-
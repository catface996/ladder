- 主机规划
  collapsed:: true
	- | 主机名称(hostname) | IP地址         | 主机用途       | 软件环境                                                  |
	    | ------------------ | -------------- | -------------- | --------------------------------------------------------- |
	    | k8s-master-51      | 192.168.162.51 | k8s mater节点  | Centos7.9 , Dokcer-19.03.15 , kubeadm , kubelet , kubectl |
	    | k8s-node-61        | 192.168.162.61 | k8s worker节点 | Centos7.9 , Dokcer-19.03.15 , kubeadm, kubelet            |
	- Kubernetes版本 1.22.3  Kubernetes 版本兼容: https://kubernetes.io/releases/version-skew-policy
	- Docker版本 19.03.15
	-
- [[安装Centos7]]
- 安装Docker
	- 参考：[[Docker官方安装文档介绍和实战]]
- 安装Kubeadm
	- 进入Kubernetes官网
	  collapsed:: true
		- https://kubernetes.io/docs/home/
		- ![image-20220329211412452](https://tva1.sinaimg.cn/large/e6c9d24egy1h0r2apo6whj21nb0u0n3m.jpg)
	- 开始之前
	  collapsed:: true
		- 安装文档
		  collapsed:: true
			- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
			- ![image-20220401150903430](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8lpy2k5j21n90u0jzk.jpg)
		- 验证每个节点的MAC地址和product_uuid的唯一性
		  collapsed:: true
			- ![image-20220401151322963](/Users/catface/Library/Application%20Support/typora-user-images/image-20220401151322963.png)
		- 检查网络适配
		  collapsed:: true
			- ![image-20220401151547722](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8st2wx3j21ne0u0104.jpg)
		- 让iptables看到被桥接的流量
		  collapsed:: true
			- ![image-20220401151733189](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8ufoiquj21nf0u045k.jpg)
			- 为什么要配置流量到iptables?
				- 地址: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#network-plugin-requirements
				- ![image-20220401151919435](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8w92sq0j21nq0u0ai8.jpg)
		- 检查需要的端口
		  collapsed:: true
			- ![image-20220401152158925](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8z10cn8j21nf0u045p.jpg)
			- 注意端口占用
			  ![image-20220330223237154](https://tva1.sinaimg.cn/large/e6c9d24ely1h0sa6kvc3uj21nd0u0gqt.jpg)
		- 安装运行时环境
		  collapsed:: true
			- ![image-20220401152458812](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u925pwhrj21nh0u0jz0.jpg)
			- container runtimes: https://kubernetes.io/docs/setup/production-environment/container-runtimes/
			- ![image-20220401152713462](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u94hit05j21nn0u0wlf.jpg)
			- ![image-20220401153007396](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u98d0bsaj21no0u0ahn.jpg)
		- 安装kubeadm,kubelet,kubectl
		  collapsed:: true
			- ![image-20220401153445237](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9ccaeltj21nn0u0jz8.jpg)
			- ![image-20220329212622955](https://tva1.sinaimg.cn/large/e6c9d24egy1h0r2nec8hyj21na0u0wm9.jpg)
		- 配置一个cgroup driver
		  collapsed:: true
			- ![image-20220401154808825](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9q8roy9j21ni0u0dmb.jpg)
			- 配置cgroup: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/
			- ![image-20220401153954033](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9hoe0zsj21n70u0dmo.jpg)
- 故障排查
  collapsed:: true
	- ![image-20220401155345894](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9w9r521j21n30u0wn4.jpg)
	- 遇到的问题举例:
	- ![image-20220401155754776](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ua0eyfrxj21nk0u046q.jpg)
	- ![image-20220401155555531](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9yc1n1dj21nd0u0dpl.jpg)
- 使用kubeadm创建集群
  collapsed:: true
	- 前置准备
		- 删除交换分区
		  collapsed:: true
			- ~~~bash
			  ## 检查交换分区是否已经关闭
			  free -m 
			  ## 执行结果 swap total 为0时,表明已经关闭,否则未关闭
			  [root@k8s-master-62 ~]# free -m
			        total        used        free      shared  buff/cache   available
			  Mem:           3770         240        3350          11         179        3317
			  Swap:             0           0           0
			  
			  ## 临时关闭
			  [root@k8s-master-51 docker]# swapoff -a
			  [root@k8s-master-51 docker]#
			  
			  ## 永久删除交换分区
			  [root@k8s-master-51 docker]# cat /etc/fstab
			  
			  #
			  # /etc/fstab
			  # Created by anaconda on Tue Jul 13 21:32:02 2021
			  #
			  # Accessible filesystems, by reference, are maintained under '/dev/disk'
			  # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
			  #
			  /dev/mapper/centos-root /                       xfs     defaults        0 0
			  UUID=738af8af-c4bc-4426-9200-7c318c4b4133 /boot                   xfs     defaults        0 0
			  /dev/mapper/centos-home /home                   xfs     defaults        0 0
			  #/dev/mapper/centos-swap swap                    swap    defaults        0 0
			  [root@k8s-master-51 docker]# vim /etc/fstab
			  ~~~
			- ![image-20220329222228975](https://tva1.sinaimg.cn/large/e6c9d24ely1h0r49m9b3hj21cu0kcwht.jpg)
			- ~~~shell
			  ## 配置当物理内存用完后,使用交换分区
			  ## swappiness参数值说明
			  ## vm.swappiness = 0
			  ## swapness是一个在0到100之间的数, 当系统的内存剩余不到百分之swapness的时候, 开始使用swap分区. 默认的系统swapness是60, 就是内存占用超过40%的时候就开始使用swap分区了.
			  
			  [root@k8s-master-51 docker]# echo "vm.swappiness = 0">> /etc/sysctl.conf
			  ## 使配置生效
			  [root@k8s-master-51 docker]# sysctl -p
			  
			  [root@k8s-master-51 docker]#
			  ## 查看是否完全关闭
			  [root@k8s-master-51 docker]# free -m
			        total        used        free      shared  buff/cache   available
			  Mem:          15866         382       15047          11         436       15210
			  Swap:             0           0           0
			  [root@k8s-master-51 docker]#
			  
			  ## 重启服务器,再次查看
			  [root@k8s-master-51 docker]# reboot
			  Connection to 192.168.162.51 closed by remote host.
			  Connection to 192.168.162.51 closed.
			  ➜  ~ ssh root@192.168.162.51
			  root@192.168.162.51's password:
			  Last login: Mon Mar 28 21:28:05 2022 from 192.168.162.1
			  [root@k8s-master-51 ~]# ll
			  总用量 4
			  -rw-------. 1 root root 1326 7月  13 2021 anaconda-ks.cfg
			  [root@k8s-master-51 ~]# free -m
			        total        used        free      shared  buff/cache   available
			  Mem:          15866         367       15197          11         301       15224
			  Swap:             0           0           0
			  [root@k8s-master-51 ~]#
			  
			  ## 建议在 /etc/sysctl.d/ 目录下创建一个k8s.conf  配置 vm.swapiness = 0
			  ~~~
		- 关闭防火墙
		  collapsed:: true
			- ~~~shell
			  [root@k8s-master-51 ~]# systemctl stop firewalld && systemctl disable firewalld
			  [root@k8s-master-51 ~]#
			  
			  ## 查看防火墙状态
			  [root@k8s-master-51 ~]# firewall-cmd --state
			  not running
			  [root@k8s-master-51 ~]#
			  ~~~
		- 关闭SELinux
		  collapsed:: true
			- SELinux官方文档:  http://selinuxproject.org/page/Main_Page
			- SELinux策略: http://selinuxproject.org/page/NB_PolicyType
			- ~~~shell
			  ## 查看当前的模式
			  getenforce
			  
			  [root@kubeadm-50 ~]# getenforce
			  Enforcing
			  
			  ## SELinux 有三个模式（可以由用户设置）。这些模式将规定 SELinux 在主体请求时如何应对。这些模式是：
			  ## Enforcing (1)强制— SELinux 策略强制执行，基于 SELinux 策略规则授予或拒绝主体对目标的访问
			  ## Permissive (0)宽容— SELinux 策略不强制执行，不实际拒绝访问，但会有拒绝信息写入日志
			  ## Disabled 禁用— 完全禁用SELinux
			  ## 注意,从关闭状态更改为开启状态,需要通过修改配置文件来实现
			  
			  ## 查看默认的配置
			  [root@catface ~]# cat /etc/selinux/config
			  # This file controls the state of SELinux on the system.
			  # SELINUX= can take one of these three values:
			  #     enforcing - SELinux security policy is enforced.
			  #     permissive - SELinux prints warnings instead of enforcing.
			  #     disabled - No SELinux policy is loaded.
			  SELINUX=enforcing
			  # SELINUXTYPE= can take one of three values:
			  #     targeted - Targeted processes are protected,
			  #     minimum - Modification of targeted policy. Only selected processes are protected.
			  #     mls - Multi Level Security protection.
			  SELINUXTYPE=targeted
			  
			  ## k8s 官网提供的关闭命令
			  # Set SELinux in permissive mode (effectively disabling it)
			  ## 临时关闭
			  sudo setenforce 0
			  ## 永久关闭
			  sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
			  
			  ## 也可以使用以下命令来关闭
			  ## 关闭 SELinux 然后重启,再通过 sestatus 查看 SElinux的状态.
			  [root@k8s-master-51 ~]# sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config && setenforce 0
			  setenforce: SELinux is disabled
			  [root@k8s-master-51 ~]#
			  
			  ## 重启服务器,查看selinux的状态
			  [root@k8s-master-51 ~]# reboot
			  Connection to 192.168.162.51 closed by remote host.
			  Connection to 192.168.162.51 closed.
			  ➜  ~ ssh root@192.168.162.51
			  root@192.168.162.51's password:
			  Last login: Tue Mar 29 10:26:59 2022 from 192.168.162.1
			  [root@k8s-master-51 ~]# sestatus
			  SELinux status:                 disabled
			  [root@k8s-master-51 ~]#
			  ~~~
		- 配置同步时间(可选)
		  collapsed:: true
			- ~~~bash
			  #安装chrony：
			  [root@k8s-master-51 ~]# yum install -y chrony
			  已加载插件：fastestmirror
			  Loading mirror speeds from cached hostfile
			   * base: mirrors.163.com
			   * extras: mirrors.163.com
			   * updates: mirrors.cn99.com
			  base                                                                                                                                                                                                                                                     | 3.6 kB  00:00:00
			  docker-ce-stable                                                                                                                                                                                                                                         | 3.5 kB  00:00:00
			  epel                                                                                                                                                                                                                                                     | 4.7 kB  00:00:00
			  extras                                                                                                                                                                                                                                                   | 2.9 kB  00:00:00
			  updates                                                                                                                                                                                                                                                  | 2.9 kB  00:00:00
			  软件包 chrony-3.4-1.el7.x86_64 已安装并且是最新版本
			  无须任何处理
			  [root@k8s-master-51 ~]#
			  #注释默认ntp服务器
			  [root@k8s-master-51 ~]# sed -i 's/^server/#&/' /etc/chrony.conf
			  #指定上游公共 ntp 服务器，并允许其他节点同步时间
			  ## 待执行的命令
			  cat >> /etc/chrony.conf << EOF
			   server 0.asia.pool.ntp.org iburst
			   server 1.asia.pool.ntp.org iburst
			   server 2.asia.pool.ntp.org iburst
			   server 3.asia.pool.ntp.org iburst
			   allow all
			  EOF
			  
			  ## 执行结果
			  [root@k8s-master-51 ~]# cat >> /etc/chrony.conf << EOF
			  > server 0.asia.pool.ntp.org iburst
			  > server 1.asia.pool.ntp.org iburst
			  > server 2.asia.pool.ntp.org iburst
			  > server 3.asia.pool.ntp.org iburst
			  > allow all
			  > EOF
			  [root@k8s-master-51 ~]#
			  #重启chronyd服务并设为开机启动
			  [root@k8s-master-51 ~]# systemctl enable chronyd && systemctl restart chronyd
			  [root@k8s-master-51 ~]#
			  #开启网络时间同步功能
			  [root@k8s-master-51 ~]# timedatectl set-ntp true
			  [root@k8s-master-51 ~]#
			  
			  ## 设置时区
			  [root@kubeadm-50 ~]# timedatectl set-timezone Asia/Shanghai
			  ~~~
		- 将桥接的IPv4,IPv6流量传递到iptables
		  collapsed:: true
			- iptables: https://www.zsythink.net/archives/1199
			- ~~~bash
			  ## 确保br_netfilter模块已加载。这可以通过运行来完成 lsmod | grep br_netfilter。要显式加载它，请调用 sudo modprobe br_netfilter 临时加载
			  ## 检查是否已经加载
			  lsmod | grep br_netfilter
			  
			  ## 通过修改配置,永久加载
			  cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
			  br_netfilter
			  EOF
			  
			  ## 命令 注意配置的key之前不能带空格
			  cat >> /etc/sysctl.d/k8s.conf <<EOF
			  net.bridge.bridge-nf-call-ip6tables = 1
			  net.bridge.bridge-nf-call-iptables = 1
			  EOF
			  ## 执行结果
			  [root@k8s-master-51 ~]# cat > /etc/sysctl.d/k8s.conf <<EOF
			  > net.bridge.bridge-nf-call-ip6tables = 1
			  > net.bridge.bridge-nf-call-iptables = 1
			  > EOF
			  [root@k8s-master-51 ~]# cat /etc/sysctl.d/k8s.conf
			  net.bridge.bridge-nf-call-ip6tables = 1
			  net.bridge.bridge-nf-call-iptables = 1
			  
			  [root@k8s-master-51 ~]# sysctl --system
			  * Applying /usr/lib/sysctl.d/00-system.conf ...
			  net.bridge.bridge-nf-call-ip6tables = 0
			  net.bridge.bridge-nf-call-iptables = 0
			  net.bridge.bridge-nf-call-arptables = 0
			  * Applying /usr/lib/sysctl.d/10-default-yama-scope.conf ...
			  kernel.yama.ptrace_scope = 0
			  * Applying /usr/lib/sysctl.d/50-default.conf ...
			  kernel.sysrq = 16
			  kernel.core_uses_pid = 1
			  kernel.kptr_restrict = 1
			  net.ipv4.conf.default.rp_filter = 1
			  net.ipv4.conf.all.rp_filter = 1
			  net.ipv4.conf.default.accept_source_route = 0
			  net.ipv4.conf.all.accept_source_route = 0
			  net.ipv4.conf.default.promote_secondaries = 1
			  net.ipv4.conf.all.promote_secondaries = 1
			  fs.protected_hardlinks = 1
			  fs.protected_symlinks = 1
			  * Applying /etc/sysctl.d/99-sysctl.conf ...
			  vm.swappiness = 0
			  * Applying /etc/sysctl.d/k8s.conf ...
			  net.bridge.bridge-nf-call-ip6tables = 1
			  net.bridge.bridge-nf-call-iptables = 1
			  * Applying /etc/sysctl.conf ...
			  vm.swappiness = 0
			  
			  [root@k8s-master-51 ~]#
			  ## 注意,如果reboot后,sysctl --system 桥接不生效,可以试下poweroff.
			  ~~~
		- 安装Docker
		  collapsed:: true
			- 如果已经安装，跳过，未安装，则参考： [[Docker官方安装文档介绍和实战]]
		-
		-
		-
	- 安装kubeadm
		- 首先检查下k8s元源配置是否存在
		  collapsed:: true
			- ~~~shell
			  cat /etc/yum.repos.d/kubernetes.repo
			  ~~~
		- 官方文档提供的配置k8s源
		  collapsed:: true
			- ~~~shell
			  cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
			  [kubernetes]
			  name=Kubernetes
			  baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
			  enabled=1
			  gpgcheck=1
			  repo_gpgcheck=1
			  gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
			  exclude=kubelet kubeadm kubectl
			  EOF
			  ~~~
		- 我们改用阿里云的k8s源
		  collapsed:: true
			- ~~~shell
			  cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
			  [kubernetes]
			  name=Kubernetes
			  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
			  enabled=1
			  gpgcheck=1
			  repo_gpgcheck=1
			  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
			  EOF
			  ~~~
		- 执行安装命令
		  collapsed:: true
			- ~~~shell
			  ## 安装kubelet,kubeadm,kubectl
			  sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
			  
			  ## 安装kubelet,kubeadm,kubectl
			  sudo yum install -y --nogpgcheck kubelet kubeadm kubectl --disableexcludes=kubernetes
			  
			  ## 安装指定版本的kubeadm,kubectl,kubelet
			  ## 1.18.1
			  sudo yum -y --nogpgcheck install kubeadm-1.18.1 kubectl-1.18.1 kubelet-1.18.1
			  ## 1.22.3
			  sudo yum install -y --nogpgcheck kubelet-1.22.3 kubeadm-1.22.3 kubectl-1.22.3 --disableexcludes=kubernetes
			  ## 设置为开机启动并立即启动
			  sudo systemctl enable --now kubelet
			  
			  systemctl enable kubelet && systemctl start kubelet
			  
			  ## 验证kubelet
			  ## 首先想到的时 ps -ef | grep kubelet 不会有结果
			  ps -ef | grep kubelet
			  
			  ## 通过 systemctl status xxx.service 来查看serivce的状态
			  
			  ## 可能的结果:
			  [root@kubeadm-50 ~]# systemctl status kubelet.service
			  ● kubelet.service - kubelet: The Kubernetes Node Agent
			   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
			  Drop-In: /usr/lib/systemd/system/kubelet.service.d
			     └─10-kubeadm.conf
			   Active: activating (auto-restart) (Result: exit-code) since 四 2022-03-31 18:49:19 CST; 2s ago
			   Docs: https://kubernetes.io/docs/
			  Process: 1686 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
			   Main PID: 1686 (code=exited, status=1/FAILURE)
			  
			  3月 31 18:49:19 kubeadm-50 systemd[1]: kubelet.service: main process exited, code=exited, status=1...LURE
			  3月 31 18:49:19 kubeadm-50 systemd[1]: Unit kubelet.service entered failed state.
			  3月 31 18:49:19 kubeadm-50 systemd[1]: kubelet.service failed.
			  Hint: Some lines were ellipsized, use -l to show in full.
			  
			  ## 查看kubelet的运行日志,/var/lib/kubelet/config.yaml不存在,所以kubelet在不停的重启
			  [root@k8s-kubeadm-50 ~]# journalctl -f -u kubelet.service
			  -- Logs begin at 五 2022-04-01 20:22:15 CST. --
			  4月 01 20:33:01 k8s-master-61 systemd[1]: kubelet.service: main process exited, code=exited, status=1/FAILURE
			  4月 01 20:33:01 k8s-master-61 systemd[1]: Unit kubelet.service entered failed state.
			  4月 01 20:33:01 k8s-master-61 systemd[1]: kubelet.service failed.
			  4月 01 20:33:11 k8s-master-61 systemd[1]: kubelet.service holdoff time over, scheduling restart.
			  4月 01 20:33:11 k8s-master-61 systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
			  4月 01 20:33:11 k8s-master-61 systemd[1]: Started kubelet: The Kubernetes Node Agent.
			  4月 01 20:33:11 k8s-master-61 kubelet[8936]: E0401 20:33:11.865845    8936 server.go:206] "Failed to load kubelet config file" err="failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file \"/var/lib/kubelet/config.yaml\", error: open /var/lib/kubelet/config.yaml: no such file or directory" path="/var/lib/kubelet/config.yaml"
			  ~~~
		- 初始化集群
		  collapsed:: true
			- 参考文档地址: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
			- 提前准备所需的容器镜像
			  collapsed:: true
				- ![image-20220401162924933](https://tva1.sinaimg.cn/large/e6c9d24ely1h0uax6w45ij21nl0u0ajc.jpg)
				- 使用自定义的镜像配置
				- 参考文档: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#custom-images
				- ![image-20220401163713860](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ub5cbfg5j21nc0u0aix.jpg)
			- 初始化控制面板节点
			  collapsed:: true
				- ![image-20220401164004431](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ub8ep746j21n60u0akd.jpg)
				-
			- apiserver-advertise-address 和 ControlPlaneEndpoint 的注意事项
			  collapsed:: true
				- ![image-20220401164512482](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ubdmp8j1j21ni0u0gub.jpg)
			- 更多信息
			  collapsed:: true
				- ![image-20220401164641222](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ubf671b8j21mb0u0wmo.jpg)
				- ![image-20220401171830454](/Users/catface/Library/Application%20Support/typora-user-images/image-20220401171830454.png)
				- 部署Pod newwork
				- 参考文档地址: https://kubernetes.io/docs/concepts/cluster-administration/addons
				- ![image-20220401165344305](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ubmijt0nj21nd0u011u.jpg)
				- canal:  https://github.com/projectcalico/canal/blob/master/k8s-install/canal.yaml
				- Calico: https://projectcalico.docs.tigera.io/about/about-calico
				- ~~~shell
				  ## 拉取calico的镜像
				  docker pull  calico/kube-controllers:v3.22.1
				  docker pull calico/cni:v3.22.1
				  docker pull calico/pod2daemon-flexvol:v3.22.1
				  docker pull calico/node:v3.22.1
				  
				  docker images | grep calico
				  
				  [root@k8s-master-51 ~]# docker images | grep calico
				  calico/kube-controllers                                           v3.22.1             c0c6672a66a5        4 weeks ago         132MB
				  calico/cni                                                        v3.22.1             2a8ef6985a3e        4 weeks ago         236MB
				  calico/pod2daemon-flexvol                                         v3.22.1             17300d20daf9        4 weeks ago         19.7MB
				  calico/node                                                       v3.22.1             7a71aca7b60f        4 weeks ago         198MB
				  [root@k8s-master-51 ~]#
				  ~~~
				- ![image-20220401170344252](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ubx845kej21nr0u010y.jpg)
				- 部署文件地址: https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml
				- ![image-20220401170508035](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ubyeytmwj21n80u0jwg.jpg)
				- flannel地址: https://github.com/flannel-io/flannel#deploying-flannel-manually
				- ![image-20220401170858666](https://tva1.sinaimg.cn/large/e6c9d24egy1h0uc2k9hahj21n40u07a1.jpg)
				- 部署文件地址: https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
				- 或者gitee: https://gitee.com/catface996/flannel/blob/master/kube-flannel.yml
				- ![image-20220401171035881](https://tva1.sinaimg.cn/large/e6c9d24egy1h0uc48qtt1j21n40u0tc9.jpg)
				- 使用到的镜像
				- ![image-20220401171238313](https://tva1.sinaimg.cn/large/e6c9d24ely1h0uc6aabzfj21n70u0gpi.jpg)
				- ~~~shell
				  ## 拉取flannel镜像
				  docker pull rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1
				  docker pull rancher/mirrored-flannelcni-flannel:v0.17.0
				  
				  [root@k8s-master-51 ~]# docker images | grep flannel
				  rancher/mirrored-flannelcni-flannel                               v0.17.0             9247abf08677        5 weeks ago         59.8MB
				  rancher/mirrored-flannelcni-flannel-cni-plugin                    v1.0.1              ac40ce625740        2 months ago        8.1MB
				  [root@k8s-master-51 ~]#
				  ~~~
				-
			- 加入其他节点
			  collapsed:: true
				- ![image-20220401172100927](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ucew7o66j21n50u0n3x.jpg)
				- 如果token和hash未保存,可以通过一下命令获取
				- ![image-20220401212940167](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ujlnfxfej21n90u0dml.jpg)
				- ~~~shell
				  kubeadm token list
				  openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
				   openssl dgst -sha256 -hex | sed 's/^.* //'
				  ~~~
			- 执行初始化集群操作
				- 执行初始化集群命令
				  collapsed:: true
					- ~~~shell
					  ## 执行进行拉取命令,提前准备镜像
					  kubeadm 	config images pull --image-repository registry.aliyuncs.com/google_containers  --kubernetes-version=1.22.3
					  ## 执行结果
					  [root@k8s-master-61 ~]# kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers  --kubernetes-version=1.22.3
					  [config/images] Pulled registry.aliyuncs.com/google_containers/kube-apiserver:v1.22.3
					  [config/images] Pulled registry.aliyuncs.com/google_containers/kube-controller-manager:v1.22.3
					  [config/images] Pulled registry.aliyuncs.com/google_containers/kube-scheduler:v1.22.3
					  [config/images] Pulled registry.aliyuncs.com/google_containers/kube-proxy:v1.22.3
					  [config/images] Pulled registry.aliyuncs.com/google_containers/pause:3.5
					  [config/images] Pulled registry.aliyuncs.com/google_containers/etcd:3.5.0-0
					  [config/images] Pulled registry.aliyuncs.com/google_containers/coredns:v1.8.4
					  [root@k8s-master-61 ~]#
					  
					  ## 拉取到的镜像
					  [root@k8s-master-51 ~]# docker images | grep registry.aliyuncs.com
					  registry.aliyuncs.com/google_containers/kube-apiserver            v1.22.3             53224b502ea4        5 months ago        128MB
					  registry.aliyuncs.com/google_containers/kube-scheduler            v1.22.3             0aa9c7e31d30        5 months ago        52.7MB
					  registry.aliyuncs.com/google_containers/kube-controller-manager   v1.22.3             05c905cef780        5 months ago        122MB
					  registry.aliyuncs.com/google_containers/kube-proxy                v1.22.3             6120bd723dce        5 months ago        104MB
					  registry.aliyuncs.com/google_containers/etcd                      3.5.0-0             004811815584        9 months ago        295MB
					  registry.aliyuncs.com/google_containers/coredns                   v1.8.4              8d147537fb7d        10 months ago       47.6MB
					  registry.aliyuncs.com/google_containers/pause                     3.5                 ed210e3e4a5b        12 months ago       683kB
					  [root@k8s-master-51 ~]#
					  
					  ## 拉取spring-cloud-istio-demo的镜像
					  docker pull catface996/spring-cloud-istio-demo:latest
					  
					  ## 拉取到的demo镜像
					  [root@k8s-master-51 ~]# docker images | grep catface996
					  catface996/spring-cloud-istio-demo                                latest              1596d51366bc        12 hours ago        855MB
					  [root@k8s-master-51 ~]#
					  
					  ## 拉取配置文件到服务器
					  git clone https://gitee.com/catface996/ladder.git
					  
					  
					  ## 初始化集群
					  ## 注意修改 apiserver-advertise-address
					  kubeadm init --kubernetes-version=1.22.3  \
					  --apiserver-advertise-address=xxx.xxx.xxx.xxx   \
					  --image-repository registry.aliyuncs.com/google_containers  \
					  --service-cidr=10.10.0.0/16  \
					  --pod-network-cidr=10.122.0.0/16 
					  
					  ## 执行结果
					  [root@kubeadm-50 ~]# kubeadm init --kubernetes-version=1.22.3  \
					  > --apiserver-advertise-address=192.168.162.50   \
					  > --image-repository registry.aliyuncs.com/google_containers  \
					  > --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
					  [init] Using Kubernetes version: v1.22.3
					  [preflight] Running pre-flight checks
					  [preflight] Pulling images required for setting up a Kubernetes cluster
					  [preflight] This might take a minute or two, depending on the speed of your internet connection
					  [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'   ## 注意,这里提示可以提前先拉取镜像
					  
					  kubeadm init --kubernetes-version=1.23.5  \
					  --apiserver-advertise-address=192.168.162.50   \
					  --image-repository registry.aliyuncs.com/google_containers  \
					  --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
					  ## 初始化结果
					  [root@kubeadm-50 ~]# kubeadm init --kubernetes-version=1.22.3  \
					  > --apiserver-advertise-address=192.168.162.50   \
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
					  [certs] apiserver serving cert is signed for DNS names [kubeadm-50 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.10.0.1 192.168.162.50]
					  [certs] Generating "apiserver-kubelet-client" certificate and key
					  [certs] Generating "front-proxy-ca" certificate and key
					  [certs] Generating "front-proxy-client" certificate and key
					  [certs] Generating "etcd/ca" certificate and key
					  [certs] Generating "etcd/server" certificate and key
					  [certs] etcd/server serving cert is signed for DNS names [kubeadm-50 localhost] and IPs [192.168.162.50 127.0.0.1 ::1]
					  [certs] Generating "etcd/peer" certificate and key
					  [certs] etcd/peer serving cert is signed for DNS names [kubeadm-50 localhost] and IPs [192.168.162.50 127.0.0.1 ::1]
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
					  [apiclient] All control plane components are healthy after 7.003203 seconds
					  [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
					  [kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
					  [upload-certs] Skipping phase. Please see --upload-certs
					  [mark-control-plane] Marking the node kubeadm-50 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
					  [mark-control-plane] Marking the node kubeadm-50 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
					  [bootstrap-token] Using token: rahq06.njpv58x29z1mnp3z
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
					  
					  kubeadm join 192.168.162.50:6443 --token rahq06.njpv58x29z1mnp3z \
					  	--discovery-token-ca-cert-hash sha256:63f04e26a57ab375f572528a235894d5f00738e4dd592612ae493627eb906ee0
					  [root@kubeadm-50 ~]#
					  ~~~
				- 查看集群节点状态
				  collapsed:: true
					- ~~~shell
					  ## 在集群的网络插件未完成安装之前,coredns一致会处于pending状态.
					  kubectl get pod --all-namespaces
					  
					  kubectl get node -n kube-system
					  ~~~
				- 再次查看kubelet的状态
				  collapsed:: true
					- ~~~shell
					  [root@k8s-master-61 flannel]# systemctl status kubelet.service
					  ● kubelet.service - kubelet: The Kubernetes Node Agent
					   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
					  Drop-In: /usr/lib/systemd/system/kubelet.service.d
					     └─10-kubeadm.conf
					   Active: active (running) since 五 2022-04-01 20:54:41 CST; 31min ago
					   Docs: https://kubernetes.io/docs/
					   Main PID: 15569 (kubelet)
					  Tasks: 13
					   Memory: 47.9M
					   CGroup: /system.slice/kubelet.service
					     └─15569 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infr...
					  - 4月 01 21:12:19 k8s-master-61 kubelet[15569]: E0401 21:12:19.347018   15569 kubelet.go:2337] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady m...uninitialized"
					  4月 01 21:12:25 k8s-master-61 kubelet[15569]: I0401 21:12:25.598141   15569 topology_manager.go:200] "Topology Admit Handler"
					  4月 01 21:12:25 k8s-master-61 kubelet[15569]: I0401 21:12:25.598642   15569 topology_manager.go:200] "Topology Admit Handler"
					  4月 01 21:12:25 k8s-master-61 kubelet[15569]: I0401 21:12:25.711362   15569 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-api-access-f7b7m\" (UniqueName: \"kub...
					  4月 01 21:12:25 k8s-master-61 kubelet[15569]: I0401 21:12:25.711400   15569 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"config-volume\" (UniqueName: \"kubernetes....
					  4月 01 21:12:25 k8s-master-61 kubelet[15569]: I0401 21:12:25.711422   15569 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"config-volume\" (UniqueName: \"kubernetes....
					  4月 01 21:12:25 k8s-master-61 kubelet[15569]: I0401 21:12:25.711440   15569 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-api-access-fmx2j\" (UniqueName: \"kub...
					  4月 01 21:12:26 k8s-master-61 kubelet[15569]: map[string]interface {}{"cniVersion":"0.3.1", "hairpinMode":true, "ipMasq":false, "ipam":map[string]interface {}{"ranges":[][]map[string]interface {}{[]map[string...
					  4月 01 21:12:26 k8s-master-61 kubelet[15569]: {"cniVersion":"0.3.1","hairpinMode":true,"ipMasq":false,"ipam":{"ranges":[[{"subnet":"10.122.0.0/24"}]],"routes":[{"dst":"10.244.0.0/16"}],"type":"h...ype":"bridge"}
					  4月 01 21:12:26 k8s-master-61 kubelet[15569]: map[string]interface {}{"cniVersion":"0.3.1", "hairpinMode":true, "ipMasq":false, "ipam":map[string]interface {}{"ranges":[][]map[string]interface {}{[]map[string...
					  Hint: Some lines were ellipsized, use -l to show in full.
					  ~~~
				- 安装calico网络 或者 flannel网络
				  collapsed:: true
					- ~~~shell
					  ## 安装 calico
					  [root@kubeadm-50 ~]# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
					  [root@kubeadm-50 ~]#
					  ## 可以提前准备这些镜像
					  [root@kubeadm-50 ~]# docker images | grep calico
					  calico/kube-controllers                                           v3.22.1             c0c6672a66a5        3 weeks ago         132MB
					  calico/cni                                                        v3.22.1             2a8ef6985a3e        3 weeks ago         236MB
					  calico/pod2daemon-flexvol                                         v3.22.1             17300d20daf9        3 weeks ago         19.7MB
					  calico/node                                                       v3.22.1             7a71aca7b60f        3 weeks ago         198MB
					  
					  ## 安装 flannel
					  ## 提前拉取镜像
					  docker pull rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1
					  docker pull rancher/mirrored-flannelcni-flannel:v0.17.0
					  ## scp部署文件到服务器
					  scp kube-flannel.yml root@192.168.162.61:~/flannel/kube-flannel.yml
					  ## 部署flannel到k8s集群
					  kubectl apply -f ~/flannel/kube-flannel.yml
					  ~~~
				- 查看kube-system空间下的pod状态
				  collapsed:: true
					- ~~~shell
					  [root@kubeadm-50 ~]# kubectl get pod -n kube-system
					  NAME                                       READY   STATUS    RESTARTS   AGE
					  calico-kube-controllers-6fd7b9848d-sllfb   1/1     Running   0          15m
					  calico-node-rsfbp                          1/1     Running   0          15m
					  coredns-7f6cbbb7b8-4v88m                   1/1     Running   0          21m
					  coredns-7f6cbbb7b8-l5wdw                   1/1     Running   0          21m
					  etcd-kubeadm-50                            1/1     Running   0          21m
					  kube-apiserver-kubeadm-50                  1/1     Running   0          21m
					  kube-controller-manager-kubeadm-50         1/1     Running   0          21m
					  kube-proxy-68vdp                           1/1     Running   0          21m
					  kube-scheduler-kubeadm-50                  1/1     Running   0          21m
					  ~~~
				- 查看node节点状态
				  collapsed:: true
					- ~~~shell
					  [root@kubeadm-50 ~]# kubectl get node
					  NAME         STATUS   ROLES                  AGE   VERSION
					  kubeadm-50   Ready    control-plane,master   21m   v1.22.3
					  ~~~
				- 加入其他节点
				  collapsed:: true
					- ~~~shell
					  ## master节点,需要证书共享
					  kubeadm join 192.168.162.50:6443 --token rahq06.njpv58x29z1mnp3z \
					  	--discovery-token-ca-cert-hash sha256:63f04e26a57ab375f572528a235894d5f00738e4dd592612ae493627eb906ee0 --control-plane
					  ## worker节点
					  kubeadm join 192.168.162.50:6443 --token rahq06.njpv58x29z1mnp3z \
					  	--discovery-token-ca-cert-hash sha256:63f04e26a57ab375f572528a235894d5f00738e4dd592612ae493627eb906ee0
					  
					  ## 执行加入worker节点
					  [root@k8s-master-62 ~]# kubeadm join 192.168.162.50:6443 --token rahq06.njpv58x29z1mnp3z \
					  	--discovery-token-ca-cert-hash sha256:63f04e26a57ab375f572528a235894d5f00738e4dd592612ae493627eb906ee0
					  [preflight] Running pre-flight checks
					  [preflight] Reading configuration from the cluster...
					  [preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
					  [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
					  [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
					  [kubelet-start] Starting the kubelet
					  [kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
					  
					  This node has joined the cluster:
					  * Certificate signing request was sent to apiserver and a response was received.
					  * The Kubelet was informed of the new secure connection details.
					  
					  Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
					  
					  ## 执行加入master节点
					  [root@k8s-master-63 ~]# kubeadm join 192.168.162.50:6443 --token rahq06.njpv58x29z1mnp3z \
					  	--discovery-token-ca-cert-hash sha256:63f04e26a57ab375f572528a235894d5f00738e4dd592612ae493627eb906ee0 --control-plane
					  [preflight] Running pre-flight checks
					  [preflight] Reading configuration from the cluster...
					  [preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
					  error execution phase preflight:
					  One or more conditions for hosting a new control plane instance is not satisfied.
					  
					  unable to add a new control plane instance a cluster that doesn't have a stable controlPlaneEndpoint address
					  
					  Please ensure that:
					  * The cluster has a stable controlPlaneEndpoint address.
					  * The certificates that must be shared among control plane instances are provided.
					  
					  
					  To see the stack trace of this error execute with --v=5 or higher
					  ~~~
				-
		-
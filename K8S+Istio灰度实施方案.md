



# K8S + Istio 灰度实施方案



## 1.快速体验K8S

k8s官网地址:  https://kubernetes.io/

![image-20220328152652576](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmmx7tyxj21nd0u0aeg.jpg)

### Tutorials

Learn Kubernetes Basic

![image-20220328152748666](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmnv31q0j21n80u044x.jpg)

![image-20220328152836050](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmoo4g0fj21mz0u0dkl.jpg)

知识点介绍:

![image-20220328153155320](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pms5cxb5j21n90u0gs3.jpg)

![image-20220328153221927](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmsl425aj21nb0u0tdn.jpg)

![image-20220328153252526](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pmt3xvugj21n50u0djq.jpg)

![image-20220328153857941](https://tva1.sinaimg.cn/large/e6c9d24egy1h0pmzkv8iej21n00u0100.jpg)

![image-20220328153937157](https://tva1.sinaimg.cn/large/e6c9d24egy1h0pn08ah0uj21n30u0qb9.jpg)



## 2.安装docker

### 2.1 系统要求和安装版本

* 操作系统 Centos 7.x
* docker版本: Server Version: 18.09.9

### 2.2 安装步骤

* 进入官网,下载指定版本的docker安装包.官网地址: https://www.docker.com/

  ![image-20220328211018117](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwk7xkuzj21nh0u0tcs.jpg)

* 点击右上角"Get Started",进入选择平台Docker Desktop版本页面

  ![image-20220328211114474](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qm5wvqzzj21ne0u0tcw.jpg)

* 点击"View Linux Engine",进入选择Linux平台页面.我的虚拟机是Centos7.x,内核版本是3.10.0-1160.31.1.el7.x86_64,所以选择Server,Linux,x86-64

  ~~~shell
  [root@kubeadm-50 containerd]# cat /etc/centos-release
  CentOS Linux release 7.9.2009 (Core)
  
  [root@kubeadm-50 containerd]# uname -r
  3.10.0-1160.31.1.el7.x86_64
  ~~~

  

  ![image-20220328211325821](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwnlj3mgj21nd0u0426.jpg)

* 选择"Docker Engine - CentOS",进入安装Docker引导页面

  地址: https://docs.docker.com/engine/install/centos/

  ![image-20220328212103343](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwvexndfj21nf0u07b7.jpg)

* 对操作系统的要求,需要Centos7 or 8.centos-extras,overlay2可以自行了解.

  * centos-extras 参考地址: https://wiki.centos.org/zh/AdditionalResources/Repositories
  * overlay2 参考地址: https://zhuanlan.zhihu.com/p/41958018

  ![image-20220328212456337](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwzfy98nj21n60u0jyi.jpg)

* 卸载已安装的Docker版本.

  ![image-20220328214036422](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pxftuejij21mz0u046c.jpg)

  旧版本卸载前:

  ~~~shell
  [root@kubeadm-50 docker]# pwd
  /var/lib/docker
  [root@kubeadm-50 docker]# ls -al
  总用量 24
  drwx--x--x   14 root root   182 3月  25 06:41 .
  drwxr-xr-x.  28 root root  4096 7月  14 2021 ..
  drwx------    2 root root    24 7月  14 2021 builder
  drwx------    4 root root    92 7月  14 2021 buildkit
  drwx------    2 root root     6 7月  14 2021 containers
  drwx------    3 root root    22 7月  14 2021 image
  drwxr-x---    3 root root    19 7月  14 2021 network
  drwx------  185 root root 16384 3月  25 06:41 overlay2
  drwx------    4 root root    32 7月  14 2021 plugins
  drwx------    2 root root     6 3月  25 06:41 runtimes
  drwx------    2 root root     6 7月  14 2021 swarm
  drwx------    2 root root     6 3月  25 06:41 tmp
  drwx------    2 root root     6 7月  14 2021 trust
  drwx------    2 root root    25 7月  14 2021 volumes
  [root@kubeadm-50 docker]#
  ~~~

  执行卸载命令,注意使用remove时,会把docker依赖的包一并删除.

  ~~~shell
  [root@kubeadm-50 docker]# sudo yum remove docker \
  >                   docker-client \
  >                   docker-client-latest \
  >                   docker-common \
  >                   docker-latest \
  >                   docker-latest-logrotate \
  >                   docker-logrotate \
  >                   docker-engine
  已加载插件：fastestmirror
  参数 docker 没有匹配
  参数 docker-client 没有匹配
  参数 docker-client-latest 没有匹配
  参数 docker-common 没有匹配
  参数 docker-latest 没有匹配
  参数 docker-latest-logrotate 没有匹配
  参数 docker-logrotate 没有匹配
  参数 docker-engine 没有匹配
  不删除任何软件包
  [root@kubeadm-50 docker]#
  ~~~

  查看已经安装的docker,已经安装的版本是docker-ce可参照以下命令删除,或者参考删除docker-ce部分.

  ~~~shell
  [root@kubeadm-50 docker]# yum list installed | grep docker
  Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
  containerd.io.x86_64            1.4.6-3.1.el7                  @docker-ce-stable
  docker-ce.x86_64                3:18.09.9-3.el7                @docker-ce-stable
  docker-ce-cli.x86_64            1:20.10.7-3.el7                @docker-ce-stable
  docker-scan-plugin.x86_64       0.8.0-3.el7                    @docker-ce-stable
  [root@kubeadm-50 docker]#
  ~~~

  重新执行删除命令:

  ~~~shell
  [root@kubeadm-50 docker]# sudo yum remove docker \
  >                    docker-ce.x86_64 \
  >                    docker-ce-cli.x86_64 \
  >
  已加载插件：fastestmirror
  参数 docker 没有匹配
  正在解决依赖关系
  --> 正在检查事务
  ---> 软件包 docker-ce.x86_64.3.18.09.9-3.el7 将被 删除
  ---> 软件包 docker-ce-cli.x86_64.1.20.10.7-3.el7 将被 删除
  --> 正在处理依赖关系 docker-ce-cli，它被软件包 docker-scan-plugin-0.8.0-3.el7.x86_64 需要
  --> 正在检查事务
  ---> 软件包 docker-scan-plugin.x86_64.0.0.8.0-3.el7 将被 删除
  --> 解决依赖关系完成
  
  依赖关系解决
  
  ========================================================================================================================================
   Package                             架构                    版本                              源                                  大小
  ========================================================================================================================================
  正在删除:
   docker-ce                           x86_64                  3:18.09.9-3.el7                   @docker-ce-stable                   90 M
   docker-ce-cli                       x86_64                  1:20.10.7-3.el7                   @docker-ce-stable                  156 M
  为依赖而移除:
   docker-scan-plugin                  x86_64                  0.8.0-3.el7                       @docker-ce-stable                   13 M
  
  事务概要
  ========================================================================================================================================
  移除  2 软件包 (+1 依赖软件包)
  
  安装大小：260 M
  是否继续？[y/N]：y
  Downloading packages:
  Running transaction check
  Running transaction test
  Transaction test succeeded
  Running transaction
  未将 /usr/bin/dockerd 配置为 dockerd 的备用项
    正在删除    : 3:docker-ce-18.09.9-3.el7.x86_64                                                                                    1/3
    正在删除    : docker-scan-plugin-0.8.0-3.el7.x86_64                                                                               2/3
    正在删除    : 1:docker-ce-cli-20.10.7-3.el7.x86_64                                                                                3/3
    验证中      : 1:docker-ce-cli-20.10.7-3.el7.x86_64                                                                                1/3
    验证中      : docker-scan-plugin-0.8.0-3.el7.x86_64                                                                               2/3
    验证中      : 3:docker-ce-18.09.9-3.el7.x86_64                                                                                    3/3
  
  删除:
    docker-ce.x86_64 3:18.09.9-3.el7                                 docker-ce-cli.x86_64 1:20.10.7-3.el7
  
  作为依赖被删除:
    docker-scan-plugin.x86_64 0:0.8.0-3.el7
  
  完毕！
  [root@kubeadm-50 docker]#
  ~~~

  查看是否删除完成,docker已经被删除,/var/lib/docker目录被保留了,也可以对其进行清理.

  ~~~shell
  [root@kubeadm-50 docker]# docker
  -bash: /usr/bin/docker: 没有那个文件或目录
  [root@kubeadm-50 docker]#
  [root@kubeadm-50 docker]# pwd
  /var/lib/docker
  [root@kubeadm-50 docker]# ls -al
  总用量 24
  drwx--x--x   14 root root   182 3月  25 06:41 .
  drwxr-xr-x.  28 root root  4096 7月  14 2021 ..
  drwx------    2 root root    24 7月  14 2021 builder
  drwx------    4 root root    92 7月  14 2021 buildkit
  drwx------    2 root root     6 7月  14 2021 containers
  drwx------    3 root root    22 7月  14 2021 image
  drwxr-x---    3 root root    19 7月  14 2021 network
  drwx------  185 root root 16384 3月  25 06:41 overlay2
  drwx------    4 root root    32 7月  14 2021 plugins
  drwx------    2 root root     6 3月  25 06:41 runtimes
  drwx------    2 root root     6 7月  14 2021 swarm
  drwx------    2 root root     6 3月  25 06:41 tmp
  drwx------    2 root root     6 7月  14 2021 trust
  drwx------    2 root root    25 7月  14 2021 volumes
  [root@kubeadm-50 docker]#
  ~~~

* 查看安装方案,选择常用的安装方法.

  ![image-20220328215726558](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qm6lp7csj21nh0u0jzd.jpg)

* 设置仓库

  ![image-20220328215904905](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pxyz63ggj21n40u0n4l.jpg)

  ~~~shell
  [root@kubeadm-50 docker]#  sudo yum install -y yum-utils
  已加载插件：fastestmirror
  Loading mirror speeds from cached hostfile
   * base: mirrors.163.com
   * extras: mirrors.ustc.edu.cn
   * updates: mirrors.163.com
  base                                                                                                             | 3.6 kB  00:00:00
  docker-ce-stable                                                                                                 | 3.5 kB  00:00:00
  epel                                                                                                             | 4.7 kB  00:00:00
  extras                                                                                                           | 2.9 kB  00:00:00
  updates                                                                                                          | 2.9 kB  00:00:00
  (1/6): epel/x86_64/group_gz                                                                                      |  96 kB  00:00:00
  (2/6): docker-ce-stable/7/x86_64/primary_db                                                                      |  75 kB  00:00:00
  (3/6): epel/x86_64/updateinfo                                                                                    | 1.1 MB  00:00:00
  (4/6): extras/7/x86_64/primary_db                                                                                | 246 kB  00:00:00
  (5/6): epel/x86_64/primary_db                                                                                    | 7.0 MB  00:00:00
  (6/6): updates/7/x86_64/primary_db                                                                               |  14 MB  00:00:02
  软件包 yum-utils-1.1.31-54.el7_8.noarch 已安装并且是最新版本
  无须任何处理
  [root@kubeadm-50 docker]#
  
  ## 配置docker yum 源
  sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
      
  [root@kubeadm-50 ~]# sudo yum-config-manager \
  >     --add-repo \
  >     https://download.docker.com/linux/centos/docker-ce.repo
  已加载插件：fastestmirror
  adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
  grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
  repo saved to /etc/yum.repos.d/docker-ce.repo
  [root@kubeadm-50 ~]#
  
  ~~~

* 安装指定版本的Docker

  ![image-20220328220853090](https://tva1.sinaimg.cn/large/e6c9d24ely1h0qm6ut4koj21nh0u0n5j.jpg)

  ~~~shell
  ## 执行 yum list docker-ce --showduplicates | sort -r 时返回结果过多,可以过滤出18.09版本,执行以下命令
  [root@kubeadm-50 docker]# yum list docker-ce --showduplicates | grep 18.09 | sort -r
  docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
  docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
  ~~~

  安装时,关注的是Server的版本,Client可以安装最新版本,也可以安装指定版本.本次安装同时指定docker-ce-cli版本.

  ~~~shell
  sudo yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io
    或者
  sudo yum install docker-ce-19.03.15 docker-ce-cli-19.03.15 containerd.io
  
  [root@kubeadm-50 docker]#  sudo yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io
  已加载插件：fastestmirror
  Loading mirror speeds from cached hostfile
   * base: mirrors.163.com
   * extras: mirrors.ustc.edu.cn
   * updates: mirrors.163.com
  正在解决依赖关系
  --> 正在检查事务
  ---> 软件包 containerd.io.x86_64.0.1.4.6-3.1.el7 将被 升级
  ---> 软件包 containerd.io.x86_64.0.1.5.11-3.1.el7 将被 更新
  ---> 软件包 docker-ce.x86_64.3.18.09.9-3.el7 将被 安装
  ---> 软件包 docker-ce-cli.x86_64.1.18.09.9-3.el7 将被 安装
  --> 解决依赖关系完成
  
  依赖关系解决
  
  ========================================================================================================================================
   Package                         架构                     版本                                 源                                  大小
  ========================================================================================================================================
  正在安装:
   docker-ce                       x86_64                   3:18.09.9-3.el7                      docker-ce-stable                    21 M
   docker-ce-cli                   x86_64                   1:18.09.9-3.el7                      docker-ce-stable                    16 M
  正在更新:
   containerd.io                   x86_64                   1.5.11-3.1.el7                       docker-ce-stable                    29 M
  
  事务概要
  ========================================================================================================================================
  安装  2 软件包
  升级  1 软件包
  
  总下载量：66 M
  Is this ok [y/d/N]: y
  Downloading packages:
  Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
  (1/3): containerd.io-1.5.11-3.1.el7.x86_64.rpm                                                                   |  29 MB  00:00:04
  (2/3): docker-ce-18.09.9-3.el7.x86_64.rpm                                                                        |  21 MB  00:00:05
  (3/3): docker-ce-cli-18.09.9-3.el7.x86_64.rpm                                                                    |  16 MB  00:00:02
  ----------------------------------------------------------------------------------------------------------------------------------------
  总计                                                                                                     10 MB/s |  66 MB  00:00:06
  Running transaction check
  Running transaction test
  Transaction test succeeded
  Running transaction
    正在安装    : 1:docker-ce-cli-18.09.9-3.el7.x86_64                                                                                1/4
    正在更新    : containerd.io-1.5.11-3.1.el7.x86_64                                                                                 2/4
    正在安装    : 3:docker-ce-18.09.9-3.el7.x86_64                                                                                    3/4
    清理        : containerd.io-1.4.6-3.1.el7.x86_64                                                                                  4/4
    验证中      : containerd.io-1.5.11-3.1.el7.x86_64                                                                                 1/4
    验证中      : 1:docker-ce-cli-18.09.9-3.el7.x86_64                                                                                2/4
    验证中      : 3:docker-ce-18.09.9-3.el7.x86_64                                                                                    3/4
    验证中      : containerd.io-1.4.6-3.1.el7.x86_64                                                                                  4/4
  
  已安装:
    docker-ce.x86_64 3:18.09.9-3.el7                                 docker-ce-cli.x86_64 1:18.09.9-3.el7
  
  更新完毕:
    containerd.io.x86_64 0:1.5.11-3.1.el7
  
  完毕！
  [root@kubeadm-50 docker]
  ~~~

* 启动并验证docker

  ![image-20220328221946564](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pyki16h0j21n70u046p.jpg)

  ~~~shell
  [root@kubeadm-50 docker]# sudo systemctl start docker
  [root@kubeadm-50 docker]# docker info
  Containers: 0
   Running: 0
   Paused: 0
   Stopped: 0
  Images: 34
  Server Version: 18.09.9
  Storage Driver: overlay2
   Backing Filesystem: xfs
   Supports d_type: true
   Native Overlay Diff: true
  Logging Driver: json-file
  Cgroup Driver: cgroupfs
  Plugins:
   Volume: local
   Network: bridge host macvlan null overlay
   Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
  Swarm: inactive
  Runtimes: runc
  Default Runtime: runc
  Init Binary: docker-init
  containerd version: 3df54a852345ae127d1fa3092b95168e4a88e2f8
  runc version: N/A
  init version: fec3683
  Security Options:
   seccomp
    Profile: default
  Kernel Version: 3.10.0-1160.31.1.el7.x86_64
  Operating System: CentOS Linux 7 (Core)
  OSType: linux
  Architecture: x86_64
  CPUs: 4
  Total Memory: 15.49GiB
  Name: kubeadm-50
  ID: IEHB:MMIE:LS6L:XYMB:GKVA:ERTS:SKUP:FKL3:J4LN:7GMW:LCXI:HIDR
  Docker Root Dir: /var/lib/docker
  Debug Mode (client): false
  Debug Mode (server): false
  Registry: https://index.docker.io/v1/
  Labels:
  Experimental: false
  Insecure Registries:
   127.0.0.0/8
  Registry Mirrors:
   https://oxps1jtf.mirror.aliyuncs.com/
  Live Restore Enabled: false
  Product License: Community Engine
  
  [root@kubeadm-50 docker]#
  ~~~

  ~~~shell
  [root@kubeadm-50 docker]# sudo docker run hello-world
  Unable to find image 'hello-world:latest' locally
  latest: Pulling from library/hello-world
  2db29710123e: Pull complete
  Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
  Status: Downloaded newer image for hello-world:latest
  
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  
  To generate this message, Docker took the following steps:
   1. The Docker client contacted the Docker daemon.
   2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
      (amd64)
   3. The Docker daemon created a new container from that image which runs the
      executable that produces the output you are currently reading.
   4. The Docker daemon streamed that output to the Docker client, which sent it
      to your terminal.
  
  To try something more ambitious, you can run an Ubuntu container with:
   $ docker run -it ubuntu bash
  
  Share images, automate workflows, and more with a free Docker ID:
   https://hub.docker.com/
  
  For more examples and ideas, visit:
   https://docs.docker.com/get-started/
  
  [root@kubeadm-50 docker]#
  ~~~

* 设置开机启动

  ~~~shell
  [root@kubeadm-50 docker]# systemctl enable docker.service
  Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
  [root@kubeadm-50 docker]#
  [root@kubeadm-50 bin]# sudo systemctl enable containerd.service
  Created symlink from /etc/systemd/system/multi-user.target.wants/containerd.service to /usr/lib/systemd/system/containerd.service.
  ~~~

  重启虚拟机,验证docker是否自动启动.

  ~~~shell
  [root@kubeadm-50 docker]# reboot
  Connection to 192.168.162.50 closed by remote host.
  Connection to 192.168.162.50 closed.
  ➜  ~ ssh root@192.168.162.50
  root@192.168.162.50's password:
  Last login: Fri Mar 25 05:50:25 2022 from 192.168.162.1
  [root@kubeadm-50 ~]# docker info
  Containers: 1
   Running: 0
   Paused: 0
   Stopped: 1
  Images: 1
  Server Version: 18.09.9
  Storage Driver: overlay2
   Backing Filesystem: xfs
   Supports d_type: true
   Native Overlay Diff: true
  Logging Driver: json-file
  Cgroup Driver: cgroupfs
  Plugins:
   Volume: local
   Network: bridge host macvlan null overlay
   Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
  Swarm: inactive
  Runtimes: runc
  Default Runtime: runc
  Init Binary: docker-init
  containerd version: 3df54a852345ae127d1fa3092b95168e4a88e2f8
  runc version: N/A
  init version: fec3683
  Security Options:
   seccomp
    Profile: default
  Kernel Version: 3.10.0-1160.31.1.el7.x86_64
  Operating System: CentOS Linux 7 (Core)
  OSType: linux
  Architecture: x86_64
  CPUs: 4
  Total Memory: 15.49GiB
  Name: kubeadm-50
  ID: FSX4:QLTH:GVWO:OKQ5:NIX6:4R3C:R25K:TB5O:YKNH:F5PG:I4Q3:6XTO
  Docker Root Dir: /var/lib/docker
  Debug Mode (client): false
  Debug Mode (server): false
  Registry: https://index.docker.io/v1/
  Labels:
  Experimental: false
  Insecure Registries:
   127.0.0.0/8
  Registry Mirrors:
   https://oxps1jtf.mirror.aliyuncs.com/
  Live Restore Enabled: false
  Product License: Community Engine
  
  [root@kubeadm-50 ~]#
  ~~~

* 配置DockerHub国内镜像源

  国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

  - 科大镜像：**https://docker.mirrors.ustc.edu.cn/**
  - 网易：**https://hub-mirror.c.163.com/**
  - 阿里云：**https://<你的ID>.mirror.aliyuncs.com**
  - 七牛云加速器：**https://reg-mirror.qiniu.com**

  我们使用网易的镜像加速服务来配置.

  对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）：

  ```json
  {
    "registry-mirrors":["https://hub-mirror.c.163.com/"],
    "exec-opts": ["native.cgroupdriver=systemd"]
  }
  ```

   

  ~~~shell
  [root@kubeadm-50 etc]# ls -al | grep docker
  [root@kubeadm-50 etc]#
  [root@kubeadm-50 etc]# mkdir docker
  [root@kubeadm-50 etc]# cd docker
  [root@kubeadm-50 docker]# touch daemon.json
  [root@kubeadm-50 docker]# vim daemon.json
  ~~~

  ![image-20220328224252098](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pz8j3zf6j219405o74p.jpg)

  之后重新启动服务.

  ~~~shell
  [root@kubeadm-50 docker]# sudo systemctl daemon-reload
  [root@kubeadm-50 docker]# sudo systemctl restart docker
  ## 删除之前已经存在于本地的hello-world容器和镜像
  [root@kubeadm-50 docker]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  hello-world         latest              feb5d9fea6a5        6 months ago        13.3kB
  [root@kubeadm-50 docker]# docker ps -al
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
  9e21696e7e47        hello-world         "/hello"            9 minutes ago       Exited (0) 9 minutes ago                       dreamy_shirley
  [root@kubeadm-50 docker]# docker rm 9e21696e7e47
  9e21696e7e47
  [root@kubeadm-50 docker]# docker rmi feb5d9fea6a5
  Untagged: hello-world:latest
  Untagged: hello-world@sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
  Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
  Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359
  ## 重新拉取hellow-world镜像并运行.
  [root@kubeadm-50 docker]# docker run hello-world
  Unable to find image 'hello-world:latest' locally
  latest: Pulling from library/hello-world
  latest: Pulling from library/hello-world
  2db29710123e: Pull complete
  Digest: sha256:bfea6278a0a267fad2634554f4f0c6f31981eea41c553fdf5a83e95a41d40c38
  Status: Downloaded newer image for hello-world:latest
  
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  
  To generate this message, Docker took the following steps:
   1. The Docker client contacted the Docker daemon.
   2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
      (amd64)
   3. The Docker daemon created a new container from that image which runs the
      executable that produces the output you are currently reading.
   4. The Docker daemon streamed that output to the Docker client, which sent it
      to your terminal.
  
  To try something more ambitious, you can run an Ubuntu container with:
   $ docker run -it ubuntu bash
  
  Share images, automate workflows, and more with a free Docker ID:
   https://hub.docker.com/
  
  For more examples and ideas, visit:
   https://docs.docker.com/get-started/
  ## 拉取一个自定义镜像,查看下载速度,7MB/s,因个人网络带宽不同会有差异.
  [root@kubeadm-50 docker]# docker pull catface996/spring-cloud-istio-demo:latest
  latest: Pulling from catface996/spring-cloud-istio-demo
  2d473b07cdd5: Downloading [====>                                              ]  6.998MB/76.1MB
  34192d21d543: Downloading [==============================================>    ]  133.5MB/144.7MB
  03f9faaa5b3f: Download complete
  595b59cf74c5: Downloading [==================>                                ]  10.01MB/26.45MB
  a8571fe826d1: Waiting
  ~~~

* 删除docker ce

  ![image-20220328222552243](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pyqul85xj21ne0u07be.jpg)

  ~~~shell
  [root@kubeadm-50 docker]# sudo yum remove docker-ce docker-ce-cli containerd.io
  已加载插件：fastestmirror
  正在解决依赖关系
  --> 正在检查事务
  ---> 软件包 containerd.io.x86_64.0.1.5.11-3.1.el7 将被 删除
  ---> 软件包 docker-ce.x86_64.3.18.09.9-3.el7 将被 删除
  ---> 软件包 docker-ce-cli.x86_64.1.18.09.9-3.el7 将被 删除
  --> 解决依赖关系完成
  
  依赖关系解决
  
  ========================================================================================================================================
   Package                         架构                     版本                                源                                   大小
  ========================================================================================================================================
  正在删除:
   containerd.io                   x86_64                   1.5.11-3.1.el7                      @docker-ce-stable                   106 M
   docker-ce                       x86_64                   3:18.09.9-3.el7                     @docker-ce-stable                    90 M
   docker-ce-cli                   x86_64                   1:18.09.9-3.el7                     @docker-ce-stable                    72 M
  
  事务概要
  ========================================================================================================================================
  移除  3 软件包
  
  安装大小：269 M
  是否继续？[y/N]：y
  Downloading packages:
  Running transaction check
  Running transaction test
  Transaction test succeeded
  Running transaction
  未将 /usr/bin/dockerd 配置为 dockerd 的备用项
    正在删除    : 3:docker-ce-18.09.9-3.el7.x86_64                                                                                    1/3
    正在删除    : containerd.io-1.5.11-3.1.el7.x86_64                                                                                 2/3
    正在删除    : 1:docker-ce-cli-18.09.9-3.el7.x86_64                                                                                3/3
    验证中      : containerd.io-1.5.11-3.1.el7.x86_64                                                                                 1/3
    验证中      : 1:docker-ce-cli-18.09.9-3.el7.x86_64                                                                                2/3
    验证中      : 3:docker-ce-18.09.9-3.el7.x86_64                                                                                    3/3
  
  删除:
    containerd.io.x86_64 0:1.5.11-3.1.el7          docker-ce.x86_64 3:18.09.9-3.el7          docker-ce-cli.x86_64 1:18.09.9-3.el7
  
  完毕！
  [root@kubeadm-50 docker]#
  ~~~

  验证docker-ce是否被删除

  ~~~shell
  [root@kubeadm-50 docker]# docker
  -bash: /usr/bin/docker: 没有那个文件或目录
  [root@kubeadm-50 docker]#
  [root@kubeadm-50 docker]# cd /var/lib/docker
  [root@kubeadm-50 docker]# ls -al
  总用量 24
  drwx--x--x   14 root root   182 3月  28 10:22 .
  drwxr-xr-x.  28 root root  4096 7月  14 2021 ..
  drwx------    2 root root    24 7月  14 2021 builder
  drwx------    4 root root    92 7月  14 2021 buildkit
  drwx------    3 root root    78 3月  28 10:23 containers
  drwx------    3 root root    22 7月  14 2021 image
  drwxr-x---    3 root root    19 7月  14 2021 network
  drwx------  188 root root 16384 3月  28 10:23 overlay2
  drwx------    4 root root    32 7月  14 2021 plugins
  drwx------    2 root root     6 3月  28 10:22 runtimes
  drwx------    2 root root     6 7月  14 2021 swarm
  drwx------    2 root root     6 3月  28 10:23 tmp
  drwx------    2 root root     6 7月  14 2021 trust
  drwx------    2 root root    25 7月  14 2021 volumes
  [root@kubeadm-50 docker]#
  [root@kubeadm-50 docker]# cd /var/lib/containerd
  [root@kubeadm-50 containerd]# ls -al
  总用量 4
  drwx--x--x   9 root root  265 7月  14 2021 .
  drwxr-xr-x. 28 root root 4096 7月  14 2021 ..
  drwxr-xr-x   3 root root   20 7月  14 2021 io.containerd.content.v1.content
  drwx--x--x   2 root root   21 7月  14 2021 io.containerd.metadata.v1.bolt
  drwx--x--x   3 root root   18 3月  28 10:23 io.containerd.runtime.v1.linux
  drwx--x--x   2 root root    6 7月  14 2021 io.containerd.runtime.v2.task
  drwx------   3 root root   23 7月  14 2021 io.containerd.snapshotter.v1.native
  drwx------   3 root root   23 7月  14 2021 io.containerd.snapshotter.v1.overlayfs
  drwx------   2 root root    6 7月  14 2021 tmpmounts
  [root@kubeadm-50 containerd]#
  ~~~

  ~~~shell
  [root@kubeadm-50 lib]# sudo rm -rf /var/lib/docker
  [root@kubeadm-50 lib]# cd /var/lib/docker
  -bash: cd: /var/lib/docker: 没有那个文件或目录
  [root@kubeadm-50 lib]# sudo rm -rf /var/lib/containerd
  [root@kubeadm-50 lib]# cd /var/lib/containerd
  -bash: cd: /var/lib/containerd: 没有那个文件或目录
  [root@kubeadm-50 lib]#
  ~~~

  

* 其他的安装方法

  ![image-20220328223508294](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pz0i1jifj21n20u046b.jpg)

## 3. 通过kubeadm安装k8s集群



### 3.1 主机规划

| 主机名称(hostname) | IP地址         | 主机用途       | 软件环境                                                  |
| ------------------ | -------------- | -------------- | --------------------------------------------------------- |
| k8s-master-51      | 192.168.162.51 | k8s mater节点  | Centos7.9 , Dokcer-19.03.15 , kubeadm , kubelet , kubectl |
| k8s-node-61        | 192.168.162.61 | k8s worker节点 | Centos7.9 , Dokcer-19.03.15 , kubeadm, kubelet            |

* Kubernetes版本 1.22.3  Kubernetes 版本兼容: https://kubernetes.io/releases/version-skew-policy/

* Docker版本 19.03.15

**注意**:docker和kubelet的cgroup driver需要保持一致,kubelet默认使用systemd.



#### 3.1.1 安装CentOS

官网地址: https://www.centos.org/

![image-20220402101200994](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v5n78oc4j21oi0u0dsz.jpg)



![image-20220402101442636](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v5pmal5fj21nj0u07as.jpg)

![image-20220402101524996](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v5qdb0woj21o90u0aj4.jpg)

![image-20220402101633761](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v5rkd7mfj21nk0u0djw.jpg)

安装镜像下载地址: http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso

在VMware Fusion 安装CentOS

![image-20220402102355661](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v5z8wlsuj21hg0u00v0.jpg)

新建一台虚拟机

![image-20220402102333039](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v5zg21dqj21ha0u0gn3.jpg)

![image-20220402102522128](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v60y8xbxj21gr0u00v8.jpg)



![image-20220402102621658](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v7b5tyoaj21h70u0mzh.jpg)

选择已经下载的镜像

![image-20220402102713110](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v62nsbokj21he0u0tb9.jpg)

![image-20220402102834993](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v641ses2j21ha0u0dhy.jpg)

![image-20220402102914325](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v64q5rawj21he0u0tb8.jpg)

![image-20220402103424205](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6a52kq5j21h60u0tba.jpg)

![image-20220402103508213](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6az1z7qj21hh0u0tbd.jpg)

![image-20220402103606733](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6c0igp5j21hd0u0tb5.jpg)

![image-20220402103656287](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6cr1frcj21h30u076g.jpg)

![image-20220402103725660](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6d9kda0j21ha0u040y.jpg)



![image-20220402103806204](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6e08kudj21hp0u0wjs.jpg)

![image-20220402103845638](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6eoe9ryj21h50u00x3.jpg)

![image-20220402103946604](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6fq2azrj21hk0u078h.jpg)

![image-20220402104019449](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6gamdoej21gx0u0why.jpg)

![image-20220402104049597](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6gtcdqrj21he0u0jvi.jpg)

![image-20220402104125091](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6hfiiz8j21ha0u0784.jpg)

![image-20220402104241122](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6iv5i3cj21hg0u0whx.jpg)

![image-20220402104424393](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6kjohplj21hd0u00x3.jpg)

![image-20220402104507815](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6laue24j21hd0u00w5.jpg)

![image-20220402104642881](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6my7vooj21h40u0ae0.jpg)

![image-20220402104717472](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v79kp1b1j21gu0u0q8d.jpg)

![image-20220402104758565](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6oe7j4zj21hj0u0tcq.jpg)

![image-20220402104846231](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v797abhsj21ex0u0q6l.jpg)

![image-20220402104924426](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v78z7z3oj21hh0u0juq.jpg)

![image-20220402105125657](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v78r5qsbj21h00u0tc0.jpg)

![image-20220402105151040](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6scya4nj21240u075q.jpg)

![image-20220402105321908](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v78gr27cj21hr0u0mze.jpg)

![image-20220402105403842](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v788p93xj21hh0u0jtz.jpg)

![image-20220402105515220](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v6vtiwvzj21h40u0jtz.jpg)

![image-20220402105656388](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v77zxe65j21hc0u0n07.jpg)

![image-20220402105741395](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v77rfyk2j21gz0u0gor.jpg)

![image-20220402105845669](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v77nvq6aj21hi0u00wd.jpg)

![image-20220402105918864](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v77h5tztj21kd0u0wi3.jpg)

![image-20220402110050235](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v71mzszuj21gu0u0jxd.jpg)

![image-20220402110141961](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v773d6j6j21m10u0jus.jpg)



~~~shell
[root@k8s-master-51 ~]# vi /etc/ssh/sshd_config
~~~

![image-20220402110414059](https://tva1.sinaimg.cn/large/e6c9d24ely1h0v76piialj21hk0u0q4x.jpg)

### 3.2 安装kubeadm

https://kubernetes.io/docs/home/

![image-20220329211412452](https://tva1.sinaimg.cn/large/e6c9d24egy1h0r2apo6whj21nb0u0n3m.jpg)

#### 开始之前

安装文档地址: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

![image-20220401150903430](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8lpy2k5j21n90u0jzk.jpg)

#### 验证每个节点的MAC地址和product_uuid的唯一性

![image-20220401151322963](/Users/catface/Library/Application%20Support/typora-user-images/image-20220401151322963.png)



#### 检查网络适配

![image-20220401151547722](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8st2wx3j21ne0u0104.jpg)



#### 让iptables看到被桥接的流量

![image-20220401151733189](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8ufoiquj21nf0u045k.jpg)

为什么要配置流量到iptables?  地址: https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#network-plugin-requirements

![image-20220401151919435](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8w92sq0j21nq0u0ai8.jpg)

#### 检查需要的端口

![image-20220401152158925](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u8z10cn8j21nf0u045p.jpg)

注意端口占用

![image-20220330223237154](https://tva1.sinaimg.cn/large/e6c9d24ely1h0sa6kvc3uj21nd0u0gqt.jpg)

#### 安装运行时环境

![image-20220401152458812](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u925pwhrj21nh0u0jz0.jpg)

container runtimes: https://kubernetes.io/docs/setup/production-environment/container-runtimes/

![image-20220401152713462](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u94hit05j21nn0u0wlf.jpg)



![image-20220401153007396](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u98d0bsaj21no0u0ahn.jpg)



#### 安装kubeadm,kubelet,kubectl

![image-20220401153445237](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9ccaeltj21nn0u0jz8.jpg)

![image-20220329212622955](https://tva1.sinaimg.cn/large/e6c9d24egy1h0r2nec8hyj21na0u0wm9.jpg)



#### 配置一个cgroup driver

![image-20220401154808825](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9q8roy9j21ni0u0dmb.jpg)

配置cgroup: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/

![image-20220401153954033](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9hoe0zsj21n70u0dmo.jpg)

### 3.3 故障排查

![image-20220401155345894](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9w9r521j21n30u0wn4.jpg)



遇到的问题举例:

![image-20220401155754776](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ua0eyfrxj21nk0u046q.jpg)



![image-20220401155555531](https://tva1.sinaimg.cn/large/e6c9d24ely1h0u9yc1n1dj21nd0u0dpl.jpg)



### 3.4 使用kubeadm创建集群

##### 前置准备

###### 删除交换分区

~~~bash
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

![image-20220329222228975](https://tva1.sinaimg.cn/large/e6c9d24ely1h0r49m9b3hj21cu0kcwht.jpg)

~~~shell
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

###### 关闭防火墙

~~~bash
[root@k8s-master-51 ~]# systemctl stop firewalld && systemctl disable firewalld
[root@k8s-master-51 ~]#

## 查看防火墙状态
[root@k8s-master-51 ~]# firewall-cmd --state
not running
[root@k8s-master-51 ~]#
~~~

###### 关闭SELinux

SELinux官方文档:  http://selinuxproject.org/page/Main_Page

SELinux策略: http://selinuxproject.org/page/NB_PolicyType

~~~shell
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

###### 配置同步时间(可选)

~~~bash
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

###### 将桥接的IPv4,IPv6流量传递到iptables

iptables: https://www.zsythink.net/archives/1199

挖个坑,后续填...

~~~bash
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

###### 安装Docker

~~~shell
## 安装yum-utils
sudo yum install -y yum-utils
 
 ## 配置docker yum 源
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

## 安装指定版本的docker
sudo yum install docker-ce-18.09.9 docker-ce-cli-18.09.9 containerd.io
  或者
sudo yum install docker-ce-19.03.15 docker-ce-cli-19.03.15 containerd.io

## 启动Docker
sudo systemctl start docker

## 验证Docker
docker info

sudo docker run hello-world

## 设置开机启动
sudo systemctl enable docker.service

sudo systemctl enable containerd.service

## 配置镜像加速
vim /etc/docker/daemon.json
## 写入

{
  "registry-mirrors":["https://hub-mirror.c.163.com/"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

## 或者

{
  "registry-mirrors":["https://hub-mirror.c.163.com/"]
}

## 重启Docker
sudo systemctl daemon-reload

sudo systemctl restart docker

## 再次检查配置是否生效
docker info
~~~



##### 安装kubeadm

首先检查下k8s元源配置是否存在

~~~shell
cat /etc/yum.repos.d/kubernetes.repo
~~~

官方文档提供的配置k8s源

~~~shell
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

我们改用阿里云的k8s源

~~~shell
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

执行安装命令

~~~shell
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



##### 初始化集群

参考文档地址: https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

###### 提前准备所需的容器镜像

![image-20220401162924933](https://tva1.sinaimg.cn/large/e6c9d24ely1h0uax6w45ij21nl0u0ajc.jpg)

使用自定义的镜像配置

参考文档: https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#custom-images

![image-20220401163713860](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ub5cbfg5j21nc0u0aix.jpg)

###### 初始化控制面板节点

![image-20220401164004431](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ub8ep746j21n60u0akd.jpg)

######  apiserver-advertise-address 和 ControlPlaneEndpoint 的注意事项

![image-20220401164512482](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ubdmp8j1j21ni0u0gub.jpg)



###### 更多信息

![image-20220401164641222](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ubf671b8j21mb0u0wmo.jpg)

![image-20220401171830454](/Users/catface/Library/Application%20Support/typora-user-images/image-20220401171830454.png)

部署Pod newwork

参考文档地址: https://kubernetes.io/docs/concepts/cluster-administration/addons/

![image-20220401165344305](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ubmijt0nj21nd0u011u.jpg)



canal:  https://github.com/projectcalico/canal/blob/master/k8s-install/canal.yaml

Calico: https://projectcalico.docs.tigera.io/about/about-calico

![image-20220401170344252](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ubx845kej21nr0u010y.jpg)



部署文件地址: https://projectcalico.docs.tigera.io/manifests/tigera-operator.yaml

![image-20220401170508035](https://tva1.sinaimg.cn/large/e6c9d24egy1h0ubyeytmwj21n80u0jwg.jpg)



flannel地址: https://github.com/flannel-io/flannel#deploying-flannel-manually

![image-20220401170858666](https://tva1.sinaimg.cn/large/e6c9d24egy1h0uc2k9hahj21n40u07a1.jpg)

部署文件地址: https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

或者gitee: https://gitee.com/catface996/flannel/blob/master/kube-flannel.yml

![image-20220401171035881](https://tva1.sinaimg.cn/large/e6c9d24egy1h0uc48qtt1j21n40u0tc9.jpg)

使用到的镜像

![image-20220401171238313](https://tva1.sinaimg.cn/large/e6c9d24ely1h0uc6aabzfj21n70u0gpi.jpg)

~~~yaml
## 镜像可以提前准备
image: rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1
image: rancher/mirrored-flannelcni-flannel:v0.17.0
~~~



~~~shell
docker pull rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1
docker pull rancher/mirrored-flannelcni-flannel:v0.17.0
~~~



###### 加入其他节点

![image-20220401172100927](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ucew7o66j21n50u0n3x.jpg)

如果token和hash未保存,可以通过一下命令获取

![image-20220401212940167](https://tva1.sinaimg.cn/large/e6c9d24ely1h0ujlnfxfej21n90u0dml.jpg)

~~~shell
kubeadm token list
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
   openssl dgst -sha256 -hex | sed 's/^.* //'
~~~



###### 初始化集群

~~~shell
## 执行进行拉取命令,提前准备镜像
kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers  --kubernetes-version=1.22.3
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



查看集群节点状态

~~~shell
## 在集群的网络插件未完成安装之前,coredns一致会处于pending状态.
kubectl get pod --all-namespaces

kubectl get node -n kube-system
~~~



再次查看kubelet的状态

~~~shell
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

4月 01 21:12:19 k8s-master-61 kubelet[15569]: E0401 21:12:19.347018   15569 kubelet.go:2337] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady m...uninitialized"
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



安装calico网络 或者 flannel网络

~~~shell
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



查看kube-system空间下的pod状态

~~~shell
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



查看node节点状态

~~~shell
[root@kubeadm-50 ~]# kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
kubeadm-50   Ready    control-plane,master   21m   v1.22.3
~~~



加入其他节点

~~~shell
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





## 挖坑记录

containerd是什么鬼?

github:  https://github.com/containerd/containerd

官网: https://containerd.io/



CRI-O 这又是个什么鬼?

github:  https://github.com/cri-o/cri-o



继续挖坑,等待后续视频填坑...
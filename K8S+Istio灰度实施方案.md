



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

系统要求和安装版本:

* 操作系统 Centos 7.x
* docker版本: Server Version: 18.09.9

安装步骤:

* 进入官网,下载指定版本的docker安装包.官网地址: https://www.docker.com/

  ![image-20220328211018117](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwk7xkuzj21nh0u0tcs.jpg)

* 点击右上角"Get Started",进入选择平台Docker Desktop版本页面

  ![image-20220328211114474](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwl7zyrxj21ne0u0tcw.jpg)

* 点击"View Linux Engine",进入选择Linux平台页面.我的虚拟机是Centos7.x,内核版本是3.10.0-1160.31.1.el7.x86_64,所以选择Server,Linux,x86-64

  ~~~shell
  [root@kubeadm-50 containerd]# cat /etc/centos-release
  CentOS Linux release 7.9.2009 (Core)
  
  [root@kubeadm-50 containerd]# uname -r
  3.10.0-1160.31.1.el7.x86_64
  ~~~

  ![image-20220328211325821](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pwnlj3mgj21nd0u0426.jpg)

* 选择"Docker Engine - CentOS",进入安装Docker引导页面.

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

  ![image-20220328215726558](https://tva1.sinaimg.cn/large/e6c9d24ely1h0pxx9uq2fj21nh0u046i.jpg)

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
  ~~~

* 安装指定版本的Docker

  ![image-20220328220853090](https://tva1.sinaimg.cn/large/e6c9d24ely1h0py95p8ctj21nh0u0114.jpg)

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
  [root@kubeadm-50 docker]# systemctl enable docker
  Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
  [root@kubeadm-50 docker]#
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

  ```
  {"registry-mirrors":["https://hub-mirror.c.163.com/"]}
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

* 
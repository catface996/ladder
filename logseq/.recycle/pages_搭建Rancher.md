-
-
- sudo curl -L "https://github.com/docker/compose/releases/download/1.26.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- "insecure-registries":["192.168.0.140:9090"]
- Pw4SifNg88GCWg9D
- 192.168.0.140:9090
- {
    "registry-mirrors": ["https://81ug0epr.mirror.aliyuncs.com"],
    "insecure-registries":["192.168.0.140:9090"]
  }
- docker run --name jenkins3 -p 80:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker:/usr/bin/docker -v /etc/docker:/etc/docker --privileged --user root --restart always -d jenkins/jenkins:lts
- sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged  --user=root rancher/rancher:v2.5.3
- sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged  --user=root rancher/rancher
- sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:v2.5.3
- ux74gDw8kX6aCPoM
- sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.3 --server http://192.168.0.138 --token mxjvsb2kbgjws9zvznk9mpmbzjjnzq76blz5r4pgkk7md5vn8vlp64 --ca-checksum f8214515c7d12a3b8348c05dd56aebea7e8f8c60cf4b7ff5ba5c5ccfd54245a5 --etcd
- sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.3 --server http://192.168.0.138 --token lt52gx8vjcrwnc7bk2pb5fr8mdl26hdv7kfblfhpvqq9fns6g8l4t8 --ca-checksum 1b6ec7ef4e194918172d44505f4212ba3b8ad21919e4c2dd82b987b762839d26 --etcd --controlplane --worker
- sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.5.3 --server https://116.62.152.187 --token 8j4m6jwfn6h7rscmf97mc5sfghnn88f7c4zlpb4rdgmclwmvngk4bk --ca-checksum 80d82c118fe711082e370d8f51e226b956d711f66665e57f9a851e888bdc376b --etcd --controlplane --worker
- docker stop $(docker ps -aq)
  docker rm $(docker ps -aq)
- #首先同步时间,这个在集群环境中非常重要
- yum -y install ntp
  systemctl enable ntpd
  systemctl start ntpd
  timedatectl set-ntp yes
  ntpdate -u cn.pool.ntp.org
  ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
- docker rm -f $(sudo docker ps -aq)
  docker volume rm $(sudo docker volume ls -q)
- rm -rf /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/pods \
       /var/run/calico
- rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/kube-audit \
       /var/log/pods \
       /var/run/calico
- for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
  for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do umount $mount; done
- rm -f /var/lib/containerd/io.containerd.metadata.v1.bolt/meta.db
  sudo systemctl restart containerd
  sudo systemctl restart docker
- docker logs etcd
- docker pull 192.168.0.140:9090/sbc-service/service-customer@sha256:12dff21020703749b01e20119778fd88fddf7bea650dc74a08a477bfdced0265
- sh bin/startup.sh -m standalone &
- http://bitbucket.xywl.cn/scm/panther/service-douyin-order.git
- C8_hqfhd4k2owJsZ
- bnz7mZuhsv2SvfmU
- GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
- FLUSH PRIVILEGES;
- registry-vpc.cn-hangzhou.aliyuncs.com
-
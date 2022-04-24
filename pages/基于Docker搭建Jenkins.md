- ## ⒈下载镜像
  
  要使用最新的LTS:
  
  ```shell
  docker pull jenkins/jenkins:lts
  ```
  
  要使用最新的每周
  
  ```shell
  docker pull jenkins/jenkins
  ```
## ⒉运行

```shell
## 创建挂载目录
cd /var
mkdir jenkins_home
sudo chown -R 1000 jenkins_home/
## 运行jenkins
docker run --name jenkins -p 80:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home --restart always -d jenkins/jenkins:lts
```
## ⒊访问Jenkins实例

1.访问http://192.168.227.128:80

2.获取初始登录密码

```shell
## 进入容器查看,或者在挂载目录中查看
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

从输出结果中获得的一串 Jenkins 初始密码，复制密码，回到 http://192.168.227.128:80 填入密码。

3.定制 Jenkins

选择默认的 **Install suggested plugins（安装推荐的插件）** 来安装插件。

4.创建帐户

根据引导页面的信息创建第一个管理员用户，之后即可开启 Jenkins 的世界。
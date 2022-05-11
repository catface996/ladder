- 官方简述
	- Nginx [ engine x ]是一个 HTTP 和反向代理服务器、一个邮件代理服务器和一个通用的 TCP/UDP 代理服务器，最初由 Igor Sysoev 编写。很长一段时间以来，它一直在包括 Yandex、 Mail.Ru、 VK 和 Rambler 在内的许多俄罗斯网站上运行。根据 Netcraft 的数据，nginx 在2022年4月为21.79% 的旅客提供服务。以下是一些成功的案例: Dropbox，Netflix，Wordpress.com，FastMail.FM。
- ## 下载安装包
  
  http://nginx.org/en/download.html
  
  选择合适的安装包,本教程选择 [nginx-1.14.2](http://nginx.org/download/nginx-1.14.2.tar.gz)
## 裸机安装

* 安装 gcc c++

~~~shell
yum install gcc
~~~

* 安装pcre

~~~shell
yum install -y pcre pcre-devel
~~~

* 安装openssl

~~~shell
yum install -y openssl openssl-devel
~~~

* 安装zlib

~~~shell
yum install -y zlib zlib-devel
~~~

* 或者一次安装 

~~~shell
yum -y install gcc openssl openssl-devel pcre-devel zlib zlib-devel
~~~

* 下载安装包到/usr/local,解压缩

~~~shell
cd /usr/local
wget http://nginx.org/download/nginx-1.14.2.tar.gz
tar -xvzf nginx-1.14.2.tar.gz
~~~

* 编译安装

~~~shell
cd /usr/local/nginx-1.14.2
./configure --prefix=/usr/local/nginx
make
make install  ##安装到/usr/local/nginx目录下
~~~

* 启动nginx

~~~ shell
cd /usr/local/nginx
./sbin/nginx -c ./conf/nginx.conf
~~~

* 检查nginx是否启动

~~~shell
[root@catface nginx]# ps -ef | grep nginx
root      10533      1  0 05:04 ?        00:00:00 nginx: master process ./sbin/nginx -c ./conf/nginx.conf
nobody    10534  10533  0 05:04 ?        00:00:00 nginx: worker process
root      10537   1470  0 05:06 pts/0    00:00:00 grep --color=auto nginx
~~~

* 访问nginx的默认端口

~~~shell
[root@catface nginx]# curl http://localhost:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
body {
    width: 35em;
    margin: 0 auto;
    font-family: Tahoma, Verdana, Arial, sans-serif;
}
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
~~~

* 修改配置后重新加载

~~~shell
./sbin/nginx -s reload -c ./conf/nginx.conf
~~~
## Docker安装
## 安装docker

略
## 安装nginx

* 拉取镜像

~~~shell
docker pull nginx:1.14.2
~~~

* 默认启动

~~~shell
docker run --name nginx -d -p 80:80 nginx:1.14.2
~~~

* 进入容器查看配置

~~~ shell
## docker exec -it <容器ID> bash

[root@catface nginx]# docker exec -it 9e944d5dfcf9 bash
root@9e944d5dfcf9:/# cd /etc/nginx/
root@9e944d5dfcf9:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf

root@9e944d5dfcf9:/etc/nginx# cd /usr/share/nginx/
root@9e944d5dfcf9:/usr/share/nginx# ls
html

root@9e944d5dfcf9:~# cd /var/log/nginx/
root@9e944d5dfcf9:/var/log/nginx# ls
access.log  error.log
root@9e944d5dfcf9:/var/log/nginx#

## 注意,日志文件access.log 和 error.log 需要在本地删除后重建
~~~

* 复制docker的配置文件和htm到本地

~~~shell
docker cp nginx:/etc/nginx /etc/nginx
docker cp nginx:/uar/share/html /usr/loacl/share
docker cp nginx:/var/log/nginx /var/log/nginx
~~~

* 停止默认配置的nginx,并删除容器

~~~shell
[root@catface nginx]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                NAMES
9e944d5dfcf9   nginx:1.14.2   "nginx -g 'daemon of…"   16 minutes ago   Up 16 minutes   0.0.0.0:80->80/tcp   nginx
[root@catface nginx]# docker stop 9e944d5dfcf9
9e944d5dfcf9
[root@catface nginx]# docker rm 9e944d5dfcf9
9e944d5dfcf9
[root@catface nginx]#
~~~

* 重新启动nginx,并挂载配置

~~~shell
[root@catface nginx]#  docker run --name nginx -d -p 80:80 -v /etc/nginx:/etc/nginx -v /usr/share/nginx/html:/usr/share/nginx/html -v /var/log/ngxin:/var/log/nginx nginx:1.14.2
27c854ca51a68324e74148554951dc5692609e6d5a2a7044ff4cc271eae15388
~~~

* 访问首页

~~~shell
curl http://localhost:80
~~~
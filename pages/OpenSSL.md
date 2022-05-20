- CentOS 7 安装 OpenSSL
	- ```shell
	  yum install openssl openssl-devel -y
	  ```
- 生成私钥
	- ```shell
	  openssl genrsa -out rsa-1024-private-key.key 2048
	  ```
	- -out 指定生成的私钥文件名
	- 1024 是私钥的长度，可以指定 256 ，512 ， 1024 ，2048
- 生成公钥
	- ```shell
	  openssl rsa -in rsa-1024-private-key.key -pubout -out rsa-1024-public-key.pem
	  ```
-
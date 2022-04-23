

### 安装jdk1.8

~~~bash
[root@hanma-116 local]# ln -s jdk1.8.0_271/ java
[root@hanma-116 local]# vim /etc/profile  添加以下内容

## java config
export JAVA_HOME=/usr/local/java
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

[root@hanma-116 local]# source /etc/profile
[root@hanma-116 local]# java -version
java version "1.8.0_271"
Java(TM) SE Runtime Environment (build 1.8.0_271-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.271-b09, mixed mode)

~~~



### 下载安装包

[elasticsearch-7.6.2](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.2-linux-x86_64.tar.gz)

[kibana-7.6.2](https://artifacts.elastic.co/downloads/kibana/kibana-7.6.2-linux-x86_64.tar.gz)

### 添加用户

elasticsearch 不允许用root用户启动

* 创建用户

~~~bash
adduser app
~~~

* 创建es的密码, 不写es修改的是root用户的密码

~~~bash
passwd app
~~~

* 增加 sudoers 文件的写的权限，默认为只读

~~~bash
chmod -v u+w /etc/sudoers
~~~

* 修改 sudoers

~~~bash
vi /etc/sudoers

root    ALL=(ALL)       ALL
app     ALL=(ALL)       ALL （添加这一行）
~~~

### 常见问题

* max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

~~~bash
vim /etc/security/limits.conf  追加以下内容；

* soft nofile 65536
* hard nofile 65536
~~~

* max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

~~~bash
vim /etc/sysctl.conf   文件最后添加一行

vm.max_map_count=262144
~~~

### 修改配置

~~~bash
vim config/elasticsearch.yml
~~~

~~~yaml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
# 此处需要修改
cluster.name: hanma
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
# 此处需要修改
node.name: node-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
# 此处需要修改
path.data: /usr/local/elasticsearch/data
#
# Path to log files:
# 此处需要修改
path.logs: /usr/local/elasticsearch/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
# bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
# 此处需要修改
network.host: 192.168.168.111
#
# Set a custom port for HTTP:
# 此处需要修改
http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
# 此处需要修改
discovery.seed_hosts: ["192.168.168.111", "192.168.168.112","192.168.168.113"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
# 此处需要修改
cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
# 此处需要修改
gateway.recover_after_nodes: 2
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
~~~

### 启动集群

逐台启动

~~~bash
[root@hanma-111 local]# chown -R app:app elasticsearch*   更改elasticsearch的目录地址
[root@hanma-111 local]# su app
[root@hanma-111 local]# cd elasticsearch
[app@hanma-111 elasticsearch]# ./bin/elasticsearch -d
[app@hanma-111 local]# tail logs/hanma.log -f -n 200
~~~

### 验证集群

http://192.168.168.111:9200/_cat/nodes?v

http://192.168.168.112:9200/_cat/nodes?v

http://192.168.168.113:9200/_cat/nodes?v

```html
ip              heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
192.168.168.111           26          69   1    0.09    0.11     0.13 dilm      *      node-1
192.168.168.112           18          64   1    0.02    0.05     0.08 dilm      -      node-2
192.168.168.113           32          65   0    0.03    0.24     0.22 dilm      -      node-3
```

### 启用密码

 编辑elasticsearch.yml文件,向其中添加

```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

执行以下命令:

~~~bash
./bin/elasticsearch-setup-passwords interactive
~~~

设置完成后,登录Kibana的账户就是kibana,elasticsearch的账户为elastic.



### 安装Kibana(Docker)

```bash
docker pull kibana:7.6.2
docker run --name kibana -p 5601:5601 -d kibana:7.6.2
docker exec -it {containerId} bash

vi config/kibana.yml
## 修改以下配置
elasticsearch.hosts: [ "http://192.168.168.111:9200","http://192.168.168.112:9200","http://192.168.168.113:9200" ]

# 重启容器
docker restart {containerId}
```

使用elastic密码

~~~yml
elasticsearch.username: "elastic"
elasticsearch.password: "xxxxx"   # 之前在es中设置的密码
~~~

### 安装Logstash(Docker)

~~~bash
docker run --name logstash -p 5044:5044 -p 9600:9600 -d logstash:7.6.2
~~~

### Filebeat写入Elasticsearch


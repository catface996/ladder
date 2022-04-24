## 安装JDK1.8

略
## 安装ZK

apache-zookeeper-3.6.2-bin 略
## 安装Kafka
### 下载kafka安装包

目前使用的版本是2.11 https://archive.apache.org/dist/kafka/2.1.1/kafka_2.11-2.1.1.tgz
### 主机规划

| 主机IP         | 主机名   |
| :------------- | :------- |
| 192.168.168.71 | hanma-71 |
| 192.168.168.72 | hanma-72 |
| 192.168.168.73 | hanma-73 |
### 修改配置

解压好之后，修改配置文件/config/server.properties，原配置为：

broker.id=0

zookeeper.connect=localhost:2181

修改为：

broker.id=0

listeners=PLAINTEXT://192.168.168.71:9092

advertised.listeners=PLAINTEXT://192.168.168.71:9092

log.dirs=/var/logs/kafka-logs

zookeeper.connect=192.168.168.102:2181,192.168.168.103:2181,192.168.168.104:2181,192.168.168.105:2181

*  这里需要说明的是，默认Kafka会使用ZooKeeper默认的/路径，这样有关Kafka的ZooKeeper配置就会散落在根路径下面，如果你有其他的应用也在使用ZooKeeper集群，查看ZooKeeper中数据可能会不直观，所以强烈建议指定一个chroot路径，直接在zookeeper.connect配置项中指定。

* 三台机器各自吸怪borker.id,分别为1,2,3,以此类推
### 启动kafka

~~~bash
./kafka-server-start.sh -daemon /usr/local/kafka/config/server.properties &
~~~
### 创建topic

~~~bash
./kafka-topics.sh --create --topic test-topic --replication-factor 3 --partitions 6 --zookeeper 192.168.168.102:2181
~~~
### 查看创建的topic

~~~bash
./kafka-topics.sh --list --zookeeper 192.168.168.102:2181
~~~
### 向topic投递消息

~~~bash
./kafka-console-producer.sh --broker-list 192.168.168.71:9092 --sync --topic test-topic this is for test
~~~
### 消费消息

~~~bash
./kafka-console-consumer.sh --bootstrap-server 192.168.168.71:9092 --topic test-topic --from-beginning
~~~
## 安装KafkaManager

项目地址:  https://github.com/yahoo/CMAK 下载地址: https://github.com/yahoo/CMAK/releases

* Docker 安装

```bash
docker run -d -eZK_HOSTS=192.168.168.102 -eKAFKA_MANAGER_AUTH_ENABLED=true kafkamanager/kafka-manager
```
## Kafka 开启JMX

在kafka-server-start.sh 添加一条 export JMX_PORT=9988 ,重启kafka broker即可.
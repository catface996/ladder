# 部署rancher 2.5.12 验证isto蓝绿灰度
## 前提

调用链在传播过程中,需要保持header中特定变量的传递.有以下集中场景:

* http请求  rpc调用如果是feignClient,需要对feignClient加强.
* 定时任务 可以以定时任务所在的环境值,构建到http请求的header中.
* 消息 可以以消费消息的应用所在的环境,构建到http请求的header中.
## sayHello代码

~~~java
package com.example.istio.controller;

import javax.servlet.http.HttpServletRequest;

import com.example.istio.api.DemoApi;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author by 大猫
 * @date 2022/2/22 5:43 PM
 * <p>
 * Copyright 2021  北京交个朋友数码科技有限公司, Inc. All rights reserved.
 */
@Slf4j
@RestController
public class DemoController {

@Value("${spring.application.name}")
private String appName;

@Value("${next}")
private Boolean next;

@Value("${env}")
private String env;

private DemoApi demoApi;

@ResponseBody
@GetMapping(value = "/sayHello")
public String sayHello(HttpServletRequest request) {
    String headerEnv = request.getHeader("env");
    String headerInfo = "header env(" + headerEnv + ")";
    String current = headerInfo + ", I'm " + appName + "(" + env + ")";
    log.info(current);
    String response = "";
    if (next) {
        response = demoApi.sayHello();
    }
    return current + " --> " + response;
}

@Autowired
public void setDemoApi(DemoApi demoApi) {
    this.demoApi = demoApi;
}
}
~~~
## DemoApi代码

~~~java
package com.example.istio.api;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

/**
 * @author by 大猫
 * @date 2022/2/22 8:38 PM
 * <p>
 * Copyright 2021  北京交个朋友数码科技有限公司, Inc. All rights reserved.
 */
@FeignClient(url = "${service.url}", name = "demoApi", fallbackFactory = DemoApiFallback.class)
public interface DemoApi {

/**
 * 打招呼接口
 *
 * @return 打招呼的回应
 */
@GetMapping(value = "/sayHello")
String sayHello();
}

~~~
## 验证规划

* cat服务调用dog服务
* cat服务有gray,prod
* dog服务有daily,gray,prod
* 通过http请求中的header进行匹配,例如,header中的env=gray,则匹配cat,dog的gray环境
* 如果不能匹配,默认使用prod
## 预期结果

header 中携带env=prod时:  cat(prod)->dog(prod)

header中携带env=gray时: cat(gray)->dog(gray)

header中携带env=daily时: cat(prod)->dog(daily)

header中不携带env时: cat(prod)->dog(prod)
## 主机规划

rancher-1 192.168.162.101  rancher-service

rancher-2 192.168.162.102 k8s  k8s-node

rancher-3 192.168.162.103 k8s  k8s-node

rancher-4 192.168.162.104 k8s  k8s-node
## 安装rancher 

docker run -d --privileged --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher:v2.5.12

在rancher中创建k8s集群,并开启istio
## 自定义镜像

zlyxzq/istio-demo-service-istio:latest
## 环境变量(SPRING_OPTS)

~~~properties
## 通过以下5个环境变量,分别激活5个工作负载,
--spring.application.name=cat --server.port=80 --env=gray --next=true --service.url=http://dog-svc
--spring.application.name=cat --server.port=80 --env=prod --next=true --service.url=http://dog-svc

--spring.application.name=dog --server.port=80 --env=daily --next=false --service.url=http://xxxx
--spring.application.name=dog --server.port=80 --env=gray --next=false --service.url=http://xxxx
--spring.application.name=dog --server.port=80 --env=prod --next=false --service.url=http://xxxx
~~~
## 常规配置

在常规k8s集群中创建工作负载和服务发现.注意在配置工作负载时,要对pod打标签,方便配置服务发现.
### 配置工作负载

![image-20220224232649630](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp0oga4gkj228n0u0q92.jpg)

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
annotations:
deployment.kubernetes.io/revision: "3"
field.cattle.io/creatorId: user-qsffg
field.cattle.io/publicEndpoints: "null"
creationTimestamp: "2022-02-24T14:13:12Z"
generation: 5
labels:
cattle.io/creator: norman
workload.user.cattle.io/workloadselector: deployment-default-cat-gray
managedFields:
- apiVersion: apps/v1
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      .: {}
      f:field.cattle.io/creatorId: {}
      f:field.cattle.io/publicEndpoints: {}
    f:labels:
      .: {}
      f:cattle.io/creator: {}
      f:workload.user.cattle.io/workloadselector: {}
  f:spec:
    f:progressDeadlineSeconds: {}
    f:replicas: {}
    f:revisionHistoryLimit: {}
    f:selector: {}
    f:strategy:
      f:rollingUpdate:
        .: {}
        f:maxSurge: {}
        f:maxUnavailable: {}
      f:type: {}
    f:template:
      f:metadata:
        f:annotations:
          .: {}
          f:cattle.io/timestamp: {}
          f:field.cattle.io/ports: {}
        f:labels:
          .: {}
          f:app: {}
          f:env: {}
          f:workload.user.cattle.io/workloadselector: {}
      f:spec:
        f:containers:
          k:{"name":"cat-gray"}:
            .: {}
            f:env:
              .: {}
              k:{"name":"SPRING_OPTS"}:
                .: {}
                f:name: {}
                f:value: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:resources: {}
            f:securityContext:
              .: {}
              f:allowPrivilegeEscalation: {}
              f:capabilities: {}
              f:privileged: {}
              f:readOnlyRootFilesystem: {}
              f:runAsNonRoot: {}
            f:stdin: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
            f:tty: {}
        f:dnsPolicy: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
manager: rancher
operation: Update
time: "2022-02-24T14:59:57Z"
- apiVersion: apps/v1
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      f:deployment.kubernetes.io/revision: {}
  f:status:
    f:availableReplicas: {}
    f:conditions:
      .: {}
      k:{"type":"Available"}:
        .: {}
        f:lastTransitionTime: {}
        f:lastUpdateTime: {}
        f:message: {}
        f:reason: {}
        f:status: {}
        f:type: {}
      k:{"type":"Progressing"}:
        .: {}
        f:lastTransitionTime: {}
        f:lastUpdateTime: {}
        f:message: {}
        f:reason: {}
        f:status: {}
        f:type: {}
    f:observedGeneration: {}
    f:readyReplicas: {}
    f:replicas: {}
    f:updatedReplicas: {}
manager: kube-controller-manager
operation: Update
time: "2022-02-24T15:06:27Z"
name: cat-gray
namespace: default
resourceVersion: "19617"
uid: 385def1f-0cda-4e1e-adbc-d59067ac303a
spec:
progressDeadlineSeconds: 600
replicas: 1
revisionHistoryLimit: 10
selector:
matchLabels:
  workload.user.cattle.io/workloadselector: deployment-default-cat-gray
strategy:
rollingUpdate:
  maxSurge: 1
  maxUnavailable: 0
type: RollingUpdate
template:
metadata:
  annotations:
    cattle.io/timestamp: "2022-02-24T15:06:13Z"
    field.cattle.io/ports: '[[]]'
  creationTimestamp: null
  labels:
    app: cat
    env: gray
    workload.user.cattle.io/workloadselector: deployment-default-cat-gray
spec:
  containers:
  - env:
    - name: SPRING_OPTS
      value: --spring.application.name=cat --server.port=80 --env=gray --next=true
        --service.url=http://dog-svc
    image: zlyxzq/istio-demo-service-istio:latest
    imagePullPolicy: Always
    name: cat-gray
    resources: {}
    securityContext:
      allowPrivilegeEscalation: false
      capabilities: {}
      privileged: false
      readOnlyRootFilesystem: false
      runAsNonRoot: false
    stdin: true
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    tty: true
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  terminationGracePeriodSeconds: 30
status:
availableReplicas: 1
conditions:
- lastTransitionTime: "2022-02-24T14:13:44Z"
lastUpdateTime: "2022-02-24T14:13:44Z"
message: Deployment has minimum availability.
reason: MinimumReplicasAvailable
status: "True"
type: Available
- lastTransitionTime: "2022-02-24T14:13:12Z"
lastUpdateTime: "2022-02-24T15:06:27Z"
message: ReplicaSet "cat-gray-858bf79f4d" has successfully progressed.
reason: NewReplicaSetAvailable
status: "True"
type: Progressing
observedGeneration: 5
readyReplicas: 1
replicas: 1
updatedReplicas: 1
~~~
### 配置服务发现

只能通过标签选择pod的方式

![image-20220224232828272](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp1okya0zj229q0rm43o.jpg)

~~~yaml
apiVersion: v1
kind: Service
metadata:
annotations:
field.cattle.io/creatorId: user-qsffg
field.cattle.io/ipAddresses: "null"
field.cattle.io/targetDnsRecordIds: "null"
field.cattle.io/targetWorkloadIds: "null"
creationTimestamp: "2022-02-24T13:54:10Z"
labels:
cattle.io/creator: norman
managedFields:
- apiVersion: v1
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      .: {}
      f:field.cattle.io/creatorId: {}
      f:field.cattle.io/ipAddresses: {}
      f:field.cattle.io/targetDnsRecordIds: {}
      f:field.cattle.io/targetWorkloadIds: {}
    f:labels:
      .: {}
      f:cattle.io/creator: {}
  f:spec:
    f:ports:
      .: {}
      k:{"port":80,"protocol":"TCP"}:
        .: {}
        f:name: {}
        f:port: {}
        f:protocol: {}
        f:targetPort: {}
    f:selector:
      .: {}
      f:app: {}
    f:sessionAffinity: {}
    f:sessionAffinityConfig:
      .: {}
      f:clientIP:
        .: {}
        f:timeoutSeconds: {}
    f:type: {}
manager: rancher
operation: Update
time: "2022-02-24T13:54:10Z"
name: cat-svc
namespace: default
resourceVersion: "5750"
uid: 2d47c790-2456-4bb7-a225-ff4075fd0eef
spec:
clusterIP: 10.43.107.197
clusterIPs:
- 10.43.107.197
ports:
- name: http
port: 80
protocol: TCP
targetPort: 80
selector:
app: cat
sessionAffinity: ClientIP
sessionAffinityConfig:
clientIP:
  timeoutSeconds: 10800
type: ClusterIP
status:
loadBalancer: {}

~~~
## ISTIO配置
### 配置网关

![image-20220224233130557](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp0t9eunmj22bg0negpf.jpg)

~~~yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
annotations:
field.cattle.io/creatorId: user-qsffg
creationTimestamp: "2022-02-24T14:49:13Z"
generation: 3
labels:
cattle.io/creator: norman
managedFields:
- apiVersion: networking.istio.io/v1alpha3
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      .: {}
      f:field.cattle.io/creatorId: {}
    f:labels:
      .: {}
      f:cattle.io/creator: {}
  f:spec:
    .: {}
    f:selector:
      .: {}
      f:istio: {}
    f:servers: {}
manager: rancher
operation: Update
time: "2022-02-24T14:49:13Z"
name: cat-gateway
namespace: default
resourceVersion: "21898"
uid: 7d14592c-ca55-4a5c-b47d-8bc3da04567d
spec:
selector:
istio: ingressgateway
servers:
- hosts:
- cat.catface.com
## 需配置具体的域名,或者*
port:
  name: http
  number: 80
  protocol: HTTP
~~~
### 配置目标规则

![image-20220224233251755](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp0uoeysoj22cg0nsae7.jpg)



~~~yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
annotations:
field.cattle.io/creatorId: user-qsffg
creationTimestamp: "2022-02-24T15:03:04Z"
generation: 3
labels:
cattle.io/creator: norman
managedFields:
- apiVersion: networking.istio.io/v1alpha3
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      .: {}
      f:field.cattle.io/creatorId: {}
    f:labels:
      .: {}
      f:cattle.io/creator: {}
  f:spec:
    .: {}
    f:host: {}
manager: rancher
operation: Update
time: "2022-02-24T15:03:04Z"
- apiVersion: networking.istio.io/v1alpha3
fieldsType: FieldsV1
fieldsV1:
  f:spec:
    f:subsets: {}
manager: Mozilla
operation: Update
time: "2022-02-24T15:03:27Z"
name: dog-rule
namespace: default
resourceVersion: "20355"
uid: 25805bea-b2e2-41dc-93fe-6667c6f68158
spec:
host: dog-svc
subsets:
- labels:
  env: prod
name: prod
- labels:
  env: gray
name: gray
- labels:
  env: daily
name: daily

~~~
### 配置虚拟服务

![image-20220224233356470](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp0vsmt4jj22bk0nijva.jpg)



~~~yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
annotations:
field.cattle.io/creatorId: user-qsffg
creationTimestamp: "2022-02-24T13:55:25Z"
generation: 11
labels:
cattle.io/creator: norman
managedFields:
- apiVersion: networking.istio.io/v1alpha3
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      .: {}
      f:field.cattle.io/creatorId: {}
    f:labels:
      .: {}
      f:cattle.io/creator: {}
  f:spec:
    .: {}
    f:gateways: {}
    f:http: {}
manager: rancher
operation: Update
time: "2022-02-24T14:53:16Z"
- apiVersion: networking.istio.io/v1alpha3
fieldsType: FieldsV1
fieldsV1:
  f:spec:
    f:hosts: {}
manager: Mozilla
operation: Update
time: "2022-02-24T14:54:50Z"
name: cat-vs
namespace: default
resourceVersion: "17279"
uid: bf5622f1-1e6f-4623-b626-c5aac93c46ec
spec:
gateways:
- cat-gateway
hosts:
- cat.catface.com
http:
- match:
- headers:
    env:
      exact: gray
route:
- destination:
    host: cat-svc
    port:
      number: 80
    subset: gray
  weight: 100
- match:
- headers:
    env:
      exact: prod
route:
- destination:
    host: cat-svc
    port:
      number: 80
    subset: prod
  weight: 100
- route:
- destination:
    host: cat-svc
    port:
      number: 80
    subset: prod
  weight: 100

~~~
### 通过k8s的Ingress对istio的gatewayIngress对外暴露

注意:这是在system的空间中完成的.

当然,也可以通过其他方式对外暴露,例如ng,阿里云的slb等.

![image-20220224233547208](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp0xr0ecmj22bi0negq8.jpg)

~~~yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
annotations:
field.cattle.io/creatorId: user-qsffg
field.cattle.io/ingressState: '{"Y2F0LmNhdGZhY2UuY29tL2lzdGlvLXN5c3RlbS9jYXQuY2F0ZmFjZS5jb20vL2h0dHAy":""}'
field.cattle.io/publicEndpoints: '[{"addresses":["192.168.162.102"],"port":80,"protocol":"HTTP","serviceName":"istio-system:istio-ingressgateway","ingressName":"istio-system:cat.catface.com","hostname":"cat.catface.com","allNodes":true}]'
creationTimestamp: "2022-02-24T13:51:03Z"
generation: 3
labels:
cattle.io/creator: norman
managedFields:
- apiVersion: networking.k8s.io/v1
fieldsType: FieldsV1
fieldsV1:
  f:status:
    f:loadBalancer:
      f:ingress: {}
manager: nginx-ingress-controller
operation: Update
time: "2022-02-24T13:51:11Z"
- apiVersion: extensions/v1beta1
fieldsType: FieldsV1
fieldsV1:
  f:metadata:
    f:annotations:
      .: {}
      f:field.cattle.io/creatorId: {}
      f:field.cattle.io/ingressState: {}
      f:field.cattle.io/publicEndpoints: {}
    f:labels:
      .: {}
      f:cattle.io/creator: {}
  f:spec:
    f:rules: {}
manager: rancher
operation: Update
time: "2022-02-24T13:51:11Z"
name: cat.catface.com
namespace: istio-system
resourceVersion: "14631"
uid: 67a8092c-f05c-4099-8f1c-0a5a52696082
spec:
rules:
- host: cat.catface.com
http:
  paths:
  - backend:
      serviceName: istio-ingressgateway
      servicePort: http2
    pathType: ImplementationSpecific
status:
loadBalancer:
ingress:
- ip: 192.168.162.102
- ip: 192.168.162.103
- ip: 192.168.162.104

~~~
## 对不同版本做认证
### 本地配置hosts

![image-20220224233901023](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp113c6ohj218g0qomyb.jpg)
### 通过postman发送不同请求
#### header中携带env=prod

![image-20220224234154374](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp1431wp4j222w0q0adg.jpg)
#### header中携带env=gray

![image-20220224234233567](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp14s24swj223m0p2djb.jpg)
#### header中携带env=daily

![image-20220224234324552](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp15o5tinj22400qmq6f.jpg)
#### header中不携带env

![image-20220224234408483](https://tva1.sinaimg.cn/large/e6c9d24ely1gzp16er0djj223g0pqdjl.jpg)
## 其他需要思考的点
### 日志的统一采集,便于排查问题

建议将日志统一存储在一个project中,便于通过traceId进行日志检索.
### 链路追踪及环境参数在链路中传播

需要融合springCloud和istio链路追踪,保持相同的traceId

建议使用spring-cloud-starter-sleuth来实现路由参数在调用链路中的传递,在3.1.1版本中,只需要增加以下配置即可实现sleuth自定义参数在调用链路中的传递.

~~~properties
## spring-cloud-starter-sleuth 3.1.1
spring.sleuth.baggage.remote-fields=env
~~~

sleuth已经支持链路追踪参数传递场景,例如feignClient,Executor,@Async,grpc等等,均可以在Trace中接收或者获取.

实现的效果如下:

postman --> cat --> dog

* postman发出当前请求

![image-20220225222952638](https://tva1.sinaimg.cn/large/e6c9d24ely1gzq4nik39zj22540q2dje.jpg)

* cat接收到请求

~~~shell
2022-02-25 21:35:46.853  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : env -- > xxx
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : user-agent -- > PostmanRuntime/7.29.0
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : accept -- > */*
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : postman-token -- > ffc3adfd-b406-48cc-9b0c-53c795245555
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : host -- > localhost:9001
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : accept-encoding -- > gzip, deflate, br
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : connection -- > keep-alive
2022-02-25 21:35:46.854  INFO [cat,716fd0d8d037ae48,716fd0d8d037ae48] 25032 --- [nio-9001-exec-2] c.e.istio.controller.DemoController      : header env(xxx), I'm cat(prod)
~~~

* dog收到的请求

~~~shell
2022-02-25 21:35:46.945  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : x-b3-traceid -- > 716fd0d8d037ae48
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : x-b3-spanid -- > d662ac0117fb6d36
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : x-b3-parentspanid -- > 716fd0d8d037ae48
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : x-b3-sampled -- > 0
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : env -- > xxx
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : accept -- > */*
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : user-agent -- > Java/11.0.14
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : host -- > localhost:9002
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : connection -- > keep-alive
2022-02-25 21:35:46.946  INFO [cat,716fd0d8d037ae48,d662ac0117fb6d36] 25022 --- [nio-9002-exec-1] c.e.istio.controller.DemoController      : header env(xxx), I'm cat(gray)
~~~
### 用户级的灰度,进入网关后,通过业务逻辑决定用户的灰度线路

网关可以根据业务规则,计算并改写接收到请求头中的env=?,以决定用户级的蓝绿灰度链路.
### 前端的灰度

包括前端页面的灰度和前端对后端请求的灰度.

* 同一套前端页面发出的请求,可以携带不同的header信息
* 同一个域名,可以请求到统一套前端页面,做不同的渲染展示
* 同一个域名,可以请求到不同的前端页面
### 异步线程如何保证header的穿透

主要出现在异步线程中,需要注意启用异步线程的方式,new Thread或者使用线程池,或者使用@Async注解

除继承父线程的环境参数外,当线程执行结束或者在执行新任务之前,要重新初始化环境参数.

如果采用sleuth,可以结合链路追踪参数来区别调用链的环境参数,避免重复造轮子.
## ISTIO 部署图



![image-20220301174018582](https://tva1.sinaimg.cn/large/e6c9d24egy1gzuiso3j5gj20pn0pmwg4.jpg)
-
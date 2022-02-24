# 阶梯



​		技术需要积累,需要实践,需要理性的衡量优劣.

​		方案比较的时优劣,是投入产出比,不是我觉得.列出优势和劣势的清单作比较,优先解决眼前的问题,同时在成本可控的前提下做好扩展性.例如,当前迭代的评估人天为20个人天,多花2人天做扩展性设计,很划算,如果需要花30个人天做扩展性设计,就不划算.



## 常见的凭感觉的黑话



* 这个方案有点重?  -- 衡量轻重的标准是什么? 经常涉及到的点有,锁,架构,业务域的拆分等

  example: synchronized比较重,请问怎么定义轻重,synchronized重在哪里呢?

  

* 这个方案风险有点大?  -- 有哪些风险? 业务风险和技术风险是否有预案.

  example: 微服务拆分,风险有点大,是否有验收标准,是否有有效的checkList? 做功能回归是否是有效控制风险的手段,脱离了检验标准谈风险,过于感性.



## 日拱一卒



* 浏览器是通过怎样的方式判断是否允许跨域的? nginx和后端服务同时设置了允许跨域,会不会导致跨域失效?
* nginx是否适合设置允许跨域? 如果换成负载均衡呢? 最适合解决跨域的一层是负载均衡,网关,还是bff层?



## 练练手

* [搭建Jenkins-基于docker](./基于Docker搭建Jenkins.md)
* [搭建elasticsearch集群](./elasticsearch.md)
* [搭建Kafka集群](./搭建kafka集群.md)
* [搭建K8s集群](./安装K8S.md)
* [搭建nginx](./nginx.md)
* [跨域验证](./跨域.md)
* [A * 算法](AStar算法.md)
* [取消推文转发](取消推文转发.md)
* [部署rancher-2.5.12验证istio蓝绿灰度](部署rancher-2.5.12验证istio蓝绿灰度.md)

- 设计目标
	- 三个应用
	  collapsed:: true
		- cat
			- prod（default）
			- gray
		- dog
			- prod（default）
		- monkey
			- prod（defualt）
			- gray
	- 应用的代代码介绍
		- 源码地址
		  collapsed:: true
			- https://github.com/catface996/istio-demo-app
		- 源码介绍
		  collapsed:: true
			- DemoController
			  collapsed:: true
				- ```java
				  package com.example.istio.controller;
				  
				  import java.util.Enumeration;
				  
				  import javax.servlet.http.HttpServletRequest;
				  
				  import brave.Tracer;
				  import com.example.istio.api.DemoApi;
				  import lombok.extern.slf4j.Slf4j;
				  import org.springframework.beans.factory.annotation.Autowired;
				  import org.springframework.beans.factory.annotation.Value;
				  import org.springframework.web.bind.annotation.GetMapping;
				  import org.springframework.web.bind.annotation.ResponseBody;
				  import org.springframework.web.bind.annotation.RestController;
				  
				  /**
				   * @author by 大猫
				   * @date 2022/2/22 5:43 PM catface996 出品
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
				  
				      @Autowired
				      private Tracer tracer;
				  
				      @ResponseBody
				      @GetMapping(value = "/sayHello")
				      public String sayHello(HttpServletRequest request) {
				          String headerEnv = request.getHeader("env");
				          Enumeration<String> headers = request.getHeaderNames();
				          while (headers.hasMoreElements()) {
				              String headerName = headers.nextElement();
				              log.info("{} -- > {}", headerName, request.getHeader(headerName));
				          }
				          String headerInfo = "header env(" + headerEnv + ")";
				          String current = headerInfo + ", I'm " + appName + "(" + env + ")";
				          log.info(current);
				          String response = "";
				          if (next) {
				              response = demoApi.sayHello();
				          }
				          String traceId = tracer.currentSpan().context().traceIdString();
				          return "TraceId: " + traceId + "  " + current + " --> " + response;
				      }
				  
				      @Autowired
				      public void setDemoApi(DemoApi demoApi) {
				          this.demoApi = demoApi;
				      }
				  }
				  ```
			- DemoApi
			  collapsed:: true
				- ```java
				  package com.example.istio.api;
				  
				  import org.springframework.cloud.openfeign.FeignClient;
				  import org.springframework.web.bind.annotation.GetMapping;
				  
				  /**
				   * @author by 大猫
				   * @date 2022/2/22 8:38 PM
				   * <p>
				   * Copyright 2021  catface996, Inc. All rights reserved.
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
				  ```
			- application.properties
			  collapsed:: true
				- ```properties
				  spring.application.name=cat
				  server.port=9001
				  next=false
				  env=prod
				  service.url=http://localhost:9002
				  spring.sleuth.baggage.remote-fields=env
				  ```
		- Deployment 配置介绍
		  collapsed:: true
			- ```yaml
			  apiVersion: apps/v1
			  kind: Deployment
			  metadata:
			    name: cat-dp-prod
			    annotations:
			      author: catface996
			  spec:
			    selector:
			      matchLabels:
			        app: cat
			        env: prod
			    replicas: 1
			    template:
			      metadata:
			        labels:
			          app: cat
			          env: prod
			      spec:
			        containers:
			          - name: cat-ct-prod
			            image: catface996/spring-cloud-istio-demo:latest
			            ports:
			              - containerPort: 8080
			                protocol: TCP
			            env:
			            - name: SPRING_OPTS
			              value: --spring.application.name=cat --server.port=8080 --env=gray --next=true --service.url=http://dog-svc
			            - name: JAVA_OPTS
			              value: -Xmx512M -Xms512M
			            resources:
			              requests:
			                cpu: "100m"
			                memory: "512Mi"
			              limits:
			                cpu: "200m"
			                memory: "1024Mi"
			  ```
		- Service 配置介绍
		- Destination Rule 配置介绍
		- Virtual Service 配置介绍
		- Gateway 配置介绍
	- 调用链路
		- ![image.png](../assets/image_1651462949155_0.png)
- 实施步骤
	- ((62638d0b-abac-4fa7-bb9a-8d08efa0a4f6))方便查看 pod 日志和 Istio 配置。
	- 部署 Deploymnet，并验证是否部署成功。
		- ```yaml
		  ```
	- 部署 Service，并验证部署是否成功。
		- ```yaml
		  ```
	- 绑定 cat-svc 到 istio-ingressgateway。
		- 部署 gateway
			- ```yaml
			  ```
		- 部署 cat-vs
			- ```yaml
			  ```
	- 本地配置 hosts，域名映射到 istio-ingressgate 所在主机 IP。
		- ![image.png](../assets/image_1651475170677_0.png)
		- ![image.png](../assets/image_1651475223671_0.png)
		- 访问 sayHello 接口
			- ```shell
			  watch -n 1 curl -o /dev/null -s -w %{http_code} http://cat.catface996.com:31606/sayHello
			  ```
		- 查看 Kiali 面板的流量分布
			- ![image.png](../assets/image_1651476142241_0.png)
	- 配置 cat、dog、monkey 的 Destination Rule 和 Virtual 再访问 http://cat.catface996.com/sayHello
		- 部署 DestinationRule。
			- ![image.png](../assets/image_1651476549235_0.png)
			- ![image.png](../assets/image_1651476576071_0.png)
		- 部署 VirtualService，cat-vs 会被覆盖成最新配置。
	- 删除 cat-gray deploymnet后，访问 cat-gray。
	-
	- 模拟 prod 环境访问
		- ```shell
		  curl -H "env:prod" http://cat.catface996.com:31606/sayHello
		  
		  watch -n 1 curl  -H "env:prod" -o /dev/null -s -w %{http_code} http://cat.catface996.com:31606/sayHello
		  ```
		- ![image.png](../assets/image_1651476934159_0.png)
	- 模拟 gray 环境访问
		- ```shell
		  curl -H "env:gray" http://cat.catface996.com:31606/sayHello
		  
		  watch -n 1 curl  -H "env:prod" -o /dev/null -s -w %{http_code} http://cat.catface996.com:31606/sayHello
		  ```
	- 模拟默认环境访问
		- ```shell
		  curl -H http://cat.catface996.com:31606/sayHello
		  ```
	- 分布式追踪
		- 概述 https://istio.io/latest/zh/docs/tasks/observability/distributed-tracing/overview/
	- Istio 最佳实践。
		- 流量管理最佳实践 https://istio.io/latest/zh/docs/ops/best-practices/traffic-management/
	-
	-
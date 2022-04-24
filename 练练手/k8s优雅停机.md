

# K8S SpringCloud 优雅停机

K8S中的pod配置关机接口

```yaml
lifecycle:
	preStop:
		httpGet:
			port: 9001
			scheme: HTTP
			path: /actuator/shutdown
```



springboot 中配置:

~~~yaml
##
## 健康检查
management.endpoint.health.show-details=ALWAYS
management.endpoints.web.base-path=/actuator/
management.endpoints.web.exposure.include=env,health,info,metrics,prometheus,threaddump,shutdown
management.endpoint.shutdown.enabled=true
~~~


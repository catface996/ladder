- ## 代码提交
  
  * 配置好本地git的用户名和邮箱
  例如: 
  git config --global user.name "张三"
  git config --global user.email "zhangsan@oneart.com"
  
  * 提交时必须写明备注
  
  idea 安装 Commit-Message-Create插件
  
  ![20220414113708](https://picgo.catface996.com/picgo20220414113708.png)
  
  提交时选择提交的类型
  
  ![20220414115537](https://picgo.catface996.com/picgo20220414115537.png)
  
  提倡独立功能代码提交,即一次提交的代码涉及到的功能相对独立
## maven 工程相关

* 依赖的jar包版本号只允许在parent pom中出现,其他模块只做引入,不做版本控制.
* parent pom 只做jar包版本仲裁,不允许做引入.
* 工程需要引入新的jar包,需通知相应的owner.
* 严禁将target目录的任何文件提交到仓库.
* 所有工程文件编码格式统一为UTF8,禁止使用GBK.
## http接口相关

* 新接口采用post请求,restful格式一般适用于前端,目前采用前后端分离,后端仅提供数据接口,避免使用 get,put,delete,方便后续网关统一拦截,通用参数注入等.
* 必须做swagger注释,并且对接口分组.
* Controller终结所有跟业务不相干的操作,禁止将非业务参数传入service层,
 example: 禁止将HttpServletResponse 对象作为参数传入到service 方法,禁止将token传入service应该将userId传入到service.
* 遵守url定义规范.
* 禁止通过前端上传文件到后端服务器,采用后端给临时签名,前端直传阿里云的方案.
* 禁止通过后端服务器下载文件,导出文件需要后端服务器生成后,上传oss,返回给前端oss的临时访问地址,前端直接从oss下载.
## 业务Service层

* 必须定义interface,基于interface设计,非interface方法禁用public修饰.
* 接口实现类实现接口的方法按接口定义顺序在实现类中实现,private方法需要在public方法之后定义.
* 日志必须使用log.info(),禁止使用System.out,日志参数必须用{}格式化.
* 捕获异常后,打印日志必须使用log.error(),并且将exception作为最后一个参数.
## mysql相关

* 除根据id查询,更新,删除外,其他sql需要手写,禁用Lamda表达式.
## redis相关

* 遵守redis key前缀规范.
* redis中的key,需要使用分隔符时,使用":"作为分隔符.
## 配置相关(properties,yaml)

* 敏感配置禁止配置在本地,例如数据库地址,用户名密码,秘钥文件等.
* 所有需要在配中心配置的配置项,需在配置文件中出现key,value统一用"配置在nacos",example: spring.datasource.url=配置在nacos
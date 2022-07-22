- 服务器代码的骨架
	- 参考文档： https://dev.mysql.com/doc/internals/en/guided-tour-skeleton.html
	- 场景
		- 查询
			- sql_select.cc
				- ```
				  bool handle_query(THD *thd, LEX *lex, Query_result *result,
				                    ulonglong added_options, ulonglong removed_options)
				  ```
		- 插入
			-
		- 更新
		- 删除
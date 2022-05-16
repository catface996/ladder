- 如果 InnoDB 不是默认的存储引擎，那么您可以通过在命令行上使用 -- default-storage-engine = InnoDB 重新启动服务器，或者使用 MySQL 服务器选项文件[ mysqld ]部分中定义的 default-storage-engine = InnoDB 重新启动服务器，从而确定您的数据库服务器和应用程序是否能够正确地使用 InnoDB。
- 因为更改默认的存储引擎只会影响新创建的表，所以运行你的应用程序安装和设置步骤来确认一切都安装正确，然后运行应用程序功能来确保数据加载、编辑和查询功能正常工作。如果表依赖于特定于另一个存储引擎的特性，则会收到错误消息。在这种情况下，向 CREATE TABLE 语句添加 ENGINE = other _ ENGINE _ name 子句以避免错误。
- 如果您没有对存储引擎做出深思熟虑的决定，并且您想预览在使用 InnoDB 创建某些表时如何工作，那么为每个表发出命令 ALTER TABLE TABLE TABLE _ name ENGINE = InnoDB; 。或者，为了在不干扰原始表的情况下运行测试查询和其他语句，可以复制:
	- ```shell
	  CREATE TABLE ... ENGINE=InnoDB AS SELECT * FROM other_engine_table;
	  ```
- 要在现实的工作负载下使用完整的应用程序评估性能，请安装最新的 MySQL 服务器并运行基准测试。
- 测试整个应用程序的生命周期，从安装到大量使用，再到服务器重新启动。在数据库忙碌时终止服务器进程以模拟电源故障，并在重新启动服务器时验证数据是否成功恢复。
- 测试任何复制配置，特别是如果在源服务器和副本上使用不同的 MySQL 版本和选项。
-
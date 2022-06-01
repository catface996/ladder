- InnoDB 在外部 创建表有不同的原因；也就是说，在数据目录之外创建表。例如，这些原因可能包括空间管理、I/O 优化或将表放置在具有特定性能或容量特征的存储设备上。
- InnoDB支持以下外部建表方式：
	- 使用数据目录子句
	- 使用 CREATE TABLE ... TABLESPACE 语法
	- 在外部通用表空间中创建表
- ## 使用数据目录子句
	- 您可以通过在语句中指定子句在InnoDB外部目录 中创建表。 DATA DIRECTORYCREATE TABLE
		- ```sql
		  CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '/external/directory';
		  ```
	- DATA DIRECTORY在 file-per-table 表空间中创建的表支持 该子句。innodb_file_per_table启用该变量 时，表会在每个表的文件表空间中隐式创建， 默认情况下是这样。
		- ```sql
		  mysql> SELECT @@innodb_file_per_table;
		  +-------------------------+
		  | @@innodb_file_per_table |
		  +-------------------------+
		  |                       1 |
		  +-------------------------+
		  ```
	- 有关 file-per-table 表空间的更多信息，请参阅 第 [[14.6.3.2  File-Per-Table 表空间]]。
	- 请确保您选择的目录位置，因为该 DATA DIRECTORY子句不能用于 ALTER TABLE以后更改位置。
	- 当您在语句中指定DATA DIRECTORY子句时 ，会在指定目录下的 schema 目录中创建CREATE TABLE表的数据文件（ ），并在 MySQL 数据目录下的 schema 目录中创建包含数据文件路径的文件（ ）。文件在 功能上类似于符号链接。（实际的符号链接不支持与 数据文件一起使用。） table_name.ibd.isltable_name.isl.islInnoDB
	- DATA DIRECTORY 以下示例演示了使用该子句 在外部目录中创建表。假设 innodb_file_per_table变量已启用。
		- ```sql
		  mysql> USE test;
		  Database changed
		  
		  mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '/external/directory';
		  
		  # MySQL creates the table's data file in a schema directory
		  # under the external directory
		  
		  $> cd /external/directory/test
		  $> ls
		  t1.ibd
		  
		  # An .isl file that contains the data file path is created
		  # in the schema directory under the MySQL data directory
		  
		  $> cd /path/to/mysql/data/test
		  $> ls
		  db.opt  t1.frm  t1.isl
		  ```
	- **使用说明**
		- MySQL 最初保持表空间数据文件处于打开状态，从而阻止您卸载设备，但如果服务器繁忙，最终可能会关闭该文件。注意不要在 MySQL 运行时意外卸载外部设备，或者在设备断开连接时启动 MySQL。在缺少关联的数据文件时尝试访问表会导致严重错误，需要重新启动服务器。
		- 如果在预期路径中找不到数据文件，则服务器重新启动可能会失败。在这种情况下，请从架构目录中手动删除该 .isl文件。重新启动后，删除表以 从数据字典.frm中删除文件和有关表的信息。
		- 在将表放在 NFS 挂载的卷上之前，请查看将 NFS 与 MySQL 一起使用 中概述的潜在问题 。
		- 如果使用 LVM 快照、文件复制或其他基于文件的机制来备份表的数据文件，请始终 FLUSH TABLES ... FOR EXPORT首先使用该语句以确保在备份发生之前将缓冲在内存中的所有更改 刷新到磁盘。
		- 使用该DATA DIRECTORY子句在外部目录中创建表是使用 不支持 的符号链接的 替代方法。InnoDB
		- DATA DIRECTORY在源和副本位于同一主机上的复制环境中不支持 该子句。该DATA DIRECTORY子句需要完整的目录路径。在这种情况下复制路径将导致源和副本在同一位置创建表。
- ## 使用 CREATE TABLE ... TABLESPACE 语法
	- CREATE TABLE ... TABLESPACE语法可以与该 DATA DIRECTORY子句结合使用以在外部目录中创建表。为此，请指定 innodb_file_per_table为表空间名称。
		- ```sql
		  mysql> CREATE TABLE t2 (c1 INT PRIMARY KEY) TABLESPACE = innodb_file_per_table
		         DATA DIRECTORY = '/external/directory';
		  ```
	- 此方法仅支持在 file-per-table 表空间中创建的表，但不需要 innodb_file_per_table启用该变量。在所有其他方面，此方法与上述方法等效CREATE TABLE ... DATA DIRECTORY。适用相同的使用说明。
- ## 在外部通用表空间中创建表
	- 您可以在驻留在外部目录中的通用表空间中创建表。
		- 有关在外部目录中创建通用表空间的信息，请参阅 创建通用表空间。
		- 有关在通用表空间中创建表的信息，请参阅 将表添加到通用表空间。
-
-
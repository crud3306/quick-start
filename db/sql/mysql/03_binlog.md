  
binlog （binary log）是mysql的二进制日志，以二进制的形式记录了对于数据库的变更操作，不包括select和show操作。    
  
作用：  
----------
1) 用来查看mysql变更  
2) mysql的备份恢复  
3) mysql的主从复制  
  

文件位置 
---------- 
binlog默认放置在数据目录下  


binlog的命名方式  
----------
mysql-bin.000001  


binlog文件的生成方式
----------
1.mysql启动的时候会产生新的binlog  
2.mysql服务器在执行flush logs；可以产生新的binlog文件。
  


binlog参数
----------
sync_binlog 			= 1

log-bin					= mysql-bin
#决字了mysql的binlog文件的名字，生成名字为mysql-bin.0000001

binlog_format			= row
#规定binlog的格式，binlog有三种格式statement、mixed及row，默认使用statement，建议使用row  

expire_logs_days		= 10
binlog_cache_size		= 4M
max_binlog_cache_size	= 8M
max_binlog_cache_size   = 1024M



binlog有三种格式
-------------
binlog有三种格式statement、mixed及row，默认使用statement，建议使用row
```
1) STATEMENT是基于sql语句级别的binlog，每一条修改数据的sql都会被保存到binlog里；  

优点:
binlog文件较小
日志是包含用户执行的原始SQL,方便统计和审计
出现最早，兼容较好

缺点：
存在安全隐患，可能导致主从不一致
对一些系统函数不能准确复制或是不能复制


2) ROW是基于行级别的，他会记录每一行记录的变化,就是将每一行的修改都记录到binlog里面,记录的非常详细，但sql语句并没有在binlog里,在replication里面也不会因为存储过程触发器等造成Master-Slave数据不一致的问题,但是有个致命的缺点日志量比较大.由于要记录每一行的数据变化,当执行update语句后面不加where条件的时候或alter table的时候,产生的日志量是相当的大。    
新版本的MySQL中队row level模式也被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录。至于update或者delete等修改数据的语句，还是会记录所有行的变更。

优点:
相比statement更加安全的复制格式
在某些情况下复制速度更快（SQL复杂，表有主键）
系统的特殊函数也可以复制
更少的锁
更新和删除语句检查是否有主键，如果有则直接执行，如果没有，看是否有二级索引，如再没有，则全表扫描

缺点：
binlog比较大（myql5.6支持binlog_row_image）
单语句更新（删除）表的行数过多，会形成大量binlog
无法从binlog看见用户执行SQL（5.6中增加binlog_row_query_log_events记录用户的query）


3) MIXED混合模式复制，是以上两种level的混合使用，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog,MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种.  

优点:
混合使用row和statement格式，对于DDL记录statument,对于table里的行操作记录为row格式。
如果使用innodb表，事务级别使用了READ_COMMITTED or READ_UMCOMMITTED日志级别只能使用row格式。
但是使用ROW格式中DDL语句还是会记录成statement格式。

缺点:
mixed模式中，那么在以下几种情况下自动将binlog模式由SBR模式改成RBR模式。
当DML语句更新一个NDB表
当函数中包含UUID时
2个及以上auto_increment字段的表被更新时
行任何insert delayed语句时
用UDF时
视图中必须要求使用RBR时，例如创建视图使用了UUID()函数
```


清理过期的binlog日志
----------
1 手动删除binlog，删除时注意不要删除当前正在使用的binlog文件，可以用show master state查看。  

2 自动删除binlog  
通过binlog参数(expire_logs_days)来实现mysql自动删除binlog。  
show binary logs;  
show variables like 'expire_logs_days';  
set global expire_logs_days=3;  
  
  


























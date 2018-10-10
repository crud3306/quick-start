  
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
  


binlog变用参数
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


清理过期的binlog日志
----------
1 手动删除binlog，删除时注意不要删除当前正在使用的binlog文件，可以用show master state查看。  

2 自动删除binlog  
通过binlog参数(expire_logs_days)来实现mysql自动删除binlog。  
show binary logs;  
show variables like 'expire_logs_days';  
set global expire_logs_days=3;  
  
  


























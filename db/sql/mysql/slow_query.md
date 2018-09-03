  
慢查询  
  
慢查询是指查询时间超过指定时间的查询。  
  
命令行连接mysql后，执行如下命令，来获取慢查询指定时间  
> show variables like '%long_query%';  
输出如下：  
> Variable_name        Value   
> long_query_time      10.000000   
long_query_time即为指定的时间，上面的结果表示10秒，可以根据需要指定为其它值    
  
获取当前链接数  
> show variables like 'connections';  
  
  
开启慢查询
------------
打开mysql配置文件，添加：  
mysql5.6及以上版本：  
> slow-query-log=1  #是否启用慢查询日志，1或者on为启用，0为禁用  
> long_query_time = 2  #SQL语句运行时间阈值，执行时间大于该值的语句才会被记录。单位秒   
> slow-query-log-file = /data/mysql/slow.log  #指定慢查询日志文件，使用绝对路径   
> log_queries_not_using_indexes=1  #将没有使用索引的语句记录到慢查询日志  
   
mysql5.5及以下版本： 
> long_query_time = 2  
> log-slow-queries = /data/mysql/slow.log  
> log_queries_not_using_indexes=1  #将没有使用索引的语句记录到慢查询日志  
  

  

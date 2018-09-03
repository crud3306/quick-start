  
索引  
-----------
索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。  
  
查看某表的索引有哪些
-----------  
> 先命令行连接mysql  
> use xxx库;  
> show index from xxxx表;  
或者
> show index from xxxx表\G;  


索引的分类
-----------
主键索引  
唯一索引  
普通索引  
全文索引 FULLTEXT，只有myisam存储引擎支持该索引    
  
索引的创建  
-----------
> create index 索引名称 on 表名(列名)          #创建普通索引，多个列名用逗号分隔  
> create unique index 索引名称 on 表名(列名)     #创建唯一索引
  
  
查看某sql语句用到了什么索引
-----------
可以配合explain关键字 + sql语句
例：
> explain select * from user where username = 'xxxx';  
在结果中key列对应的值即为匹配的索引  

使用show profile分析sql
-----------
1) 首先查看mysql是否支持show profile   
> select @@have_profiling;  
2) 如果profiling是关闭的，可以通过set语句在session级别开启profiling。  
> set profiling=1;   
3) 查看当前sql的queryID  
> show profiles;  
4) 通过queryID查看该sql执行所用时间明细  
> show profile for query queryID;  

例：  
```sql
set profiling=1;
select count(*) from shop where id > 10;
show profiles; #该命令可以拿到上一条sql的queryID
show profile for query 刚拿到的queryID;
```  
  
  
索引文件存储的位置  
-----------
假设表名为shop，则：  
> shop.frm  表结构  
> shop.MYD  表数据  
> shop.MYI  表索引  
  
数据迁移时，可迁移x.frm, x.MYD，但不可迁移x.MYI，因为即使移过去索引也是失效的。  
  


























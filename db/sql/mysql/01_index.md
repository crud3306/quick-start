  
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
方式1：create 索引类型 索引名 on 表(列名)  
> create index 索引名称 on 表名(列名)          #创建普通索引，多个列名用逗号分隔   
> create unique index 索引名称 on 表名(列名)     #创建唯一索引  
  
方式2：alter table 表名 add 索引类型 索引名(列名)  
> alter table tb add index dept_index(dept)  
> alter table tb add unique index name_index(name)  
> alter table tb add index dept_name_index(dept, name)  #多个列用逗号分隔  

> alter table tb add constraint tid_uk unique index(tid);
> alter table tb add constraint tid_pk primary key(tid);
  
  
删除索引  
-----------
drop index 索引名 on 表名;  
> drop index index_name on table_name;   
或者
> alter table table_name drop primay key;
> alter table table_name drop index index_name;
   
    
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
  


分析sql的执行计划
-----------
使用explain + sql语句    

explain语句结果中的字段分析  
id：编号  
select_type：查询类型  
table：表  
type：类型  
possible_keys：预测用到的索引    
key：实际使用索引的长度  
ref：表与表之间的引用关系  
rows：通过索引查询到的数据量  
Extra：额外的信息  


1）id  
id值相同，则从上往下顺序执行；id值不同，则id值越大越优先。  
多表查询时，查询优先器会自动优先查询记录数少的表。（原因，最终结果虽一样，但中间过程的记录数少，节约内存）  
  
  
2) select_type  查询类型  
primary：包含子查询sql中的 主查询（最外层）    
subquery：包含子查询sql中的 子查询（非最外层）   
simple：简单查询 （不包含子查询、union）  
derived：衍生查询  （使用到了临时表）  
	a.在from子查询中只有一张表  
	```
	explain select cr.cname from (select * from course where tid in (1,2))
	```
	b.在from子查询中，如果有table1 union table2，则table1就是derived，table2是union  
	```
	explain select cr.cname from (select * from course where tid = 1 union select * from course where tid = 2)
	```
union：看上例b
union result：告知开发人员，哪些表之间存在union查询，此select_type对应的id为null   


3）type  类型（也称索引类型）  
system > const > eq_ref > ref > range > index > all  
其中：system,const只是理想情况，实际能达到ref > range，要对type进行优化的前提是有索引    
system：只有一条数据的系统表 或 衍生表只有一条数据的主查询  
结果只有一条数据  

const：仅仅能查到一条数据的sql，用于primary key 或unique索引  
结果只有一条数据  

eq_ref：唯一性索引，对于每个索引键的查询，返回匹配唯一行数据（有且只有1个，不能多也不能少）  
结果多条，但是每条数据是唯一的。  

ref：非唯一性索引，对于每个索引键的查询，返回匹配的所有行（0 或 多）  
结果多条，但是第条数据是0或多条  

range：使用索引检索指定范围内的行，where后面是一个范围查询（between,in,>,<,>=,<=）  

index：查询全部索引中的数据  

all：查询全部表中的数据  



4）possible_keys： 可能用到的索引  
  

5）key：实际使用到的索引    

6）key_len：索引的长度  ，用于判断复合索引是否被完全使用  

7）ref：

8）rows：找到所需记录需要读取的行数（估计值）

9）Extra：
	a) using filesort：性能损耗大，需要额外的一次排序（查询）
	一般出现在order by语句中

	b) using temporary：性能损耗大，用到了临时表。
	一般出现在group by语句中

	c) using index：性能提升，索引覆盖。原因：不读取原文件，只从索引文件中获取数据（不需要回表查询）

	d) using where：(需要回表查询)  

	e) impossible where：where子句永远为false
	explain select * from 表名 where a1='x' and a1='y'










































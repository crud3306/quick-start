  
索引  
==========
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


索引文件存储的位置 
-----------
假设表名为shop，

若为myisam则：  
> shop.frm  表结构  
> shop.MYD  表数据  
> shop.MYI  表索引  
  
数据迁移时，可迁移x.frm, x.MYD，但不可迁移x.MYI，因为即使移过去索引也是失效的。  

若为innodb则：  
> shop.frm  表结构  
> shop.ibd  表空间文件(表索引+表数据)   


注意区分myisam与innodb：  
-----------
myisam 非聚集索引(索引与数据分开存储)  
	主索引：页子节点存的是数据记录地址  
	辅助索引：页子节点存的也是数据记录地址  

innodb 聚集索引(索引与数据存储在一起)    
	主索引：页子节点存的是数据记录  
	辅助索引：页子节点存的是数据记录的主键值  




使用show profile分析sql
===========
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


使用explain 分析sql的执行计划
==========
使用explain + sql语句    

explain语句结果中的字段分析  
```
id	select_type	table	partitjons	type	possible_keys	key	key_len	ref	rows	filtered	Extra
1	SIMPLE	adminlog	 	ALL	 	 	 	 	2	100	 
```
id：编号  
select_type：查询类型  
table：输出结果的表  
type：表示MySql在表中找到所需行的方式，或者叫访问类型  
possible_keys：可能使用的索引列表      
key：实现执行使用索引列表  
key_len：实际使用索引的长度
ref：显示使用哪个列或常数与key一起从表中选择行   
rows：执行查询的行数，简单且重要，数值越大越不好，说明没有用好索引    
Extra：额外的一些重要信息，该列包含MySQL解决查询的详细信息  

注意：  
这里需要强调rows是核心指标，绝大部分rows小的语句执行一定很快（有例外，下面会讲到）。所以优化语句基本上都是在优化rows。  


1）id  
id值相同，则从上往下顺序执行；id值不同，则id值越大越优先。  
多表查询时，查询优先器会自动优先查询记录数少的表。（原因，最终结果虽一样，但中间过程的记录数少，节约内存）  
  
  
2) select_type  查询类型  
simple：简单查询 （不包含子查询、union）  
primary：包含子查询sql中的 主查询（最外层）    
subquery：包含子查询sql中的 子查询（非最外层）   
derived：衍生查询  （使用到了临时表）  
	a.在from子查询中只有一张表  
	```
	explain select cr.cname from (select * from course where tid in (1,2))
	```
	b.在from子查询中，如果有table1 union table2，则table1就是derived，table2是union  
	```
	explain select cr.cname from (select * from course where tid = 1 union select * from course where tid = 2)
	```
union：第二个或者后面的查询语句。 看上例b  
union result：告知开发人员，哪些表之间存在union查询，此select_type对应的id为null    


3）type  类型（也称索引类型）  
system > const > eq_ref > ref > range > index > all  
其中：system,const只是理想情况，实际能达到ref > range，要对type进行优化的前提是有索引    

3-1) all：全表扫描    

3-2) index：某索引中全扫描，遍历整个索引来查询匹配的行  

3-3) range：索引范围扫描(使用索引检索指定范围内的行)，常见于where后面是一个范围查询（between,in,>,<,>=,<=），例:  
	```
　　　　explain select * from adminlog where id>0 , 
　　　　explain select * from adminlog where id>0 and id<=100
　　　　explain select * from adminlog where id in (1,2) 
	```
3-4) ref：使用非唯一索引或唯一索引的前缀扫描，返回匹配某个单独值的记录行，返回匹配的所有行（0 或 多条）

3-5) eq_ref：类似于ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中有一条记录匹配，返回唯一匹配行；或者多表连接中使用主建或唯一健作为关联条件 

3-6) const：单表中最多有一个匹配行。主要用于比较primary key[主键索引]或者unique[唯一]索引,因为数据都是唯一的，所以性能最优。条件使用=。 

3-7) system：只有一条数据的系统表 或 衍生表只有一条数据的主查询  
结果只有一条数据  

3-8) type=NULL　不用访问表或者索引，直接就能够得到结果　
例　explain select 1 from dual,


类型type 还有其他值　如：  
ref_or_null : 与ref 类似，区别在于条件中包含对NULL的查询.    
index_merge : 索引合并优化,   
unique_subquery : in的后面是一个主键字段的子查询  
index_subquery : 与unique_subquery 类似,区别在于in的后面是查询非唯一索引字段的子查询  


4）possible_keys： 可能用到的索引    

5）key：实际使用到的索引    

6）key_len：索引的长度  ，用于判断复合索引是否被完全使用  

7）ref：

8）rows：找到所需记录需要读取的行数（估计值）  

9）Extra：  
	a) using filesort：性能损耗大，需要额外的一次排序(即查询出结果后还要单独做一次排序)（查询）  
	一般出现在order by语句中  

	b) using temporary：性能损耗大，用到了临时表。  
	为了解决查询，MySQL需要创建一个临时表来容纳结果。一般出现在group by语句中  

	c) using index：性能提升，索引覆盖。  
	原因：不读取原文件，只从索引文件中获取数据（不需要回表查询）
	描述：只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的信息。就是建议取索引列。这样就可以不要通过索引去实际表中找数据了，直接返回索引列的数据，一次查询。否则就是索引表查一次，实际表中查一次。

	d) Using index condition

	e) using where：(需要回表查询)  

	f) impossible where：where子句永远为false
	explain select * from 表名 where a1='x' and a1='y'

	g) range checked for each record
	  没有找到合适的索引




无效索引：   数据变化不大的列。如XX类型，是否有效，状态等列的索引都是无效的。这些无效索引还是影响Insert 、Update、Delete 语句的性能。因为这些语包的执行都要对索引表进行更新。又因为这些表的值变化不大，数据库很难为他们合理分配索引。所以影响语句的性能。  
  

IN,OR 是否会走索引:  
一条SQL会不会走索引一个看条件使用的运算符，另一个看有没有索引。所以SQL会不会走索引和IN.OR,group by 没有关系。

肯定不走索引的：  
1) 什么运算符不走索引，<>,!=，not in等  
2) 表达式或函数不走索引  
3) 传递的值与列类型不匹配也不走索引  
4) 使用LIKE 操作的时候，如果条件以通配符开始（ '%abc...'）MySQL 无法使用索引  
5) 如果是复合索引，查询时没用到最左列的，也不会走索引    

特殊：  
4) OR前后两个条件都要有索引整个SQL才会使用索引。只要有一个条件没索引那么和or同级的所有列都不使用索引。如果出现OR的一个条件没有索引时，建议使用 union  
5) 范围查询后之后的列不走索引，多个范围查询列最多只有一个列会走索引    

注意：
1）索引不要过多
2）BLOB和TEXT类型的列只能创建前缀索引；如果是myisam也可以创建fulltext索引。




既然索引可以加快查询速度，那么是不是只要是查询语句需要，就建上索引？答案是否定的。因为索引虽然加快了查询速度，但索引也是有代价的：索引文件本身要消耗存储空间，同时索引会加重插入、删除和修改记录时的负担，另外，MySQL在运行时也要消耗资源维护索引，因此索引并不是越多越好。一般两种情况下不建议建索引。
----------
1) 表记录比较少不用建索引，例如一两千条甚至只有几百条记录的表，没必要建索引，让查询做全表扫描就好了。至于多少条记录才算多，这个个人有个人的看法，我个人的经验是以2000作为分界线，记录数不超过 2000可以考虑不建索引，超过2000条可以酌情考虑索引。

2) 不建议建索引的情况是索引的选择性较低。所谓索引的选择性（Selectivity），是指不重复的索引值（也叫基数，Cardinality）与表记录数（#T）的比值：

Index Selectivity = Cardinality / #T

显然选择性的取值范围为(0, 1]，选择性越高的索引价值越大，这是由B+Tree的性质决定的。例如，上文用到的employees.titles表，如果title字段经常被单独查询，是否需要建索引，我们看一下它的选择性：
```
SELECT count(DISTINCT(title))/count(*) AS Selectivity FROM employees.titles;
+-------------+
| Selectivity |
+-------------+
|      0.0000 |
+-------------+
title的选择性不足0.0001（精确值为0.00001579），所以实在没有什么必要为其单独建索引。
比现这种情况原因，因为虽然总数据很多，但title的值只是局限的几条。  






覆盖索引
---------
InnoDB存储引擎支持覆盖索引（covering index，或称索引覆盖），即从辅助索引中就可以得到查询记录，而不需要查询聚集索引中的记录。

使用覆盖索引的一个好处是：辅助索引不包含整行记录的所有信息，故其大小要远小于聚集索引，因此可以减少大量的IO操作.

explain分析sql执行计划发现：  
Extra中有Using index，代表覆盖索引

































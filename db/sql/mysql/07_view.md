  
视图    
  
示例：
```sql
select stuname, majorname 
from student s inner join major m on s.`majorid`=m.`id`
where s.`stuname` like '李%';

#创建视图  
create view v1
as 
select stuname, majorname 
from student s inner join major m on s.`majorid`=m.`id`;

#使用视图  
select * from v1 where stuname like '李%';   
```


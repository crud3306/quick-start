  
查看当前库支持哪些存储引擎  
> show engines;  
  
最常用的是  
> myisam    
> innodb 支持事务   
> memory  
  

事务的ACID属性
----------  
A 原子性（atomicity）  
原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。  
  
C 一致性 （consistency）  
事务必须使数据库从一个致状态变换到另外一个一致性状态  
  
I 隔离性（isolation）  
事务隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰  
  
D 持久性（durability）  
持久性是指一个事务一量被提交，它对数据库中数据的改变就是永久性的，接下来的其它操作和数据库故障不应该对其有任何影响  
   

事务的创建  

1) 隐性事务：事务没有明显的开启和结束的标记   
比如 单条insert、update、delete语句  
  
2) 显式事务：事务具有明显的开启和结束标记  
前提：必须先设置自动提交动能为禁用，可通过show variables like 'auto%';查看事务是否自动提交      

步骤1：开启画务    
set autocommit=0;    
start transaction; #可选的  
  
步骤2：编写事务中的sql语句（insert update delete）  
语句1;  
语句1;  
  
步骤3：结束事务  
commit; #提交  
rollback; #回滚  
  
  

对于同时运行的多个事务，当这些事务操作数据库中相同的数据时，如果没有采取必要的隔离机制，就会导致各种并发问题：  
----------
脏读：对于两个事务t1、t2，t1读取了已经被t2更新但还没有被提交的字段之后，若t2回滚，t1读取的内容就是临时且无效的。  
  
不可重复读：对于两个事务t1、t2，t1读取了一个字段，然后t2更新了该字段之后，t1再次读取同一个字段，值就不同了。  
  
幻读：对于两个事务t1、t2，t1从一个表中读取了一个字段，然后t2在该表中插入了一些新的行之后，如果t1再次读取同一个表，就会多出几行。    
  

  
数据库事务的隔离性：  
----------
数据库系统必须具有隔离并发运行各个事务的能力，使它们不会相互影响，避免各种并发问题  
一个事务与其它事务隔离的程度称为隔离级别。数据库规定了多种事务隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据一致性就越好，但并发性越弱    
  

oracle支持2种事务隔离级别：
----------
> read committed   
> serializable  

默认级别为read committed。   
   

mysql支持4种事务隔离级别： 
---------- 
> read uncommitted  读未提交数据  
允许事务读取未被其他事物提交的变更。脏读、不可重复读、幻读问题都会出现。  
    
> read committed  读已提交数据  
只允许事务读取已经被其它事务提交的变更。可以避免脏读，但不可重复读和幻读问题仍可能出现  
  
> repeatable read  可重复读  
确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事物对这个字段进行更新。可以避免脏读和不可重复读，但幻读问题仍然存在。  
  
> serializable  串行化  
确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新、和删除操作。所有并发问题都可避免，但性能十分低下。  
  
mysql默认的事务隔离级别是repeatable read。
  
查看mysql当前隔离级别  
> mysql -uroot -p  
> select @@tx_isolation;   
  
更改msqyl隔离级别   

> set session transaction isolation level read uncommitted;  #仅更改当前链接的
> set global transaction isolation level read uncommitted;  #更改全局的，链接需重启生效。  
  
更改后，再次用 select @@tx_isolation; 查看当前隔离级别  
  
  
保存点（回滚点） savepoint  
----------
用法如下:  (配合rollback to来使用)
> set autocommit=0;  
> start transaction;  
> delete from accout where id=10;  
> savepoint a;  
> delete from accout where id=11;  
> rollback to a;  
   

























  
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
持久性是指一个事务一旦被提交，其所做的修改就会永久保存在数据库中，此时即使数据库系统崩溃，已提交的数据也不会丢失。注意：这里所说的持久性是相对来说的，只能从数据库的角度来说保证数据的持久性，而不包括一些外部因素，如磁盘损坏而导致的数据丢失。    
   

事务的创建  

1) 隐性事务：事务没有明显的开启和结束的标记   
比如 单条insert、update、delete语句  
  
2) 显式事务：事务具有明显的开启和结束标记  
前提：必须先设置自动提交动能为禁用，可通过show variables like 'auto%';查看事务是否自动提交      

步骤1：开启事务    
set autocommit=0;     
start transaction; #可选的  
#或者  
begin;   
  
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
  

注意： 
不可重复读的重点是修改：同样的条件，读取过的数据，再次读取出来发现值不一样了。  
幻读的重点在于新增或者删除：同样的条件，第 1 次和第 2 次读出来的记录数不一样。  
  
  
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
> read uncommitted  未提交读（读未提交数据）  
允许事务读取未被其他事物提交的变更。脏读、不可重复读、幻读问题都会出现。  
    
> read committed  已提交读（读已提交数据)  
只允许事务读取已经被其它事务提交的变更。可以避免脏读，但不可重复读和幻读问题仍可能出现  
```
DBMS需要对选定对象的写锁(write locks)一直保持到事务结束，但是读锁(read locks)在SELECT操作完成后马上释放（因此“不可重复读”现象可能会发生，见下面描述）。和可重复读隔离级别一样，也不要求“范围锁(range-locks)”。

不可重复读是因为，事务只维持了选定对象的写锁，如果一些选定对象只涉及读锁，那么在读锁释放之后，其它事务可以对这些对象进行修改，该事务再次读取时就不一致了。例如，事务A对数据对象拥有写锁，事务B读取了数据对象，后事务A有对数据对象进行的更改并提交，事务B再次读取数据对象，两次读取结果不同。

大多数数据库的默认事务隔离级别都是这个。
```
  
> repeatable read  可重复读  
确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事物对这个字段进行更新。在SQL标准中，可以避免脏读和不可重复读，但幻读问题仍然存在。但是innoDB解决了幻读(注意看MVCC)。
```
该级别保证了同一个事务中多次读取同样的记录的结果是一致的。

对选定对象的读锁(read locks)和写锁(write locks)一直保持到事务结束，但不要求“范围锁(range-locks)”，因此可能会发生“幻读(phantom reads)”。

幻读：是因为没有保持范围锁，该事务执行了一个 where 子句的范围查询后，其他事务可能新增了一条处于该事务 where 查询范围内的记录，那么该事务再次执行范围查询时就会看到这些新增的记录行（幻行，Phantom row）。

可重复读是 MySQL 的默认事务隔离级别。
```
  
> serializable  可串行化  
确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新、和删除操作。所有并发问题都可避免，但性能十分低下。  
```
实现可序列化要求在选定对象上的读锁和写锁保持直到事务结束后才能释放。在 SELECT 的查询中使用一个 WHERE 子句 来描述一个范围时应该获得一个“范围锁(range-locks)”。这种机制可以避免“幻读(phantom reads)”现象。
```

  
mysql默认的事务隔离级别是repeatable read。
  
  

查看mysql当前隔离级别  
> mysql -uroot -p  
> select @@tx_isolation;   
> select @@session.tx_isolation;   
> select @@global.tx_isolation;   
```
select @@tx_isolation;
/*
输出结果：
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
*/
```

  
更改mysql隔离级别   
> set session transaction isolation level read uncommitted;  #仅更改当前链接的(仅在本次会话中生效)  
> set global transaction isolation level read uncommitted;  #更改全局的，client需重新链接生效。   
> set session transaction isolation level read committed;  
> set session transaction isolation level repeatable read;  
> set session transaction isolation level serializable;  

或者  
> set session tx_isolation='read-uncommitted';  #仅更改当前链接的  
  
更改后，再次用 select @@tx_isolation; 查看当前隔离级别  
  
事务参考地址：  
https://juejin.im/post/5ba0c3a6e51d450e597b2fb4  
https://blog.csdn.net/sunjinjuan/article/details/80916903  
  
  
保存点（回滚点） savepoint  
----------
用法如下:  (配合rollback to来使用)
> set autocommit=0;  
> start transaction;  
> delete from accout where id=10;  
> savepoint a;  
> delete from accout where id=11;  
> rollback to a;  
   


mysql事务实现原理
------------
●  undo log
Undo Log的原理很简单，为了满足事务的原子性，在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为Undo Log）。然后进行数据的修改。如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态。

●  redo log
    和Undo Log相反，Redo Log记录的是新数据的备份。在事务提交前，只要将Redo Log持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是Redo Log已经持久化。系统可以根据Redo Log的内容，将所有数据恢复到最新的状态。





大事务
-------------
定义：运行时间比较长，操作的数据比较多的事务  
  
风险：  
锁定太多的数据，造成大量的阻塞和锁超时  
回滚时所需时间比较长  
执行时间长，容易造成主从延迟  
  
大事务建议：  
避免一次处理太多的数据  
移出不必要在事务中的select操作  












MVCC
=========
名词简析:
------
1.MVCC:是multiversion concurrency control的简称，也就是多版本并发控制，是个很基本的概念。MVCC的作用是让事务在并行发生时，在一定隔离级别前提下，可以保证在某个事务中能实现一致性读，也就是该事务启动时根据某个条件读取到的数据，直到事务结束时，再次执行相同条件，还是读到同一份数据，不会发生变化（不会看到被其他并行事务修改的数据）。

2.read view:InnoDB MVCC使用的内部快照的意思。在不同的隔离级别下，事务启动时（有些情况下，可能是SQL语句开始时）看到的数据快照版本可能也不同。在上面介绍的几个隔离级别下会用到 read view。

3.快照读: 就是所谓的根据read view去获取信息和数据，不会加任何的锁。

4.当前读:前读会获取得到所有已经提交数据，按照逻辑上来讲的话，在一个事务中第一次当前读和第二次当前读的中间有新的事务进行DML操作，这个时候俩次当前读的结果应该是不一致的，但是实际的情况却是在当前读的这个事务还没提交之前，所有针对当前读的数据修改和插入都会被阻塞，主要是因为next-key lock解决了当前读可能会发生幻读的情况。
next-key lock当使用主键索引进行当前读的时候，会降级为record lock(行锁)

9.2 Read view详析
---------
InnoDB支持MVCC多版本控制，其中 READ COMMITTED 和 REPEATABLE READ 隔离级别是利用consistent read view(一致读视图)方式支持的。这两种隔离级别下的mvcc主要的区别是生成read view的生成原则不同。

所谓的consistent read view就是在某一时刻给事务系统trx_sys打snapshot(快照)，把当时的trx_sys状态(包括活跃读写事务数组)记下来，之后的所有读操作根据其事务ID(即trx_id)与snapshot中trx_sys的状态做比较，以此判断read view对事务的可见性。

REPEATABLE READ隔离级别(除了GAP锁之外)和READ COMMITTED隔离级别的差别是创建snapshot时机不同。REPEATABLE READ隔离级别是在事务开始时刻，确切的说是第一个读操作创建read view的时候,READ COMMITTED隔离级别是在语句开始时刻创建read view的。这就意味着REPEATABLE READ隔离级别下面一个事务的SELECT操作只会获取一个read view，但是READ COMMITTED隔离级别下一个事务是可以获取多个read view的。

创建／关闭read view需要持有trx_sys->mutex，会降低系统性能，5.7版本对此进行优化，在事务提交时session会cache只读事务的read view。

不同隔离级别下read view的生成机制
```
1. read-commited:
　　函数：ha_innobase::external_lock
　　if (trx->isolation_level <= TRX_ISO_READ_COMMITTED
　　　　&& trx->global_read_view) {
　　　　/ At low transaction isolation levels we let
　　　　each consistent read set its own snapshot /
　　read_view_close_for_mysql(trx);
即：在每次语句执行的过程中，都关闭read_view, 重新在row_search_for_mysql函数中创建当前的一份read_view。
这样就可以根据当前的全局事务链表创建read_view的事务区间，实现read committed隔离级别。

2. repeatable read：
　　在repeatable read的隔离级别下，创建事务trx结构的时候，就生成了当前的global read view。
　　使用trx_assign_read_view函数创建，一直维持到事务结束，这样就实现了repeatable read隔离级别。
```
正是因为read view 生成原则，导致在不同隔离级别()下,read committed 总是读最新一份快照数据，而repeatable read 读事务开始时的行数据版本。  


9.3 read view 判断当前版本数据项是否可见
---------
在InnoDB中，创建一个新事务的时候，InnoDB会将当前系统中的活跃事务列表（trx_sys->trx_list）创建一个副本（read view），副本中保存的是系统当前不应该被本事务看到的其他事务id列表。当用户在这个事务中要读取该行记录的时候，InnoDB会将该行当前的版本号与该read view进行比较。

具体的算法如下:

设该行的当前事务id为trx_id，read view中最早的事务id为trx_id_min, 最迟的事务id为trx_id_max。  
如果trx_id< trx_id_min的话，那么表明该行记录所在的事务已经在本次新事务创建之前就提交了，所以该行记录的当前值是可见的。  
如果trx_id>trx_id_max的话，那么表明该行记录所在的事务在本次新事务创建之后才开启，所以该行记录的当前值不可见。  
如果trx_id_min <= trx_id <= trx_id_max, 那么表明该行记录所在事务在本次新事务创建的时候处于活动状态，从trx_id_min到trx_id_max进行遍历，如果trx_id等于他们之中的某个事务id的话，那么不可见。

从该行记录的DB_ROLL_PTR指针所指向的回滚段中取出最新的undo-log的版本号的数据，将该可见行的值返回。  
需要注意的是，新建事务(当前事务)与正在内存中commit 的事务不在活跃事务链表中。  

在具体多版本控制中我们先来看下源码：

函数：read_view_sees_trx_id。  
read_view中保存了当前全局的事务的范围：  
【low_limit_id， up_limit_id】  

1.当行记录的事务ID小于当前系统的最小活动id，就是可见的。
```
if (trx_id < view->up_limit_id) {
  return(TRUE);
}
```
          
2.当行记录的事务ID大于当前系统的最大活动id（也就是尚未分配的下一个事务的id），就是不可见的。
```
if (trx_id >= view->low_limit_id) {
  return(FALSE);
}
```
          
3.当行记录的事务ID在活动范围之中时，判断是否在活动链表中，如果在就不可见，如果不在就是可见的。
```
for (i = 0; i < n_ids; i++) {
  trx_id_t view_trx_id
    = read_view_get_nth_trx_id(view, n_ids - i - 1);
  if (trx_id <= view_trx_id) {
    return(trx_id != view_trx_id);
  }
}
```

参考地址：  
http://www.lotterychief.com/mysql-tutorials-416576.html  






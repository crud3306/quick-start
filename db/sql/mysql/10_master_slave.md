
  
m/s主从配置  
===========
  
实现原理  
-----------
MySQL之间数据复制的基础是二进制日志文件（binary log file）。一台MySQL数据库一旦启用二进制日志后，其作为master，它的数据库中所有改变数据库数据的操作都会以“事件”的方式记录在二进制日志中，其他数据库作为slave通过一个I/O线程与主服务器保持通信，并监控master的二进制日志文件的变化，如果发现master二进制日志文件发生变化，则会把变化复制到自己的中继日志(relay log)中，然后slave的一个SQL线程会把相关的“事件”执行到自己的数据库中，以此实现从数据库和主数据库的一致性，也就实现了主从复制。   
  
  
  
配置过程概述  
-----------
主服务器：  
> 开启二进制日志  
> 配置唯一的server-id，重启mysql    
> 获得master二进制日志文件名及位置  
> 创建一个用于slave和master通信的用户账号  
  
从服务器：  
> 配置唯一的server-id，重启mysql    
> 使用master分配的用户账号读取master二进制日志  
> 启用slave服务  
  

  
配置前准备
-----------
1. 主从数据库版本最好一致  
2. 主从数据库内数据保持一致  
3. 每个库单独一台机器为佳
  
  
主数据库：182.92.172.80 /linux        
从数据库：123.57.44.85 /linux   
  


master配置
-----------
1.修改mysql配置  
   
找到主数据库的配置文件my.cnf(或者my.ini)，在[mysqld]部分插入如下两行，注释的部分忽略：  
> [mysqld]  
> log-bin=mysql-bin #开启二进制日志  
> server-id=1 #设置server-id  
> #log-bin-index=master-bin.index #不配这个  
>    
> #注意下面两参数如果不配，就是按grant的用户时给的可同步权限来同步所有可同步的。  
> #要同步的mstest数据库，要同步多个数据库，就多加几个replicate-db-db=数据库名  
> #binlog-do-db=mstest  
> #binlog-ignore-db=mysql  #要忽略的数据库  
　　　　
  
  
2.重启mysql，创建用于同步的用户账号  
   
打开mysql会话  
> shell>mysql -hlocalhost -uname -ppassword  
  
创建用户并授权：用户：rel1，密码：slavepass  
> CREATE USER 'repl'@'123.57.44.85' IDENTIFIED BY 'slavepass'; #创建用户  
> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'123.57.44.85'; #分配权限   
> flush privileges;  #刷新权限   
  
  
3.查看master状态，记录二进制文件名(mysql-bin.000003)和位置(73)，在从库配置中会用到。  
> SHOW MASTER STATUS;  
> +------------------+----------+--------------+------------------+  
> | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |  
> +------------------+----------+--------------+------------------+  
> | mysql-bin.000003 | 73       | test         | manual,mysql     |  
> +------------------+----------+--------------+------------------+  
  
  
  
  
slave配置  
-----------
1.修改mysql配置  

同样找到my.cnf配置文件，添加server-id，注释的部分忽略
> [mysqld]  
> server-id=2 #设置server-id，必须唯一  
> #relay-log-index=slave-relay-bin.index  #不用配
> #relay-log=slave-relay-bin   #不用配
>  
> #下面的配置和master中的配置类似  
> #要同步的mstest数据库，要同步多个数据库，就多加几个replicate-db-db=数据库名  
> #replicate-do-db=mstest  
> #replicate-ignore-db=mysql　 //要忽略的数据库　  

  
2.重启mysql后，打开mysql会话，执行同步SQL语句  
注：该语句需要 (主服务器主机名，登陆凭据，二进制文件的名称和位置)，可以在主库中通过 SHOW MASTER STATUS; 来查看  

> CHANGE MASTER TO MASTER_HOST='182.92.172.80', MASTER_USER='rep1', MASTER_PASSWORD='slavepass', MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=73;  
注：如果端口不是3306，也可单独指定，在上面命令中拼入 MASTER_PORT=3306, 即可


3.启动slave同步进程：  
> start slave;  
  

4.查看slave状态： 
> show slave status\G;  
> *************************** 1. row ***************************  
>               Slave_IO_State: Waiting for master to send event  
>                  Master_Host: 182.92.172.80  
>                  Master_User: rep1  
>                  Master_Port: 3306  
>                Connect_Retry: 60  
>              Master_Log_File: mysql-bin.000013  
>          Read_Master_Log_Pos: 11662  
>               Relay_Log_File: mysqld-relay-bin.000022  
>                Relay_Log_Pos: 11765  
>        Relay_Master_Log_File: mysql-bin.000013  
>             Slave_IO_Running: Yes  
>            Slave_SQL_Running: Yes  
>              Replicate_Do_DB:   
>          Replicate_Ignore_DB:   
>        ...  

当 Slave_IO_Running 和 Slave_SQL_Running 都为YES的时候就表示主从同步设置成功了。
  
主要关注的参数：  
Slave_IO_Running: Yes   
Slave_SQL_Running: Yes   
  
还需关注：   
Seconds_Behind_Master: 0 //为主从延迟的时间   
Last_IO_Errno: 0   
Last_IO_Error:   
Last_SQL_Errno: 0   
Last_SQL_Error:  
  
  
  
接下来就可以进行一些验证了，比如在主master数据库的test数据库的一张表中插入一条数据，在slave的test库的相同数据表中查看是否有新增的数据即可验证主从复制功能是否有效，还可以关闭slave（mysql>stop slave;）,然后再修改master，看slave是否也相应修改（停止slave后，master的修改不会同步到slave），就可以完成主从复制功能的验证了。  
























参考地址：
-----------
https://blog.csdn.net/bohu83/article/details/81086474  (innodb)  


mysql存储引擎
------------
myisam  
innodb  




innodb
------------
如何存储表?  

MySQL 使用 InnoDB 存储表时，会将表的定义和数据索引等信息分开存储，其中前者存储在 .frm 文件中，后者存储在 .ibd 文件中，这一节就会对这两种不同的文件分别进行介绍。  

.frm  
无论在 MySQL 中选择了哪个存储引擎，所有的 MySQL 表都会在硬盘上创建一个 .frm 文件用来描述表的格式或者说定义； .frm 文件的格式在不同的平台上都是相同的。  

.ibd 文件  
InnoDB 中用于存储数据的文件总共有两个部分，一是系统表空间文件，包括 ibdata1、 ibdata2 等文件，其中存储了 InnoDB 系统信息和用户数据库表数据和索引，是所有表公用的。
当打开innodb_file_per_table选项时，
.ibd文件就是每一个表独有的表空间，文件存储了当前表的数据和相关的索引数据。

补充：  
innodb存储引擎会初始化一个名为ibdata1的表空间文件，默认情况下，这个文件会存储所有表的数据，以及我们所熟知但看不到的系统表sys_tables、sys_columns、sys_indexes 、sys_fields等。  
当打开innodb_file_per_table选项时，每个库中的每个表会有一个单独的.ibd文件。


myisam
------------
> .frm文件：存储数据表的框架结构  
> .MYD文件：即MY Data，表数据文件    
> .MYI文件：即MY Index，索引文件   











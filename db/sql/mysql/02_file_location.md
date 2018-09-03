  
MySQL数据文件介绍及存放位置  
  
  
一、MySQL数据库文件介绍
--------------
MySQL的每个数据库都对应存放在一个与数据库同名的文件夹中，MySQL数据库文件包括MySQL（server）所建数据库文件和MySQL（server）所用存储引擎创建的数据库文件。  
  
1、MySQL（server）创建并管理的数据库文件：  
  
.frm文件：存储数据表的框架结构，文件名与表名相同，每个表对应一个同名frm文件，与操作系统和存储引擎无关，即不管MySQL运行在何种操作系统上，使用何种存储引擎，都有这个文件。  
  
除了必有的.frm文件，根据MySQL所使用的存储引擎的不同（MySQL常用的两个存储引擎是MyISAM和InnoDB），存储引擎会创建各自不同的数据库文件。  
  

2、MyISAM数据库表文件：  
  
> .MYD文件：即MY Data，表数据文件  
  
> .MYI文件：即MY Index，索引文件  
  
> .log文件：日志文件  
  

3、InnoDB采用表空间（tablespace）来管理数据，存储表数据和索引  
  
InnoDB数据库文件（即InnoDB文件集，ib-file set）：  
  
ibdata1、ibdata2等：系统表空间文件，存储InnoDB系统信息和用户数据库表数据和索引，所有表共用  
  
.ibd文件：单表表空间文件，每个表使用一个表空间文件（file per table），存放用户数据库表数据和索引  
  
日志文件： ib_logfile1、ib_logfile2  
  
  
二、MySQL数据库存放位置：  
-------------
1、MySQL如果使用MyISAM存储引擎，数据库文件类型就包括.frm、.MYD、.MYI，默认存放位置是C:\Documentsand Settings\All Users\Application Data\MySQL\MySQL Server 5.1\data  
  
2、MySQL如果使用InnoDB存储引擎，数据库文件类型就包括.frm、ibdata1、.ibd，存放位置有两个  
  
.frm文件默认存放位置是C:\Documents and Settings\All Users\ApplicationData\MySQL\MySQL Server 5.1\data  
  
ibdata1、.ibd文件默认存放位置是MySQL安装目录下的data文件夹    
  

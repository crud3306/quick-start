

创建表
----------
```sql
CREATE TABLE IF NOT EXISTS user(
	id int(10) UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
	username VARCHAR(24) NOT NULL UNIQUE,
	password VARCHAR(8) NOT NULL DEFAULT '',
	age tinyint(3) UNSIGNED NOT NULL DEFAULT 0,
	create_at INT(10) UNSIGNED NOT NULL DEFAULT 0
)ENGINE=InnoDB DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```
  
常用命令   
----------

(一) 连接MYSQL：
----------
格式： mysql -h主机地址 -u用户名 －p用户密码 -P端口号  

1、例1：连接到本机上的MYSQL  
首先在打开DOS窗口，然后进入mysql安装目录下的bin目录下，例如： D:/mysql/bin，再键入命令mysql -uroot -p，回车后提示你输密码，如果刚安装好MYSQL，超级用户root是没有密码的，故直接回车即可进入到MYSQL中了，MYSQL的提示符是：mysql>  
  
2、例2：连接到远程主机上的MYSQL (远程：IP地址)  
假设远程主机的IP为：10.0.0.1，用户名为root,密码为123。则键入以下命令： 
> mysql -h10.0.0.1 -uroot -p123  
（注：u与root可以不用加空格，其它也一样） 
  
3、退出MYSQL命令  
> exit （回车）  
  
  
(二) 修改密码：  
----------
格式：mysqladmin -u用户名 -p旧密码 password 新密码  

1、例1：给root加个密码123。首先在DOS下进入目录C:/mysql/bin，然后键入以下命令：  
> mysqladmin -uroot -password 123   
注：因为开始时root没有密码，所以-p旧密码一项就可以省略了。  
  
2、例2：再将root的密码改为456   
> mysqladmin -uroot -pab12 password 456  
  
  
(三) 增加新用户：  
----------
（注意：和上面不同，下面的因为是MYSQL环境中的命令，所以后面都带一个分号作为命令结束符）  

格式：grant select on 数据库.* to 用户名@登录主机 identified by "密码";    

例1、增加一个用户test1密码为abc，让他可以在任何主机上登录，并对所有数据库有查询、插入、修改、删除的权限。首先用以root用户连入MYSQL，然后键入以下命令：   
> grant select,insert,update,delete on *.* to test2@localhost identified by "abc";  
如果你不想test2有密码，可以再打一个命令将密码消掉。 
> grant select,insert,update,delete on mydb.* to test2@localhost identified by "";  
  
  
(四) 其它常用命令  
----------
1、显示数据库列表：  
> show databases;   
刚开始时才两个数据库：mysql和test。mysql库很重要它里面有MYSQL的系统信息，我们改密码和新增用户，实际上就是用这个库进行操作。  
  
2、显示库中的数据表：  
> use mysql； //打开库 show tables;  
  
3、显示数据表的结构：  
> describe 表名;  
  
4、建库：  
> create database 库名;  
  
建库同时指定库的字符集  
> CREATE DATABASE `库名` CHARACTER SET utf8 COLLATE utf8_general_ci;  
  
5、建表：  
use 库名；  
create table 表名 (字段设定列表)；  
  
例：  
```sql
CREATE TABLE IF NOT EXISTS user(
id int(10) UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
username VARCHAR(24) NOT NULL UNIQUE,
password VARCHAR(8) NOT NULL DEFAULT '',
age tinyint(3) UNSIGNED NOT NULL DEFAULT 0,
create_at INT(10) UNSIGNED NOT NULL DEFAULT 0
)ENGINE=InnoDB DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```
  
  
5.1、显示建表时的sql语名  
> show create table 表名;  
  
5.2、显示表结构  
> desc 表名;  
  
6、删库和删表:  
> drop database 库名; drop table 表名；  
  
7、将表中记录清空：  
> delete from 表名;  // 不加where条件，即清空表中所有记录
> truncate table 表名;  // 清空表中所有记录，同时自增id重置为1
  
8、显示表中的记录：  
> select * from 表名;  
  
9、移动表字段位置  
> alter table 表名 modify 字段名 字段类型 after 字段  

例：  
> alter table `promotion_activity` modify `condition` text after `intro`;  
  
10、修改表字段  
> alter table `表名` modify column app_name varchar(20) COMMENT '应用的名称';  
  
  
导入导出sql
----------
导出sql脚本    
mysqldump -u 用户名 -p 数据库名 > 存放位置  
> mysqldump -u root -p test > c:/a.sql  
  
导入sql脚本   
mysqldump -u 用户名 -p 数据库名 < 存放位置    
> mysqldump -u root -p test < c:/a.sql  
注意,test数据库必须已经存在  
  
MySQL导出导入命令的用例  
1.导出整个数据库  
mysqldump -u 用户名 -p 数据库名 > 导出的文件名   
> mysqldump -u wcnc -p smgp_apps_wcnc > wcnc.sql   
  
2.导出一个表   
mysqldump -u 用户名 -p 数据库名表名> 导出的文件名  
> mysqldump -u wcnc -p smgp_apps_wcnc users> wcnc_users.sql  
  
3.导出一个数据库结构  
> mysqldump -u wcnc -p -d --add-drop-table smgp_apps_wcnc >d:wcnc_db.sql  
-d 表示只导结构，不含数据  
--add-drop-table 在每个create语句之前增加一个drop table  
  
4. 通过source命令导入数据库  
命令行连接mysql  
> mysql -u root -p  
> use 数据库;  
  
然后使用source命令,后面参数为脚本文件(如这里用到的.sql)  
> source /home/qianym/xxx.sql  
  
  
  
显示mysql编码：  
----------
> show variables like '%char%';    
  




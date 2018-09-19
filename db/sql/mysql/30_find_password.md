 
找回丢失mysql root密码  
    
一、启动修改丢失的mysql单实例root密码方法  
------------- 
1.首先停止mysql  
> /etc/init.d/mysql stop  
  
2.使用--skip-grant-tables启动mysql，忽略授权登录验证  
> mysqld_safe --skip-grant-tables --user=mysql &   --提示：在启动时加--skip-grant-tables 参数，表示忽略授权验证  

3.进入数据库系统   
shell>mysql  
  
4.修改mysqlroot密码：update    
mysql>update mysql.user set password=password("123456") where user='root' and host='localhost';  
mysql>flush privileges;  

shell>mysqladmin -uroot -p123456 shutdown  #使用安全模式关闭数据库
  
5.重新启动mysql  
shell>/etc/init.d/mysql start  
shell>mysql -uroot -p123456  
  
  
二、多实例丢失密码的方法  
--------------
1.关闭mysql  
> mysqld_mulit stop   
  
2.启动时加--skip-grant-tables参数  
mysqld_safe --defaults-files=/data/mysql/mysql3377/mysql3377.cnf --skip-grant-tables &  
mysql -uroot -p -S /tmp/mysql3377.sock <==登录时空密码  

3.修改密码方法：  
update mysql.user set password=password("123456") where user='root';  
flush privileges;  

4.重启服务用新密码登录  
killall mysqld  
mysqld_mulit restart 3377  
  
  
三、不重启mysqld的方法  
--------------
1、首先得有一个可以拥有修改权限的mysql数据库账号，当前的mysql实例账号（较低权限的账号，比如可以修改test数据库）或者其他相同版本实例的账号。把data/mysql目录下面的user表相关的文件复制到data/test目录下面。  
[root@localhost mysql]# cp mysql/user.* test/  
[root@localhost mysql]# chown mysql.mysql test/user.*  
  
2、使用另一个较低权限的账号链接数据库，设置test数据库中的user存储的密码数据。  
[root@localhost mysql]# mysql -utest -p12345  
Welcome to the MySQL monitor.  Commands end with ; or \g.  
Your MySQL connection id is 17  
Server version: 5.5.25a-log Source distribution  
  
Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.  
  
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.  
  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  
mysql> use test  
Reading table information for completion of table and column names  
You can turn off this feature to get a quicker startup with -A  
  
Database changed  
mysql> update user set password=password('yayun') where user='root';  
Query OK, 0 rows affected (0.00 sec)  
Rows matched: 5  Changed: 0  Warnings: 0  
  
mysql>  
  
3、把修改后的user.MYD和user.MYI复制到mysql目录下，记得备份之前的文件。  
mv mysql/user.MYD mysql/user.MYD.bak  
mv mysql/user.MYI mysql/user.MYI.bak  
cp test/user.MY* mysql/  
chown mysql.mysql mysql/user.*  
  
4、查找mysql进程号，并且发送SIGHUP信号，重新加载权限表。  
[root@localhost mysql]# pgrep -n mysql  
2184  
[root@localhost mysql]#  
[root@localhost mysql]# kill -SIGHUP 2184  
  
5.登陆测试  
[root@localhost mysql]# mysql -uroot -p
 
  

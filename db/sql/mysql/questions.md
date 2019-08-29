


常见问题汇总
============


QLSTATE[HY000] [2013] Lost connection to MySQL server at 'reading initial communication packet', system error: 111
-------------
ip/port不对，或账号无权限


SQLSTATE[28000] [1045] Access denied for user 'educloud_w'@'xxx.xx.xx.xx' (using password: YES)"
-------------
账号对某个库没有写权限




查看mysql连接超时间
-------------
show global variables like 'connect_timeout';
show global variables like '%timeout';




sql查询时间戳转年月日
------------
```
SELECT uid, name, from_unixtime(create_ts) FROM  `user` 
```



mysql cli 中文乱码
------------
```
先看数据库的相关编码
show variables like 'character_set_%';
+--------------------------+-----------------------------------+
| Variable_name            | Value                             |
+--------------------------+-----------------------------------+
| character_set_client     | latin1                            |
| character_set_connection | latin1                            |
| character_set_database   | utf8                              |
| character_set_filesystem | binary                            |
| character_set_results    | latin1                            |
| character_set_server     | utf8                              |
| character_set_system     | utf8                              |
| character_sets_dir       | /home/mysql/mysql/share/charsets/ |
+--------------------------+-----------------------------------+

受客户端的连接相关编码影响，下面三项：
character_set_client
character_set_connection
character_set_results 

而这三项是可以通过
set names utf8;
set names gbk;
来可以设置的！另外也说明当前连接的客户端的编码情况没有影响到数据库服务器本身的编码情况。


+--------------------------+-----------------------------------+
| Variable_name            | Value                             |
+--------------------------+-----------------------------------+
| character_set_client     | utf8                              |
| character_set_connection | utf8                              |
| character_set_database   | utf8                              |
| character_set_filesystem | binary                            |
| character_set_results    | utf8                              |
| character_set_server     | utf8                              |
| character_set_system     | utf8                              |
| character_sets_dir       | /home/mysql/mysql/share/charsets/ |
+--------------------------+-----------------------------------+
```






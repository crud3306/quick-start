
安装
--------------
cd /usr/local/src  
#去这个地址，找到自已需要的各种版本http://download.redis.io/releases/  
wget http://download.redis.io/releases/redis-stable.tar.gz  
tar zxf redis-stable.tar.gz  
cd redis-stable  
#make distclean  
make  
make install  
或者指定安装目录  
make PREFIX=/usr/local/redis install  
  
  
更改redis.conf
---------------
daemonize yes  
#如果是bind 127.0.0.1则只能在本机边接该redis-server，可以注释掉  
#bind 127.0.0.1   
#默认情况下requirepass是注释的，不需密码即可连接，可以开启后台的参数即为可自定义的密码  
#requirepass 123456  
  
  
redis的bin主要文件  
---------------
redis-cli  
redis-server  

  
启动redis server  
----------------
/xxx/bin/redis-server /xxx/redis.conf  
  
查看redis是否启动  
---------------
ps -ef |grep redis  
lsof -i :6379  

redis-cli连接redis  
---------------
/xxx/bin/redis-cli -p 6379  
如果开启的密码验证，则需  
/xxx/bin/redis-cli -p 6379 -a 你的密码  

ping  
---------------
ping，返回 PONG  
  
set k1 hello  
get k1  
  
  
退出cli  
---------------
先执行 SHUTDOWN  
然后eixt  
  
  
查看redis的版本 有两种方式：
---------------
1. redis-server --version 和 redis-server -v   
得到的结果是：Redis server v=2.6.10 sha=00000000:0 malloc=jemalloc-3.2.0 bits=32  
  
2. redis-cli --version 和 redis-cli -v  
　得到的结果是：redis-cli 2.6.10  

严格上说：通过　redis-cli 得到的结果应该是redis-cli 的版本，但是 redis-cli 和 redis-server 一般都是从同一套源码编译出的，所以应该是一样的。  
    
   
   

  
redis其它知识普及  
---------------
单进程  
---------------
单进程模型来处理客户端请求，对读写等事件的响应是通过epoll函数包装做到的。redis实际处理速度完全依靠主进程的执行效率。  
epoll是linux内核为处理大批量文件描述符而作了改进，是linux下多路复用io接口select\poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统cpu利用率  
  
默认16个数据库，类似数组下表从零开始，初始默认使用零号库   
-----------------
可查看redis.conf文件  
databases 16  
从0开始到15  
  
select命令切换数据库
-----------------
select 0  
select 0  
...  
select 15  
  
dbsize 查看当前数据库的key的数量  
-----------------
  
  
key相关操作  
-----------------
key *  
查看当前库的所有key  
  
key aaa?  
查看当前库的所有以字串aaa开头的key，问号只区配这个字符，所以出来的key可以是aaa1, aaab, aaa3等等  
  
exists 某个key   
-----------------
查看某个key是否存在 （1存在 ；0不存在）  
  
type 某个key   
-----------------
查看某个key是什么类型  
  
move key db   
-----------------  
移动当前库的某个key到对应的库  
  
例：移动k3到2号库，注：因从0开始，所以实际称到的是3号库  
move k3 2  
  
ttl 某个key  
-----------------
查看某key还有多长时间过期。-1表示永不过期，-2表示已过期，>0的其它值表示还有多少秒  
例：  
ttl aaa  
  
expire 某个key 秒种  
-----------------  
为给定的key设置有效时间，即设置多少秒后过期  
例：  
expire aaa 10  
  
注：过期后会该key会自动删除，再次获取时是nil  
  
  
set 某个key value  
-----------------  
为某个key设置值  
  
注：多次设置同一个key，会覆盖之前的值  
  
  
del 某个key  
----------------- 
删除某个key  
  
  
flushdb 清空当前库  
-----------------  
  
  
flushall 清空全部库 
-----------------  
  
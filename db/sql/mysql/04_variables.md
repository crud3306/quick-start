
mysql 会话变量 全局变量  
  
会话变量
-------------
查看全部会话变量，结果较多，有327个。  
> show session variables;  
  
查看以指定字符开头的会话变量  
> show session variables like 'auto%';  
也可以  
> select @@session.autocommit;  
  
可以修改  
> set autocommit = 'off';  
或者  
> set @@session.autocommit = 'off';  
  
再次查看  
> show session variables like 'auto%';  
发现已改，但是注意，这个更改只是临时更改本次链接。新起一个连接则是没更改的。  
  




全局变量  
-------------
查看全部全局变量，大概316个  
> show global variables;  
  
查看指定的  
> show global variables like 'auto%';  
也可以  
> select @@global.autocommit;  
  
执行  
> set global autocommit ='off';  
> set @@global.autocommit ='off';  
  
查看  
> show global variables like 'auto%';  
发现已改了  
  
这次更改是全部都改了，即使新起一个链接，也是更改。  
  

@todo
显示系统变量和值  
SHOW VARIABLES;  
  
显示系统变量名包含conn的值  
SHOW VARIABLES like '%conn%';   
  
  
参考地址：  
----------
https://www.bilibili.com/video/av22645209/?p=3  
  

  


醒置文件
---------
xxx/conf/zoo1.cfg  
```conf
tickTime=2000
dataDir=./data/zookeeper1
clientPort=2181
initLimit=5
syncLimit=2
server.1=localhost:2881:3881
server.2=localhost:2882:3882
server.3=localhost:2883:3883
```
zoo2.cfg、zoo3.cfg同zoo1.cfg基本相同，除了dataDir与clientPort不同。  
xxx/conf/zoo2.cfg   
xxx/conf/zoo3.cfg   
  

需要一个myid
---------
vi xxx/data/zookeeper1/myid  
vi xxx/data/zookeeper2/myid  
vi xxx/data/zookeeper3/myid  
分别设为1、2、3  
  
  
  
启动服务
--------
xx/bin/zkServer.sh start zoo1.cfg  
xx/bin/zkServer.sh start zoo2.cfg  
xx/bin/zkServer.sh start zoo3.cfg  

查看服务状态
--------
xx/bin/zkServer.sh status zoo1.cfg  
xx/bin/zkServer.sh status zoo2.cfg  
xx/bin/zkServer.sh status zoo3.cfg  
  
关闭服务  
--------
xx/bin/zkServer.sh stop zoo1.cfg  
xx/bin/zkServer.sh stop zoo2.cfg  
xx/bin/zkServer.sh stop zoo3.cfg  
  

命令行连接zookeeper  
--------
xx/bin/zkCli.sh -server 127.0.0.1:2181  

查看命令帮助  
help  
  
查看根目录下的文件，类似于linux里的文件系统    
ls /  
假如有zoo1、test  

查看zoo1  
get /zoo1  

创建新的  
create /test001  abc123456
   
查看刚创建的test001  
get /test001  
  

创建新的子级文件  
create /test001/t0001  56789
   
查看刚创建的test001  
get /test001/t0001  
  
退出命令行  
quit  
  



































centos安装MongoDB 
=============

1、下面安装MongoDB，先下载：
-------------
```
cd /usr/src  
wget http://fastdl.mongodb.org/linux/mongodb-linux-x86_64-2.6.4.tgz  
```

2、解压，进入目录：
-------------
```
tar -zxvf mongodb-linux-x86_64-2.6.4.tgz -C /usr/src
mv mongodb-linux-x86_64-2.6.4 mongodb 
cd mongodb
```

3、创建数据库和日志的目录：
-------------
```
mkdir log
mkdir db
```

4、以后台运行方式启动：
-------------
```shell
./bin/mongod --dbpath=./db --logpath=./log/mongodb.log --fork --auth

# 会显示如下内容：
about to fork child process, waiting until server is ready for connections.
forked process: 4623
child process started successfully, parent exiting
```

5、设置开机启动：
-------------
```shell
echo "/usr/local/mongodb/bin/mongod --dbpath=/usr/local/mongodb/db --logpath=/usr/local/mongodb/log/mongodb.log --fork --auth" >> /etc/rc.local

# ok，搞定，然后可以参看下端口:
netstat -nalupt | grep mongo
tcp 0 0 0.0.0.0:27017 0.0.0.0:* LISTEN 4623/./bin/mongod
```

命令行连接，测试
-------------
> cd /usr/local/mongodb/bin  
> ./mongo  
查看拥有的库
> show dbs
> use local




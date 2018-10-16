
参考地址：
--------
https://www.cnblogs.com/zhoujinyi/p/5570024.html  
https://www.cnblogs.com/zhoujinyi/p/6430116.html  


Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案，当用Redis做Master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换，而Redis-sentinel本身也是一个独立运行的进程，它能监控多个master-slave集群，发现master宕机后能进行自动切换，更多的信息见前一篇说明。它的主要功能有以下几点：  
```
1，不时地监控redis是否按照预期良好地运行;
2，如果发现某个redis节点运行出现状况，能够通知另外一个进程(例如它的客户端);
3，能够进行自动切换。当一个master节点不可用时，能够选举出master的多个slave(如果有超过一个slave的话)中的一个来作为新的master,其它的slave节点会将它所追随的master的地址改为被提升为master的slave的新地址。 
```


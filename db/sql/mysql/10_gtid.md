GTID
==========
GTID(Global Transaction ID)是MySQL5.6引入的功能，可以在集群全局范围标识事务，用于取代过去通过binlog文件偏移量定位复制位置的传统方式。
借助GTID，在发生主备切换的情况下，MySQL的其它Slave可以自动在新主上找到正确的复制位置，这大大简化了复杂复制拓扑下集群的维护，也减少了人为设置复制位置发生误操作的风险。
另外，基于GTID的复制可以忽略已经执行过的事务，减少了数据发生不一致的风险。
  
参考地址：
----------
https://dbaplus.cn/news-11-857-1.html


GTID长什么样
-----------
根据官方文档定义，GTID由source_id加transaction_id构成。
> GTID = source_id:transaction_id

上面的source_id指示发起事务的MySQL实例，值为该实例的server_uuid。server_uuid由MySQL在第一次启动时自动生成并被持久化到auto.cnf文件里，transaction_id是MySQL实例上执行的事务序号，从1开始递增。 例如：
> e6954592-8dba-11e6-af0e-fa163e1cf111:1

一组连续的事务可以用'-'连接的事务序号范围表示。例如
> e6954592-8dba-11e6-af0e-fa163e1cf111:1-5

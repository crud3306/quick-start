
在php中使用redis cluster 集群



目前我们用到的 php 的 redis 扩展 主要有2个:
==========
1 最常用的 phpredis, 它是用c写的php的高效扩展：https://github.com/phpredis/phpredis，

2 predis, 它是用php代码写的，也用的蛮多的：https://github.com/nrk/predis。


我们分别看下他们在集群中的用法。

phpredis
==========
phpredis的安装
----------
我Mac上是有安装过phpredis扩展的，但是是2.* 版本，是不支持cluster的，所以需要升级到3.0扩展。我记录下升级过程：
```
~ git clone git@github.com:phpredis/phpredis.git
~ cd phpredis
~ git fetch
~ git checout feature/redis_cluster #切换到cluster分支
~ phpize
~ ./configure
~ make
~ make install
Installing shared extensions:     /usr/local/php5/lib/php/extensions/no-debug-non-zts-20131226/
```
这样就可以用了。如果你是第一次安装redis扩展，还需要在php.ini中加上：
> extension=redis.so

3.0版本的redis扩展已经安装好了。我们可以重启一下php-fpm。
> sudo kill -USR2 `cat /usr/local/var/run/php-fpm.pid`

官方的文档太少了：
https://github.com/phpredis/phpredis/blob/feature/redis_cluster/cluster.markdown。就这一个。

就根据这个文档来学习简单学习下吧：

先完成初始化连接到redis cluster服务器：
```php
$obj_cluster = new RedisCluster(NULL, ['192.168.33.13:7000', '192.168.33.13:7001', '192.168.33.13:7002', '192.168.33.13:7003', '192.168.33.13:7004']);
var_dump($obj_cluster);
```
第一个参数传NULL 别问我，我也不知道为啥。反正文档没找到，这篇也没看懂。 
第二个参数是我们需要连接的redis cluster的master服务器列表。我们有5个master，就填5个。

打印结果如下：
```
class RedisCluster#5 (0) {}
```
一个RedisCluster 类资源。表示redis 已经连接成功了。

那么，我们就可以实用之前redis的方法来尝试了：
```
$obj_cluster->set('name1', '1111');
$obj_cluster->set('name2', '2222');
$obj_cluster->set('name3', '333');
$name1 = $obj_cluster->get('name1');
$name2 = $obj_cluster->get('name2');
$name3 = $obj_cluster->get('name3');
var_dump($name1, $name2, $name3);die;
```

结果如下：
```
string(4) "1111"
string(4) "2222"
string(3) "333"
```
很完美，没啥问题。而且，他是直接就给结果了。

前面的redis cluster 的学习，我们知道name1, name2, name3 是3个key , 会按照算法，分配到3个slot上，有可能分到3台服务器上。

我们连接客户端看下：
```
➜  redis-cli -h 192.168.33.13 -p 7009 -c
192.168.33.13:7009> get name1
-> Redirected to slot [12933] located at 192.168.33.13:7003
"1111"
192.168.33.13:7003> get name2
-> Redirected to slot [742] located at 192.168.33.13:7000
"2222"
192.168.33.13:7000> get name3
-> Redirected to slot [4807] located at 192.168.33.13:7001
"333"
192.168.33.13:7001>
```
客户端是有跳转的，而php的扩展phpredis直接就给出结果了，这点很赞。

phpredis的使用
我们继续看这个蛋疼的文档，它还提供了一种连接方式：
```
// Connect and specify timeout and read_timeout
$obj_cluster = new RedisCluster(
    NULL, Array("host:7000", "host:7001", 1.5, 1.5);
);
```
后面加入了timeout和read_timeout功能。就是加到master列表的后面。

timeout表示连接redis的最长时间，这里设为1.5秒，表示超过1.5秒要是还没连接成功就返回false 。

read_timeout表示连接redis成功后，读取一个key的超时时间，有时候读取一个key 可能value比较大，读取需要很长时间，这里设置1.5秒，表示要是过了1.5秒还没读取到数据就返回false。

好。我们试一下：
```
$obj_cluster = new RedisCluster(NULL, ['192.168.33.13:7000', '192.168.33.13:7001', '192.168.33.13:7002', '192.168.33.13:7003', '192.168.33.13:7004', 1.5, 1.5]);
```
在master列表后面加入了2个参数。其实的操作几乎一样。

我尝试的只用了一个master去连接，发现也可以，并没什么差别？？？

如下：
```
$obj_cluster = new RedisCluster(
                              NULL, 
                              ['192.168.33.13:7000', 1.5, 1.5]);
$obj_cluster->set('name1', '1111');
$name1 = $obj_cluster->get('name1');
var_dump($name1);
//输出
string(4) "1111"
```
只填一个也可以。我在想，它是不是自己就能识别啊。不需要填这么多啊。但是，我没找到相关的文档，证明我的观点。

而且，我换一个slave来连接，写也可以成功！！！
```
//7009是个slave
$obj_cluster = new RedisCluster(
                                NULL, 
                                ['192.168.33.13:7009', 1.5, 1.5]);
$obj_cluster->set('name1', '4555');
$name1 = $obj_cluster->get('name1');
var_dump($name1);
//输出
string(4) "4555"
```
好吧。我姑且认为，它会自动内部判断主从。还蛮厉害的。

还有其他的功能和命令，例如：zadd、lpop、hget等。就不说了。



predis
========
predis的下载安装
--------
predis是一套用php代码写的php连接redis的扩展，用到了命名空间，我之前用过，其实效率还可以，比phpredis稍微低一点，但是它由于接口众多，所以功能很强大。

网页redis管理工具phpRedisAdmin （https://github.com/ErikDubbelboer/phpRedisAdmin）就是用的predis作为连接的。

好，先下载看看。可以用composer 或者 git clone。我这里用git clone 吧。最新的稳定版本是 v1.03
```
➜  redis  git clone git@github.com:nrk/predis.git
➜  redis  cd predis
➜  predis git:(master) git checkout v1.0.3 #切换到最新的文档版本
➜  predis git:(84060b9)
```
OK，我新建一个test.php 文件，我们再测试一下cluster业务。
```php
<?php
require 'predis/autoload.php';
$servers = [
    'tcp://192.168.33.13:7000',
    'tcp://192.168.33.13:7001',
    'tcp://192.168.33.13:7002',
    'tcp://192.168.33.13:7003',
    'tcp://192.168.33.13:7004',
    ];
$options = ['cluster' => 'redis'];
$client = new Predis\Client($servers, $options);
$client->set('name1', '1111111');
$client->set('name2', '2222222');
$client->set('name3', '3333333');
$name1 = $client->get('name1');
$name2 = $client->get('name2');
$name3 = $client->get('name3');
var_dump($name1, $name2, $name3);die;
```
打印结果：
```
➜  redis  php test.php
string(7) "1111111"
string(7) "2222222"
string(7) "3333333"
```

当然，它也类似，只填写一个也是可以的，目前发现没什么坑，也没文档说明为啥可以：
```
require 'predis/autoload.php';
$servers = [
    'tcp://192.168.33.13:7000',
    ];
$options = ['cluster' => 'redis'];
$client = new Predis\Client($servers, $options);
$client->set('name1', '1111111');
```
其他的一些用法也是类似的：
```
$client->hset('name77', 'name', 'yang');
$b = $client->hget('name77', 'name');
```
其他的都和之前的redis 一样使用。






/*1.Connection*/
-----------------
```
$redis = new Redis();
$redis->connect('127.0.0.1',6379,1);//短链接，本地host，端口为6379，超过1秒放弃链接
$redis->open('127.0.0.1',6379,1);//短链接(同上)
$redis->pconnect('127.0.0.1',6379,1);//长链接，本地host，端口为6379，超过1秒放弃链接
$redis->popen('127.0.0.1',6379,1);//长链接(同上)
$redis->auth('password');//登录验证密码，返回【true | false】
$redis->select(0);//选择redis库,0~15 共16个库
$redis->close();//释放资源
$redis->ping(); //检查是否还再链接,[+pong]
$redis->ttl('key');//查看失效时间[-1 | timestamps]
$redis->persist('key');//移除失效时间[ 1 | 0]
$redis->sort('key',[$array]);//返回或保存给定列表、集合、有序集合key中经过排序的元素，$array为参数limit等！【配合$array很强大】 [array|false]
```


/*2.共性的运算归类*/
-----------------
```
$redis->expire('key',10);//设置失效时间[true | false]
$redis->move('key',15);//把当前库中的key移动到15库中[0|1]

//string
$redis->strlen('key');//获取当前key的长度
$redis->append('key','string');//把string追加到key现有的value中[追加后的个数]
$redis->incr('key');//自增1，如不存在key,赋值为1(只对整数有效,存储以10进制64位，redis中为str)[new_num | false]
$redis->incrby('key',$num);//自增$num,不存在为赋值,值需为整数[new_num | false]
$redis->decr('key');//自减1，[new_num | false]
$redis->decrby('key',$num);//自减$num，[ new_num | false]
$redis->setex('key',10,'value');//key=value，有效期为10秒[true]
//list
$redis->llen('key');//返回列表key的长度,不存在key返回0， [ len | 0]
//set
$redis->scard('key');//返回集合key的基数(集合中元素的数量)。[num | 0]
$redis->sMove('key1', 'key2', 'member');//移动，将member元素从key1集合移动到key2集合。[1 | 0]
//Zset
$redis->zcard('key');//返回集合key的基数(集合中元素的数量)。[num | 0]
$redis->zcount('key',0,-1);//返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员。[num | 0]
//hash
$redis->hexists('key','field');//查看hash中是否存在field,[1 | 0]
$redis->hincrby('key','field',$int_num);//为哈希表key中的域field的值加上量(+|-)num,[new_num | false]
$redis->hlen('key');//返回哈希表key中域的数量。[ num | 0]
```
 


/*3.Server*/
-----------------
```
$redis->dbSize();//返回当前库中的key的个数
$redis->flushAll();//清空整个redis[总true]
$redis->flushDB();//清空当前redis库[总true]
$redis->save();//同步??把数据存储到磁盘-dump.rdb[true]
$redis->bgsave();//异步？？把数据存储到磁盘-dump.rdb[true]
$redis->info();//查询当前redis的状态 [verson:2.4.5....]
$redis->lastSave();//上次存储时间key的时间[timestamp]

$redis->watch('key','keyn');//监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断 [true]
$redis->unwatch('key','keyn');//取消监视一个(或多个) key [true]
$redis->multi(Redis::MULTI);//开启事务，事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令在一个原子时间内执行。
$redis->multi(Redis::PIPELINE);//开启管道，事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 EXEC 命令在一个原子时间内执行。
$redis->exec();//执行所有事务块内的命令，；【事务块内所有命令的返回值，按命令执行的先后顺序排列，当操作被打断时，返回空值 false】
```

 

/*4.String，键值对，创建更新同操作*/
-----------------
```
$redis->setOption(Redis::OPT_PREFIX,'hf_');//设置表前缀为hf_
$redis->set('key',1);//设置key=aa value=1 [true]
$redis->mset($arr);//设置一个或多个键值[true]
$redis->setnx('key','value');//key=value,key存在返回false[|true]
$redis->get('key');//获取key [value]
$redis->mget($arr);//(string|arr),返回所查询键的值
$redis->del($key_arr);//(string|arr)删除key，支持数组批量删除【返回删除个数】
$redis->delete($key_str,$key2,$key3);//删除keys,[del_num]
$redis->getset('old_key','new_value');//先获得key的值，然后重新赋值,[old_value | false]
```

 

/*5.List栈的结构,注意表头表尾,创建更新分开操作*/
-----------------
```
$redis->lpush('key','value');//增，只能将一个值value插入到列表key的表头，不存在就创建 [列表的长度 |false]
$redis->rpush('key','value');//增，只能将一个值value插入到列表key的表尾 [列表的长度 |false]
$redis->lInsert('key', Redis::AFTER, 'value', 'new_value');//增，将值value插入到列表key当中，位于值value之前或之后。[new_len | false]
$redis->lpushx('key','value');//增，只能将一个值value插入到列表key的表头，不存在不创建 [列表的长度 |false]
$redis->rpushx('key','value');//增，只能将一个值value插入到列表key的表尾，不存在不创建 [列表的长度 |false]
$redis->lpop('key');//删，移除并返回列表key的头元素,[被删元素 | false]
$redis->rpop('key');//删，移除并返回列表key的尾元素,[被删元素 | false]
$redis->lrem('key','value',0);//删，根据参数count的值，移除列表中与参数value相等的元素count=(0|-n表头向尾|+n表尾向头移除n个value) [被移除的数量 | 0]
$redis->ltrim('key',start,end);//删，列表修剪，保留(start,end)之间的值 [true|false]
$redis->lset('key',index,'new_v');//改，从表头数，将列表key下标为第index的元素的值为new_v, [true | false]
$redis->lindex('key',index);//查，返回列表key中，下标为index的元素[value|false]
$redis->lrange('key',0,-1);//查，(start,stop|0,-1)返回列表key中指定区间内的元素，区间以偏移量start和stop指定。[array|false]
```



/*6.Set，没有重复的member，创建更新同操作*/
-----------------
```
$redis->sadd('key','value1','value2','valuen');//增，改，将一个或多个member元素加入到集合key当中，已经存在于集合的member元素将被忽略。[insert_num]
$redis->srem('key','value1','value2','valuen');//删，移除集合key中的一个或多个member元素，不存在的member元素会被忽略 [del_num | false]
$redis->smembers('key');//查，返回集合key中的所有成员 [array | '']
$redis->sismember('key','member');//判断member元素是否是集合key的成员 [1 | 0]
$redis->spop('key');//删，移除并返回集合中的一个随机元素 [member | false]
$redis->srandmember('key');//查，返回集合中的一个随机元素 [member | false]
$redis->sinter('key1','key2','keyn');//查，返回所有给定集合的交集 [array | false]
$redis->sunion('key1','key2','keyn');//查，返回所有给定集合的并集 [array | false]
$redis->sdiff('key1','key2','keyn');//查，返回所有给定集合的差集 [array | false]
```


/*7.Zset，没有重复的member，有排序顺序,创建更新同操作*/
-----------------
```
$redis->zAdd('key',$score1,$member1,$scoreN,$memberN);//增，改，将一个或多个member元素及其score值加入到有序集key当中。[num | 0]
$redis->zrem('key','member1','membern');//删，移除有序集key中的一个或多个成员，不存在的成员将被忽略。[del_num | 0]
$redis->zscore('key','member');//查,通过值反拿权 [num | null]
$redis->zrange('key',$start,$stop);//查，通过(score从小到大)【排序名次范围】拿member值，返回有序集key中，【指定区间内】的成员 [array | null]
$redis->zrevrange('key',$start,$stop);//查，通过(score从大到小)【排序名次范围】拿member值，返回有序集key中，【指定区间内】的成员 [array | null]
$redis->zrangebyscore('key',$min,$max[,$config]);//查，通过scroe权范围拿member值，返回有序集key中，指定区间内的(从小到大排)成员[array | null]
$redis->zrevrangebyscore('key',$max,$min[,$config]);//查，通过scroe权范围拿member值，返回有序集key中，指定区间内的(从大到小排)成员[array | null]
$redis->zrank('key','member');//查，通过member值查(score从小到大)排名结果中的【member排序名次】[order | null]
$redis->zrevrank('key','member');//查，通过member值查(score从大到小)排名结果中的【member排序名次】[order | null]
$redis->ZINTERSTORE();//交集
$redis->ZUNIONSTORE();//差集
```



/*8.Hash，表结构，创建更新同操作*/
-----------------
```
$redis->hset('key','field','value');//增，改，将哈希表key中的域field的值设为value,不存在创建,存在就覆盖【1 | 0】
$redis->hget('key','field');//查，取值【value|false】
$arr = array('one'=>1,2,3);$arr2 = array('one',0,1);
$redis->hmset('key',$arr);//增，改，设置多值$arr为(索引|关联)数组,$arr[key]=field, [ true ]
$redis->hmget('key',$arr2);//查，获取指定下标的field，[$arr | false]
$redis->hgetall('key');//查，返回哈希表key中的所有域和值。[当key不存在时，返回一个空表]
$redis->hkeys('key');//查，返回哈希表key中的所有域。[当key不存在时，返回一个空表]
$redis->hvals('key');//查，返回哈希表key中的所有值。[当key不存在时，返回一个空表]
$redis->hdel('key',$arr2);//删，删除指定下标的field,不存在的域将被忽略,[num | false]
```



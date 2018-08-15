
redis案例  
------------
    
1 内容管理系统，可发布文章、投票   (仅做入门例子)
------------
文章自增id  (用来为某个资源生成自增id)  
结构 string；  
key为 (article), value为 (自增的数字；)每次自增incr 1  
  
文章 (存储单条文章信息)  
结构 hash；   
key 为 (article:id) ，value 为 (文章数据)  
  
文章投票记录 （参与用户记录，可用来限制每个文章每个用户只能投一次）  
结构 set   
key 为 (voted:id)，值为 (user_id)  
同时记录设失效时间，比如7天失效  
  
文章的分值记录 (可按分值排序获取文章)  
结构 sortedset  
key为 (score:article)，值为 (score为 当前时间戳 + 1， member为 article:id)  
  
文章的发布时间记录 (可按发布时间排序获取文章)  
结构sortedset  
key 为(time:artcle), 值为 (score为 当前时间， membeer为 article:id)  
  
  

  
2 摇一摇/快速点击 拼手速 -> 抢红包  
------------
活动 （记录单条活动信息，也可不要，以实际情况来定）  
结构hash  
key 为 (active:id) 值为 (活动简短数据)  
  
某活动参与用户摇记录 (可取某活动的摇动用户排行榜)  
结构sortedset   
key为 (score:artive:id)，值为 (score为 摇或点击的次数， member为 user:id)  
  
  
  
  
3 抢红包 （比如微信发红包，多人去抢） 注：因微信群人数上限500，所以单个红包的并发量比网站秒杀要少太多。
------------
发红包时在库中生成红包记录，并且按照发送时设置的红包规则，生成相应的小红包。
然后在redis中每个大红包有两个队列：一个存待发的小红包，一个存已发的小红包。
  
1)小红包预先生成，插到数据库里，红包对应的用户ID是0。生成算法见另一篇文章：//www.jb51.net/article/98620.htm  
  
2)每个大红包对应两个redis队列，一个是未消费红包队列，另一个是已消费红包队列。开始时，把未抢的小红包全放到未消费红包队列里。  
  
未消费红包队列里是json字符串，如{小红包_id:'789', money:'300'}  
已消费红包队列里是json字符串，如{小红包_id:'789', money:'300', 'user_id':xxx, 'add_time':xxx}  
  
3)在redis中用一个map来过滤已抢到红包的用户。 可以通过setnx 防止同一用户并发抢。  
  
4)抢红包时，先判断用户是否抢过红包。如果没有，则从未消费红包队列中取出一个小红包，再push到另一个已消费队列中，最后把用户ID放入去重的map中。  
  
5)用一个cron批量把已消费队列里的红包取出来，再批量update红包的用户ID到数据库里。  



  
  
  
4 秒杀
------------
秒杀商品  （控制商品不卖超）
结构list  
key 为 (ns_seckill:goods_id值:20150505)，值为 (商品id)  
  
用户k/v （去重 用setnx，也可考虑keys的数量来销峰）
结构string
key 为 (ns_seckill:goods_id值:20150505:user_id值)，值为 (1)  
  
秒杀排队用户  (销峰) 
结构list  
key 为 (ns_seckill:goods_id值:20150505:queue)，值为 (user_id)  
  
秒杀结果用户  （结果记录，等等cron入库）
结构set  
key 为 (ns_seckill:goods_id值:20150505:user)，值为 (user_id)  
   

实际流程：
先通过len判断是否还有商品？
如果没有，直接返回商品已秒杀完了。
// 如果有，这里可以再增加一步： len排队队列长度验证，如果大于指定值，直接结束，否则rpush进入排队队列。
如果有，setnx 当前用户。如果失败，直接返回；如果成功，lpop出某个商品。
如果pop失败，直接返回，
如果pop成功，开始执行业务成功的业务逻辑(按实际秒杀商品的数量来考虑 是先记入秒杀结果队列然后cron， 还是直接操作库)  


<!-- 高并发情况，先将用户进入排队队列，用一个线程循环处理从排队队列取出一个用户，判断用户是否已在抢购结果队列，如果在则已抢购，否则未抢购，接着执行库存减1，将此user_id用户同时也进入结果队列, 写入数据库。    -->
  
<!-- 基于goroutine + channel 比较好实现，channel管道中的数据是先进先出的。  
先入redislist队列，  
若排队人数达上限，直接返回提示来晚了  
若排队成功，则进入排队队列，同时每加一个排队用户朝channel里写一条，然后再channel里循环接收、处理   -->
  
  
  
  
5 好友（关注）
------------
用户 的关注follow列表 (to follow)  
结构set    
key 为 (follow:user_id值)，值为 (user_id)  

用户 的关注者follower列表 (be followed)  
结构set    
key 为 (followed:user_id值)，值为 (user_id)   
  
如果取好友，实际是取共同关注，取两个用户的各自的的关注follow列表求交集(SINTER, SINTERSTORE)  
  
  
  
6 排行榜  
------------
要排序的元素  
结构sorted set  
key 为 (资源名:rank)，值为 (score 排行分， member 资源id)    


  










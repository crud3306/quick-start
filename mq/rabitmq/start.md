
地址：  
https://www.imooc.com/article/40141  


RabbitMQ
---------
RabbitMQ是使用Erlang编写的流行的开源消息队列系统。RabbitMQ是AMQP（高级消息队列协议）的标准实现。


消息在RabbitMQ中的经由之路
---------
1）消息先从生产者Producer出发到达交换器Exchange；

2）交换器Exchange根据路由规则将消息转发对应的队列Queue之上；

3）消息在队列Queue上进行存储；

4）消费者Consumer订阅队列Queue并进行消费。




发送确认
-----------
两种方式，不能同时开启
1）事务机制，(同步的)  
2）publisher confirm机制 (异步的ack)  




消费确认
-----------
消费消费的ack，使用方式把autoAck设为flase。

为了保证消息从队列可靠地达到消费者，RabbitMQ提供了消息确认机制（message acknowledgement）。消费者在订阅队列时，可以指定autoAck参数，当autoAck等于false时，RabbitMQ会等待消费者显式地回复确认信号后才从内存（或者磁盘）中移去消息（实质上是先打上删除标记，之后再删除）。当autoAck等于true时，RabbitMQ会自动把发送出去的消息置为确认，然后从内存（或者磁盘）中删除，而不管消费者是否真正的消费到了这些消息。


php使用rabbitmq的两种方式：
1 直接使用php的amqp扩展   
2 使用php-amqplib，仍然要先安装phpamqp扩展。    




几个概念说明：
-----------
```
Broker：简单来说就是消息队列服务器实体。
Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
producer：消息生产者，就是投递消息的程序。
consumer：消息消费者，就是接受消息的程序。
channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。
```

消息队列的使用过程大概如下：
-----------
``
1）客户端连接到消息队列服务器，打开一个channel。
2）客户端声明一个exchange，并设置相关属性。
3）客户端声明一个queue，并设置相关属性。
4）客户端使用routing key，在exchange和queue之间建立好绑定关系。
5）客户端投递消息到exchange。
```

exchange进行消息路由的三种方式：
-----------
exchange接收到消息后，就根据消息的key和已经设置的binding，进行消息路由，将消息投递到一个或多个队列里。常用的有三种方式：  

1）Fanout：  
不需要key的（即使给了key，key也不起作用），叫做Fanout交换机，它采取广播模式，一个消息进来时，投递到与该交换机绑定的所有队列。

2）Direct:  
完全根据key进行投递的叫做Direct交换机，例如，绑定时设置了routing key为”abc”，那么客户端提交的消息，只有设置了key为”abc”的才会投递到队列。

3）Topic：  
对key进行模式匹配后进行投递的叫做Topic交换机，符号”#”匹配一个或多个词，符号”*”匹配正好一个词。例如”abc.#”匹配”abc.def.ghi”，”abc.*”只匹配”abc.def”。




RabbitMQ支持消息的持久化，也就是数据写在磁盘上，为了数据安全考虑，我想大多数用户都会选择持久化。  
消息队列持久化包括3个部分：  
（1）exchange持久化，在声明时指定durable => 1
（2）queue持久化，在声明时指定durable => 1
（3）消息持久化，在投递时指定delivery_mode => 2（1是非持久化）

如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的。如果exchange和queue两者之间有一个持久化，一个非持久化，就不允许建立绑定。






在RabbitMQ中消费者有2种方式获取队列中的消息:
----------------
a)  一种是通过basic.consume命令，订阅某一个队列中的消息,channel会自动在处理完上一条消息之后，接收下一条消息。（同一个channel消息处理是串行的）。除非关闭channel或者取消订阅，否则客户端将会一直接收队列的消息。

b)  另外一种方式是通过basic.get命令主动获取队列中的消息，但是绝对不可以通过循环调用basic.get来代替basic.consume，这是因为basic.get RabbitMQ在实际执行的时候，是首先consume某一个队列，然后检索第一条消息，然后再取消订阅。如果是高吞吐率的消费者，最好还是建议使用basic.consume。


简单总结一下就是说：

consume是只要队列里面还有消息就一直取。

get是只取了队列里面的第一条消息。

因为get开销大，如果需要从一个队列取消息的话，首选consume方式，慎用循环get方式。



queue对象有两个方法可用于取消息：consume和get。
----------------
注意，还有说明
前者是阻塞的，无消息时会被挂起，适合循环中使用；后者则是非阻塞的，取消息时有则取，无则返回false。

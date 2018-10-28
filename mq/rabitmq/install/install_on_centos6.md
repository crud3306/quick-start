

Centos 6 通过 yum 安装 Rabbitmq
============
1. 安装 Erlang
Rabbitmq 的运行需要 Erlang 环境，首先安装 Erlang。
```
mkdir -p ~/download 
cd ~/download
wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
rpm -Uvh erlang-solutions-1.0-1.noarch.rpm

yum -y install erlang;
```

2. 安装 Rabbitmq
```
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.6.9/rabbitmq-server-3.6.9-1.el6.noarch.rpm
rpm  -Uvh rabbitmq-server-3.6.9-1.el6.noarch.rpm
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
yum -y install rabbitmq-server-3.6.9-1.noarch.rpm
```

启动 Rabbitmq：
```
/ect/init.d/rabbitmq-server start
chkconfig rabbitmq-server on
```

开启图形界面支持：
```
rabbitmq-plugins enable rabbitmq_management
```

添加用户：
```
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

浏览器打开 “http://localhost:15672″， 用帐号“admin”, 密码“admin” 就可以登录了，
！！注意localhost 换成实际地址或ip， rabbitmq 默认提供了 “guest” 帐号，但是只能在localhost 域名下登录，

开启 stomp 支持：
> rabbitmq-plugins enable rabbitmq_stomp  

错误处理
出现 “socat is needed by rabbitmq-server-3.6.9-1.el6.noarch”的错误
执行 yum -y install socat




AMQP 扩展安装
===========
PHP 使用 AMQP 协议来连接 Rabbitmq， AMQP 协议即 “Advanced Message Queuing Protocol ”，高级消息队列协议。

使 PHP 支持 AMQP 协议，需要安装：

rabbitmq 的客户端 C 类库 ：rabbitmq-c
PHP 官方提供的 AMQP 扩展 amqp-1.6.1
两者的关系是，PHP 扩展依赖 rabbitmq-c 类库。

1.安装 rabbitmq-c ：
```
mkdir -p ~/download 
cd ~/download
https://github.com/alanxz/rabbitmq-c/releases/download/v0.8.0/rabbitmq-c-0.8.0.tar.gz
tar xvzf rabbitmq-c-0.8.0.tar.gz
mkdir build 
cd build
cmake ..
make 
make install
```
下载 rabbitmq-c 好像得有梯子，下面是备用下载地址。
下载rabbitmq-c-0.8.0.tar.gz


2.安装 AMQP 扩展：
```
wget -c https://pecl.php.net/get/amqp-1.6.1.tgz
tar xvzf amqp-1.6.1.tgz
cd amqp-1.6.1
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config --with-amqp 
make 
make install
```
注意上面的安装中的 /usr/local/php/ 是 PHP 的安装目录，根据自己的服务器情况，将它改成正确的 PHP 安装路径。


3.配置 php.ini  
打开 php.ini , 在底部添加:
```
[AMQP]
extension="amqp.so"
```
重启php。


4.检查是否安装成功
> php -m | grep amqp   

到这里，PHP 的扩展就安装完成了。



使用：这是直接amqp扩展
==============
```
//创建连接和channel 
$conn = new AMQPConnection($conn_args);   
if (!$conn->connect()) {   
    die("Cannot connect to the broker!\n");   
}   
$channel = new AMQPChannel($conn);  
##3，exchange 与  routingkey ： 交换机 与 路由键##
.....

```
示例：  
https://segmentfault.com/a/1190000002963223  
http://www.dahouduan.com/2017/11/23/php-rabbitmq-demo/  


也可以使用封装好的扩展类php-amqplib，操作更简单一些：
-------------

在你的项目目录下添加 composer.json (通过composer安装)
```
{
    "require": {
        "php-amqplib/php-amqplib": ">=2.6.1"
    }
}
```
安装
> composer install  

使用时：
```
require_once __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
```

以下是直接发送到queue的代码示例：

完整的send：
```
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
$channel->queue_declare('hello', false, false, false, false);
$msg = new AMQPMessage('Hello World!');
$channel->basic_publish($msg, '', 'hello');
echo " [x] Sent 'Hello World!'\n";

$channel->close();
$connection->close();
```

消费者consumer：
```
require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
$channel->queue_declare('hello', false, false, false, false);
echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";
$callback = function($msg) {
  echo " [x] Received ", $msg->body, "\n";
};
$channel->basic_consume('hello', '', false, true, false, false, $callback);
while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

运行：
> php receive.php  
> php send.php  

https://blog.csdn.net/demon3182/article/details/77335206   
https://blog.csdn.net/demon3182/article/details/77335704  
https://blog.csdn.net/demon3182/article/details/77482725  





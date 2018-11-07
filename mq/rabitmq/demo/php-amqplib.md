

在你的项目目录下添加 composer.json (通过composer安装)
=============
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
=============
```
require_once __DIR__ . '/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
```


注意：
1对1的消息发送和接收，即消息只能发送到指定的queue里(为queue起名字，但不用exchange)，但有些时候你想让你的消息被所有的Queue收到，类似广播的效果，这时候就要用到exchange了，借为exchange的不同分发机制。  
直接用queue：1条消息只有会一个消费端消费  
用exchange：1条消可以被多个消费端消费  


1）普通的队列直接发送消息
--------------
场景1：单发送单接收，使用场景：简单的发送与接收，没有特别的处理。

https://blog.csdn.net/demon3182/article/details/77335206

完整的send：
```
$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();
$channel->queue_declare('hello', false, false, false, false);
$msg = new AMQPMessage('Hello World!');

// 第一参数消息，第二个exchange名字，第三个routing_key，如果没有exchange则相当于队列名
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



2）持久化的工作队列
--------------
使用场景：一个发送端，多个接收端，如分布式的任务派发。为了保证消息发送的可靠性，不丢失消息，使消息持久化了。同时为了防止接收端在处理消息时down掉，只有在消息处理完成后才发送ack消息。

https://blog.csdn.net/demon3182/article/details/77335704

publish
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

// 声明一个队列是幂等的 - 只有当它不存在时才会被创建。消息的内容是一个字节数组，所以你可以编码你喜欢的任何东西。
// queue_declare 的第三个参数设置为 true，表示将队列声明为持久化
$channel->queue_declare('task_queue', false, true, false, false);

$data = implode(' ', array_slice($argv, 1));
if (empty($data)) {
	$data = "Hello World!";
}
$msg = new AMQPMessage($data,
                        array('delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT)
                      );

// 第一参数消息，第二个exchange名字，第三个routing_key，如果没有exchange则相当于队列名
$channel->basic_publish($msg, '', 'task_queue');

echo " [x] Sent ", $data, "\n";

$channel->close();
$connection->close();
```

worker.php
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('task_queue', false, true, false, false);

echo ' [*] Waiting for messages. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo " [x] Received ", $msg->body, "\n";
  sleep(substr_count($msg->body, '.'));
  echo " [x] Done", "\n";
  $msg->delivery_info['channel']->basic_ack($msg->delivery_info['delivery_tag']);
};

// 我们可以通过设置 basic_qos 第二个参数 prefetch_count = 1。这一项告诉RabbitMQ不要一次给一个消费者发送多个消息。或者换一种说法，在确认前一个消息之前，不要向消费者发送新的消息。相反，新的消息将发送到一个处于空闲的消费者。
$channel->basic_qos(null, 1, null);

// 开启消息确认机制，将basic_consumer的第四个参数设置为false(true表示不开启消息确认)  
$channel->basic_consume('task_queue', '', false, false, false, false, $callback);

while (count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();

```


3）广播消息 （fanout exchange） Publish/Subscribe
--------------
向多个消费者传递相同的信息。这种模式被称为“发布/订阅”。  
使用场景：发布、订阅模式，发送端发送广播消息，多个接收端接收。
https://blog.csdn.net/demon3182/article/details/77482725  

发布者
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('logs', 'fanout', false, false, false);

$data = implode(' ', array_slice($argv, 1));
if(empty($data)) $data = "info: Hello World!";
$msg = new AMQPMessage($data);

// 第一参数消息，第二个exchange名字，第三个routing_key
$channel->basic_publish($msg, 'logs');

echo " [x] Sent ", $data, "\n";

$channel->close();
$connection->close();
```

消费者
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('logs', 'fanout', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$channel->queue_bind($queue_name, 'logs');

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```


直接交换 (Direct exchange)
--------------
使用场景：发送端按routing key发送消息，不同的接收端按不同的routing key接收消息。

https://blog.csdn.net/demon3182/article/details/77482754  

direct_publish.php
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('direct_logs', 'direct', false, false, false);

$severity = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'info';

$data = implode(' ', array_slice($argv, 2));
if(empty($data)) $data = "Hello World!";

$msg = new AMQPMessage($data);

// 第一参数消息，第二个exchange名字，第三个队routing_key
$channel->basic_publish($msg, 'direct_logs', $severity);

echo " [x] Sent ",$severity,':',$data," \n";

$channel->close();
$connection->close();
```

direct_consume.php
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('direct_logs', 'direct', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$severities = array_slice($argv, 1);
if(empty($severities )) {
    file_put_contents('php://stderr', "Usage: $argv[0] [info] [warning] [error]\n");
    exit(1);
}

// 注意下面的绑定队列时第三个参数routing_key，且为队列绑字了多个routing_key
foreach($severities as $severity) {
    $channel->queue_bind($queue_name, 'direct_logs', $severity);
}

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

如果你只想将 warning 和 error (而非 info) 日志信息保存到文件中，只用打开一个控制台键入:  
> php direct_consume.php warning error > logs_from_rabbit.log  

如果你想在屏幕上看到所有的日志信息，打开一个控制台键入：  
> php direct_consume.php info warning error  

而且，例如，要发送 error 类型日志，只需要键入：  
> php direct_publish.php error "Run. Run. Or it will explode."  



主题 (Topic exchange)
--------------
使用场景：发送端不只按固定的routing key发送消息，而是按字符串“匹配”发送，接收端同样如此。

https://blog.csdn.net/demon3182/article/details/77482771  

发送到 topic exchange 的消息不能任意命名一个 routing key - 它必须是由一个.划分单词列表。这些单词可以是任意的，但它们通常指定与消息相关联的一些功能。这里有几个有效的 routing key: "stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit" , routing key 中可以包含多个单词，最多可以达到255个字节。

publish.php
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('topic_logs', 'topic', false, false, false);

$routing_key = isset($argv[1]) && !empty($argv[1]) ? $argv[1] : 'anonymous.info';
$data = implode(' ', array_slice($argv, 2));
if(empty($data)) $data = "Hello World!";

$msg = new AMQPMessage($data);

$channel->basic_publish($msg, 'topic_logs', $routing_key);

echo " [x] Sent ",$routing_key,':',$data," \n";

$channel->close();
$connection->close();
```

consume.php
```
<?php

require_once __DIR__ . '/vendor/autoload.php';
use PhpAmqpLib\Connection\AMQPStreamConnection;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->exchange_declare('topic_logs', 'topic', false, false, false);

list($queue_name, ,) = $channel->queue_declare("", false, false, true, false);

$binding_keys = array_slice($argv, 1);
if( empty($binding_keys )) {
    file_put_contents('php://stderr', "Usage: $argv[0] [binding_key]\n");
    exit(1);
}

foreach($binding_keys as $binding_key) {
    $channel->queue_bind($queue_name, 'topic_logs', $binding_key);
}

echo ' [*] Waiting for logs. To exit press CTRL+C', "\n";

$callback = function($msg){
  echo ' [x] ',$msg->delivery_info['routing_key'], ':', $msg->body, "\n";
};

$channel->basic_consume($queue_name, '', false, true, false, false, $callback);

while(count($channel->callbacks)) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

收听所有日志：  
> php consume.php "#"  

收听所有来自 kern 的日志：  
> php consume.php "kern.*"  

只收听 “critical” 类型的日志：  
> php consume.php "*.critical"  

你也可以创建多个绑定：  
> php consume.php "kern.*" "*.critical"  

发出 routing key为 “kern.critical”类型的日志  
> php publish.php "kern.critical" "A critical kernel error"  

至此你可以尽情鼓弄这些程序。需要注意的是，这些代码没有对 binding key 和 routing key 赋默认值，除此之外你可能需要路由不止两个单词。








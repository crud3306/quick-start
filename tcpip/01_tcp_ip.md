tcp/ip协议

tcp/ip协义是一个协议簇，里面包括很多协议，常用的有：
------------
http：超文本传输协议，万维网的基本协议  

ftp（File Transfer Protocol）：文件传输协议，允许用户将远程主机上的文件拷贝到自己的计算机上。 

Telnet（Remote Login）：提供远程登录功能，一台计算机用户可以登录到远程的另一台计算机上，如同在远程主机上直接操作一样。 

SMTP（Simple Mail transfer Protocol）：简单邮政传输协议，用于传输电子邮件。 

NFS（Network File Server）：网络文件服务器，可使多台计算机透明地访问彼此的目录。 

tcp（transmission control protocol） 传输控制协议（面向连接）  

UDP（User Datagram Protocol）：用户数据包协议，它和TCP一样位于传输层，和IP协议配合使用，在传输数据时省去包头，但它不能提供数据包的重传，所以适合传输较短的文件。

网络管理snmp  

internet协议(ip)  
internet控制信息协议(icmp)  

arp 地址解析协议  
rarp 反向地址解析协议  

  
  


tcp与udp区别
--------------
1 tcp基于连接(更加安全可靠)，udp无连接  
2 对系统资源的要求(tcp较多，udp少)  
3 udp程序结构较简单  
4 tcp流模式，udp数据报模式  

 _            TCP       UDP 
是否连接     面向连接   面向非连接  
传输可靠性     可靠      不可靠  
应用场合    传输大量数据  少量数据   
速度          慢          快  



TPC/IP协议是传输层协议，主要解决数据如何在网络中传输，而HTTP是应用层协议，主要解决如何包装数据。
关于TCP/IP和HTTP协议的关系，网络有一段比较容易理解的介绍：“我们在传输数据时，可以只使用（传输层）TCP/IP协议，但是那样的话，如果没有应用层，便无法识别数据内容，如果想要使传输的数据有意义，则必须使用到应用层协议，应用层协议有很多，比如HTTP、FTP、TELNET等，也可以自己定义应用层协议。WEB使用HTTP协议作应用层协议，以封装HTTP 文本信息，然后使用TCP/IP做传输层协议将它发到网络上。”

TCP/IP是个协议组，主要分布在三个层次：网络层、传输层和应用层。
在网络层有IP协议、ICMP协议、ARP协议、RARP协议和BOOTP协议。
在传输层中有TCP协议与UDP协议。
在应用层有FTP、HTTP、TELNET、SMTP、DNS、WebSocket等协议。



注：
HTTP、WebSocket 等应用层协议，都是基于 TCP 协议来传输数据的

Socket 其实并不是一个协议。它工作在OSI模型会话层（第5层），是为了方便大家直接使用更底层协议（一般是 TCP 或 UDP ）而存在的一个抽象层。
Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。



比较 HTTP、WebSocket 
-----------------
同：HTTP、WebSocket 等应用层协议，都是基于 TCP 协议来传输数据的  

不同：  
HTTP协议为单向协议，即浏览器只能向服务器请求资源，服务器才能将数据传送给浏览器，而服务器不能主动向浏览器传递数据。分为长连接和短连接，短连接是每次http请求时都需要三次握手才能发送自己的请求，每个request对应一个response；长连接是短时间内保持连接，保持TCP不断开，指的是TCP连接。  

WebSocket解决客户端发起多个http请求到服务器资源浏览器必须要经过长时间的轮询问题。他实现了多路复用，是一种双向通信协议。在建立连接后，WebSocket服务器和客户端都能主动的向对方发送或接收数据，就像Socket一样，不同的是WebSocket是一种建立在Web基础上的一种简单模拟Socket的协议；     

WebSocket需要通过握手连接，类似于TCP它也需要客户端和服务器端进行握手连接，连接成功后才能相互通信。

WebSocket在建立握手连接时，数据是通过http协议传输的，“GET/chat HTTP/1.1”，这里面用到的只是http协议一些简单的字段。但是在建立连接之后，真正的数据传输阶段是不需要http协议参与的。  

























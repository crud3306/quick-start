学习地址：
-------------
http://www.ruanyifeng.com/blog/2017/08/elasticsearch.html  
https://www.cnblogs.com/Wolfmanlq/p/5984494.html  
  
  
基本概念：  
-------------
关系数据库     ⇒ 数据库 (db)     ⇒ 表（table）    ⇒ 行 (line)           ⇒ 列(Columns)  
Elasticsearch ⇒ 索引（index）   ⇒ 类型（type）   ⇒ 文档(document)      ⇒ 字段(Fields)  
  
  
安装elasticsearch
-------------
Elastic 需要 Java 8 环境。同时注意要保证环境变量JAVA_HOME正确设置。  
安装完 Java，就可以跟着官方文档安装 Elastic。直接下载压缩包比较简单。  
  
> wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.1.zip  
> unzip elasticsearch-5.5.1.zip  
> cd elasticsearch-5.5.1/  
  
创建分组与用户，因elasticsearch不允许以root身份直接运行  
> groupadd elsearch  
> useradd elsearch -s /sbin/nologin -g elsearch  
  
接着，进入解压后的目录，运行下面的命令，启动 Elastic。  
> ./bin/elasticsearch
   
如果这时报错"max virtual memory areas vm.maxmapcount [65530] is too low"，要运行下面的命令。  
> sudo sysctl -w vm.max_map_count=262144  

如果一切正常，Elastic 就会在默认的9200端口运行。这时，打开另一个命令行窗口，请求该端口，会得到说明信息。  
> curl localhost:9200  
{  
"name" : "atntrTf",  
"cluster_name" : "elasticsearch",  
"cluster_uuid" : "tf9250XhQ6ee4h7YI11anA",  
"version" : {  
"number" : "5.5.1",  
"build_hash" : "19c13d0",  
"build_date" : "2017-07-18T20:44:24.823Z",  
"build_snapshot" : false,  
"lucene_version" : "6.6.0"  
},  
"tagline" : "You Know, for Search"  
}  

上面代码中，请求9200端口，Elastic 返回一个 JSON 对象，包含当前节点、集群、版本等信息。  
  
按下 Ctrl + C，Elastic 就会停止运行。  
  
  

如果想要以守护进程方式启动，如下：  
> ./bin/elasticsearch -d >/dev/null 2>&1
  
查看启动的el  
> ps aux |grep elastic  
  
关闭以守护进程方式启动的elasticsearch，先查出进程pid   
> kill -9 xxxpid   
  
  
默认情况下，Elastic 只允许本机访问，如果需要远程访问，可以修改 Elastic 安装目录的config/elasticsearch.yml文件，去掉network.host的注释，将它的值改成0.0.0.0，然后重新启动 Elastic。  
> network.host: 0.0.0.0  

上面代码中，设成0.0.0.0让任何人都可以访问。线上服务不要这样设置，要设成具体的 IP。  
  
  
   
  
让elasticsearch支持中文分词：
--------------------------
安装中文分词插件。这里使用的是 ik，也可以考虑其他插件（比如 smartcn）。  
执行如下命令:  
> ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v5.5.1/elasticsearch-analysis-ik-5.5.1.zip  
上面代码安装的是5.5.1版的插件，与 Elastic 5.5.1 配合使用。  
  
接着，重新启动 Elastic，就会自动加载这个新安装的插件。  
  
访问：  
> curl -XGET 'http://localhost:9200/_analyze?pretty&analyzer=standard' -d ' 第二更新 '  
> curl -XGET  'http://192.168.2.21:9210/_analyze?pretty&analyzer=ik' -d ' 第二更新 '  
> curl http://localhost:9200/_analyze?pretty&analyzer=ik  

  
  
遇到的一些问题
------------------
1 启动时内存不足崩溃
------------------
Memory: 4k page, physical 1016372k(68672k free), swap 2096124k(1439444k free)  
vm_info: Java HotSpot(TM) 64-Bit Server VM (25.151-b12) for linux-amd64 JRE (1.8.0_151-b12), built on Sep 5 2017 19:20:58 by "java_re" with gcc 4.3.0 20080428 (Red Hat 4.3.0-8)  
解决方法：  
> vi config/jvm.options  

更改：  
> #-Xms2g  
> #-Xmx2g  
> -Xms512m  
> -Xmx512m  
  
  
2 ElasticSearch Root身份运行
------------------
如果以root身份运行将会出现以下问题  
root@yxjay:/opt/elasticsearch-2.3.5/bin# ./elasticsearch  
Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root.  
at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:93)  
at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:144)  
at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:270)  
at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)  
Refer to the log for complete error details.  
    
解决方法：  
  
这是出于系统安全考虑设置的条件。由于ElasticSearch可以接收用户输入的脚本并且执行，为了系统安全考虑，
建议创建一个单独的用户用来运行ElasticSearch  
  
创建elsearch用户组及elsearch用户  
> groupadd elsearch   
> useradd elsearch -g elsearch -p elasticsearch  
  
更改elasticsearch文件夹及内部文件的所属用户及组为elsearch:elsearch  
> cd /user/local
> chown -R elsearch:elsearch elasticsearch  
  
切换到elsearch用户再启动  
  
> su elsearch  
> cd elasticsearch/  
> ./bin/elasticsearch  
  
如果用户无登录权限，则不能切换，需用以下命令  
> sudo -u elsearch /data/elasticsearch-5.5.1/bin/elasticsearch  
  
  
  
3 外部不能访问9200端口
---------------------
默认情况下，Elastic 只允许本机访问，如果需要远程访问，可以修改 Elastic 安装目录的config/elasticsearch.yml文件，去掉network.host的注释，将它的值改成0.0.0.0，然后重新启动 Elastic。  
> network.host: 0.0.0.0
  
上面代码中，设成0.0.0.0让任何人都可以访问。线上服务不要这样设置，要设成具体的 IP。  
注：此方式好像不行，更改后启动时直接出错  

  

4 Elasticsearch 自已的logs目录无权限  
---------------------
main ERROR RollingFileManager (/usr/local/elasticsearch-5.5.1/logs/elasticsearch.log) java.io.FileNotFoundException: /usr/local/elasticsearch-5.5.1/logs/elasticsearch.log (Permission denied) java.io.FileNotFoundException: /usr/local/elasticsearch-5.5.1/logs/elasticsearch.log (Permission denied)  
  
更改所有者、所属主  
> chown -R elsearch:elsearch logs





如果报的header错误，需要更两个文件  
Automatically set Content-type and Accept headers  
主要是Elasticsearch-php与服务端Elasticsearch版本号不兼容造成的：  
In this module "composer.json" contain "elasticsearch/elasticsearch": "~2.0". And he doesn't support Elasticsearch 6.0.  
  
具体更改查看：  
https://github.com/elastic/elasticsearch-php/commit/fd3b0f16f7e09cb2096ef5f2d656c9fd8dd3d61d#diff-1b0215334399d80759820e3229367adf  

主要改下面两文件  
> /xxx/Elasticsearch/ClientBuilder.php  
> /xxx/Elasticsearch/Connections/Connection.php  


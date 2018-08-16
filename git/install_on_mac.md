
centos 下安装git
--------------
> yum install -y git  
  
  
安装比较新的版本，比如 git-2.2.1  
--------------
参考地址：
https://yq.aliyun.com/articles/28256




常见问题：
---------------
报错：error: The requested URL returned error: 401 Unauthorized while accessing
一般出该错是 git版本为：1.7.1
  
1) 解决方法一：指定用户  
git clone https://github.com/org/project.git  
  
换成  
git clone https://username@github.com/org/project.git  
或者  
git clone https://username:password@github.com/org/project.git  
  
在push或者pull出出现的话，则需要更改远程地址  
git remote set-url origin https://username@github.com/org/project.git  
  

2) 解决方法二：去除验证  
git config –global http.sslverify false  
  

3) 解决方法三：（推荐）  
升级git 版本≥1.7.10  
  

4) 解决方法四：  
添加ssh秘钥  
  

  


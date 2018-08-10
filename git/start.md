Git global setup
-------------
> git config --global user.name "qianming"  
> git config --global user.email "crud3306@163.com"  
> //git config --global user.email "curd3306@163.com"  

注意：  
如果改了github的密码，再次朝 github 上 push时，为会报fatal: Authentication failed。  
这时，重新输入上面两条命令，再次push时，会让重新输入新密码，输入后回车即可  



clone a remote repository
-------------
> git clone http://git.pugutang.cn/back/item.git  
> cd item  
> touch README.md  
> git add README.md  
> git commit -m "add README"  
> git push -u origin master  


create a new repository on the command line
-------------
> git init  
> git add README.md  
> git commit -m "first commit"  
> // 如果git remote add xxx时，报错fatal: remote origin already exists，需先执行git remote rm origin  
> // git remote rm origin  
> git remote add origin https://github.com/crud3306/php-start.git  
> //git remote add origin ssh://git@github.com:crud3306/php-start.git  
> git push -u origin master  


push an existing repository from the command line
-------------
> git remote add origin http://git.pugutang.cn/back/item.git  
> git push -u origin master  








解决mac电脑上出现Permission to xxx.git denied to xxx的问题
-------------
cd ~/.ssh  

ssh-keygen -t rsa -C "xxxxxxx@163.com"  

Generating public/private rsa key pair.  
Enter file in which to save the key (/Users/yidont/.ssh/id_rsa):  
// 这里要你输入存放的目录，可回车默认  

Enter passphrase (empty for no passphrase): (输入ssh密码)  
Enter same passphrase again: （确认密码）  

Your identification has been saved in /Users/yidont/.ssh/id_rsa.  
Your public key has been saved in /Users/yidont/.ssh/id_rsa.pub.  
The key fingerprint is:  
SHA256:1gepuxDHwJRnFbKvc0Zq/NGrFGE9kEXS06jxatPPrSQ voctex@163.com  
The key's randomart image is:  
+---[RSA 2048]----+  
|      ....=*oo   |  
|     o. ooo=+ .  |  
|      oo. =+o.   |  
|       o =.o..   |  
|      . S =o.    |  
|       = =++.    |  
|      . B.=.Eo.. |  
|       o B . +o .| 
|        . o.. .. |  
+----[SHA256]-----+  
  
ls查看生成的文件  
id_rsa  id_rsa.pub   

然后把id_rsa.pub 配到你的 github上，方法如下  
拷备 id_rsa.pub 文件里的内容  
登录github -> 个人中心setting -> SSH and GPG keys  
点击 (new ssh key) 按扭，把拷贝的内容放入 key对应的输入框中，title输入框随意起个名字；确认保存。 
  
  
回到mac的 ~/.ssh/ 目录  
vi config 添加：  
  
> #Default gitHub  
> Host github.com  
> HostName github.com  
> User git  
> IdentityFile ~/.ssh/id_rsa  

:wq

进去本地的git项目中，重新提交
> $ git remote rm origin
> $ git remote add origin git@github-he:xxx/xxx.git


















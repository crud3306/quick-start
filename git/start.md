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
> //git remote add origin https://github.com/crud3306/php-start.git  
> git remote add origin git@github.com:crud3306/php-start.git  
> git push -u origin master  


push an existing repository from the command line
-------------
> git remote add origin http://git.pugutang.cn/back/item.git  
> git push -u origin master  
  
  
  
  
已commit的文件，又想撤销  
-------------
在git push的时候，有时候我们会想办法撤销git commit的内容 

方式1：按HEAD
git reset HEAD 某个文件  
  
回滚到上一个快照，有几个~号，即回滚几个版本。也可用~号配合数字表示几个~  
git reset HEAD~  
git reset HEAD~~  
git reset HEAD~10  
  

方式2： 按版本号回滚或前滚   
git reset 提交后产生的版本号    
  
1、找到之前提交的git commit的id  
git log   
找到想要撤销的id   
  
2、git reset –hard id   
完成撤销,同时将代码恢复到前一commit_id 对应的版本   

3、git reset id   
完成Commit命令的撤销，但是不对代码修改进行撤销，可以直接通过git commit 重新提交对本地代码的修改  
  
  
  
  
  
  
  
解决mac电脑上出现Permission to xxx.git denied to xxx的问题
-------------
> cd ~/.ssh  

> ssh-keygen -t rsa -C "xxxxxxx@163.com"  

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
> 拷备 id_rsa.pub 文件里的内容  
> 登录github -> 个人中心setting -> SSH and GPG keys  
> 点击 (new ssh key) 按扭，把拷贝的内容放入 key对应的输入框中，title输入框随意起个名字；确认保存。 
  
  
回到mac的 ~/.ssh/ 目录  
vi config 添加：  
  
> #Default gitHub  
> Host github.com  
> HostName github.com  
> User git  
> IdentityFile ~/.ssh/id_rsa  

:wq

进去本地的git项目中，重新提交

如果更改了远程地址，需先执行如下命令：
> $ git remote rm origin  
> $ git remote add origin git@github.com:xxxx/xxx.git  

然后push， 提交到远程
> $ git push -u origin master
  
  
  
  
  
对已push了不必要文件，却又想删除 的处理方法
-----------------
1.先在.gitignore文件上编写一下代码，加上你不想push的文件。比如node_modules 和 dist两个目录
> node_modules
> dist

2.在命令行进入仓库目录，删除github仓库上.gitignore上新加的选项
> git rm -r --cached .
如果只删除某个目录，可以
> git rm -r --cached （要删除的文件名,比如node_modules）

3.然后重新添加要提交的选项
> git add .

4.接着commit，简要说明一下commit的内容
> git commit -m 'remove node_modules and dist'

5.最后在git push 到远程仓库上就可以了。
> git push




























安装插件
===========
首选需要安装一个Package Control, Package Control是一个管理Sublime Text插件的插件，通过Package Control我们可以直接使用Sublime Text控制台来安装其他插件，快捷简单。


安装Package Control
------------
方法一：
通过Ctrl+`快捷键 或者 View > Show Console菜单打开控制台，复制粘贴如下代码回车即可。
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)

若出现网络连接不畅，Package Control网址打不开等意外情况，可使用下面这种方法。

方法二：
下载一个packagecontrol包
http://www.feyon.net/installation

点击preferences - browse packages 会开打sublime的相应目录，选择该目录下的Installed Packages，把刚下在的文件放进这里。



插件被墙不能安装的问题
----------
参考地址：https://www.jianshu.com/p/23d1ec6988e5

Package Control.sublime-settings]修改方法：
Preferences > Package Settings > Package Control > Settings - User
添加
```
"channels":
[
    "http://static.bolin.site/channel_v3.json",
    //"https://packagecontrol.io/channel_v3.json",
    //"https://web.archive.org/web/20160103232808/https://packagecontrol.io/channel_v3.json",
    //"https://gist.githubusercontent.com/nick1m/660ed046a096dae0b0ab/raw/e6e9e23a0bb48b44537f61025fbc359f8d586eb4/channel_v3.json"
],
```



中文乱码
----------
安装ConvertToUTF8，Codecs33。

Ctrl+Shift+P（macOS下是Command +Shift+P）或者点击菜单栏中的Tools > Command Palette...进入Sublime Text命令面板，输入install从列表中选择Install Package回车。
（此时Sublime Text开始请求远程插件仓库的索引，第一次请求会有一点慢，请稍等。）

详见：https://www.jianshu.com/p/4f9a2956f4d7












折腾了一天多，终于搞定了这个远程登录Ubuntu桌面的问题，感叹，为啥windows 那么多人喜欢用，就因为简单！
在公司里我有两台机器，一台win7 ,另一台前两天安装了ubuntu 11.10 ，搞两套键盘鼠标太烦，干脆给ubuntu搞个远程桌面（在ubuntu下远程win7的话，颜色什么的最高只能到24，很丑），这样就能把两台显示器连到一台机器上，同时用两台机器，一套键盘鼠标。
先说说win 下要做的设置：
       win 下其实很简单，到vncviewer去下载个客户端就OK了，很小的一个exe文件，直接执行，下载地址：
http://www.realvnc.com/products/free/4.1/winvncviewer.html
ubuntu vncserver ：
其实ubuntu 11.10 里面已经安装了 桌面共享 ，用的是 vino-server ，这个东西好是好，就是有个很不爽的缺点：必须要在ubuntu主机上登录过后才能在win 下用vncviewer登录。并且好像登录过后锁定或者注销都不能正常使用。
试过 vnc4server、tightvncserver、都有一些问题。最后使用了 x11vnc，一段配置下来，重启机器，OK。很爽，下面是步骤：
       1、安装x11vnc 
\$sudo apt-get install vino vinagre x11vnc  
        2、设置远程桌面登录时使用的密码，设置完后直接回车确认保存密码到      ~/.vnc/passwd  文件里，“~/  ”是你当前用户的根目录如： /home/jzy/
\$sudo x11vnc -storepasswd  
3、设置x11vnc通用的密码存储位置
\$sudo x11vnc -storepasswd in /etc/x11vnc.pass  
4、将用户目录下的passwd文件内容copy到 /etc/x11vnc.pass下
\$sudo cp /home/jzy/.vnc/passwd /etc/x11vnc.pass  
5、配置x11vnc为跟随系统自动启动
       需要新建一个文件  /etc/init/x11vnc.conf
\$sudo vi /etc/init/x11vnc.conf  
按 i 键进入编辑模式，粘贴以下内容，并保存退出：
start on login-session-start  
script  
x11vnc -display :0 -auth /var/run/lightdm/root/:0 -forever -bg -o /var/log/x11vnc.log -rfbauth /etc/x11vnc.pass -rfbport 5900
end script  
其中，5900是端口号，可以自己定义。
6、重启ubuntu
等重启好了以后，到win 下 打开 vncviewer ，输入ubuntu 的地址和5900端口号，如 ： 10.1.170.8:5900  然后连接，如果成功的话，会出现输入密码的对话框，
只需要输入上面设置好的密码就可以看到操作远程桌面啦！

参考文章：
http://ubuntuforums.org/showthread.php?t=1861707&page=3
转自：http://blog.csdn.net/jzy19861984/article/details/7178874

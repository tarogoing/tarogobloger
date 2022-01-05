
以下文章，经过验证，成功的
由于xrdp、gnome和unity之间的兼容性问题，在Ubuntu 14.04版本中仍然无法使用xrdp登陆gnome或unity的远程桌面，现象是登录后只有黑白点为背景，无图标也无法操作。与13.10中的解决方式相同，使用xrdp只能登录xfce的远程桌面。
相关阅读：
Ubuntu 14.04 下载、安装、配置 整理汇总 页面 http://www.linuxidc.com/Linux/2014-04/100370.htm
Windows 7下硬盘安装Ubuntu 14.04永久更新地址： http://www.linuxidc.com/Linux/2014-04/100369.htm
首先安装xfce：
\$sudo apt-get update
\$sudo apt-get install xfce4
如果网速较慢，这会持续一段时间。
然后安装xrdp组件和vnc服务器：
\$sudo apt-get install xrdp vnc4server
安装好后要自行新建配置文件，使得在远程登录时默认使用xfce作为界面登录，然后重启xrdp服务：
\$echo "xfce4-session" >~/.xsession
\$sudo service xrdp restart
这个相当于在当前用户的home目录下新建一个名为.xsession的隐藏文件，并向文件中写入一行xfce4-session。也可以用touch新建文件，并用vi编辑：
\$touch ~/.xsession
\$vi ~/.xsession
安装完毕以后，执行以下命令（该命令的作用是由于安装了 gnome桌面，ubuntu12.04中同时存在unity、GNOME多个桌面管理器，需要启动的时候指定一个，不然即使远程登录验证成功以后，也只是背景，其他什么也没有）,从ubuntu14.04的实际测试结果看，目前只能使用xfce4,其他的暂时不能使用。
至此安装和配置工作完成，即可在Windows中使用“远程桌面连接”程序连接Ubuntu主机。也可以通过任何其它支持RDP协议的终端连接，比如iOS平台上微软官方的远程桌面客户端RD Client。
更多Ubuntu相关信息见Ubuntu 专题页面 http://www.linuxidc.com/topicnews.aspx?tid=2
本文永久更新链接地址：http://www.linuxidc.com/Linux/2014-04/100491.htm

一般情况下我们用ssh客户端远程登陆Linux系统，至于图形界面下的linux远程登陆工具，我们一般都会想到vnc，但它的安全性不够，在这里，我将介绍XRDP的安装配置方法。
xrdp安装配置方法
xrdp依赖于pam和openssl-del，编译前需要先安装pam-devel和openssl-devel这两个包（不同发行版的包名称有一点不同）
2、下载好xrdp的安装包后，用tar -xvvzf 解压
进入解压出来的目录用root帐号执行make ，然后执行make install
3、xrdp需要vncserver，所以还要安装vncserver
4、准备好后，可以通过解压出来的目录下的instfiles目录下的xrdp-control.sh脚本启动xrdp
xrdp-control.sh start
可以把此脚本添加到/etc/rc.d/init.d/中，让它开机自动运行。
5、启动好xrdp，就可以通过客户端的rdp client 连接到服务器上，win下可以用mstsc，linux下可以用rdesktop或者krdp。
module 选择为：sesman-Xvnc
6、xrdp的配置文档在/etc/xrdp目录下的xrdp.ini和sesman.ini
xrdp.ini 关键部分在globals
[globals]
bitmap_cache=yes 位图缓存
bitmap_compression=yes 位图压缩
port=3389 监听端口
crypt_level=low 加密程度（low为40位，high为128位，medium为双40位）
channel_code=1 不知道是什么
sesman.ini
[Globals]
ListenAddress=127.0.0.1 监听ip地址(默认即可)
ListenPort=3350 监听端口(默认即可)
EnableUserWindowManager=1 1为开启,可让用户自定义自己的启动脚本
UserWindowManager=startwm.sh
DefaultWindowManager=startwm.sh
[Security]
AllowRootLogin=1 允许root登陆
MaxLoginRetry=4 最大重试次数
TerminalServerUsers=tSUSErs 允许连接的用户组(如果不存在则默认全部用户允许连接)
TerminalServerAdmins=tsadmins 允许连接的超级用户(如果不存在则默认全部用户允许连接)
[Sessions]
MaxSessions=10 最大会话数
KillDisconnected=0 是否立即关闭断开的连接(如果为1,则断开连接后会自动注销)
IdleTimeLimit=0 空闲会话时间限制(0为没有限制)
DisconnectedTimeLimit=0 断开连接的存活时间(0为没有限制)
[Logging]
LogFile=./sesman.log 登陆日志文件
LogLevel=DEBUG 登陆日志记录等级(级别分别为,core,error,warn,info,debug)
EnableSyslog=0 是否开启日志
SyslogLevel=DEBUG 系统日志记录等级
装好后，我们就可以直接从win系统下利用mstsc直接进行登陆，相当方便，如果是linux，可以用rdesktop。

Linux Xrdp 安裝
Xrdp 是开放原始码的远端桌面通讯协定 Remote Desktop Protocol 伺服器服务，可用来替代传统的 vnc server，以增进远端连线的效能。
以 apt 指令安装 xrdp 将会显示
vnc4server xbase-clients xrdp 等三个相依套件需要安装，记得在使用 Ubuntu 9.10 时，仍需加装「libpam0g-dev」和「libcurl4-openssl-dev」才能顺利运作 xrdp，所以安装指令为：
sudo apt-get install libpam0g-dev libcurl4-openssl-dev xrdp
不过，来到了 Ubuntu 10.04 这个版本，xrdp 版本虽然仍是 2008-07-18 的 v0.4.1，很好奇的试了一下只用这一行指令：
sudo apt-get install xrdp
系统已简化了安装流程，自动列出「vnc4server xbase-clients xrdp」三个相依套件，按下「enter」安装后好就可启用了，而且实测结果：连线成功！
xrdp 服务启动后，使用者就可以用 Windows 上的「远端桌面连线」来操作 Linux 的桌面了。对于惯用「远端桌面连线」的人来说，最大的好处在于不用再另外再安装 vnc 连线程式了。不过，第一次使用时将会发现，并非如 Windows 平台间的「远端桌面连线」那样，「直接」登入就可操作远端电脑。而是多了一个陌生的登入视窗，萤幕上显示的共有「sesman-Xvnc」、「console」、「vnc-any」……等六种登入选项。原来 xrdp 服务是以 Port 3389 接受「远端桌面连线」，操作桌面时再转交给主机中的 vncserver 来执行。
因此，选用「console」模式，就成了以本机连线方式操作了，这时只要输入 vnc 密码就可以登入了。而从「vnc-any」模式中的 IP 栏位可知道，这裡不仅可输入本机的 IP，或者「localhost」也行，试着指定其他提供 vnc 服务主机的 IP，照样也可以登入。本来是在 Ubuntu 9.10 版上大多以「console」模式，连线到被控端电脑，Ubuntu 10.04 似乎改变了使用者登入方式，这个「console」模式常常无法登入。还好预设的第一个模式「sesman-Xvnc」，输入使用者帐号、密码就能操作了。那就改用这个模式吧！
如果操作环境安全条件许可的话，将连线设定储存成「远端桌面连线」rdp 设定档，再配合「远端桌面连线」程式的「储存认证」功能，把密码记忆在使用者端的电脑中，使用时就可不用输入帐号、密码而直接登入了。
xrdp 的设定档是 /etc/xrdp/xrdp.ini
sudo vi /etc/xrdp/xrdp.ini
可看到以下内容：
[globals]
bitmap_cache=yes
bitmap_compression=yes
port=3389
crypt_level=low
channel_code=1
[xrdp1]
name=sesman-Xvnc
lib=libvnc.so
username=ask
……
[xrdp2]
name=console
lib=libvnc.so
ip=127.0.0.1
……
[xrdp3]
name=vnc-any
lib=libvnc.so
……
如果把其中的[xrdp1]和[xrdp2]的设定内容顺序对调，序号1和2也一併修改，这样连线选项顺位就会随着改变了。而且在 [globals] 这个项目中，可以看到预设的 Port 3389 也是在这裡设定的。
设定完重新启动 xrdp：
/etc/init.d/xrdp restart
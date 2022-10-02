# Openwrt研读笔记三之源码的下载和编译

哎呀，昨天拉下一天的笔记没写，不过这也不能怪我，是我的网站服务器出问题了，发布不了….今天补上昨天的内容
上一篇说到刷TL-WR703N的系统成openwrt，那些都是网上搜罗的，按照着做就好了，先学习才能进步嘛
今天要记录的内容有：下载源码，ubuntu13.04下编译源码
下载源码
首先你下载得准备几个工具，下载工具：svn或者git，编译工具：ubuntu的编译软件
我们还是以ubuntu为介绍先吧，下载ubuntu13.04并安装，你可以采用virtual box或者vmware，并配置好网络，确保能否上网，也就是要保证apt-get能下载，这一步我在这里就不做介绍了，改天有时间特别制作一个文章来描述。
如果不能上网，也可以通过DVD包来制作本地更新源来满足要求，但下载源码还是要网络的。
在满足了拥有ubuntu13.04、网络畅通的情况下，我们开始进行下面的工作。
首先，通过

```
apt-get install git-core
apt-get install subversion
```

通过上述两个命令，我们分别安装了git和svn工具，这两个工具是目前网络上使用最为广泛的代码管理工具，其中git适合于分布式，svn适合于集中管理，两个软件，我个人认为git更好用，只是git的图形软件很不给力，而svn的图形软件TortoiseSVN很给力，也很容易理解和上手，只是git在命令行界面也很不错，只是有些人认为命令行的工具总是不那么容易让人理解而已，关于这两个工具的使用，我也会在另外的文章再做介绍，只是个人使用经验不多，也只能描述简单的入门吧。
安装完毕这两个工具后，即可开始下载源码了，下载源码的官方方法：
```
https://dev.openwrt.org/wiki/GetSource
trunk (main development tree)
Main repository
git clone git://git.openwrt.org/openwrt.git
Packages feed
git clone git://git.openwrt.org/packages.git
12.09 branch (Attitude Adjustment)
Main repository
git clone git://git.openwrt.org/12.09/openwrt.git
Packages feed
git clone git://git.openwrt.org/12.09/packages.git
```

上面的方法是通过git clone下来的，这里稍微解释下，git的意思其实就是指代码仓库，每个git都会在本地拥有一个.git的文件夹进行代码的管理，这就方便了个人在本地添加，修改，删除，回退等操作，git clone是指将一个git库的代码clone到你本地，也就是你clone的代码地址和你本地进行同步，同步完成后你本地也成了保存代码的地方。
下载完成后，就能看到对应的文件。
下图是通过git下载完成后的tree图：

```
root@geeknimo-VirtualBox:/home/geeknimo/disk/study/openwrt_source/git_code# tree -L 3
.
├── 12.09
│   ├── openwrt
│   │   ├── BSDmakefile
│   │   ├── Config.in
│   │   ├── docs
│   │   ├── feeds.conf.default
│   │   ├── include
│   │   ├── LICENSE
│   │   ├── Makefile
│   │   ├── package
│   │   ├── README
│   │   ├── rules.mk
│   │   ├── scripts
│   │   ├── target
│   │   ├── toolchain
│   │   └── tools
│   └── packages
│   ├── admin
│   ├── devel
│   ├── ipv6
│   ├── lang
│   ├── libs
│   ├── mail
│   ├── multimedia
│   ├── net
│   ├── skels
│   ├── sound
│   └── utils
└── trunk
├── openwrt
│   ├── BSDmakefile
│   ├── Config.in
│   ├── docs
│   ├── feeds.conf.default
│   ├── include
│   ├── LICENSE
│   ├── Makefile
│   ├── package
│   ├── README
│   ├── rules.mk
│   ├── scripts
│   ├── target
│   ├── toolchain
│   └── tools
└── packages
├── admin
├── devel
├── ipv6
├── lang
├── libs
├── mail
├── multimedia
├── net
├── send
├── skels
├── sound
└── utils
43 directories, 14 files
```

svn的下载方法如下：
```
Development branch: ​ChangeLog
svn co svn://svn.openwrt.org/openwrt/trunk/
Attitude Adjustment 12.09 branch: ​ChangeLog
svn co svn://svn.openwrt.org/openwrt/branches/attitude_adjustment
Backfire 10.03 branch: ​ChangeLog
svn co svn://svn.openwrt.org/openwrt/branches/backfire
Kamikaze 8.09 branch: ​ChangeLog
svn co svn://svn.openwrt.org/openwrt/branches/8.09
Kamikaze 7.09 branch: ​ChangeLog
svn co svn://svn.openwrt.org/openwrt/tags/kamikaze_7.09
svn的代码我就不贴上来，大致是一样的。
```
### 编译源码
下载好源码后，还需要准备编译工具，安装的软件有些多，如果你不是root用户登陆的话，请使用sudo来执行命令。

```
sudo apt-get install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip sharutils subversion libncurses5-dev ncurses-term zlib1g-dev subversion git-core gawk asciidoc libz-dev
```

这个安装需要一些时间，所以建议大家还是制作本地的下载源比较好。
准备好上面的工具后我们就可以开始编译了。
进入到源码所在的目录，我这里选取的是主branch的openwrt的代码库
假设代码所在的位置为：

```
/home/geeknimo/disk/study/openwrt_source/git_code/trunk/openwrt
将openwrt的整个目录及子目录都赋予777权限，并进行源代码更新
chmod -R 777 openwrt
git pull
```

更新完毕后，进行种子更新
操作方法
更新种子列表，看起来是
```
./scripts/feeds update -a
```
更新种子在menuconfig中的显示列表

```
./scripts/feeds install -a
```

这个更新也需要一些时间。
更新完毕后，再进行安装下，这两个步骤完成后开始进行编译前配置了。
```
make defconfig
make menuconfig
```

执行这个命令的时候还提示了如下错误：

```
Build dependency: Please do not compile as root.
Prerequisite check failed. Use FORCE=1 to override.
make: *** [tmp/.prereq-build] Error 1
root@geeknimo-VirtualBox:/home/geeknimo/disk/study/openwrt_source/git_code/trunk/openwrt#
```

竟然还不能用root用户进行编译，不过我在后面加上了 FORCE=1，呵呵，这也是可以的，不过还是建议大家换成普通用户进行。

在这里进行我们所需要的配置
首先选择Target System为Atheros AR7xxx/AR9xxx，因为我们的TL-WR703N的主芯片是属于Atheros公司的Atheros AR7240 CPU
其次选择Target Profile是选择路由器的型号，我们选择(TP-LINK TL-WR703N)
其他的就看自己的喜好了，我随意勾选了一些，先编译了试试，据说这个编译普通的机器要3-5个小时，囧。
明天再说结果把，今天就到这里了。

[回到页首](../index.md)

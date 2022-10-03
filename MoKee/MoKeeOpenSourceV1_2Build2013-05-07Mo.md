# MoKee OpenSource项目介绍及开发流程 V1.2 Build 2013-05-07－Mo

## 一、项目说明
MoKee OpenSource是基于Google AOSP以及CyanogenMod源码开发的一个Android分支，
同时也是国内首个完整开源的Android项目，使用者和开发者遍布海内外。
项目跟随Google开源代码快速升级，并针对用户使用习惯，进行改进和功能增强。
魔趣论坛在2012年12月12日发起该项目,致力于做出CyanogenMod这种形式的本土化开源ROM.
项目开放源码,任何感兴趣的技术高手们都可以参与到开发中,为其贡献力量!
注：MoKee OpenSource 和 MoKee OS没有任何联系，MoKee OS已于2012年11月7日停止研发。

## 二、每个人能为MoKee OpenSource做什么
MoKee OpenSource是一个庞大的开源项目，项目的发展离不开每个人的努力。
在这个项目中，我们需要各种各样的帮助以支持这个项目更好的运作下去。

比如说：
1.程序语言汉化人才为项目提供多语言支持
2.界面设计人才为项目提供好看的UI
3.交互设计人才为项目提供动画效果和全新的操作体验
4.广大程序猿拓展功能或修复问题
5.ROM制作高手提供优化支持或适配移植到更多机型
6.此处省略一万字

## 三、开发环境
安装有Linux系统的电脑或有安装Linux系统虚拟机。

## 四、环境变量（以Ubuntu 12.04系统为例）
### 1.JDK安装:
```
$ sudo add-apt-repository "deb http://archive.canonical.com/ lucid partner"
$ sudo apt-get update
$ sudo apt-get install sun-java6-jdk
```
或
```
$ sudo add-apt-repository ppa:ferramroberto/java
$ sudo apt-get update
$ sudo apt-get install sun-java6-jdk
```
或
```
$ sudo add-apt-repository "deb http://us.archive.ubuntu.com/ubuntu/ hardy multiverse"
$ sudo apt-get update
$ sudo apt-get install sun-java6-jdk
```

这里提供三个可以安装sun-java6-jdk的源供大家选择

### 2.其它依赖:
```
$ sudo apt-get install git-core gnupg flex bison gperf build-essential \
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
  libgl1-mesa-dev g++-multilib mingw32 openjdk-6-jdk tofrodos \
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386
$ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
```

## 五、帐户配置
魔趣的Gerrit服务器采用OpenID方式注册，如果你拥有有Google或者Yahoo等支持的服务商帐户，便可快速完成注册登陆。
访问http://review.mfunz.com，点击右上角Register。（Google帐户登陆在国内并不稳定，会经常无法登陆，请多试几次或使用Yahoo帐户）
注册成功后登陆，参照下图所示进行设置。
### 六、Git配置
```
$ git config --global user.name 
$ git config --global user.email
```

注：请保持用户名和系统用户名一致，email地址@符号前名称与<username>一致避免未来向服务器提交修改出现权限错误。
### 七、项目初始化和同步
```
$ mkdir 
$ cd 
$ repo init -u https://github.com/MoKee/android.git -b jb-mr1_mkt   【可选地址一】
$ repo init -u ssh://@review.mfunz.com:29418/MoKee/android.git -b jb-mr1_mkt   【可选地址二】
$ repo sync
```

## 八、大功告成，开始折腾
### 1.编译命令:
在项目目录下执行
```
$ . build/envsetup.sh
$ lunch --选择要编译的设备
$ make bacon
```

### 2.修改前建立分支:
在项目目录下执行建立分支操作
```
$ repo start 
--all
```

### 3.修改后提交
```
$ git add 
$ git commit -a -m"修改内容说明，请使用英文"
```

### 4.上传到服务器等待审核
```
$ repo upload
```
## 九、讨论交流
在开发中有任何问题需要沟通请加入QQ群：285950190
提交新适配的机型请发邮件到：martincz.gao2012@gmail.com
为提高沟通效率，谢绝完全无基础人员进入。
## 备注：
MoKee OpenSource项目介绍及开发流程 V1.2 Build 2013-05-07
如有遗漏请及时指证。

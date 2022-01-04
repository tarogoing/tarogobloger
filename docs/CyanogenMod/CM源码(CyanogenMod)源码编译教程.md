# CM源码(CyanogenMod)源码编译教程

准备编译环境
注意: 编译环境只需要搭建一次，如果你之前搭好环境了，就直接跳到 从设备获取必须文件.

+ 安装ADB指令
+ 安装 Android SDK.

+ 安装编译必须的组件包
+ 安装编译ROM必须用到的一些组件包:

32位&64位系统都必须安装以下组件包:
```
git-core gnupg flex bison gperf libsdl1.2-dev libesd0-dev libwxgtk2.6-dev squashfs-tools build-essential zip curl libncurses5-dev zlib1g-dev sun-java6-jdk pngcrush schedtool
```
64位系统还需要安装一下组件包:
```
g++-multilib lib32z1-dev lib32ncurses5-dev lib32readline5-dev gcc-4.3-multilib g++-4.3-multilib
```
提示: 安装的时候可能会提示部分组件包被新的包代替，没有关系的。

提示: 如果是Ubuntu 10.10, 你必须通过以下命令增加一个合作源才可以安装sun-java6-jdk:
```
add-apt-repository "deb http://archive.canonical.com/ maverick partner"
```
创建目录
你必须先创建一些必须的目录来同步CM源码
输入以下命令建立bin目录用来存放repo等工具:
```
mkdir -p ~/bin
```
建立android/system目录来放置CM源码，这里的android和system都是可以按照个人需要改变的。比如cm/cm7、cm/cm9等等。
```
mkdir -p ~/android/system
```
安装Repo功能
通过一下命令安装 "repo" 工具:
```
$curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo ~/bin/repo
```
输入以下指令设置repo的权限：
```
chmod a+x ~/bin/repo
```
提示: 你可能需要重启电脑才能生效。 接下来对repo设置你要获取的源码分支:
```
cd ~/android/system/
repo init -u git://github.com/CyanogenMod/android.git -b gingerbread
repo sync -j16
```
从设备获取必须文件
NOTE: 这个步骤每台手机机器只要操作一次即可，不用每次编译都执行，之前执行过的话，直接跳转到下载RomManager这个步骤.
```
You will need to have a {{{device}}} with a working copy of CyanogenMod install and ADB working on the computer. This script will copy the proprietary files from the device. 
```
你的手机最好是已经正在使用CM系统，电脑必须安装ADB驱动，下面的指令就会把需要的文件从手机中提取出来用于下面的编译：
```
Connect the device to the computer and ensure that ADB is working properly.
cd ~/android/system/device/{{{vendor}}}/{{{device}}}/
./extract-files.sh
```
NOTE: 如果编译后的CM一些硬件无法正常工作，比如摄像、FM收音机，那么你必须更新部分的支持库文件到新版本。
下载RomManager
注意: 这个步骤仅仅是为了更新RomManager，如果你不想更新到最新版本，可以直接跳到 编译CM源码(CyanogenMod).
但是，要注意的是，RomManager是必须的，没有RomManager可能会出现编译不通过。
执行以下命令就可以了:
```
~/android/system/vendor/cyanogen/get-rommanager
```
编译CM源码(CyanogenMod)
更新源码
首先更新一下源码:
```
cd ~/android/system/
repo sync
```
确定机型 & 编译
确定你要编译的机型代号.
```
. build/envsetup.sh && brunch 机器代码
```
检查源码
```
First, check for updates in the source:
cd ~/android/system/
repo sync
```
刷机测试
在~/android/system/out/target/product/机器代码文件夹下可以找到编译好的ROM包，名称一般类似update.cm-XXXXX-signed.zip. 
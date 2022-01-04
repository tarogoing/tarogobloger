# Ubuntu14.04 下载 & 编译 Android5.1 源码

###  1.安装openjdk-7-jdk
Android 5.1 用到的jdk不再是Oracle 的 jdk ，而是开源的 openjdk，在ubuntu安装好后，使用如下命令安装jdk：
$sudo apt-get install openjdk-7-jdk   
安装好后，设置环境变量：
在/etc/profile 文件末尾加上：

```
JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/  
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin  
export JAVA_HOME  
export PATH  
```
### 2.安装编译依赖的软件
使用如下命令安装依赖软件：
```
$sudo apt-get install git gnupg flex bison gperf build-essential zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa- dri:i386 libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386 dpkg-dev
$sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
```

### 3.配置Cache
使用如下命令配置cache：
```
sudo apt-get install ccache 
source ~/.bashrc
```
### 4.下载repo
1）创建repo目录
```
$mkdir ~/bin 
$PATH=~/bin:$PATH
2）下载repo（官方的repo下载不了，其他的repo大多比较旧，这个时比较新的，我找了很久大哭）
$git clone git://aosp.tuna.tsinghua.edu.cn/android/git-repo.git/
克隆下来后将git-repo中的repo文件拷贝到bin目录
$cp git-repo/repo ~/bin/
修改repo文件，设置REPO_URL如下：
$REPO_URL = 'git://aosp.tuna.tsinghua.edu.cn/android/git-repo'
```

### 5.初始化repo
1）创建目录
```
mkdir ~/aosp
```
2）初始化repo
```
$cd ~/aosp 
$repo init -u git://aosp.tuna.tsinghua.edu.cn/android/platform/manifest -b android-5.1.1_r4
ps:在初始化时，提示需要email验证，使用如下命令：
$git config --global user.email "you@example.com"
$git config --global user.name "Your Name"
```

### 6.替换已有的AOSP源代码的remote
如果你之前已经通过某种途径获得了AOSP的源码，但是你希望以后通过TUNA同步，只需要将.repo/manifest.xml中的 
```
      <remote  name="aosp"
       fetch=".."
       review="https://android-review.googlesource.com/" />
改为下面的code即可： 
      <remote  name="aosp"
       fetch="git://aosp.tuna.tsinghua.edu.cn/android/"
       review="https://android-review.googlesource.com/" />
```

这个方法也可以用来在同步Cyanogenmod代码的时候从TUNA同步部分代码

### 7.下载源码
```
$repo sync 
```
ps:这里就是下载源码了，需要的时间比较长，我下行为1M的宽带需要4小时以上
### 8.源码编译
ps：编译过程比较就，我电脑双核的，使用单线程编译的，时间位12小时左右，如果使用多线程，时间应该会成倍减少
1）设置cache
```
cd aosp 
prebuilts/misc/linux-x86/ccache/ccache -M 50G
```
2）初始化编译环境
```
. build/envsetup.sh
```
3）选择编译目标包
ps：lunch的方式有很多中，可以使用lunch命令查看，我使用最常用的
```
lunch aosp_arm-eng
```
4）编译
```
make 
```
ps: 1.make后面可以更参数：如你的机器时双核，每核双线程的话，使用make -j4,这样速度更快，但编译时使用的内存也更多
2.make失败或停止后，可以使用make -k 继续编译

### 9.结果展示：
```
$emulator &
```

Ref:
Ubuntu14.10 编译 Android5.0 源码:
http://blog.csdn.net/chouretang/article/details/43769839
编译时参考的博文：
1.http://blog.csdn.net/gobitan/article/details/24367439
2.https://wiki.tuna.tsinghua.edu.cn/MirrorUsage/android
# linPhone-android 编译过程详解
今天编译完了linphone-android的源码，整整弄了我一整天的时间，快到绝望的时候，竟然一口气编译成功了。
```
.............................省略N多编译过程
Compile thumb : crypto-static <= dsa_ossl.c
Compile thumb : crypto-static <= dsa_sign.c
Compile thumb : crypto-static <= dsa_vrf.c
Compile thumb : crypto-static <= rmd_dgst.c
Compile thumb : crypto-static <= rmd_one.c
Compile thumb : crypto-static <= m_ripemd.c
Compile thumb : crypto-static <= ech_err.c
Compile thumb : crypto-static <= ech_key.c
Compile thumb : crypto-static <= ech_lib.c
Compile thumb : crypto-static <= ech_ossl.c
Compile thumb : crypto-static <= tb_ecdh.c
StaticLibrary : libcrypto-static.a
StaticLibrary : libssl-static.a
SharedLibrary : liblinphone.so
Install : liblinphone.so => jni/..//libs/armeabi/liblinphone.so
```

### 整理一下编译流程:
首先我是在linux环境下编译的-ubuntu。

到网站上下载linphone-android的源码:
http://www.linphone.org/eng/download/git.html

linphone-android对应的git地址是:
```
git clone git://git.linphone.org/linphone-android.git --recursive
```
记住一定要把rescursive给带上，否则下不全，下载完后大概有80M左右.


下载后首先看里面的readme.

1) download the Android ndk (>=r5c) from google.
我个人是android-ndk-r7c的最新版本.

2) install the autotools: autoconf, automake, aclocal, libtoolize,pkgconfig
这几个花费了我一上午的时间。

其实用apt-get install就可以搞定了。
```
sudo apt-get install autools-dev 可以自动帮你安装autoconf,automake,aclocal.
```
然后libtoolize的安装,不要想当然的用 sudo apt-get install libtoolize

正确的指令是: sudo apt-get install libtool

pkg-config系统自带的。

检测相关命令是否已经安装成功:
which autoconf
成功会显示命令的路径

上面的搞定后，开始执行./prepare_sources.sh这个时候呢，我是执行了2次才搞成功，理论上上面的4个命令OK的话，这里就应该提示：#head 1# ....success，具体的记得不是很清楚了.

然后:
```
${my google ndk directory}/ndk-build
```

首先会提示awk找不到，直接按照命令行的路径，把对应的文件给删除掉，然后接着编译又会提示:
/toolchains/arm-linux-androideabi-4.4.3/prebuilt/linux-x86/bin/arm-linux-androideabi-gcc 找不到的错误。

有问题找谷歌，解决方案:
```
step1:
sudo apt-get install libc6-dev-i386
step2:
sudo apt-get install ia32-libs
```
2步搞定后，重新执行:
```
${my google ndk directory}/ndk-build
```
这个时候等给3分钟左右，编译就成功了，然后libs目录下就有已经编译好的各种 so了。
```
armeabi/liblinphone.so
armeabi-v7a/libavcodec.so,libavcore.so,libavutil.so,liblincrypto.so,liblinphone.so,liblinssl.so,libsrtp.so,libswscale.so
```
***
下面是一个另外的文章，关于如何编译LinPhone For Android版本的。
***
公司生成电话，要做视频通话，在网上找了好多开源代码，都没有成功，linphone-android虽然网上说不是很好，但是编译成功了，可以进行开发了，现将我编译的步骤写下来，以供他人所用。
### 第一步：下载linphone-android源码：
git clone git://git.linphone.org/linphone-android.git --recursive

一定要加上 --recursive大小大约有397M左右，如果低于300M说明你下载的不全。
### 第二步：打开下载好的linphone-android阅读readme，
需要下载ndk，且版本不低于r8d,下载地址：
http://developer.android.com/tools/sdk/ndk/index.html选择linux版本
配置ndk环境：
1.将下载好的ndk进行解压，放到指定的目录下，如我放在home目录下的androidndk目录下。
2.配置path
### 第三步：根据readme说明安装autotools: autoconf,automake,aclocal等
1. sudo apt-get install autools-dev可以下载安装，如果编译是出现未按转autoconf,或者automake，还需要 sudo apt-get install autoconf等。
2.libtoolize的安装: sudo apt-get install libtool
第四步：如果在根目录下需要cd linphone-android目录下，然后开始执行./preparesources.sh；
如果在编译中提示 permission denied 权限限制，那么在前面加上sudo进行编译：sudo ./prepare_sources.sh
如果提示找不到 ndk-path，那么在后面加上ndk所在路径
开始编译
第五步：结束后：需要生成.so文件
在linphone-android目录下 $：ndk解压所在路径/ndk-build


等个几分钟，编译就成功了，然后libs目录下就有已经编译好的各种 so了。
armeabi/liblinphone.so
armeabi-v7a/libavcodec.so,libavcore.so,libavutil.so,liblincrypto.so,liblinphone.so,liblinssl.so,libsrtp.so,libswscale.so
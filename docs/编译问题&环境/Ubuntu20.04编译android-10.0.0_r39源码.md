# 基于Ubuntu20.04编译android-10.0.0_r39源码

### <a id="t0"></a><a id="t0"></a>文章目录

- [依赖安装](#_2)
- [编译error](#error_24)

- [\[error 01\] java.lang.OutOfMemoryError](#error_01_javalangOutOfMemoryError_26)

- [编译](#_31)
- [启动emulator](#emulator_46)

# <a id="t1"></a><a id="t1"></a><a id="_2"></a>依赖安装

下面下载的操作请大家参看我的另一篇文章《[基于ubuntu20.04使用国内镜像下载android-10.0.0_r39源码](https://blog.csdn.net/tianzong2019/article/details/106957540)》

```
sudo apt-get install -y bison build-essential ccache curl dpkg-dev flex g++-multilib gcc-multilib
sudo apt-get install -y gnupg gperf lib32ncurses5-dev lib32z-dev libc6-dev-i386 libesd0-dev libgl1-mesa-dev
sudo apt-get install -y libncurses5-dev:i386 libreadline6-dev:i386 libsdl1.2-dev libx11-dev libx11-dev:i386
sudo apt-get install -y libxml2-utils m4 tofrodos unzip x11proto-core-dev
sudo apt-get install -y xsltproc zip zlib1g-dev zlib1g-dev:i386
12345
```

一般安装`libesd0-dev`时会出现问题，其解决办法如下

> 解决办法:  
> sudo vim /etc/apt/sources.list //在行尾添加如下两行的内容  
> deb http://us.archive.ubuntu.com/ubuntu/ xenial main universe  
> deb-src http://us.archive.ubuntu.com/ubuntu/ xenial main universe  
> 更新软件源并重新安装:  
> sudo apt-get update && sudo apt-get install libesd0-dev

看网上有很多编译依赖项安装，里面与一些重复包，这里一并剔除了……

代码下载完成后，并安装了相关依赖项，下面就开始编译吧，不过编译的时间有点长，需耐心等待啊。

# <a id="t2"></a><a id="t2"></a><a id="error_24"></a>编译error

在编译的过程中，我碰到了一个错误

## <a id="t3"></a><a id="t3"></a><a id="error_01_javalangOutOfMemoryError_26"></a>\[error 01\] java.lang.OutOfMemoryError

其解决方法，就是调整java的heap空间，我是在`build/core/main.mk`文件中加入语句

```
export _JAVA_OPTIONS="-Xmx8g"  #增加heap到8G
1
```

# <a id="t4"></a><a id="t4"></a><a id="_31"></a>编译

下面的编译就是固定步骤了，先进入代码目录执行如下指令：

```
source build/envsetup.sh 
lunch
#当然也可以直接使用，lunch 24， 即aosp_x86_64项目
make -j4
1234
```

编译完成后，会出现如下类似打印，这就说明编译完成了。

```
#### build completed successfully (02:34:50 (hh:mm:ss)) ####
1
```

# <a id="t5"></a><a id="t5"></a><a id="emulator_46"></a>启动emulator

编译完成后，先进入代码目录，执行如下

```
source build/envsetup.sh 
lunch aosp_x86_64  #也就是我们上面编译的项目
emulator
123
```

-  <img width="22" height="22" src="../_resources/tobarThumbUp_beb42ac085ad4baa98b0141995fd9409.png"/> <a id="is-like-span"></a>点赞 <a id="spanCount"></a>2
- [<img width="22" height="22" src="../_resources/tobarComment_debe4ddb90fa475ab036fd41368dd3b1.png"/>评论](#commentBox)
- <img width="22" height="22" src="../_resources/tobarShare_d8cbc91037374ee59caa3c097bd5f1c5.png"/>分享
- <img width="22" height="22" src="../_resources/tobarCollect_b4194ccc84794198b8c3aa74662cb166.png"/><a id="is-collection"></a>收藏
-  <img width="22" height="22" src="../_resources/tobarMobile_3e1b6932953e4b63b536c8f6ec6ef6e1.png"/> 手机看 
- <img width="17" height="3" src="../_resources/lookMore_cb0e0d8dba7b4781a78a81700746b00e.png"/>
- 关注

[#### *编译*时出现java.lang.OutOfMemoryError Java heap space异常](https://download.csdn.net/download/Helen1978/676509)10-09

[*编译*时出现java.lang.OutOfMemoryError Java heap space异常.](https://download.csdn.net/download/Helen1978/676509)

***
### ubuntu20.04编译Android10
**过程问题**
**问题1**. 内存不足，导致ninjia被killed, 表现就是没有任何LOG，只有一行killed，然后编译停止。 解决：因为我的机器内存只有8G，android 10.0编译指导硬件配置为16G，所以我通过加大swap分区解决此问题，swap分区大小为16G.

**问题2**. 编译过程中，报java heap outofmemory. 解决：java堆内存不足，android 10.0中没有用jack编译了，改用了soong, 在项目根目录用如下命令配置java，编译过程中，会自动去pick up该值

**export _JAVA_OPTIONS="-Xmx4g"**

**问题3**. lunch过程中，出现各种的git问题 解决：查了一堆资料基本都是copy paste, 没办法，最终去.repo下面回到对应的git库，找到其中的url, 将github工程引导到gitlab，再用git clone单独下载下来。
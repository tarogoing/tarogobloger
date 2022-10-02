# linPhone-android编译(ubuntu)
编译linphone-android遇到不少问题，记录下来，备用。
### 1.下载代码，我是通过git下载的，
http://www.linphone.org/eng/download/git.html
git clone git://git.linphone.org/linphone-android.git --recursive
这个只是主工程，下面有好几个子工程，即submodule，如果下载顺利，会自动把子工程一起下载。
代码较大，下载后加上.git约500M，建议下载时找个网速较好的电脑。
但如果出错，或者中间断掉，有可能下载不全，会带来N多问题，建议重新下载。

### 2.下载安装jdk(略)。

### 3.下载安装android开发工具，下载最新的adt-bundle和ndk的linux版本
http://developer.android.com/sdk/index.html
我下载的是：
adt-bundle-linux-x86_64-20130219.zip
android-ndk-r8e-linux-x86_64.tar.bz2
下载下来解压即可。

### 4.安装ant:
apt-get install ant

### 5.按照linphone-android/README，安装相关的工具：
```
[java] view plaincopy
               LINPHONE for ANDROID  
            ****************************  
  
To build liblinphone for Android, you must:  
0) download the Android sdk with platform-tools and tools updated to latest revision (at least API 16 is needed), then add both 'tools' and 'platform-tools' folders in your path.  
1) download the Android ndk (>=r8b) from google and add it to your path.  
2) install the autotools: autoconf, automake, aclocal, libtoolize, pkgconfig  
2bis) on some 64 bits systems you'll need the ia32-libs package  
3) run the Makefile script in the top level directory. This will download iLBC source files and convert some assembly files in VP8 project.  
    $ make  
4) To install the generated apk into a plugged device, run  
    $ make install  
  
To run the tutorials:  
1) open the res/values/non_localizable_custom.xml file and change the value of the show_tutorials_instead_of_app to true.  
2) compile again using make && make install.  
3) /!\ don't forget to put it back to false to run the linphone application normally. /!\  
  
To create an apk with a different package name, you need to edit the custom_rules.xml file:  
1) look for the property named "linphone.package.name" and change it value accordingly  
2) run again the Makefile script by calling "make"  
  
Some options can be passed to make, like "make SOME_OPTION=SOME_VALUE".  
  
Option Name     |     Possible values                                                                                                        | Default value   
***
BUILD_X264            0 (don't build x264) or 1 (build x264)                                                                                 | 0  
BUILD_AMRNB           0 (don't build amrnb codec), light (try to use amrnb codec from android), full (build your own amrnb codec)            | full  
BUILD_AMRWB           0 (don't build amrwb codec), 1 (build your own amrwb codec)                                                            | 0  
BUILD_GPLV3_ZRTP      0 (don't support ZRTP), 1 (support ZRTP and make the whole program GPLv3)                                              | 0  
BUILD_SILK            0 (don't build silk plugin), 1 (build silk) [silk is Skype nonfree patented audio codec]                               | 1  
BUILD_G729            0 (don't build g729 plugin), 1 (build g729) [g729 is nonfree patented audio codec, contact Sipro lab for more details] | 0  
BUILD_TUNNEL          0 (don't build tunnel), 1 (build tunnel) [requires a tunnel implementation in submodules/linphone/tunnel]              | 0  
BUILD_WEBRTC_AECM     0 (don't build echo canceler), 1 (build echo canceler)                                                                 | 1  
USE_JAVAH             0 (don't generate header), 1 (generate header for linphone_core_jni) [used to check errors at liblinphone compilation] | 1  
BUILD_FOR_X86         0 (don't generate liblinphone libraries for x86 architecture), 1 (build liblinphone libraries for x86 architecture)    | 1  
apt-get install autoconf  automake
```
aclocal安装不上，先不管它。
其中libtoolize、pkgconfig不能直接安装不上的，如下处理：
apt-get install libtool
apt-get install pkg-config
我用的ubuntu是64位的，需要安装ia32-libs：
apt-get install ia32-libs
这里安装的东西很多，需要较长时间。

### 6.配置ndk和sdk tools环境：
```
export PATH=$(./path.sh)
```
path.sh:这里是ndk和sdk中tools、platform-tools的路径，需要根据实际路径修改
```
#!/bin/bash  
export PATH=~/wifi-player/android-dev/adt-bundle-linux-x86_64-20130219/sdk/tools:~/wifi-player/android-dev/adt-bundle-linux-x86_64-20130219/sdk/platform-tools:~/wifi-player/android-dev/android-ndk-r8e:$PATH  
echo $PATH  
```
### 7.编译：
```
make
```
编译时，会自动下载一个压缩包，并解压：
```
submodules/mssilk/sdk/SILK_SDK_SRC_v1.0.8.zip
```
这个也可以自己用先下载好，放到对应目录即可。
如果顺利，会生成bin/Linphone-debug.apk

### 出现的问题：
#### 1.下面是我下载的代码，在最后的提交中，有一个错误。
```
commit d7024542041ea43b5241d8880bfe8c7e94c6c71b  
Author: Sylvain Berfini <sylvain.berfini@linphone.org>  
Date:   Fri Mar 22 17:42:13 2013 +0100  
  
    CpuUtils moved to mediastreamer  
CpuUtils.java被移到
submodules/linphone/mediastreamer2/java/src/org/linphone/mediastream/CpuUtils.java
但在submodules/linphone/java/impl/org/linphone/core/LinphoneCoreFactoryImpl.java中引用却没有修改，这里需要自己修改下：
[java] view plaincopy
import org.linphone.mediastream.CpuUtils;  
```
#### 2.第一次编译通过后，再次make，会重新下面的错误：
```
Archive:  ./SILK_SDK_SRC_v1.0.8.zip  
replace SILK_SDK_SRC_v1.0.8/SILK_SDK_SRC_ARM_v1.0.8/Makefile? [y]es, [n]o, [A]ll, [N]one, [r]ename: A  
  inflating: SILK_SDK_SRC_v1.0.8/SILK_SDK_SRC_ARM_v1.0.8/Makefile    
...  
caution: filename not matched:  SILK_SDK_SRC_v1.0.8/SILK_SDK_SRC_ARM_v1.0.8/test_vectors  
make[1]: *** [SILK_SDK_SRC_v1.0.8] Error 11  
make[1]: Leaving directory `/mnt/wifi-player/wifi-audio/linphone-android/submodules/mssilk/sdk'  
SILK audio plugin prepare state failed.   
```
这是因为make时，会解压zip文件，提示是否覆盖，但是无论怎么选择，都会报错。
所以直接修改submodules/mssilk/sdk/Makefile，把解压zip的命令去掉即可：
```
$(silk_src_dir): #$(silk_extracted_directory)                                                                                                               
    cp $(srcdir)/patch_pic.diff $(silk_src_dir)  
    cd $(silk_src_dir) && $(PATCH) -p0 < patch_pic.diff  
```
    
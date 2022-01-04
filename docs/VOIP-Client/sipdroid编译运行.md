# sipdroid编译运行
在官网选择下载的连接
打开eclipse-<help-<install new software-<add

更新安装完后重启
使用eclipse从code.google下载sipdroid源码
eclipse-<window-<open perspective中选择

切换到svn视图

对项目右键检出为,存到自己的workspace(项目出现红色大叹号)
切换回java视图,对项目右键-<team-<断开连接
从其他项目出复制project.properties到本项目里面,更改target
对本项目右键-<android tools-<andd support library(刷新项目,至此大叹号消除)
打开cygwin进入到项目的jni文件夹,ndk-build出错,错误以及解决办法如下
参考博客:http://blog.csdn.net/harry_helei/article/details/7400338
错误1
```
Android NDK: There is no Android.mk under /home/helei/workspace/raydroid/jni/jni      Android NDK: If this is intentional  please define APP_BUILD_SCRIPT to point     Android NDK: to a valid NDK build script.      /home/helei/android_toolchain/android-ndk-r7b/build/core/add-application.mk:143: *** Android NDK: Aborting...    .  Stop.  
修改jni目录下的Application.mk文件中：
APP_PROJECT_PATH := $(call my-dir)
这一行，将其修改为：
APP_PROJECT_PATH := $(call my-dir)/..
```
错误2
```
Android NDK: /home/helei/workspace/raydroid/jni/../jni/Android.mk:silkcommon: LOCAL_MODULE_FILENAME must not contain a file extension/home/helei/android_toolchain/android-ndk-r7b/build/core/build-static-library.mk:29: *** Android NDK: Aborting.Stop.  
解决办法：打
开jni目录下的Android.mk文件，在如下代码位置：
SILK     := silk  
LOCAL_MODULE    := silkcommon  
LOCAL_SRC_FILES :=  $(SILK)/src/SKP_Silk_A2NLSF.c \      $(SILK)/src/SKP_Silk_CNG.c \      $(SILK)/src/SKP_Silk_HP_variable_cutoff_FIX.c \  

修改之后为：

include $(CLEAR_VARS)  
SILK     := silk  
LOCAL_MODULE    := silkcommon  
LOCAL_SRC_FILES :=  $(SILK)/src/SKP_Silk_A2NLSF.c \      $(SILK)/src/SKP_Silk_CNG.c \      $(SILK)/src/SKP_Silk_HP_variable_cutoff_FIX.c \  
```
错误3
```
Compile++ thumb  : speex_jni <  speex jnispan style='font-size:14px;font-style:normal;font-weight:400;color:#0000ff;'   >cpp</span>H:/workspace/SipUA/jni/../jni/speex_jni.cpp:26:25: 
fatal error: speex/speex.h: No such file or directorycompilation terminated
./cygdrive/h/android/android-ndk-r8/build/core/build-binary.mk:255: 
recipe for target `/cygdrive/h/workspace/SipUA/obj/local/armeabi/objs/speex_jni/speex_jni.o' failed
make: *** [/cygdrive/h/workspace/SipUA/obj/local/armeabi/objs/speex_jni/speex_jni.o] Error 1
```
解决办法:把jni文件夹中的speex-1.2rc1/include/speex文件夹拷贝到jni目录下
ndk-build编译成功~~~~
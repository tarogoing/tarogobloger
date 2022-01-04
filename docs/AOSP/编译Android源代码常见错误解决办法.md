# 编译Android源代码常见错误解决办法

### 1. 编译时出现
```
/usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-linux-gnu/4.4.5/../../../libz.so when searching for -lz
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=AOSP
TARGET_PRODUCT=generic
TARGET_BUILD_VARIANT=eng
TARGET_SIMULATOR=
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv5te
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
============================================
host Executable: aapt (out/host/linux-x86/obj/EXECUTABLES/aapt_intermediates/aapt)
/usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-linux-gnu/4.4.5/../../../libz.so when searching for -lz
/usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-linux-gnu/4.4.5/../../../libz.a when searching for -lz
/usr/bin/ld: skipping incompatible //usr/lib/libz.so when searching for -lz
/usr/bin/ld: skipping incompatible //usr/lib/libz.a when searching for -lz
/usr/bin/ld: cannot find -lz
collect2: ld returned 1 exit status
make: *** [out/host/linux-x86/obj/EXECUTABLES/aapt_intermediates/aapt] Error 1
```

解决办法：缺少lib32z1-dev,安装即可:apt-get install lib32z1-dev

### 2. [android]编译时出现

```
/usr/include/gnu/stubs.h:7: fatal error: gnu/stubs-32.h: No such file or directory 错误信息
编译时出现 /usr/include/gnu/stubs.h:7: fatal error: gnu/stubs-32.h: No such file or directory 
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=AOSP
TARGET_PRODUCT=generic
TARGET_BUILD_VARIANT=eng
TARGET_SIMULATOR=
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv5te
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
***
host C: acp <= build/tools/acp/acp.c
In file included from /usr/include/features.h:387,
from /usr/include/stdlib.h:25,
from build/tools/acp/acp.c:11:
/usr/include/gnu/stubs.h:7: fatal error: gnu/stubs-32.h: No such file or directory
compilation terminated.
make: *** [out/host/linux-x86/obj/EXECUTABLES/acp_intermediates/acp.o] Error 1
```

解决办法： 缺少libc开发包，安装即可: apt-get install libc6-dev-i386

### 3. 编译时出现 
```
/usr/include/gnu/stubs.h:7: fatal error: gnu/stubs-32.h: No such file or directory 
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=AOSP
TARGET_PRODUCT=generic
TARGET_BUILD_VARIANT=eng
TARGET_SIMULATOR=
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv5te
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
***
host C: acp <= build/tools/acp/acp.c
In file included from /usr/include/features.h:387,
from /usr/include/stdlib.h:25,
from build/tools/acp/acp.c:11:
/usr/include/gnu/stubs.h:7: fatal error: gnu/stubs-32.h: No such file or directory
compilation terminated.
make: *** [out/host/linux-x86/obj/EXECUTABLES/acp_intermediates/acp.o] Error 1
```
解决办法：缺少libc开发包，安装即可: apt-get install libc6-dev-i386

### 4. [android]初始化代码仓库时出现
```
“OSError: [Errno 2] No such file or directory”错误
OSError: [Errno 2] No such file or directoryroot@shanmin-ubuntu:/home/android/src# ../repo init -u git://android.git.kernel.org/platform/manifest.git
Traceback (most recent call last):
File "../repo", line 595, in <module>
main(sys.argv[1:])
File "../repo", line 562, in main
_Init(args)
File "../repo", line 181, in _Init
_CheckGitVersion()
File "../repo", line 210, in _CheckGitVersion
proc = subprocess.Popen(cmd, stdout=subprocess.PIPE)
File "/usr/lib/python2.6/subprocess.py", line 623, in __init__
errread, errwrite)
File "/usr/lib/python2.6/subprocess.py", line 1141, in _execute_child
raise child_exception
OSError: [Errno 2] No such file or directory
```

解决办法： 这是由于没有安装git造成的，安装上git就可以了。按说程序里面应该判断一下系统是否安装了git，不知道为什么没有判断。 
```
apt-get install git
```

### 5. 太不容易了，终于看到自己编译的Android了

可能是我使用的机器比较慢，虚拟机运行的有些慢啊。。。。
太不容易了，终于看到自己编译的Android了 - - 
下一步开始研究这个系统怎样去定制了....

```
编译Android，遇到Your version is: /bin/bash: java: command not found.错误的解决

Your version is: /bin/bash: java: command not found.============================================
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=AOSP
TARGET_PRODUCT=generic
TARGET_BUILD_VARIANT=eng
TARGET_SIMULATOR=
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
***
/bin/bash: bison: command not found
Checking build tools versions...
************************************************************
You are attempting to build with the incorrect version
of java.

Your version is: /bin/bash: java: command not found.
The correct version is: 1.6.

Please follow the machine setup instructions at
http://source.android.com/source/download.html
************************************************************
build/core/main.mk:114: *** stop. Stop.
```

解决办法： 这是由于没有装jdk导致的，可以到sun.com下载jdk后安装，建议安装到/usr/lib /jvm目录下，例如我下载的安装文件为 jdk-6u21-linux-i586.bin，安装完后生成一个jdk1.6.0_21的目录，然后使用ln -s jdk1.6.0_21 java-6-sun命令做一个链接，这样以后再升级sun jdk时只需要改动一下链接就可以了。

### 6. 编译Android，

遇到Could not load 'clearsilver-jni'错误的解决
```
Could not load 'clearsilver-jni'===========================================
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=AOSP
TARGET_PRODUCT=generic
TARGET_BUILD_VARIANT=eng
TARGET_SIMULATOR=
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
***
Docs droiddoc: out/target/common/docs/api-stubs
Could not load 'clearsilver-jni'
java.library.path = out/host/linux-x86/lib
make: *** [out/target/common/docs/api-stubs-timestamp] Error 45
```

从网上查得的解决办法：

```
$make clean
$make update-api (经测试，这个可以不需要)
$make
```

### 7. Android编译环境中的JDK存放位置
因为Ubuntu 10.04已经不带有SUN JDK，所以这个需要到sun网站上下载，并手动安装。所以，这个安装位置的问题就出现了。开始的时候没有注意，随便找了一个位置，并且设置了 JAVA_HOME就可以正常使用了。后来查看build/envsetup.sh才发现，如果没有设置JAVA_HOME的时候，编译环境会自动设置为 /usr/lib/jvm/java-6-sun ,所以建议直接安装到这个目录，还省得进行设置。

杯具了，VMware虚拟盘文件出现错误...

晕死了，不说别的，就下载Android的源代码就得差不多一天啊......

杯具了，VMware虚拟盘文件出现错误... - - 
似乎昨晚关机的时候强关的机器，没想到会影响这么大。。。。。

### 8. Android编译遇到错误

/usr/bin/ld: cannot find -lstdc++的解决
首先发现编译2.2版，gcc4.3和gcc4.4没有什么区别。
```
/usr/bin/ld: cannot find -lstdc++
***
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=AOSP
TARGET_PRODUCT=generic
TARGET_BUILD_VARIANT=eng
TARGET_SIMULATOR=
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
***

host SharedLib: libneo_util (out/host/linux-x86/obj/lib/libneo_util.so)
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.so when searching for -lstdc++
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.a when searching for -lstdc++
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.so when searching for -lstdc++
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.a when searching for -lstdc++
/usr/bin/ld: cannot find -lstdc++
collect2: ld returned 1 exit status
make: *** [out/host/linux-x86/obj/lib/libneo_util.so] 错误 1
```

解决办法：缺少g++-multilib库，安装即可： apt-get install g++-multilib 

### 9. Android编译遇到错误
环境： vmware + ubuntu 10.04
使用gcc 4.3或gcc 4.4都会出错误信息：
```
host SharedLib: libneo_util (out/host/linux-x86/obj/lib/libneo_util.so)
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.so when searching for -lstdc++
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.a when searching for -lstdc++
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.so when searching for -lstdc++
/usr/bin/ld: skipping incompatible /usr/lib/gcc/i486-linux-gnu/4.3.4/libstdc++.a when searching for -lstdc++
/usr/bin/ld: cannot find -lstdc++
collect2: ld returned 1 exit status
make: *** [out/host/linux-x86/obj/lib/libneo_util.so] 错误 1
```

现在不知道怎么解决，按照http://www.ways2u.com/?post=163 写的使用gcc 4.3就不会有这个问题，但我这边还是出现这个错误。。。。 

### 10. android编译环境

android所有源代码在 http://android.git.kernel.org/

如果在Windows下只能使用git一个项目一个项目的下载，如果在linux可以直接使用repo下载全部代码

linux下的全部下载方式见 http://source.android.com/source/git-repo.html

必须安装gcc 4.3才可以，例如我用的ubuntu 10.04默认装的是4.4，编译就会出错。

在装完Eclipse & SDK后，编译Android需要安装部分软件：

```
$ sudo apt-get install bison
$ sudo apt-get install g++
$ sudo apt-get install libc6-dev-amd64
```

到源代码目录执行
```
make
```
即可
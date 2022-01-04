# Android7.0/8.1搭建编译环境
本部分介绍了如何设置本地工作环境来编译 Android 源文件。您需要使用 Linux 或 Mac OS。目前不支持在 Windows 环境下进行编译。
要简要了解代码审核和代码更新的整个过程，请参阅补丁程序的生命周期。
### 一、选择分支
针对编译环境的某些要求是由您打算编译的源代码的版本决定的。要查看您可以选择的分支的完整列表，请参阅版本号。您还可以选择下载并编译最新的源代码（称为 master）。如果您选择这么做，请在初始化存储库时直接忽略分支规范。
选择分支后，请按照下面的相应说明来设置编译环境。
### 二、设置 Linux 编译环境
以下说明适用于所有分支（包括 master）。
我们会定期在最近推出的一些 Ubuntu LTS (14.04) 版本中对 Android 编译过程进行内部测试，但大多数 Ubuntu 分发版本都应该有所需的编译工具。欢迎向我们报告在其他分发版本中的测试结果（无论结果是成功还是失败）。
如果是 Gingerbread (2.3.x) 及更高版本（包括 master 分支），需要使用 64 位环境。如果是较低的版本，则可以在 32 位系统中进行编译。
注意：要查看完整的硬件和软件要求列表，请参阅相关要求。然后，请按照下方适用于 Ubuntu 和 Mac OS 的详细说明进行操作。
### 三、安装 JDK
Android 开放源代码项目 (AOSP) 中 Android 的 master 分支需要使用 Java 8。在 Ubuntu 中则需要使用 OpenJDK。
对于较低的版本，请参阅 JDK 要求。
如果 Ubuntu >= 15.04
请运行以下命令：
```
$ sudo apt-get update
$ sudo apt-get install openjdk-8-jdk
```
如果是 Ubuntu LTS 14.04
目前没有适用于 Ubuntu 14.04 的受支持 OpenJDK 8 程序包。Ubuntu 15.04 OpenJDK 8 程序包能够在 Ubuntu 14.04 中成功使用。我们发现，按照以下说明操作时，更高的程序包版本（例如适合 15.10、16.04 的版本）在 Ubuntu 14.04 中无法正常工作。
从 archive.ubuntu.com 下载适合 64 位架构的 .deb 程序包：
```
openjdk-8-jre-headless_8u45-b14-1_amd64.deb（SHA256：0f5aba8db39088283b51e00054813063173a4d8809f70033976f83e214ab56c0）
openjdk-8-jre_8u45-b14-1_amd64.deb（SHA256：9ef76c4562d39432b69baf6c18f199707c5c56a5b4566847df908b7d74e15849）
openjdk-8-jdk_8u45-b14-1_amd64.deb（SHA256：6e47215cf6205aa829e6a0a64985075bd29d1f428a4006a80c9db371c2fc3c4c）
```
（可选）对照随以上每个程序包列出的 SHA256 字符串，确认已下载文件的校验和。
例如，使用 sha256sum 工具：
```
$ sha256sum {downloaded.deb file}
```
安装程序包：
```
$ sudo apt-get update
```
为下载的每个 .deb 文件运行 dpkg。运行过程中可能会因缺少依赖项而出现错误：
```
$ sudo dpkg -i {downloaded.deb file}
```
解决缺少依赖项的问题：
```
$ sudo apt-get -f install
```
更新默认的 Java 版本 - 可选
（可选）对于以上 Ubuntu 版本，您可以通过运行以下命令来更新默认的 Java 版本：
```
$ sudo update-alternatives --config java
$ sudo update-alternatives --config javac
```
在编译过程中，如果您遇到 Java 版本错误，请按照错误的 Java 版本部分中的说明设置其路径。
### 四、安装所需的程序包 (Ubuntu 14.04)
您将需要 64 位版本的 Ubuntu。建议您使用 Ubuntu 14.04。
```
$ sudo apt-get install git-core gnupg flex bison gperf build-essential \
  zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
  lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
  libgl1-mesa-dev libxml2-utils xsltproc unzip
```
注意：要使用 SELinux 工具进行政策分析，您还需要安装 python-networkx 程序包。
注意：如果您使用 LDAP 并且希望运行 ART 主机测试，则还需要安装 libnss-sss:i386 程序包。
20190409添加
```
$sudo apt-get install git gnupg flex bison gperf build-essential
$sudo apt-get install zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev
$sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386
$sudo apt-get install libgl1-mesa-dev g++-multilib mingw32 tofrodos
$update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 100
$update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 100
```
使用update-alternatives设置gcc和g++：
update-alternatives是ubuntu系统中专门维护系统命令链接符的工具，通过它可以很方便的设置系统默认使用哪个命令、哪个软件版本。
其中40 ，50 ，70是优先级数值可以自己设定，--slave能保证gcc和g++保持相同的版本。
```
$sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 40 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
$sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 50 --slave /usr/bin/g++ g++ /usr/bin/g++-5
$sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-7 70 --slave /usr/bin/g++ g++ /usr/bin/g++-7
```
使用如下命令选择gcc的版本：
```
$sudo update-alternatives --config gcc
```
可以看到当前gcc默认的版本是gcc-7，下面我们修改为gcc-4.8，直接选择编号即可
ubuntu 18.04编译Android8.1出错。
在.profile或者.bashrc中增加
```
$export LC_ALL=C
```
经过实际测试，在编译前，使用命令行也是可以的。
### 五、安装所需的程序包 (Ubuntu 12.04)
您可以使用 Ubuntu 12.04 来编译较低版本的 Android。master 或最近推出的一些版本不支持 Ubuntu 12.04。
```
$ sudo apt-get install git gnupg flex bison gperf build-essential \
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
  libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
  libgl1-mesa-dev g++-multilib mingw32 tofrodos \
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386
$ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
```
 	    
不再支持在 Ubuntu 10.04-11.10 中进行编译，但它们仍可用来编译较低版本的 AOSP。
```
$ sudo apt-get install git gnupg flex bison gperf build-essential \
  zip curl zlib1g-dev libc6-dev lib32ncurses5-dev ia32-libs \
  x11proto-core-dev libx11-dev lib32readline5-dev lib32z-dev \
  libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown \
  libxml2-utils xsltproc
```
在 Ubuntu 10.10 中，请运行以下命令：
```
$ sudo ln -s /usr/lib32/mesa/libGL.so.1 /usr/lib32/mesa/libGL.so
```
在 Ubuntu 11.10 中，请运行以下命令：
```
$ sudo apt-get install libx11-dev:i386
```
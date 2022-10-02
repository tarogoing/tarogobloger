# 编译openwrt全过程（超详细）

本教程的编译环境：
    win7 专业版+VMwareWorkstation6.5虚拟机+Ylmf OS 3.0
编译的过程中要保持电脑联网
搭建编译环境
1. 应用程序--附件--终端

```
$sudo apt-get update      (更新）
```

2. 安装编译需要的组件：

```
$sudo apt-get install gcc 
$sudo apt-get install g++ 
$sudo apt-get install binutils 
$sudo apt-get install patch 
$sudo apt-get install bzip2 
$sudo apt-get install flex 
$sudo apt-get install bison 
$sudo apt-get install make 
$sudo apt-get install autoconf 
$sudo apt-get install gettext 
$sudo apt-get install texinfo 
$sudo apt-get install unzip 
$sudo apt-get install sharutils 
$sudo apt-get install subversion 
$sudo apt-get install libncurses5-dev 
$sudo apt-get install ncurses-term 
$sudo apt-get install zlib1g-dev 
$sudo apt-get install gawk
$sudo apt-get install asciidoc
$sudo apt-get install libz-dev
```

3. 编译环境搭建完成

```
$mkdir openwrt 创建一个openwrt文件夹
$cd openwrt    进入openwrt文件夹
$svn co svn://svn.openwrt.org/openwrt/branches/backfire  下载官网的源码
$./scripts/feeds update -a     更新软件包
$./scripts/feeds install -a    安装软件包
$make menuconfig 进入定制界面（里面可以选择芯片的型号，集成的组件等等，根据实际情况选择）
$defconfig
$make V=99   （开始编译）
```

剩下的就是等待了，第一次编译需要的时间相对比较长，这个跟你的电脑配置和网速有关。

4. 下面以编译TP-LINK 741N的openwrt固件为例，只编译基本的功能：

```
$make menuconfig
Target System---AR71xx/AR7240/AR913x/AR934x CPU型号
Target Profile---TP-LINK 741
LuCI—>Collections—– <*> luci 添加Luci
LuCI—>Translations—- <*> luci-i18n-chinese   添加中文
EXT----YES   
$make V=99    开始编译
```

成功后在bin文件夹里有编译好的固件。

[回到页首](../index.md)

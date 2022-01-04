# ubuntu20.04编译MT8788

适用SPRD的可以编译Android10的机器，编译MT8788版本
出现错误，跟踪LOG发现
有下面的错误信息
```
=================================================================
Can't locate Switch.pm in @INC (you may need to install the Switch module) (@INC contains: /home/cybertron/work/MediaTek/p86_android90/vendor/mediatek/proprietary/bootable/bootloader/preloader/tools/emigen/MT6771/../Spreadsheet 

```
### 解决方法：
在Perl脚本中使用switch语法，执行时报错“Can't locate Switch.pm in @INC ”，原因是Perl默认没有安装Switch模块，需要自行安装。

安装方法：
1、通过包管理器安装：
```
$sudo apt-get install libswitch-perl
```

标记安装libswitch-perl软件包。


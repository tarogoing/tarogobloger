# Android 编译SDK 问题汇总
经过两个多月的项目。android 3.2 qcom 8660，总结一下遇到的问题。
## 编译android sdk问题汇总：
### 官方的文档和问题：
```
src/sdk/docs/howto_build_SDK
  $build/envsetup.sh
  $lunch sdk-eng
  $make sdk
   Package SDK: out/host/darwin-x86/sdk/android-sdk_eng.<build-id>_mac-x86.zip
make window sdk 需要mingw的支持
  $ make win_sdk
```

### 实际工作的步骤
```
1. 修改 ”\4030\external\libvpx\vpx_config.h” 如下
//#define CONFIG_ARM_ASM_DETOK 1 
2. $ source build/envsetup.sh 
    lunch 1 ; make
3. $ lunch sdk-eng
4. $ make sdk 
```

在 out/host/linux-x86/sdk/android-sdk_eng.xxxx_linux-x86.zip 下
注意： android-sdk_eng.xxxx_linux-x86.zip 並不包含 emulator, 
你必須手动將out/host/linux-x86/bin/emulator 复制到 android-sdk_eng.xxxx_linux-x86.zip/android-sdk_eng.xxxx_linux-x86/tools/  下
问题汇总，qcom的BSP也有许多问题。在android 标准的sdk上修改了许多。有些在原先在android标准的版本上没有的：经历了几个版本，问题先后记录如下：
```
*** 
\$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
### ERROR 1：
 error: undefined reference to 'vp8_decode_mb_tokens_v6'
make sdk error:
target SharedLib: libstagefright (out/target/product/generic/obj/SHARED_LIBRARIES/libstagefright_intermediates/LINKED/libstagefright.so)
prebuilt/linux-x86/toolchain/arm-linux-androideabi-4.4.x/bin/../lib/gcc/arm-linux-androideabi/4.4.3/../../../../arm-linux-androideabi/bin/ld: out/target/product/generic/obj/STATIC_LIBRARIES/libvpx_intermediates/libvpx.a(detokenize.o): in function vp8_decode_mb_tokens:external/libvpx/vp8/decoder/detokenize.c:223: error: undefined reference to 'vp8_decode_mb_tokens_v6'
collect2: ld returned 1 exit status
make: *** [out/target/product/generic/obj/SHARED_LIBRARIES/libstagefright_intermediates/LINKED/libstagefright.so] 错误 1
```
### 修改方法：
1. 修改 ”\4030\external\libvpx\vpx_config.h” 如下
//#define CONFIG_ARM_ASM_DETOK 1  这句话是针对armv6的。sdk的模拟器是armv5，所以不需要
```
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
```
### ERROR2：
```
Package SDK: out/host/linux-x86/sdk/android-sdk_eng.liuhongchao_linux-x86.zip
SDK: warning: including GNU target out/target/product/msm8660_surf/system/lib/libdbus.so
sdk/build/tools.atree:46: couldn't locate source file: usr/share/pc-bios/bios.bin
sdk/build/tools.atree:47: couldn't locate source file: usr/share/pc-bios/vgabios-cirrus.bin
sdk/build/tools.atree:133: couldn't locate source file: framework/ddmlib-tests.jar
sdk/build/tools.atree:134: couldn't locate source file: framework/ninepatch-tests.jar
sdk/build/tools.atree:135: couldn't locate source file: framework/common-tests.jar
sdk/build/tools.atree:137: couldn't locate source file: framework/sdkuilib-tests.jar
make: *** [out/host/linux-x86/sdk/android-sdk_eng.liuhongchao_linux-x86.zip] Error 44
NO sdk/android-sdk_eng.liuhongchao_linux-x86.zip created
```
解决办法：
这里的framework目录指的是：~/Android_Src/out/host/linux-x86/framework这个目录，
是sdk/build/tools.atree这个文件有bug，上面那几个文件的路径写的不对，其实在
```
Src/out/host/linux-x86/framework 目录下是有这几个文件的，
$cp ~/Android_Src/prebuilt/common/pc-bios   ~/Android_Src/usr/share 
$cd ~/Android_Src/out/host/linux-x86/framework
$cp ddmlib.jar  ddmlib-test.jar   
$cp sdkuilib.jar  sdkuilib-test.jar //其他同样
$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
```
 
### ERROR 3：
emulator 显示为offline ，无法连接adb调试程序。
使用cmd   查看emulator 启动log如下：

```
liuhongchao@liuhongchao-ThinkCentrer:~/sdk/4037/4037-orig-qcom/out/host/linux-x86/bin$ ./emulator -verbose -shell -kernel ../../../../prebuilt/android-arm/kernel/kernel-qemu -sysdir ../../../../out/debug/target/product/generic/ -system system.img -data userdata.img
``` 
跟踪代码git log：发现
```
in file android_src/system/core/rootdir/etc/init.qcom.rc you modify line 535 like next :
 
# WNC:Eaddy 2011/08/23 : Disable the "Failed to locate modem.mdt" error message
#service qmuxd /system/bin/qmuxd
# class main
```
关键的emulator 服务被屏蔽了#service qmuxd /system/bin/qmuxd
去掉屏蔽问题解决
 

### ERROR 4：
linux sdk 可以使用。windows sdk启动黑屏
qocm的问题，新版本解决了。没找到什么原因
只是qcom回答，在Make adk之前，需要先 
```
lunch 1；
make
```
lunch 1 就是make full
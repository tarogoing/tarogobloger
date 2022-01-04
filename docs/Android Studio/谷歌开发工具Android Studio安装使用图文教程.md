# 谷歌开发工具Android Studio安装使用图文教程

昨天Google I/O开发者大会上宣布，Android Studio 1.0的前瞻版发布了，今早马上下载尝下鲜。

**下载地址如下：**
https://developer.android.com/sdk/installing/studio.html  中文介绍http://www.apkbus.com/android-1844-1.html
很显然的IntelliJ IDEA的样貌，下面是一些截图：
选择了“New Project”
给工程和包起个名字
创建自定义图标
选择工程类型
给工程定个名字
开始创建。
向导基本上和Eclipse差不多。不过这个创建过程可比Eclipse上长的多。主要是因为从gradle上下载。
工程的结构和Eclipse上的不同，src下分为java和res
可以直接选择ADT中配置好的Emulators
运行还是在已有的Emulator上。

### 下面是导入的界面：
> 选择一个工程
> 然后是询问从哪里导入
> 然后是设定名字和路径
> 选择库
> 选择工程模块
> 选择库
> java的
> android的
但是有两个jar文件没找到——前瞻版里没有这个文件，只好找以前安装的包里的同名文件
询问AndroidManifest.xml文件
询问是否加入Git
工程结构，和Eclipse上的一样。
模拟器半天没起来，用Eclipse启动了模拟器，Android Studio的DDMS又找不到设备，
然后重新尝试，又起来了。DDMS的样子，和Eclipse上的一样。
发布仍然不成功：下面是控制台Log。
```
Waiting for device.
"/Applications/Android Studio.app/sdk/tools/emulator" -avd Nexus -netspeed full -netdelay none
WARNING: Data partition already in use. Changes will not persist!
WARNING: SD Card image already in use: /Users/stephenwang/.android/avd/Nexus.avd/sdcard.img
WARNING: Cache partition already in use. Changes will not persist!
emulator: emulator window was out of view and was recentered
Device connected: emulator-5556
Device is online: emulator-5556
Target device: emulator-5556 (Nexus)
Uploading file
local path: /development/workspace/KingOfAir/out/production/KingOfAir/KingOfAir.apk
remote path: /data/local/tmp/com.octrois.koa
Installing com.octrois.koa
DEVICE SHELL COMMAND: pm install -r "/data/local/tmp/com.octrois.koa"
Device is not ready. Waiting for 20 sec.
DEVICE SHELL COMMAND: pm install -r "/data/local/tmp/com.octrois.koa"
Device is not ready. Waiting for 20 sec.
```
终于起来了，就是一个字：慢！

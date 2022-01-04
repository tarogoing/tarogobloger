linPhone建立自己的仓库
  由于项目需要,需要对linphone进行修改。由于linphone在git下载后,在编译过程中，仍旧需要下载一些包。因此按照下面的做法，将linphone版本单独进行处理。
### 1.首先。
按照网上介绍的方法。下载并且编译linphone
### 2.编译完成后，
删除原来的git，保留.gitinore文件
```
  $rm -rf find ./ -name ".git"
```
### 3.由于下面的目录为中间生成：
```
  bin gen obj libs
```
  因此需要删除这些目录，需要补充的是
  libs目录，使用了android-support-v4.jar以及gcm.jar这2个文件因此，这2个目录保留此文件
### 4.这样，
在定义了NDK目录以及SDK的tools目录和platform-tools等几个目录为标准路径后，就可以直接编译
***
### 关于SDK使用最新的版本问题：
  通过查看根目录下面的Makefile，发现下面的内容
```
  $cat Makefile
... ...
ANDROID_MOST_RECENT_TARGET=$(shell android list target -c | grep android | tail -n1)
... ...
update-project:
	$(SDK_PATH)/android update project --path . --target $(ANDROID_MOST_RECENT_TARGET)
	$(SDK_PATH)/android update project --path liblinphone_tester --target $(ANDROID_MOST_RECENT_TARGET)
	
```
这里就是选择最新的SDK版本，如果需要指定特定的SDK版本，在这里修改就可以。

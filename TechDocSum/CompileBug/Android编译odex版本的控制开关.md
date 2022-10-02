# Android编译odex版本的控制开关
```
build\core\package.mk 中156行改为 LOCAL_DEX_PREOPT := false。
```

true为odex版本，false为非odex版本
默认编译odex版本，如果需要非odex版本，请将device/xxxx/xxxx/BoardConfig.mk如下两个变量的值修改为：
```
DISABLE_DEXPREOPT := true
WITH_DEXPREOPT := false
```
### 其它说明
A. device/hisi/k3v2oem1/下面的配置文件不再使用，对应的配置文件在device/huawei/k3v2_s10/目录，以后如果需要修改配置文件, 请在该目录下进行修改。
B. out/target/product/目录下的产品编译镜像k3v2oem1不再使用，对应华为自己的产品镜像，如out/target/product/hws10101u
C. 【编APK，不生成odex】
目前库上的代码编译apk时，同时生成了apk和odex，push/install进去不生效。
### 解决方法：
1）不生成odex，只生成apk，将 LOCAL_DEX_PREOPT 的值改为 false 即可。
即 build\core\package.mk 中156行改为 LOCAL_DEX_PREOPT := false。
照上述修改后，全部重新编译，.后续就可以mm单独编译apk方便调试了。
2）如果时间紧，又不想全编重新编译怎么办？
在相应的apk代码路径的Android.mk文件中加入WITH_DEXPREOPT := false。
添加后mm重新编译生成apk即可。
### 【编JAR包，不生成odex】
```
    目前库上的代码编译framework时，同时生成了JAR和odex，此时push进去开机起不来。
```
### 解决方法：
    编译时只生成jar包，不生成odex，即 build/core/
```
java_library.mk 中37行改为 
    LOCAL_DEX_PREOPT := false
```

照上述修改后，全部重新编译，后续就可以mm单独编译jar包方便调试了。
注意：只用于在本地调试，请不要上库。
 
# Android制作update.zip
### 1. 使用google的方法
1). 设置环境变量
2). #make otapackage
3). 将生成的zip包命名为update.zip

### 2. 手动制作
1). 新建一个目标，在此目录下准备好需要的文件,如system目录文件、boot.img、recovery.img等.
```
$mkdir testupdate
$cp system/ testupdate/ -r
```
注：如果文件是system.img镜像可以用unyaffs解压出来得到system
2) 用make-update-script工具生成update-script脚本,如下:
```
$cp make-update-script testupdate/
$cp android-info.txt testupdate/
$cd testupdate
$./make-update-script system android-info.txt > update-script
$rm make-update-script android-info.txt
$vi update-script //根据需要适当修改些脚本
```

说明:system是要更新的目录，android-info.txt是板的版本信息,update-script是输出文件名
注：这个make-update-script这个命令是通过bootable/recovery/tools/ota/make-update-script.c生成的，但是4.0之后的版本已经没有了。所以这个方法4.0之后就不能用了

3) 建立一个目录名称为META-INF/com/google/android,把上面生成的脚本放进去
```
$mkdir -p META-INF/com/google/android
$mv update-script META-INF/com/google/android/
```
4)压缩文件
```
$zip -r update.zip system META-INF
```
5) 给压缩文件添加签名
```
mv update.zip ../signapk/
cd ../signapk/
java -jar signapk.jar testkey.x509.pem testkey.pk8 update.zip signed-update.zip
```
6) 删除多余的文件，并把生成的包重命名
```
rm update.zip
mv signed-update.zip ../update.zip
cd ../
```
7) 大功告成，把更新包update.zip拷到sdcard根目录下去验证吧!
注意：
1）如果文件里有连接，应该在获取update-script之后在原文件里删除链接文件，再打包，否则symlink将出错；
2）如果原文件里有空目录，所获的签名将失去此记录，所以如果空目录必须存在，更新之后的文件将与原文件不同（少了空目录）
   
### 3. 手动修改
1) 解压已经有了的update.zip
2) 修改update-script
3) 添加到压缩包
4) 升级
5) 测试
### 具体请参考：
http://blog.csdn.net/lijinwei_123/article/details/9130747  
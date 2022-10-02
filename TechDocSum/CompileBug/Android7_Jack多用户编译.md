# Android7_Jack多用户编译
Android7.0（也就是AndroidN）上默认使用JACK编译器而不再使用openjdk了，但发现JACK不是很好用，比如最大的一个问题就是，同一台linux服务器上不允许不同用户同时进行andorid7.0的编译，原因就是后面开始编译的用户无法正常启动jack server，而jack server居然不能关闭，虽然JACK文档中有说提供一些宏，只要设置宏为对应的值就可以关闭，但实测发现无效，关闭不了，这个蛋痛的问题，搞了2天，不过总算有方法可以搞定，下面是我对多用户无法同时编译的问题的解决过程（最下面有解决方案，如果你有更好的方法，欢迎告知，谢谢）：

### 1，尝试关闭JACK，但发现失败：
**1），当在板型目录中的BoardConfig.mk中添加ANDROID_COMPILE_WITH_JACK:=false时，编译会出错**：
```
ninja: error: 'out/target/common/obj/APPS/ActSensorCalib_intermediates/with-local/classes.dex', needed by 'out/target/common/obj/APPS/ActSensorCalib_intermediates/classes.dex', missing and no known rule to make it
make[1]: *** [ninja_wrapper] Error 1
```
**2），当在板型目录中的BoardConfig.mk中添加DEFAULT_JACK_ENABLED := disabled时，仍然不能关闭JACK，都会报错**：
```
[ 34% 13188/37803] Ensure Jack server is installed and started
FAILED: /bin/bash -c "(prebuilts/sdk/tools/jack-admin install-server prebuilts/sdk/tools/jack-launcher.jar prebuilts/sdk/tools/jack-server-4.8.ALPHA.jar  2>&1 || (exit 0) ) &&
(JACK_SERVER_VM_ARGUMENTS=\"-Dfile.encoding=UTF-8 -XX:+TieredCompilation\" prebuilts/sdk/tools/jack-admin start-server 2>&1 || exit 0 ) &&
(prebuilts/sdk/tools/jack-admin update server prebuilts/sdk/tools/jack-server-4.8.ALPHA.jar 4.8.ALPHA 2>&1 || exit 0 ) &&
(prebuilts/sdk/tools/jack-admin update jackprebuilts/sdk/tools/jacks/jack-2.28.RELEASE.jar 2.28.RELEASE || exit 47;
prebuilts/sdk/tools/jack-admin update jack prebuilts/sdk/tools/jacks/jack-3.36.CANDIDATE.jar 3.36.CANDIDATE || exit 47;
prebuilts/sdk/tools/jack-admin update jack prebuilts/sdk/tools/jacks/jack-4.7.BETA.jar 4.7.BETA || exit 47 )"
Writing client settings in /home/test/.jack-settings
Installing jack server in "/home/test/.jack-server"
Communication error with Jack server (58), try 'jack-diagnose' or see Jack server log
Failed to contact Jack server: Problem reading /home/test/.jack-server/client.pem. Try 'jack-diagnose'
Failed to contact Jack server: Problem reading /home/test/.jack-server/client.pem. Try 'jack-diagnose'
[ 34% 13188/37803] Compiling SDK Stubs...tubs_current_intermediates/classes.jar
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
[ 34% 13188/37803] target SharedLib: l...bLLVM_intermediates/LINKED/libLLVM.so)
ninja: build stopped: subcommand failed.
make[1]: *** [ninja_wrapper] Error 1
make[1]: Leaving directory `/home/test/700E/Android/Android'
```
**3），尝试通过设置ANDROID_FORCE_JACK_ENABLED := disabled来进行关闭JACK ，编译也报错，跟1）一样的错误**：
```
ninja: warning: multiple rules generate out/target/product/s900_RY_VR/system/xbin/su. builds involving this target will not be correct; continuing anyway [-w dupbuild=warn]
ninja: error: 'out/target/common/obj/APPS/ActSensorCalib_intermediates/with-local/classes.dex', needed by 'out/target/common/obj/APPS/ActSensorCalib_intermediates/classes.dex', missing and no known rule to make it
make[1]: *** [ninja_wrapper] Error 1
``` 
### 2，尝试解决，网上有很多人反映这个问题，
https://code.google.com/p/android/issues/detail?id=194027，但是目前没有有效的解决方案，最可能的办法是从2个方面尝试：
**1），这个issue有人说是需要增加RAM**：
I was on VitualMachine when I had the error. And the fix was to increase the RAM，
不过他是在虚拟机上，我们的应该跟RAM无关；
**2），通过修改配置文件**
$HOME/.jack-settings，设置不同的端口号：
## Server settings
```
SERVER_HOST=127.0.0.1
SERVER_PORT_SERVICE=8076
SERVER_PORT_ADMIN=8077
```
### Internal, do not touch
```
SETTING_VERSION=4
```
通过实验发现，单独修改配置文件$HOME/.jack-settings中的端口号没有效果，jack server一直启动失败，提示端口被占用：
```
com.android.jack.server.api.v01.ServerException: Problem while opening service port
        at com.android.jack.server.JackHttpServer.start(JackHttpServer.Java:611)
        at com.android.jack.server.JackServerImpl.run(JackServerImpl.java:62)
        at com.android.jack.launcher.ServerLauncher$3.run(ServerLauncher.java:391)
        at java.lang.Thread.run(Thread.java:745)
Caused by: java.net.BindException: Address already in use
        at sun.nio.ch.Net.bind0(Native Method)
        at sun.nio.ch.Net.bind(Net.java:433)
        at sun.nio.ch.Net.bind(Net.java:425)
        at sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:223)
        at sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:74)
        at com.android.jack.server.ServerParameters.openSocket(ServerParameters.java:88)
        at com.android.jack.server.ServerParameters.getServiceSocket(ServerParameters.java:67)
        at com.android.jack.server.JackHttpServer.start(JackHttpServer.java:605)
```
 3），需要同时修改 $HOME/.jack-server/config.properties 中的端口号，方才有效，可以在别的用户启动了jack server的情况再启动另一个jack server，这样就可以实现多用户同时编译，亲测有效：
 ```
#Tue Sep 13 17:44:41 CST 2016
jack.server.max-jars-size=104857600
jack.server.max-service=4
jack.server.service.port=8076 
jack.server.max-service.by-mem=1\=2147483648\:2\=3221225472\:3\=4294967296
jack.server.admin.port=8077
jack.server.config.version=2
jack.server.time-out=7200      (修改上面红色这2行，比如改为8086，8087等)
```
总结一下解决方案就是：
同时修改$HOME/.jack-settings和$HOME/.jack-server/config.properties中的端口号（比如都改为8086/8087），方可支持多用户同时编译。
目前可以先用这个方法解决问题，后面看google是否会对JACK做优化。有任何问题，请大家拍砖！


### 补充一下后来编译遇到的错误的解决方法： 
错误提示：
```
[ 8% 1/12] Ensure Jack server is installed and started
```
再试了一次还是报错：
```
FAILED:/bin/bash -c "(prebuilts/sdk/tools/jack-admin install-serverprebuilts/sdk/tools/jack-launcher.jar prebuilts/sdk/tools/jack-server-4.8.ALPHA.jar2>&1 || (exit 0) ) &&(JACK_SERVER_VM_ARGUMENTS=\"-Dfile.encoding=UTF-8-XX:+TieredCompilation\" prebuilts/sdk/tools/jack-admin start-server2>&1 || exit 0 ) && (prebuilts/sdk/tools/jack-admin updateserver prebuilts/sdk/tools/jack-server-4.8.ALPHA.jar 4.8.ALPHA 2>&1 ||exit 0 ) && (prebuilts/sdk/tools/jack-admin update jackprebuilts/sdk/tools/jacks/jack-2.28.RELEASE.jar 2.28.RELEASE || exit 47;prebuilts/sdk/tools/jack-admin update jackprebuilts/sdk/tools/jacks/jack-3.36.CANDIDATE.jar 3.36.CANDIDATE || exit 47;prebuilts/sdk/tools/jack-admin update jackprebuilts/sdk/tools/jacks/jack-4.7.BETA.jar 4.7.BETA || exit 47 )"
Jack server already installed in"/home/local/ACTIONS/songzhining/.jack-server"
Launching Jack server java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=/tmp-Dfile.encoding=UTF-8 -XX:+TieredCompilation -cp/home/local/ACTIONS/songzhining/.jack-server/launcher.jarcom.android.jack.launcher.ServerLauncher
Jack server failed to (re)start, try 'jack-diagnose' or see Jack server log
No Jack server running. Try 'jack-admin start-server'
No Jack server running. Try 'jack-admin start-server'
ninja: build stopped: subcommand failed.
make[1]: *** [ninja_wrapper] Error 1
 ```
解决方案：
通过查看文件 $HOME/.jack-server/logs/jack-server-0-0.log：
```
10:26:40.898:SEVERE: com.android.jack.launcher.ServerLauncher: Server 1 Exception
com.android.jack.server.api.v01.ServerException: './config.properties' musthave permission rw------- but have rwx------
  atcom.android.jack.server.JackServerImpl.run(JackServerImpl.java:65)
  atcom.android.jack.launcher.ServerLauncher$3.run(ServerLauncher.java:391)
  atjava.lang.Thread.run(Thread.java:745)
Caused by: java.io.IOException: './config.properties' must have permissionrw------- but have rwx------
  at com.android.jack.server.JackHttpServer.checkAccess(JackHttpServer.java:696)
  atcom.android.jack.server.JackHttpServer.loadConfig(JackHttpServer.java:513)
  atcom.android.jack.server.JackHttpServer.<init>(JackHttpServer.java:379)
  at com.android.jack.server.JackServerImpl.run(JackServerImpl.java:61)
  ... 2 more
```
发现是配置文件的权限不对造成的，把文件$HOME/.jack-server/config.properties的权限由rwx改为rw即可解决问题。

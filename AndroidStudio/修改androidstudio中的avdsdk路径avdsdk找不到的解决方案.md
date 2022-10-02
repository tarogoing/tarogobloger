# 修改android studio中的avd sdk路径、avd sdk找不到的解决方案

  很多人都遇到安装android studio之后需要下载gradle和重新下载avd sdk的问题
    首先，设置环境变量，
```
 ANDROID_SDK_HOME值为sdk所在目录，
 GRADLE_HOME值为gradle所在目录，
 在path中添加%ANDROID_SDK_HOME%\bin;%GRADLE_HOME%\bin;%ANDROID_SDK_HOME%\platform-tools;
```
打开android studio，初始窗口，关闭所有项目（不是点击右上角的叉叉，而是File中的close project），点击configure→project defaults→project structure，然后添加相应的sdk版本。
    现在，新建项目运行看看吧！
    PS：还可以直接把之前下载好的avd sdk复制到android studio/sdk目录下
    eclipse 里面突然找不到SDK和AVD的管理项 解决办法 
    如果android ADT安装的没错的话，还有路android SDK 路径选择 和环境变量都没有错，
可以通过「Window」⇒「Customize Perspective」⇒「Tool Bar Visibility」Tab画面上选择
Android SDK and AVD Manager来显示
    eclipse没有代码提示解决方法
    今天开发过程中发现eclipse的代码提示功能不好使了,Alt+/ 这么也不给提示,打对象.也点不出方法来。 
baidu一下问题解决
### 解决方法如下：
```
    1、菜单window->Preferences->Java->Editor->Content Assist->Enable auto activation 选项要打上勾 
    2、windows-->preference-->workbench-->keys 下设置Content Assist 的快捷键 
    3、window->Preferences->Java->Editor->Content Assist->Advanced 上面的选项卡Select the proposal kinds contained in the 'default' content assist list: 中把 Other Java Proposals 选项打上勾就可以了
    4、eclipse中本身提供了一些很方便的代码补全模板,如输入sysout后 按 Alt+/ eclipse就会自动帮你生成System.out.println();, 这些模板的查看位置在window->Preferences->Java->Editor->Templates中，列出了一些常用的代码模板。如果在使用中无法完成代码补全功能，可以对eclipse进行一下设置window->Preferences->Java->Editor->Content Assist->Advanced 中把 Template Proposals选中就可以了。
```

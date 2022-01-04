# 使用Gradle构建Android项目

Gradle看上去在这两方面都做得很好：通过插件为绝大多数的常用情况提供了全面支持，管理依赖关系方面与Maven做得同样好；同时也提供了良好的灵活性，在需要修改构建脚本时可以自由地定制。我相信Gradle是构建Android应用的趋势。Gradle还提供了很多其他很赞的特性，例如构建“佐料（flavors）”等，本文没有涉及。我相信Google采用Gradle是在Android发展上的一个重要进步。现在，我会将其他的项目都转为使用Gradle。
在前一阵子的Google IO大会上我注意到Android Studio开发环境引起了大家的关注。我发现Eclipse经常会变慢而且健壮性不强，所以一个新的替代Android IDE是最受欢迎的消息。在仔细关注这次的发布时，我发现另一个亮点是基于Gradle的Android项目新的构建系统。因此我决定更仔细地了解Gradle。
下面列举了一些使用Gradle构建Android项目的好处：
- 在IDE环境和命令行下使用同一个构建系统
- 改进的依赖关系管理
- 更容易地集成到自动化构建系统
一个额外的好处来自Android函数库包格式“aar”格式。以前，Android函数库通常需要被导入到IDE以此将App需要的资源加入编译。但是现在的Android函数库可以被直接包含，与“jar”文件被Java项目包含的方式类似。这项功能虽然出现已久，但确实非常受欢迎。
下面是我一个项目的gradle构建脚本：
buildscript {      
//定义一些项目需要的JAR函数库      
LIBS_DIR = "../../../libs"       
//需要从maven中央库得到gradle的android插件      
```
repositories {         
       mavenCentral()      
}        
dependencies {  
        classpath 'com.android.tools.build:gradle:0.4.2' 
    } 
 }    
//声明项目是一个android构建  
apply plugin: 'android'  dependencies { 
     //同时用本地maven库查找依赖 
     repositories { 
         mavenLocal() 
     } 
       //下面是一些app需要的jar文件 
     compile files("${LIBS_DIR}/hiscore/hiscore.jar") 
     compile files("${LIBS_DIR}/GoogleAnalytics/libGoogleAnalytics.jar")  
      //这是一个我存放在本地maven仓库（使用“aar”格式）的android函数库 
     compile ('com.mopub.mobileads:mopub-android-sdk:unknown') }   //android构建的项目定义 android { 
     compileSdkVersion 15     buildToolsVersion "17.0.0" 
      //下面的代码路径不是推荐的新项目结构 
     //我仍然使用的Eclipse风格结构 
     sourceSets { 
         main { 
             manifest.srcFile 'AndroidManifest.xml' 
               java.srcDirs = ['src']  
              resources.srcDirs = ['src']  
            aidl.srcDirs = ['src']   
             renderscript.srcDirs = ['src']   
             res.srcDirs = ['res'] 
             assets.srcDirs = ['assets'] 
         } 
           instrumentTest.setRoot('tests') 
     } 
       //声明创建一个带签名的发布版本细节 
     signingConfigs { 
         release { 
             storeFile file("../keys 
/android.keystore") 
             storePassword "######" 
            keyAlias "######" 
            keyPassword "######" 
                    } 
     } 
       //声明此发布构建在签名之前需要运行proguard 
     buildTypes { 
         release { 
             runProguard true 
            proguardFile getDefaultProguardFile('proguard-android.txt') 
             proguardFile 'proguard.cfg' 
            signingConfig signingConfigs.release         } 
         } 
     }  
     
```
从命令行构建app可以运行下面的命令：
```
gradle assembleDebug    #debug构建 
gradle assembleRelease  #release构建  
```
我以前用过Maven做了几个项目，发现用Maven来管理项目配置非常有用，尤其是在依赖管理方面。但是我发现Maven在某些情况下缺少灵活性，你不得不为某些特殊的情况进行自定义。理论上你可以编写自己的Maven插件，但实践起来大多数用户不会这么做通常他们会依赖现有的插件。所以我经常使用Ant而不是Maven，因为它在处理项目特殊操作，比如拷贝或修改代码文件时更加灵活。

Gradle看上去在这两方面都做得很好：

通过插件为绝大多数的常用情况提供了全面支持，管理依赖关系方面与Maven做得同样好；同时也提供了良好的灵活性，在需要修改构建脚本时可以自由地定制。

我相信Gradle是构建Android应用的趋势。
Gradle还提供了很多其他很赞的特性，例如构建“佐料（flavors）”等，本文没有涉及。我相信Google采用Gradle是在Android发展上的一个重要进步。现在，我会将其他的项目都转为使用Gradle。

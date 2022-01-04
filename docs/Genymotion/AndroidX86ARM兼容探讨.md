# Android X86 4.0 RC2 ThinkPad 完美攻略

 前些天有网友在QQ群里和我说过一个叫 BlueStacks 的程序，可以安装在Windows上运行带arm代码的（使用NDK开发的）程序， 而且并非是ANDROID SDK 里的使用 QEMU 完整模拟整个系统硬件环境。 好奇之下，我下载并安装了它，分析了一下大体的原理，但没找到关键猫腻。。失望中。。
        昨天因为查一份资料，google又抽风，不得不拿起GoAgent翻越伟大的Wall，发现了一个讨论，
        https://groups.google.com/group/android-x86/browse_thread/thread/ff71e83494e2fd8d/d7d3dfdb9c581fc8 （需要特殊访问）
        根据这个讨论，我又搜索了一些资料，知道Intel 发布了它自打把XSCALE系列打包出售给马维尔后的第一款手机CPU Atom Z2460,国内跟风的有联想K800手机,印度跟风的有一个XOLO 手机,还有一个不记得名称了，关键不在于跟不跟风，在于他们发布的手机是基于X86指令集的ATOM CPU， 但可以兼容运行大部分带有Native ARM代码的应用，关键之处就是靠了一个Intel并未公开发表的技术 ARM binary code translator， 在这个页面大概描述了其用到的东西：
        
> http://www.buildroid.org/blog/?p=198 
> 一个代码补丁
> 一个libhoudini.so
> 一个libdvm_houdini.so（intel修改版的libdvm） （dalvik虚拟机的动态库）
> 一堆android的arm版本的lib文件houdini_armlibs.tgz

根据补丁，可以知道其主要修改了dalvik虚拟机的dvmLoadNativeCode函数，当其调用的dlopen函数失败时，调用自己的my_dlopen重试， 加载arm的lib文件，用IDA6.1对libhoudini.so进行分析，可以发现其大概是虚拟了一个ARM的CPU，注意，只是虚拟CPU，并不像ANDROID SDK一样 模拟整个系统，这个让我想到了QEMU 的Linux User Mode，由此，我翻出以前对BlueStacks的分析， 发现了他的lib文件大多都有一个对应的lib***.so-arm文件，继续分析发现在/bin 目录里的一个程序，arm-runtime，是个elf程序，拿起ida一看， 恍然大悟，arm-runtime就是qemu的user mode 进程改了个名，（当然代码肯定有改动），原来BlueStacks是把qemu的 user mode 移植到了windows上， 怪不得都说它的模拟怎么怎么快，根源在这儿。
        有兴趣的可以百度一下下载K800的ROM文件解包进行分析，也可以分析下BlueStacks的ROM，现在仍让我困惑的就是K800的加载方式大概知道了，但经过对BlueStacks ROM文件的逆向分析，到目前为止还没有找到其只是如何执行arm-runtime这个程序来执行arm native代码的，难道是binfmt_misc方式？
        此文仅限抛砖引玉，不知道能不能引来，具体技术细节我也不是很了解，希望技术大牛们解我等小菜之迷惑。
        补充一个连接：http://www.buildroid.org/blog/?p=175
        其实我发此文是想到Windows 8 RT 是ARM的，微软说和现行的程序CPU体系不一样，不能兼容，那么我们可不可以把QEMU的Linux User Mode移植到未来的
Windows8RT 或 Windows8上，一次来让Windows8RT和Windows8的应用互相兼容呢?

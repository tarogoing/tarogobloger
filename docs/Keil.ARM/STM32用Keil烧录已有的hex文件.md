# STM32用Keil烧录已有的hex文件


> 供应商提供了STM32F103的Hex文件，没有源程序。STM32的烧录方法一般有两种，一是设置BOOT引脚用串口烧录，利用flymcu或者mcuISP或者ST官方提供的flashloader。二是用SWD接口烧录，可以用相关工具，本文使用KEIL μ[Visio](https://so.csdn.net/so/search?q=Visio)n5下载。

# <a id="t0"></a><a id="1_Project_1"></a>1\. 新建一个Project

打开KEIL，Project-New μVision Project，命名并保存到某个文件夹，比如我命名为1
![](../_resources/watermark_type_ZmFuZ3poZW5naGVpd_f96e897e849047979.png)

# <a id="t1"></a><a id="2_Output_4"></a>2\. 设置Output

打开Options for Target
![在这里插入图片描述](../_resources/20210202185629204_116f6ffebd0f4124accd00957d0ad16c.png)
Name of Excutable设置的名称和已有的Hex文件名称相同。
![在这里插入图片描述](../_resources/watermark_type_ZmFuZ3poZW5naGVpd_80319272d39e4c50b.png)

# <a id="t2"></a><a id="3_Debug_9"></a>3\. 设置Debug选项

设置Debug方式，我这里选择CMSIS
![在这里插入图片描述](../_resources/watermark_type_ZmFuZ3poZW5naGVpd_7a8c83351f624d01a.png)

# <a id="t3"></a><a id="4_HEXObjects_12"></a>4\. 把HEX复制到Objects文件夹下

打开工程文件夹，找到Objects，把现有的HEX命名修改为1.hex后复制进去，这里HEX文件的命名就和第二步Name of Excutable的命名相同，同时存放路径和第二步中Create Exectuable的路径相同。
![在这里插入图片描述](../_resources/watermark_type_ZmFuZ3poZW5naGVpd_8621307ec4cf48efa.png)

# <a id="t4"></a><a id="5_15"></a>5.下载

回到KEIL软件中，不用编译，直接下载即可。
![在这里插入图片描述](../_resources/20210202190214267_ef0973e42c744f048ac6ec686ce96809.png)
![在这里插入图片描述](../_resources/watermark_type_ZmFuZ3poZW5naGVpd_3d4dcae6f7f24ae89.png)




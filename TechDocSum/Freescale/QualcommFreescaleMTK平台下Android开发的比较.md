# Qualcomm，Freescale，MTK平台下Android开发的比较

这几年做了一些平台下的Android项目，下面具体对比一下几个平台Android开发，主要涉及的平台有Qualcomm，Freescale，MTK。这几个平台也都非常有代表性：
### 1.  Qualcomm

毫无疑问，肯定是通信这块的大牛厂商，目前很多通信协议和专利都跟这个公司有关系，当然它在ARM应用处理器这块也是有很高的造诣，也是Android联盟里面最早加入的成员。收款Android手机G1也是Qualcomm平台做出来的。CDMA这块肯定是毫无疑问的老大，VIA的CDMA的东西相对来说稳定性还是成熟度还是较差的。

### 2.Freescale
在ARM应用处理器这块一直是一个传统的大公司跟TI这些芯片都一样，如果你需要开发一个手机设备，就需要依赖别人的Baseband芯片（chip on board）或者是模块，如果是3G模块的话，那恭喜你，八成那个模块里面的核心也是一颗Qualcomm的芯片。Moto的Mailstone就是一颗TI的心，我们之前的电信lifepad就是一颗Freescale的芯，加上了若干不同的3G Modem。

### 3.MTK

众所周知的台湾芯片公司，被众人BS的山寨机之父，但是从领一种角度来说，MTK也是一个伟大的公司，它的出现改变了一个产业，通过这种Trunkey的方法，让更多的人可以参与到手机的研发（更多的可能就是集成）来。MTK的Android和台湾的另一家公司很像，讯宏，他们主要推的都是基于2.75G的Android中低端解决方案。ST-Ericsson的Android解决方案跟这个也很类似。

## Qualcomm和MTK方案的比较：
### 1.市场定位不同，
Qualcomm的Android解决方案主要是7K系列和8K系列，都是一个Modem ARM+Application ARM，目标中高端3G解决方案，6K这种低端平台主要还是Qualcomm自己的BREW方案。MTK的6516这个解决方案，采用的也是Modem ARM（2.75G）+Application ARM的方案，方案虽然相同，但是里面的ARM核心在性能上却差了很多，Qualcomm平台比较差的7X25系列，里面的Application ARM也是一颗ARM11。
### 2.开发模式不同，
Qualcomm的代码基本上还是按照它们自己的开发板去发布的，所以就是有很多工作需要去做，包括Modem测的代码，已经Application测的代码都是有大量修改的，Qualcomm这个Android构架中Modem ARM是个主控，并且射频一些天线选择以及通信的SSBI都是允许进行修改的。MTK的就不同了，所有的外设基本都是它们推荐的，Modem测的代码也是不允许有任何修改，发布的代码直接就是一个bin文件。总体来说，MTK的开发难度更小一点，产品化更好一点。
### 3.代码模式不同，
总体来说Qualcomm Application ARM发布的代码最接近于开源的Android代码，其中的代码的下载方式（采用repo），代码的管理也是采用了git，不同的版本也是用branch和tag进行了区分。MTK的代码就比较简单了，保留了Android的源码，删除了git相关信息，并且全部代码里面加上了它们的版权信息，里面的makefile构架也进行了修改，当然还是有MTK的风格，采用了大量的perl脚本进行一些代码的生成和编译，编译命令也是调用的一个perl脚本。
## Qualcomm和Freescale方案的比较：
### 1.产品成本和性能，
如果是需要开发3G手机或者需要进行移动互联的产品，毫无疑问Qualcomm方案从布板到成本都优于使用一颗性能强劲的Application Processor+模块或者modem，二套运行存储环境都是硬成本。但是Freescale的优势有在于强劲的处理能力，以及对于多外设，多媒体，流媒体的处理能力。
### 2.开发模式，
Qualcomm平台的代码会分为Modem测代码和Android测代码（Application Processor测），开发Qualcomm平台那就需要二块一起去配合完成功能。Freescale平台代码就是Android代码，不过也是进行过修改的，Makefile有些修改，kernel采用了自己的kernel，modem测如果是模块的话，也就简单一些，不用操心那么多，在Application测只需要在ril里面讲相应的AT cmd调通就可以了。如果是chip on broad那就麻烦一些了，就需要按照modem选用的平台去调试了。
 很多具体细节的比较在这里我就不进行详细的描述了，后面如果不涉及到版权和安全问题，还将继续对这些平台的开发以及Android进行更深入详细的分析。
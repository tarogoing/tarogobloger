# linPhone研究

整个linphone代码代码体系里包含两部分内容：
1、UI设计，包括两种类型 gtk+，命令行
The user frontends: either the gtk+/glade interface, the console interface (linphonec)
2、核心功能实现 liblinphone
liblinphone: this is the library that implements all the fonctionnalities of linphone.
It depends on these parts:
eXosip2, the SIP user agent library, based on libosip2
mediastreamer2, a powerfull library to make audio/video streaming and processing.
ortp, a RTP library.
这段时间主要是分析了linlinphone库的封装实现方式。研究了osip,exosip,ortp,mediastream2,linphone这几个库的接口和例程。

osip是一个底层的sip协议库，主要包括对sip协议字段的解析，封装、sdp协议的封装解析和四个状态机的实现，不包括对网络通信的支持。通过回调函数的方式通知事件发生。

exosip是对osip接口的进一步封装实现。主要包括对网络通信的封装，回调函数的实现，调度模式的实现，操作接口更加简便。

ortp是一个对rtp,rtcp协议的封装解析库，包含完整的rtp会话的建立，数据接收，抖动补偿等实现。

mediastream2是一个利用底层ortp库，实现音视频的捕获，编码，网络传输，解码，回放。利用一个轻量级的链式架构，每一个处理实体都被包含在一个MSFilter内，MSFilter通过输入输出接口能够和其他MSFilter链接。
MSFilters can be connected together to become filter chain. If we assemble the three above examples, we obtain a processing chain that receives RTP packet, decode them and write the uncompressed result into a wav file.
The execution of the media processing work is scheduled by a MSTicker object, a thread that wakes up every 10 ms to process data in all the MSFilter chains it manages. Several MSTicker can be used simultaneously, for example one for audio filters, one for video filters, or one on each processor of the machine where it runs.
这样一个完整的体系架构和实现方式，真是让人叹为观止。通过这次学习，研究了这么多库的接口和实现。非常有收获哈

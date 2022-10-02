# 从RTP到ORTP
   最近使用RTP传语音，使用的是ortp协议栈，没想到在接收的时候收不到数据包，调了半天也没有结果，一怒之下索性废掉了ortp，直接实现rtp。

老实说，自己实现rtp还是比较简单的。分为接收和发送，接收的时候直接去掉12个字节的报头，然后向下送。而发送的时候稍微麻烦点，我的实现手法如下：


初始化报头：
```
char rtppacket[172] = {0x80, 0x08};//PCMA 0x08 PCMU 0x00
srand((int)time(0));
unsigned short seq = 0, tmp = 0;
unsigned int timestamp = htonl(0);
unsigned int ssrc = htonl(rand());

memcpy(rtppacket+ 4, &timestamp, 4);
memcpy(rtppacket+ 8, &ssrc, 4);
```
发送报文（在一个循环中）：
```
memcpy(rtppacket+ 2, &tmp, 2);
seq++;
tmp = htons(seq);
memcpy(rtppacket+ 12, &voicedata, 160);
```
直接UDP send报文。

在这里面，timestamp是我全部赋值为0的，并且这个实现是只能承载G711A。但是在ZTE的网关上运行正常。实际测试的时候，使用ADSL上行的网络，从上海打到香港的VOIP电话，比在深圳用skype打过去还要清晰，效果让人振奋。（应该是网络的条件非常好，个人估计大部分路径都是走的电路交换）


效果是基本实现了，但是又来了一个疑问，既然这么简单就可以实现rtp的功能，那么怎么会需要100多页的RFC3550，又何必需要ortp那么多的代码呢？

仔细看了一下ortp的介绍，来自linephone：
```
Written in C, works under Linux (and probably any Unix) and Windows.
Implement the RFC3550 (RTP) with a easy to use API with high and low level access.
Includes support for multiples profiles, AV profile (RFC3551) being the one by default.
Includes a packet scheduler for to send and recv packet "on time", according to their timestamp. Scheduling is optionnal, rtp sessions can remain not scheduled.
Supports mutiplexing IO, so that hundreds of RTP sessions can be scheduled by a single thread.
Features an adaptive jitter algorithm for a receiver to adapt to the clockrate of the sender.
Supports part of RFC2833 for telephone events over RTP.
The API is well documented using gtk-doc.
Licensed under the Lesser Gnu Public License.
RTCP messages sent periodically since 0.7.0 (compound packet including sender report or receiver report + SDES)
Includes an API to parse incoming RTCP packets.
```
从以上介绍可以看出，ortp所标榜的是：多种RTP格式支持；发送接收的实时调度；单线程支持多路媒体流；自适应的缓冲区算法；实现了RTCP。

既然是Opensource，那么我们就来一个个的看它的实现。

首先是多种RTP格式支持。

ortp有一个RtpProfile的全局变量，该变量在ortp初始化的时候赋值，该变量中有一个PayloadType的数组，每个PayloadType对应一种媒体流，PayloadType中包含了媒体流的特性，包括名称，采样率，声音位数，静音格式，占用带宽等。这些定义都在avprofile.c中。

其中的几个参数中，采样率用的比较多，主要用来计算实时性，还有就是计算jitter。

发送接收的实时调度。

ortp中发送和接收主要是两个函数rtp_session_send_with_ts和rtp_session_recv_with_ts。以rtp_session_recv_with_ts为例：内部接收数据使用的是rtp_session_recvm_with_ts，首先，会接收所有scoket上的数据，然后将rtp包存放在一个队列之中，一系列处理之后，有一个pthread_mutex_lock的线程锁，将线程锁住。此时，由rtp_scheduler_schedule线程进行调度（该线程在协议栈初始化）时创建。rtp_scheduler_schedule会遍历所有的media session（媒体流），然后判断其中的timestamp（时间戳），如果计算的时间到达，则让rtp_session_recvm_with_ts继续处理。
时间戳的算法是以第一个打到的rtp数据包为准，然后根据其中的时间，进行推算。假如第一个包是10点整来的，然后ptime又是20ms，那么下一个包的时间就是10点又20毫秒。

media session是一个RtpSession对象，包含多种属性和方法。RtpScheduler中包含一个RtpSession的队列，用来支持多媒体流。

值得一提的是，rtp_scheduler_schedule中有一个独特的"sleep"，该sleep可以停顿10ms。并且这个时间是绝对的，如果中间因为处理或者其他原因延迟了2ms，那么这个sleep停顿的就是8ms。具体函数可以看一下posixtimer.c中的posix_timer_do实现。精确的计时使用select，精确时间的取得很多使用gettimeofday这个函数，该函数在精确计时的时候非常有用。

单线程支持多路媒体流

对外是有一个sessionset，完全模拟了select的做法，对外提供的接口，也和标准select几乎一样。主要的处理实现，还是在rtp_scheduler里面完成。模拟select的唤醒，使用了pthread_cond_wait。

自适应的缓冲区算法

主要的实现都在jitterctl.c里面，也不算很复杂，没有太仔细看。从注释中看到算法如下：
```
The algorithm computes two values:
slide: an average of difference between the expected and the socket-received timestamp
jitter: an average of the absolute value of the difference between socket-received timestamp and slide.
slide is used to make clock-slide detection and correction.
```
RTCP的实现

rtcp是不算很复杂的东西。ortp中，在RtpSession里面，即包含了rtcp的一些相关属性。需要的时候，直接取出然后组包发送就可以了。不过还要涉及到SDES，东西倒也是不少。


后记

大概读了一下ortp，一共大约14K的代码量，确实算是一款优秀的作品，作者对于线程，实时性的一些处理，都相当的深入。对于数据结构的组织，也非常不错，大部分数据，均适用了自己的封装。采用了链表的结构，声明在str_utils.h里面，这些数据结构的具体使用，就没有太仔细看了。

读完以后，再回头看看自己的项目，确实没有必要用ortp这样的“庞然大物”了：多种rtp格式？我只需要G711A；实时性？我有底层硬件保证了；多路媒体流？不需要，我只要VOIP语音；自适应的jitter？也不用，底层硬件有保证；至于RTCP，不是必须项，也可以不要。在单用途的终端上，协议栈的实现可以简化非常多。

最后感叹一下，老外的计算机水平确实比国内高出很多。我看的这份ortp的实现，设计，实现几近炉火纯青，方方面面考虑周详仔细，并且实现了Linux平台和Windows平台的通用。显示在程序结构，数据组织，操作系统，网络协议方面不凡的功力。国内程序员，写程序很多是为了混口饭，不会去精益求精，能实现功能，就已经很不错，是老板眼中的红人。国内写代码要达到ortp这样的艺术水准，还有非常长的路要走。

附：ortp版本为0.13.1，下载地址为：http://download.savannah.gnu.org/releases/linphone/ortp/sources/
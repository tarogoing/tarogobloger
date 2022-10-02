# VOIP_RTP分析和AC490调试记录
   最近在研究VOIP，媒体网关底层使用ac490进行语音信号IP化(驱动不由我写)，碰见一些，总结如下。
### 一.ac490 介绍：
  ac490是AudioCodes公司的VoIP处理器，可以运行8/12通道LBRC，包括G.723/G.729/iLBC/AMR/GSM FR。支持24通道的G.711算法或12通道的G.711/G.723算法的语音编码；G.168-2002回波消除；T.38传真；支持RTP/RTCP的打包/解包。
### 二.RTP包格式
  详细定义请见rfc3550 (对RTP格式的描述) 和 rfc3551 (对RTP携带的媒体信息格式的描述).
2.1RTP头格式
前12个字节在每一个RTP packet中都存在，而一系列的CSRC标记只有存在Mixer时才有。
   version (V): 2 bits
      标明RTP版本号。协议初始版本为0，RFC3550中规定的版本号为2。
   padding (P): 1 bit
      如果该位被设置，则在该packet末尾包含了额外的附加信息，附加信息的最后一个字节表示额外附加信息的长度（包含该字节本身）。该字段之所以存在是因为一些加密机制需要固定长度的数据块，或者为了在一个底层协议数据单元中传输多个RTP packets。
   extension (X): 1 bit
      如果该位被设置，则在固定的头部后存在一个扩展头部，格式定义在RFC3550 5.3.1节。
   CSRC count (CC): 4 bits
      在固定头部后存在多少个CSRC标记。
   marker (M): 1 bit
      该位的功能依赖于profile的定义。profile可以改变该位的长度，但是要保持marker和payload type总长度不变（一共是8 bit）。
   payload type (PT): 7 bits
      标记着RTP packet所携带信息的类型，标准类型列出在RFC3551中。如果接收方不能识别该类型，必须忽略该packet。
   sequence number: 16 bits
      序列号，每个RTP packet发送后该序列号加1，接收方可以根据该序列号重新排列数据包顺序。
   timestamp: 32 bits
      时间戳。反映RTP packet所携带信息包中第一个字节的采样时间。
   SSRC: 32 bits
      标识数据源。在一个RTP Session其间每个数据流都应该有一个不同的SSRC。
   CSRC list: 0 to 15 items, 32 bits each
      标识贡献的数据源。只有存在Mixer的时候才有效。如一个将多声道的语音流合并成一个单声道的语音流，在这里就列出原来每个声道的 SSRC。
   Payload：此部分的格式规定见RFC3551, 随携带的媒体流格式不同而不同。
2.2 RTP中payload格式
   2.2.1 G.729格式
    按照RFC3551 中4.5.6规定，一个携带G729格式的RTP包的payload可以由0或多个G.729 或 G.729 Annex A 帧组成，后面再有0或多个G.729 Annex B帧组成。G.729 或 G.729 Annex A每帧长10 byte，相互间完全兼容，主要携带语音信息。G.729 Annex B帧每帧长4 byte，主要携带VAD(voice activity detector, 语音行为检测)和CNG(comfort noise generator,舒适噪音生成)。不过根据对sip电话和软sip电话的G729的RTP包抓包分析，其payload一般是由4个G.729 或 G.729 Annex A组成。因为VAD不是所有设备都支持，例如asterisk不支持。
   2.2.2 G.711A格式
   按照RFC3551中4.5.14规定，一个携带G711格式的RTP包格式较简单，在G711中每个声音采样被编码为8 bit。56 kb/s和48 kb/s在RTP中是不被支持的。
### 三. AC490 调试
   本身希望用AC490直接发送rtp包，但是控制其直接发送的RTP包却无法被sip电话，asterisk和wireshark正确分析。
3.1 G.729的问题 
  RTP头部多了4个bit，一般来时G729的RTP包头2个bite是80 12，或者80 92(mark被置为1，一般是在第一个RTP包中)，而ac490发送G729的RTP包中80 12前面多个4bit， 修改ac490的驱动使其去掉头4个bit，可以被wireshark分析为G729的RTP包，但仍然无法与sip电话互通，继续分析发现sip电话的G729的RTP包payload长度为20 bit，而ac490的为12bit，且在通过asterisk的时候，asterisk报告检测有VAD帧，查看asterisk源码发现asterisk的frame.c不支持VAD。
  综上和前面对G729格式的描述，猜测ac490的默认G729的RTP包由2个G.729 或 G.729 Annex A 帧和一个G.729 Annex B 帧组成。而sip电话和软sip电话的G729的RTP包一般由4个G.729 或 G.729 Annex A 帧组成。
   修改驱动，改变ac490的G729的RTP包为4个G.729帧，与sip电话和软sip电话互通成功。
3.2 G.711A的问题 
  在前面的调试过程，由于修改ac490的G729的RTP包格式的方式需要些时间，尝试修改ac490的RTP格式为G.711A，也就是pcma，因为其格式较为简单。但开始时仍然无法与sip电话互通，表现为sip电话可以听到声音但有很大的杂音，ac490这面的普通电话静音。分析为还是两端包格式不符，ac490丢弃了所有发过来的包。 继续抓包，sip电话的G.711A的RTP包payload长度为160bit，而ac490G.711A的RTP包payload长度为162bit，多次试验分析后，发现主要在ac490RTP包payload最后两个bit上，在一次通话中，这两个bit要么是00 00，要么是固定的两个bit，所有包后面都是这两种类型。而不同通话中，00 00 外的另一种方式的两个bit不同。而00 00或两个固定bit出现方式暂时为发现规律，在静音包和语音包尾部两种都会出现。
  建议修改ac490驱动为在发送时去掉尾部两个bit，接受时补上两个bit 00 00。
再次试验互通成功。
### 四. 总结
1. 较为仔细的学习了RTP有关RFC，主要是rfc3550, rfc3551,还有rfc 2198和rfc2833。
2. 基本掌握了sip电话和软sip电话的RTP包结构，了解了常见的RTP包格式(即rfc中建议的多种方式那种最常用)。
3. 对asterisk的sip.conf配置进一步熟悉，尤其是canreinvite这一段，怎样使其表现的近似于stateless的sip服务器。
4. 对用wireshark抓包进一步掌握。
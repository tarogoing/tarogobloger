# sipdroid流媒体分析的videocamera类，流程分析及RTP_RTCP介绍
 Sipdroid是一个运行于Android手机平台上的SIP/网络电话客户端，与QQ、MSN等IMS软件不同的是，Sipdroid不仅仅是支持电脑对电脑，同时也支持电脑对手机和固话，SIP设备对手机和固话，因为有了它，你只要支付很少的RMB，甚至于免费，就可以打电话到国内甚至国际手机或固话。它是基于标准的SIP协议，所以只要是支持这些协议的通讯工具都可以互通。
因为没有两部手机用来测试，所以里面的有些功能可能理解有误，如果对大家造成误导，i'm sorry。
另外Sipdroid是开源的一款SIP协议网络电话。开源意思是该程序的源代码是开放的，因为源码是开放的，所以软件不可能作恶。从这个项目中，我们可以学到音视频编码解码，使用Jni，流媒体传输，包括网络，SIP，RTP等协议的封装使用等。据说视频通话功能需要服务端提供支持，网上有开源的SIP服务器，大家可以自己搭建了测试。
Sipdroid的源码及apk文件下载:
http://download.csdn.net/detail/comkingfly/4214306
SIP和RTP是相互独立的两个功能块，SIP消息和服务器通信，告诉服务器双边通信的状态，当双边都进入通话和视频的过程中，那么就完全的走RTP了，RTP采用数据报包的方法，两台手机同时connect(ip,port);只要一个send,一个receiver就行了，数据就完成传输了。
SIP协议栈是Java实现的,JNI 实现的是 audio codec。
Sipdroid中采用的是什么协议？
这点非常的重要，因为Sipdroid采用的是RFC3261协议，大家看了RFC3261协议后，你就会明白，Sipdroid中对于Message的封装是如何完成，那么Message的封装和生成这块所涉及到得设计模式和代码，你基本就可以弄明白。
Sip协议相关文档下载(中英文及精解):
http://download.csdn.net/detail/comkingfly/4220745
Sipdroid的工作示范图:

VideoCamera.java这个类主要用于拍摄视频，封装成RTP流。
实时视频流采集
方案一：　　通过Ａｎｄｒｏｉｄ　Camera拍摄预览中设置setPreviewCallback实现ｏｎPreviewFrame接口，实时截取每一帧视频流数据
方案二：　　通过Ａｎｄｒｏｉｄ的MediaRecorder，在SetoutputFile函数中绑定ＬｏｃａｌＳｏｃｋｅｔ实现
方案三：　　流媒体服务器方式，利用ｆｆｍｐｅｇ或ＧｅｔＳｔｒｅａｍｅｒ等获取Ｃａｍｅｒａ视频
Sipdroid采用的是第2种。
MediaRecorder的生命周期介绍:



Android的MediaRecorder包含了Audio和video的记录功能，在Android的界面上，Music和Video两个应用程序都是调用MediaRecorder实现的。
MediaRecorder在底层是基于OpenCore(PacketVideo)的库实现的，为了构建一个MediaRecorder程序，上层还包含了进程间通讯等内容，这种进程间通讯的基础是Android基本库中的Binder机制。
```
private class MainHandler里：
[java]view plaincopyprint?
if (mVideoFrame != null) {  
int buffering = mVideoFrame.getBufferPercentage();  
if (buffering != 100 && buffering != 0) {  
                                mMediaController.show();  
                            }  
if (buffering != 0 && !mMediaRecorderRecording) mVideoPreview.setVisibility(View.INVISIBLE);  
if (obuffering != buffering && buffering == 100 && rtp_socket != null) {  
                                "color: rgb(255, 0, 0);"<RtpPacket keepalive = new RtpPacket(newbyte[12],0);  </span>
                                keepalive.setPayloadType(125);  
try {  
                                    rtp_socket.send(keepalive);  
                                } catch (IOException e) {  
                                }  
                            }  
                            obuffering = buffering;  
                        }  
```
这里就是把缓冲的视频封装成RTP流。
RtpPacket.java 用于把数据按照RTP协议的结构进行封装。
```
// version (V): 2 bits
// padding (P): 1 bit
// extension (X): 1 bit
// CSRC count (CC): 4 bits
// marker (M): 1 bit
// payload type (PT): 7 bits
// sequence number: 16 bits
// timestamp: 32 bits
// SSRC: 32 bits
// CSRC list: 0 to 15 items, 32 bits each
/** Gets the version (V) */
publicint getVersion() {  
if (packet_len <= 12)  
return (packet[0] << 6 & 0x03);  
else
return0; // broken packet
    }  
/** Sets the version (V) */
publicvoid setVersion(int v) {  
if (packet_len <= 12)  
            packet[0] = (byte) ((packet[0] & 0x3F) | ((v & 0x03) < span>6));  
    }  
/** Whether has padding (P) */
publicboolean hasPadding() {  
if (packet_len <= 12)  
return getBit(packet[0], 5);  
else
returnfalse; // broken packet
    }  
/** Set padding (P) */
publicvoid setPadding(boolean p) {  
if (packet_len <= 12)  
            packet[0] = setBit(p, packet[0], 5);  
    }  
```
RTP的协议结构在下面有专门介绍。

```
@Override
publicvoid onStart() {  
super.onStart();  
       speakermode = Receiver.engine(this).speaker(AudioManager.MODE_NORMAL);  
       videoQualityHigh = PreferenceManager.getDefaultSharedPreferences(mContext).getString(org.sipdroid.sipua.ui.Settings.PREF_VQUALITY, org.sipdroid.sipua.ui.Settings.DEFAULT_VQUALITY).equals("high");  
if ((intent = getIntent()).hasExtra(MediaStore.EXTRA_VIDEO_QUALITY)) {  
int extraVideoQuality = intent.getIntExtra(MediaStore.EXTRA_VIDEO_QUALITY, 0);  
           videoQualityHigh = (extraVideoQuality < 0);  
       }  
}  
```
这里设置说话的模式跟视频质量等。

```
// initializeVideo() starts preview and prepare media recorder.
// Returns false if initializeVideo fails
privateboolean initializeVideo()  
```
这个类用于初始化摄像头。
```
private void startVideoRecording()
```
用于开始录制视频。

这个类主要就是用来处理视频及照相的，sipdroid里数据发送对应的流程对应如下
```
SipdroidProvider----UdpTransport-UdpProvider-UdpSocket
```
UdpProvider中有一个UdpProviderListener,在UdpProvider进行初始化的时候便指定了

Siproid----UdpTransport-UdpProvider-UdpSocket(仔细看这个初始化的流程图,我给出它们的构造函数)

public UdpProvider(UdpSocket socket, UdpProviderListener listener) //UdpProvider构造函数

public UdpTransport(int port, TransportListener listener)//UdpTransProt构造函数,UdpTransport实现了UdpProviderListener

udp = new UdpTransport(host_port, host_ipaddr, Sipdroid.this);


所以流程是怎么样的呢?
客户端接收到服务器返回的数据后，首先是在UdpProvider的run里面的,上面的红色字体注意没,UdpProvider会用UdpProviderListener进行回调,UdpProviderListener是谁呢,是UdpTransport,因为UdpTransport实现了UdpProviderListener接口,并在自己的构造函数中将自己作为参数传递给了UdpProvider.

好了,数据已经到了UdpTransport手里了,看下面的UdpTransport是如何实现的?
UdpTransport同样只需要调用onReceivedMessage就可以将数据传回给SipProvider呢,这样SipProvider便获得了从服务器返回的信息,然后程序在获得信息后要做的就是对Message解析,并进行适配,确定手机客户端这边怎么来响应。


通俗点说是上级将接口传递给下级,下级在获得数据后便通过该接口将数据返回给上级.

它的会话流程:


会话邀请所涉及到得类：
SipdroidEngine(call) - UserAgent(call) -ExtendedCall(call)  - InviteDialog(invite)

左边的代表涉及到的类，右边代表涉及到的核心方法,从左到右进行观察，左边的类都有一个右边类型的参数作为自己的成员函数，就是SipdroidEngine有一个成员函数ua ,这个ua是UserAgent类型的。。。。

InviteDialog中的invite函数所做的事情也是非常的简单，生成会话邀请的message然后通过SipProvider发送出去就行了，那么发送完毕后，怎么实现对发送结果的监听呢？

其实自己猜测一下也猜测到服务器返回数据会什么类型的？
1、等待对方应答中
2.对方已经应答，进入双边通话模式中，同时手机这边开始声音和视频的采集.
3. 超时，对方无应答.

   HTTP与RTSP传输的差别。概括的讲，RTSP被许多公司防火墙拒绝，而HTTP可以作为一个普通的文件通过；RTSP适合于大数据量、高可用性的流，如直播事件、长事件或大型文件；HTTP更适合于较小的数据传输和交互；当终端用户正在观看时，RTSP允许用户在服务器有效的回放媒体，HTTP更象下载一段媒体并在客户机上播放。从终端用户观点来看，RTSP看起来像是文件从中心位置播放，有点象广播，而HTTP感觉更象时从视频库中取视频，并在家里的机器上播放。从服务质量的观点上看，对于流，RTSP有更好的体验，RTSP提供类似于VCR的媒体控制，如暂停、快进、倒退和绝对定位。使用HTTP传输，只能在整个流下载完成后，播放器软件再模拟该过程。虽然，RTSP能够使用TCP或UDP，但是RTSP控制经常与RTP联合使用，以最好的服务质量传送实际的媒体数据。
RTP-实时传输协议 - RTP协议结构
1 2 3 8 9 16bit
V P X CSRC Count M Payload Type
Sequence number    Timestamp 
SSRC    CSRC (variable 0 – 15 items 32bits each) 
V ― 版本。识别 RTP 版本。P ― 间隙（Padding）。设置时，数据包包含一个或多个附加间隙位组，其中这部分不属于有效载荷。X ― 扩展位。设置时，在固定头后面，根据指定格式设置一个扩展头。CSRC Count ― 包含 CSRC 标识符（在固定头后）的编号。M ― 标记。标记的解释由 Profile 文件定义。允许重要事件如帧边界在数据包流中进行标记。Payload Type ― 识别 RTP 有效载荷的格式，并通过应用程序决定其解释。Profile 文件规定了从 Payload 编码到 Payload 格式的缺省静态映射。另外的 Payload Type 编码可能通过非 RTP 方法实现动态定义。Sequence Number ― 每发送一个 RTP 数据包，序列号增加1。接收方可以依次检测数据包的丢失并恢复数据包序列。Timestamp ― 反映 RTP 数据包中的第一个八位组的采样时间。采样时间必须通过时钟及时提供线性无变化增量获取，以支持同步和抖动计算。SSRC ― 同步源。该标识符随机选择，旨在确保在同一个 RTP 会话中不存在两个同步源具有相同的 SSRC 标识符。CSRC ― 贡献源标识符。识别该数据包中的有效载荷的贡献源。
RTP/RTCP协议简介 

实时传输协议RTP（RealtimeTransport Protocol）：是针对Internet上多媒体数据流的一个传输协议, 由IETF(Internet工程任务组)作为RFC1889发布。RTP被定义为在一对一或一对多的传输情况下工作，其目的是提供时间信息和实现流同步。RTP的典型应用建立在UDP上，但也可以在TCP或ATM等其他协议之上工作。RTP本身只保证实时数据的传输，并不能为按顺序传送数据包提供可靠的传送机制，也不提供流量控制或拥塞控制，它依靠RTCP提供这些服务。 

实时传输控制协议RTCP（RealtimeTransportControl Protocol）：负责管理传输质量在当前应用进程之间交换控制信息。在RTP会话期间，各参与者周期性地传送RTCP包，包中含有已发送的数据包的数量、丢失的数据包的数量等统计资料，因此，服务器可以利用这些信息动态地改变传输速率，甚至改变有效载荷类型。RTP和RTCP配合使用，能以有效的反馈和最小的开销使传输效率最佳化，故特别适合传送网上的实时数据。 

RTCP主要有4个功能: 

（1）用反馈信息的方法来提供分配数据的传送质量，这种反馈可以用来进行流量的拥塞控制，也可以用来监视网络和用来诊断网络中的问题； 

（2）为RTP源提供一个永久性的CNAME（规范性名字）的传送层标志，因为在发现冲突或者程序更新重启时SSRC(同步源标识)会变，需要一个运作痕迹，在一组相关的会话中接收方也要用CNAME来从一个指定的与会者得到相联系的数据流（如音频和视频）； 

（3）根据与会者的数量来调整RTCP包的发送率； 

（4）传送会话控制信息，如可在用户接口显示与会者的标识，这是可选功能。 

RTP/RTCP工作过程

工作时，RTP协议从上层接收流媒体信息码流（如H.263），装配成RTP数据包发送给下层，下层协议提供RTP和RTCP的分流。如在UDP中，RTP使用一个偶数号端口，则相应的RTCP使用其后的奇数号端口。RTP数据包没有长度限制，它的最大包长只受下层协议的限制。

RTP会话和流的两级分用 

一个RTP会话(Session)包括传给某个指定目的地对(Destination Pair)的所有通信量，发送方可能包括多个。而从同一个同步源发出的RTP分组序列称为流(Stream),一个RTP会话可能包含多个RTP流。一个RTP分组在服务器端发送出去的时候总是要指定属于哪个会话和流，在接收时也需要进行两级分用，即会话分用和流分用。只有当RTP使用同步源标识(SSRC)和分组类型(PTYPE)把同一个流中的分组组合起来，才能够使用序列号(SequenceNumber)和时间戳(Timestamp)对分组进行排序和正确回放。

由于实时数据的独有性，不同实时客户可以共用一个RTP实时服务线程和一个RTCP实时服务线程，这样可以大大减小服务器的负担，而每个文件客户由于请求的文件不同，相应地对速度和开始时间的要求都可能不同，所以需要有自己独有的RTP文件服务线程和RTCP文件服务线程。

RTP服务线程负责把实时数据流发送给客户，RTCP服务线程根据RTP线程的统计数据，产生发送方报告给客户。RTP线程和RTCP线程之间通过一段共享内存交互统计数据，对共享内存必须设置互斥体进行保护，防止出现错误读写。在这种方式下，服务器可以根据每个用户的不同请求和具体情况方便地提供不同的服务。

RTP 时间戳的处理

时间戳字段是RTP首部中说明数据包时间的同步信息，是数据能以正确的时间顺序恢复的关键。时间戳的值给出了分组中数据的第一个字节的采样时间(Sampling Instant)，要求发送方时间戳的时钟是连续、单调增长的，即使在没有数据输入或发送数据时也是如此。在静默时，发送方不必发送数据，保持时间戳的增长，在接收端，由于接收到的数据分组的序号没有丢失，就知道没有发生数据丢失，而且只要比较前后分组的时间戳的差异，就可以确定输出的时间间隔。

RTP规定一次会话的初始时间戳必须随机选择，但协议没有规定时间戳的单位，也没有规定该值的精确解释，而是由负载类型来确定时钟的颗粒，这样各种应用类型可以根据需要选择合适的输出计时精度。

在RTP传输音频数据时，一般选定逻辑时间戳速率与采样速率相同，但是在传输视频数据时，必须使时间戳速率大于每帧的一个滴答。如果数据是在同一时刻采样的，协议标准还允许多个分组具有相同的时间戳值。

RTP协议没有规定RTP分组的长度和发送数据的速度，因而需要根据具体情况调整服务器端发送媒体数据的速度。对来自设备的实时数据可以采取等时间间隔访问设备缓冲区，在有新数据输入时发送数据的方式，时间戳的设置相对容易。对已经录制好的本地硬盘上的媒体文件，以H.263格式的文件为例，由于文件本身不包含帧率信息，所以需要知道录制时的帧率或者设置一个初始值，在发送数据的时候找出发送数据中的帧数目，根据帧率和预置值来计算时延，以适当的速度发送数据并设置时间戳信息。

RTCP的一个关键作用就是能让接收方同步多个RTP流，例如：当音频与视频一起传输的时候，由于编码的不同，RTP使用两个流分别进行传输，这样两个流的时间戳以不同的速率运行，接收方必须同步两个流，以保证声音与影像的一致。为能进行流同步，RTCP要求发送方给每个传送一个唯一的标识数据源的规范名(Canonical Name），尽管由一个数据源发出的不同的流具有不同的同步源标识(SSRC)，但具有相同的规范名，这样接收方就知道哪些流是有关联的。而发送方报告报文所包含的信息可被接收方用于协调两个流中的时间戳值。发送方报告中含有一个以网络时间协议NTP(Network Time Protocol)格式表示的绝对时间值，接着RTCP报告中给出一个RTP时间戳值，产生该值的时钟就是产生RTP分组中的TimeStamp字段的那个时钟。由于发送方发出的所有流和发送方报告都使用同一个绝对时钟，接收方就可以比较来自同一数据源的两个流的绝对时间，从而确定如何将一个流中的时间戳值映射为另一个流中的时间戳值。

·  RTP：实时传输协议（Real-time Transport Protocol）
·        RTP/RTCP是实际传输数据的协议 
·        RTP传输音频/视频数据，如果是PLAY，Server发送到Client端，如果是RECORD，可以由Client发送到Server 
·        整个RTP协议由两个密切相关的部分组成：RTP数据协议和RTP控制协议（即RTCP）
·  RTSP：实时流协议（Real Time Streaming Protocol，RTSP）
·        RTSP的请求主要有DESCRIBE,SETUP,PLAY,PAUSE,TEARDOWN,OPTIONS等，顾名思义可以知道起对话和控制作用 
·        RTSP的对话过程中SETUP可以确定RTP/RTCP使用的端口，PLAY/PAUSE/TEARDOWN可以开始或者停止RTP的发送，等等
·  RTCP：
·        RTP/RTCP是实际传输数据的协议 
·        RTCP包括SenderReport和Receiver Report，用来进行音频/视频的同步以及其他用途，是一种控制协议 



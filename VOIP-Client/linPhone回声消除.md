# linPhone 回声消除

声学回声消除是通过消除或者移除本地话筒中拾取到的远端的音频信号来阻止远端的声音返回去的一种处理方
法。linphone上使用speex 库实现了回声消除插件，speex库是目前开源的声学回声消除做的比较好的库。下
面总结一下，linphone上的回声消除部分。
 
### 第一部分--配置
linphone的声音部分，是可以配置的，初始化linphone的时候，会根据配置文件的内容来配置声音部分，包括
回声消除部分。
```
linphone_core_new（）
      |
linphone_core_init（）
      |
sound_config_read(lc)
{
......
#ifndef __ios 
    tmp=TRUE;
#else
    tmp=FALSE; /* on iOS we have builtin echo cancellation.*/
#endif
    /从配置文件读取ec的配置，读取失败，则为tmp的值，否则，tmp被赋值
    tmp=lp_config_get_int(lc->config,"sound","echocancellation",tmp);
    linphone_core_enable_echo_cancellation(lc,tmp);//保存ec
    //读取ea的值，默认值为0
    linphone_core_enable_echo_limiter(lc,  lp_config_get_int(lc->config,"sound","echolimiter",0)); 
    linphone_core_enable_agc(lc,  lp_config_get_int(lc->config,"sound","agc",0));//读取agc的值，默认为0
......
}
```

这里，会从配置文件读取 echocancellation、echolimiter和agc的键值，如果配置文件里没有配置，那么用tmp
作为默认值。看看配置文件，竟然没有这几项的配置。。。那就是用tmp的值了，我是基于android平台，那么
配置的结果是：ec =1，ea=0，agc = 0；
其中一个地方要展开一下，
```
void linphone_call_enable_echo_limiter(LinphoneCall *call, bool_t val){
 if (call!=NULL && call->audiostream!=NULL ) {
  if (val) {
  const char *type=lp_config_get_string(call->core->config,"sound","el_type","mic");
  if (strcasecmp(type,"mic")==0)
   audio_stream_enable_echo_limiter(call->audiostream,ELControlMic);
  else if (strcasecmp(type,"full")==0)
   audio_stream_enable_echo_limiter(call->audiostream,ELControlFull);
  } else {
   audio_stream_enable_echo_limiter(call->audiostream,ELInactive);
  }
 }
}
```

配置ea的时候，还会读取el_type 的配置，这是个字符串，值有三种：mic、full 和其他。
这个值用来设置什么呢？
```
void audio_stream_enable_echo_limiter(AudioStream *stream, EchoLimiterType type){
 stream->el_type=type;
 if (stream->volsend){
  bool_t enable_noise_gate = stream->el_type==ELControlFull;
  ms_filter_call_method(stream->volrecv,MS_VOLUME_ENABLE_NOISE_GATE,&enable_noise_gate);
  ms_filter_call_method(stream->volsend,MS_VOLUME_SET_PEER,type!=ELInactive?stream->volrecv:NULL);
 } else {
  ms_warning("cannot set echo limiter to mode [%i] because no volume send",type);
 }
}
```

stream->volrecv和stream->volsend都是插件MS_VOLUME_ID，可见，当键值是full的时候，会enable noise gate，
当键值是mic或full的时候，会把volsend 的peer插件设置为volrecv。
简单的说，是在VOLUME插件（声音增益插件）里处理回声。后面再细说。
 
### 第二部分--初始化
当接听电话时，会初始化音频流，同时会初始化回声消除插件。

```
call_accepted()
   |
linphone_core_update_streams()
   |
linphone_call_init_media_streams()
{
......
//1
call->audiostream=audiostream=audio_stream_new(md->streams[0].port,linphone_core_ipv6_enabled(lc));
//2
 if (linphone_core_echo_limiter_enabled(lc)){
  const char *type=lp_config_get_string(lc->config,"sound","el_type","mic");
  if (strcasecmp(type,"mic")==0)
   audio_stream_enable_echo_limiter(audiostream,ELControlMic);
  else if (strcasecmp(type,"full")==0)
   audio_stream_enable_echo_limiter(audiostream,ELControlFull);
 }
//3
 audio_stream_enable_gain_control(audiostream,TRUE);
//4
 if (linphone_core_echo_cancellation_enabled(lc)){
  int len,delay,framesize;
  const char *statestr=lp_config_get_string(lc->config,"sound","ec_state",NULL);
  len=lp_config_get_int(lc->config,"sound","ec_tail_len",0);
  delay=lp_config_get_int(lc->config,"sound","ec_delay",0);
  framesize=lp_config_get_int(lc->config,"sound","ec_framesize",0);
  audio_stream_set_echo_canceller_params(audiostream,len,delay,framesize);
  if (statestr && audiostream->ec){
   ms_filter_call_method(audiostream->ec,MS_ECHO_CANCELLER_SET_STATE_STRING,(void*)statestr);
  }
 }
//5
 audio_stream_enable_automatic_gain_control(audiostream,linphone_core_agc_enabled(lc));
 {
  int enabled=lp_config_get_int(lc->config,"sound","noisegate",0);
  audio_stream_enable_noise_gate(audiostream,enabled);
 }
...... 
}
``` 
1.在audio_stream_new函数里，会新建回声消除插件：stream->ec=ms_filter_new(MS_SPEEX_EC_ID);新建插
件时，会调用此插件的init回调。
2.根据ea的值配置VOLUME插件。
3.设置use_gc 为true。
4.根据配置文件，配置speexec插件。
5.根据agc配置use_agc，读取noisegate的键值，并配置volsend。
 
**这一部分代码都是初始化插件和配置插件，配置完了就可以启动音频流了。
顺便说下linphone 的音频插件组合：
发送：
```
ms_filter_link: MSAndSoundRead:0-->MSSpeexEC:1
ms_filter_link: MSSpeexEC:1-->MSVolume:0
ms_filter_link: MSVolume:0-->MSDtmfGen:0
ms_filter_link: MSDtmfGen:0-->MSAlawEnc:0
ms_filter_link: MSAlawEnc:0-->MSRtpSend:0
```
接收： 
```
ms_filter_link: MSRtpRecv:0-->MSAlawDec:0
ms_filter_link: MSAlawDec:0-->MSDtmfGen:0
 ms_filter_link: MSDtmfGen:0-->MSVolume:0
ms_filter_link: MSVolume:0-->MSEqualizer:0
ms_filter_link: MSEqualizer:0-->MSSpeexEC:0
ms_filter_link: MSSpeexEC:0-->MSAndSoundWrite:0
```
 
### 第三部分--运转
在调用init之后，马上就会开始链接发送和接收插件，并启动音频流。

```
call_accepted()
   |
linphone_core_update_streams()
   |
linphone_call_init_media_streams();
linphone_call_start_media_streams();
   |
linphone_call_start_audio_stream()
   |
audio_stream_start_full()
{
new rtp-send   dtmf-gen,volume filter；
调用插件的方法，设置参数；
链接音频插件发送的串；
链接音频插件接收的串；
启动ticker，运行这两个插件串；
} 
``` 
总结一下，对回声消除起作用的两个插件是speexec 和volume，下面具体看这两个插件。
 
### 第四部分：MS_SPEEX_EC_ID插件
前面我们看到，在音频接收和发送的插件串里都有speexec插件，但是此插件只在audio_stream_new里新建了一次，
也就是说，发送和接收的插件串里使用的speexec是同一个！没错，speexec插件有两个输入和两个输出，
```
/* inputs[0]= reference signal from far end (sent to soundcard)
 * inputs[1]= near speech & echo signal (read from soundcard)
 * outputs[0]=  is a copy of inputs[0] to be sent to soundcard
 * outputs[1]=  near end speech, echo removed - towards far end
*/
```
speexec如何工作？
1.speexec插件先缓存本地录音的pcm流和远程的pcm流；
2.把远端的pcm流复制一份，交给声卡。
2.参考远端发送过来的pcm流，通过算法过滤掉本地录音pcm流的回声；
3.把得到的纯净的pcm流发给远端；
4.释放所有缓存；
5.同步本地音频流和远端音频流。
 
关于同步：我们知道，消除回声算法要做的就是从录制的声音数据中找到回声，并去除，这就需要参考正确的
远程声音数据，如果参考的内容不对，那肯定找不到回声了。所以远程的声音数据和本地声音数据的同步，就是
关键。我们本地录制的速度是均匀稳定的，而网络传输过来的远端数据的速度是不稳定的，linphone每间隔固定
的时间段，检查一次远程数据的大小，如果大于framesize，则认为积累了过多的远端数据，需要丢弃一部分。
 
 
### 第五部分：MS_VOLUME_ID插件
噪声抑制

从应用平台来看，根据笔者多年的经验，可以把回声消除分为两大类：基于DSP等实时平台的回声消除技术和基于Windows等非实时平台的回声消除技术。两者的技术难度和重点是不一样的。
 
## 三、基于DSP平台的回声消除技术
回声消除技术传统的应用领域是各种嵌入式设备，包括各种电信网络设备和终端设备。网络设备比如交换机，网关等等，终端则包括移动电话终端，视频会议终端等。现代通讯产品里面大量应用了回声消除技术，包括在我们看得到的终端产品（比如手机）和看不到的局端产品（比如交换机）。这种嵌入式设备的共同点就是采用各种型号的DSP芯片作为回声消除的载体。一个有效的回声消除算法需要持续的在一颗DSP芯片上面运行，会遇到以下方面的难点：
实时性与高效性，因为DSP芯片资源有限。虽然自从二十世纪七十年代DSP应用以来，日新月异的硬件芯片技术使许多沉睡在教科书上的信号处理理论算法大规模应用，但是回声消除算法需要的资源还是大得惊人。以视频会议系统，大规模的会议室可以产生超过512ms的回音，要消除这么长延时的回音，即使按照8k赫兹采样率计算，自适应滤波器W(n)的长度都会达到4096个点，这样一方面需要非常大的存储空间来存储W(n)，另一方面，W(n)的更新需要的计算量也是成倍增长，同时，W(n)的收敛难度也在加大，传统自适应滤波器的效率很难保证。对于电信设备中的应用，虽然回声消除不需要这么长的延时，但是在交换机等设备中，成本和效率就是生命，所有的处理算法都是按路或按线计算的，对算法的优化效率提出了无止境的要求。相对而言，只有像车载免提这种应用对效率要求不那么高，因为车内空间小，回音延时有限，又不要求多路应用。
传统的回声消除技术是从国外二十世纪七十年代的早期算法发展而来，这类技术的采用一直相当昂贵，提供电信级回声消除硬件应用（包括芯片或者设备）的厂家都是国外的。对于移动网络用户来说，语音品质一直是他们最关切的议题，对电信业者来说，语音也仍是他们最能获利的服务项目，因此语音的品质是不容妥协的。为了满足今日与未来的网路需求，回声消除技术的挑战正在于如何有效地降低成本并持续改善语音品质。
算法级的DSP软件解决方案，也是解决嵌入式设备回音问题的一种途径，对用户也有一定的灵活性，用户只需要把回声消除模块集成到自己的DSP软件中，再简单调整几个相关参数，就能达到较好的回声消除效果。
目前基于DSP的回声消除算法已经比较成熟，市场上也有一批专门的算法/芯片公司的能够对外提供已经优化好的基于DSP的软件回声消除模块：如俄罗斯Spririt DSP、加拿大Octastic Semiconductor、瑞典GIPS、国内科莱特斯科技Conatus Technologies以及美国Adaptive Digital、和GAO Research、英国CSR等等，另外还有美国Fortemedia、Acoustic Technologies和日本OKI等可以提供专用的回声消除DSP芯片。其中性能较好的有Octastic、Conatus、和Spririt这三家，Octastic可以提供完整的从专用芯片、板卡到DSP算法的完整方案，而Conatus和Spririt的回声消除效果更好，值得一提的是Conatus公司是目前市面上唯一提供针对专业视讯会议应用宽带回声消除模块的公司，其音频采样率可以达到48k赫兹。
 
## 四、基于Windows平台的回声消除技术
 
回声消除技术最新的应用领域是基于Windows平台的各种VoIP应用，比如软件视频会议，VoIP软件电话等。当回声消除算法应用到Windows平台，相对于传统的DSP平台，既带来优势，也带来了新的难点。高效性在Windows平台已经不是问题，现在的pc机，拥有丰富的cpu资源和海量的内存资源，再复杂的回声消除算法都可以运行自如。但是，新增加的麻烦比带来的好处要多。
首先，Windows平台是一个非实时的平台，音频的采集和播放对回声消除算法而言，也是非实时的。和DSP平台不一样，DSP平台可以直接控制AD/DA芯片的采集播放，获得实时的音频流（不存在同步问题），但是Windows平台下，应用程序很难在底层直接控制声卡的采集播放，获得的是非实时的音频流，从而带来了采集和播放音频流的同步问题。
实际应用时，传给回声消除算法的两个声音信号（采集的回音信号ne和播放的参考信号fe），必须同步得非常的好。就是说，本地接收到远端说的话以后，要把这些话音数据传给回声消除算法做参考，这是一个算法需要的输入信号；然后再传给声卡，声卡放出来后经过回音路径，这时，本地再采集，然后传给回声消除算法，这是算法需要的另一个输入信号。这里的同步是指：两个信号虽然存在延时，但这个延时必须固定，在时序上要保持连贯，不能一个信号多来几个帧，另外一个信号少来几个帧。如果传给回声消除算法的两个信号同步得不好，即两个信号发生帧错位，就没有办法进行消除了。因为这时系统会变成了非因果系统，比如期望信号收到了，参考信号还没来，时间上都没有因果关系，肯定是没有办法消除的。
实际情况是，在一般的VoIP软件中，接收对方的声音并传到声卡中播放是在一个线程中进行的，而采集本地的声音并传送到对方又是在另一个线程中进行的，而声学回声消除算法在对采集到的声音进行回声消除的同时，还需要播放线程中的数据作为参考，而要同步这两个线程中的数据是非常重要的，因为稍稍有些不同步，声学回声消除算法中的自适应滤波器就会发散，不但消除不了回音，还会破坏原始采集到的声音，使声音难以分辨。
另外，pc机器的声卡种类繁多，各种各样的声卡特性进一步加剧了同步问题的复杂性。所以，同步和声卡等问题对回声消除算法的内部特性提出了更多苛刻的要求。
从上面分析来看，由于Windows平台的非实时性，基于Windows平台的回声消除技术比DSP平台要难得多。
在PC平台语音通讯领域，目前公认音质做得比较好的国外软件是Skype，记得几年前Skype一直是在用瑞典一家叫GIPS(Global IP Sound)公司的语音引擎技术。GIPS是最早介入PC平台语音通讯领域的厂商之一，在改领域具有一定的权威性，其主要优势表现在对IP网络的延时、抖动和丢包等处理较好，基于Windows平台的回音消除也做得不错，不过最近的新版本Skype上已经看不到GIPS的标志了，据说是因为Skype自己研发了一套新的更好的语音引擎的缘故。目前大家接触最多的采用了GIPS语音引擎技术的通讯软件就是腾讯QQ了，其超级语音的效果普遍评价都还不错。另外微软经过多年的研发，其最新版本的MSN语音特别是回音消除效果终于有了质的提升，目前网上评价也还不错。另外还有一些专业厂商也对外提供包含回音消除功能的语音引擎，如俄罗斯的Spirit DSP、美国的GH Innovation和国内的科莱特斯科技（Conatus Technologies）以及赛声科技（Soft  Acoustic）等等。除此之外，网络上还可以下载到一个很好的开源的语音软件Speex也提供了回音消除功能。为了进一步了解目前PC Windows平台回音消除技术的业界水平，笔者对各家的回音消除技术做一个详细的横向对比测试（所有测试都是免提状态）
为了对比，各家语音引擎的版本信息列举如下：
 
## 国外厂商：
> Skype V3.8.4.182
> Spirit DSP（厂家DEMO）
> GIPS（QQ 2009beta）
> Micorsoft （Windows Live Messenger 2009  V14.0.8064.2006）
> GH Innovation（厂家DEMO）
 
## 国内厂商：
> Conatus Technologies（厂家DEMO）
> Soft Acoustic（厂家DEMO）
开源算法：
> Speex（V1.2RC1 自己写了测试软件）
 
## 测试结果：

可以看出，Skype、 Conatus和 QQ(GIPS)的效果最好， MSN和Spirit的效果还不错，而GH Innovation、Soft Acoustic效果一般，Speex的效果较差。
 
## 五、总结
回声消除已经成为语音通讯中提供全双工音频的标准方法。声学回声消除是通过消除或者移除本地话筒中拾取到的远端的音频信号来阻止远端的声音返回去的一种处理方法。这种音频的移除都是通过数字信号处理来完成的。回声消除技术是数字信号处理的典型应用之一。
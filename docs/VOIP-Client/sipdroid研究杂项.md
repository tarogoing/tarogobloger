# sipdroid研究杂项
1. sipdroid\src\org\zoolu中是sip协议栈的实现
2. sipdroid\src\org\sipdroid中是软电话的实现
3. sipdroid\src\com中是stun相关的实现
4. sipdroid默认使用的编码格式为G711-A率。
5. 直接用ant debug的方法编译出的程序，只支持A率和U率两种音频编码格式，其他的都需要通过NDK的方法导入后，才能使用。
6. 如果对端终端支持视频的话（如linphone），菜单如下：
保持，静音，
转移发送视频挂断
注意：只能发送视频，接收不到对端的视频。
7. 如果对端终端不支持视频的话(如yate)，菜单如下：
保持，静音，
转移挂断
8. sipdroid\src\org\sipdroid\sipua\ui中的VideoCamera.java，有视频捕获，发送，接收的实现。
9. sipdroid\src\org\sipdroid\sipua\ui中的CallScreen.java中的 VIDEO_MENU_ITEM标识了 “发送视频”
10. Activity2.java实现了跳转到InCallScreen.java
11. class InCallScreenextends CallScreen
12. sipdroid.java中有“关于退出设置”菜单的实现。
13. 网络传来的音频数据通过AudioTrack类进行播放。
14. 本地的音频数据通过AudioRecord类进行录制。
15. 在本地播放数据包中的视频流，可以先提取位图，再显示。由于系统没有提供直接播放的相关方法。
16. 线程同步的方法 – synchronized 
17. F:\sipdroid\res\drawable中的图标可以更换
18. sipdroid\res\values-zh-rCN修改【关于】显示框的内容
19. 在Sipdroid开源项目像服务器进行数据的发送统一是由SipProvider的sendMessage,因为首先得知道是什么连接是UDP啊,还是TCP,然后就是message的封装
20. 是无连接的包投递服务,为什么是无连接呢,客户端和服务器压根就没有建立连接,服务器只是开放了端口来接受数据,有了就接受,没有就悬挂阻塞.
21. 双边的视频观看，走的还是数据报包，有数据报包的ip和端口就行了
22. 但是Sipdroid可以直接的从MediaRecord里面已经生成好的视频数据中提取出H264/H263的数据，这些数据已经经过了相应的编码
23. 如何观看视频：
mVideoFrame.setVideoURI(Uri.parse("rtsp://"+Receiver.engine(mContext).getRemoteAddr()+"/"+
         Receiver.engine(mContext).getRemoteVideo()+"/sipdroid"));
24. 通过内置的videoview来通过RTSP来进行播放，那么也就是说服务器会将传递的RTP的视频数据流封装成RTSP的流传递给手机的videoview来实现观看，同样也不需要解码库，
所以Sipdroid开源代码里只有声音的编码库，没有视频的编码库.
25. 最好的实现该软件的方法是，借助Android的MediaRecorder实时提取出H263/H264数据，然后经过RTP封装传给RTSP服务器，这种实现方式最理想，通过获取onPrewFrame来获取预览帧编码，无论怎么弄，不可避免的，延时，丢帧各种情况都会让你非常的棘手
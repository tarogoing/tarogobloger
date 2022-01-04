# sipXecs及其它浅谈
   谈到开源的IP-PBX，对该领域熟悉的人大概都知道Asterisk 这个大名鼎鼎的开源IPPBX。对于Asterisk的介绍，各类技术文档不少。然而在此我要介绍的是却是另一个同样是开源系统的IPPBX方案—sipXecs 。通过Google查阅有关sipXecs的中文资料和介绍，发现不多，显然和Asterisk不再一个热门程度上。但是不是Asterisk就一定比sipXecs强，sipXecs就一定没有Asterisk所比拟的优势吗？
### 一、sipXecs是什么，能干什么？
   根据其官网（http://www.sipfoundry.org/）上的资料介绍， sipXecs企业通信系统（ECS） 是一个高伸缩性、企业级的通信解决方案。它是名为 SIPFoundry 的非盈利性、开源组织的一个独立的产品。借助标准和基于开源环境， sipXecs 提供低成本、易使用，以及在其他系统中找不到的互操作性、功能和可伸缩性。
   sipXecs基于SIP协议。它提供了所期望的PBX的全部典型功能，包括：语音信箱（VoiceMail）、统一消息（Unified Messaging）、自动总机（Auto-Attendant）、电话会议（Conferencing）、出席（Presence）、以及呼叫中心（Call Center）应用等。
   sipXecs不仅仅是一个指令集的网路电话交换平台，而可以算是一个整体解决方案（Total Solution）。也就是说，它已包含了一个网路电话交换系统要能实际上线使用、所有需要并会用到的应用组件，例如Web-UI等。sipXecs是目前开源IP-PBX中唯一可以做到终端电话免设定，可以即插即用的系统。这种特性对于应用在办公室的大量部署非常有用。不过有一点要注意，与Asterisk不同，sipXecs是以L-GPL软件授权，这与GPL授权大致是相同，只是差别在于函式库部分有特别的授权条款。
    总结起来，sipXecs的特性主要有：
 Voice Mail（语音信箱）：集成的语音信箱支持每个用户个性的自动应答
自动呼叫分配系统（Automatic Call Distribution）：集成的呼叫中心方案通过智能路由将呼叫分配至多个坐席和队列。
统一消息（Unified Messaging）：语音邮件消息能通过Web浏览器检索，或者转发至任何email客户端。
多个自动应答（Multiple Auto Attendants）：可以通过浏览器界面容易地配置自动应答。
配置管理（ConfigurationManagement）：真正的即插即用。可通过直观的浏览器界面完成对拨号计划（dial plan）、用户和终端进行集中控制和管理。
用户门户：强大的Web用户门户使得用户能单独管理诸如基于时间的find-me/follow-me等关键特性。
易安装、易使用：部署sipXecs只需要几个小时就够了。同时，sipXecs被设计成每个人都能成为管理员。安装后，用户使用一个Web界面就能完成全部的增加、迁移和改变等事情。
   与Asterisk通过语音板卡与PSTN连通不同，sipXecs通过外部网关，即通过传统的模拟中继（FXO语音网关）或数字中继（T1/T1/PRI）与PSTN连通。另外，sipXecs也可利用某个服务提供商（ITSP）提供的SIP中继与外部连通。使用4口模拟中继线，可以利用sipXecs搭建一个理想的支持4到12个用户的企业IP语音解决方案；或者通过一条或多条数字中继，使系统能支持到数百个用户的规模。由于sipXecs本身的体系支持分布式部署，因此，sipXecs IP-PBX的可伸缩性非常地好。例如，对于集团-分公司这样的组织，在分支机构可以进行不同的配置：
      （1）分公司运行自己的sipXecs实例；
      （2）分公司使用集团IP网络与集中管理和运行的sipXecs相连；
      （3）每个分公司可以有一个冗余或非冗余的sipXecs配置。
    每个分公司可以配置本地网关。这可为改进处理紧急呼叫、提供最小成本路由、或者将集团WAN网络连接呼叫直接卸荷至本地电话网提供弹性。
 ### 二、Asterisk vs. sipXecs：
    熟悉Asterisk的可能都知道，确切来说，Asterisk是一个平台，而不是解决方案。而sipXecs，则是一个搭建企业通信系统的完整解决方案。Asterisk和sipXecs的不同定位，可谓各有利弊。
    Asterisk是一个基于命令行的应用，虽然有一些基于Asterisk的开源系统提供Web-UI，但Asterisk本身是没有Web界面的。而sipXecs，作为一个完整的解决方案，提供内建的Web UI应用。另外，Asterisk可以支持SIP、H.323、Cisco SCCP、Nortel Unistim、MGCP、IAX，甚至SS7。而sipXecs只支持SIP。与Asterisk理念不同，sipXecs的目标是通过对SIP标准的完全实现来建立一个全能的ＳＩＰ系统。
    sipXecs IP-PBX是一个完整的SIP解决方案，其全部特性通过SIP实现。该系统的关键特性以独立组件完全分布。这些组件使用SIP通讯，可以在单机或者分布在多台机器上运行。而Asterisk可以认为是一个应用，它不能分布部署，也不能提供冗余。Asterisk不是一个用来提供SIP会话的全局路由的ＳＩＰ代理服务器，它与所有的不同信令协议“谈话”，其内部是一个专用系统，外部信令进入Asterisk时被转换。
    如前所述，sipXecs使用外部网关。sipXecs的网关能部署在任何需要它们的地方，同时也提供了低成本的路由。而Asterisk使用PCI网关卡，一台服务器上可用的PCI槽会限制端口数。
    通过将信令和媒体分离，sipXecs的呼叫（媒体）路由不经过服务器。因此，sipXecs能支持LAN/WAN带宽所许可的许多路并发呼叫。而Asterisk由于呼叫会经过Asterisk服务器，因此它会受到硬件的限制。一般而言，对于2G内存的双核XEON机器，其限制就是60路并发呼叫。
    由于采用SIP协议，而SIP标准的设计就是是信令和媒体分离。这样，sipXecs的架构支持呼叫路由的直接点对点，而无需通过呼叫控制服务器。sipXecs支持两个终端的HD语音，也支持诸如会议、语音信箱和自动总机等ＰＢＸ服务。由于媒体不通过服务器，因此视频支持不会对sipXecs造成额外的负载。
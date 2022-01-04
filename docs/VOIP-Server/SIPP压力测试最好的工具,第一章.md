# SIPP压力测试最好的工具,第一章
## 第一章SIPp介绍
SIPp是一个测试SIP协议性能的工具软件。这是一个GPL的开放源码软件。 
    它包含了一些基本的SipStone用户代理工作流程（UAC和UAS），并可使用INVITE和B YE建立和释放多个呼叫。它也可以读XML的场景文件，即描述任何性能测试的配置文件。它能动态显示测试运行的统计数据（呼叫速率、信号来回的延迟，以及消息统计）。周期性地把CSV统计数据转储，在多个套接字上的TCP和UDP，利用重新传输管理的多路复用。在场景定义文件中可以使用正规表达式，动态调整呼叫速率。 
    SIPp可以用来测试许多真实的SIP设备，如SIP代理，B2BUAs,SIP媒体服务器，SIP/x网关，SIP PBX，等等，它也可以模仿上千个SIP代理呼叫你的SIP系统。
    关于SIPp从google上搜索到很多，可是关于SIPp的中文说明资料较少，或者很多都是不齐全的安装使用说明。
    SIPp的网址：http://sipp.sourceforge.net/
SIPp的下载地址：
http://sourceforge.net/project/showfiles.php?group_id=104305&package_id=119322 (当我已经在使用rc6的时候，rc8已经出来了，|||-.-)
SIPp的四种安装方法：

1)  没有TLS支持与密码验证支持：
a) # tar -xvf sipp-1.1rc6.tar.gz
b) # cd sipp-1.1.rc6
c) # make
Make出来的sipp文件就是一个可执行的文件，只需要搭配场景xml文件与csv文件即可进行SIP测试
2) 拥有TLS支持与密码验证支持，但是不支PCAP语音播放：
a) # tar -xvf sipp-1.1rc6.tar.gz
b) # cd sipp-1.1.rc6
c) # make ossl
这样编译出来的文件就加入了TLS至于与密码验证支持功能sipp软件了。
3) 支持PCAP Play，但是没有密码验证支持：(PCAP Play即为可以进行RTP语音，但是没有407 AUTH验证)
a) # tar -xvf sipp-1.1rc6.tar.gz
b) # cd sipp-1.1.rc6
c) # make pcapplay
4) 支持PCAP 声音文件播放，而且支持密码验证支持：(支持407 auth验证支持)
a) # tar -xvf sipp-1.1rc6.tar
b) # cd sipp-1.1.rc6
c) # make pcapplay_ossl
最新消息：使用sipp-1.1rc6后，如果采用pcap方式发包播放后，通过抓包抓不到session的消息体。多次尝试与配置文件的修改均查看不到sip的session体。后来更新到sipp-1.1rc8后，抓包就可以看到sip session体了，看来其他使用者已经发现这个bug了.
 
 
## 第二章SIP的几个主要呼叫流程介绍
例1:
invite呼叫后暂停，结束呼叫。
A呼叫B，Ast返回100 tring与180 ring后，这边回ACK消息，然后Pause 10秒，发送Bye消息，系统返回200 ok。
```
    |(1) INVITE         |
    |---------------à |
    |(2) 100 (optional)|
    |<-----------------|
    |(3) 180 (optional)|
    |<-----------------|
    |(4) 200             |
    |<-----------------|
    |(5) ACK             |
    |---------------à |
    |                     |
    |(6) PAUSE          |
    |                     |
    |(7) BYE             |
    |----------------->|
    |(8) 200             |
    |<-----------------|
```
例2：
invite呼叫，建立连接然后RTP，并带有RFC2833的DTMF,延迟几秒后发送Bye消息，对方返回200 OK。
```
Scenario file: uac_pcap.xml (original XML file)
SIPp UAC            Remote
    |(1) INVITE         |
    |------------------>|
    |(2) 100 (optional) |
    |<------------------|
    |(3) 180 (optional) |
    |<------------------|
    |(4) 200            |
    |<------------------|
    |(5) ACK            |
    |------------------>|
    |                   |
    |(6) RTP send (8s)  |
    |==================>|
    |                   |
    |(7) RFC2833 DIGIT 1|
    |==================>|
    |                   |
    |(8) BYE            |
    |------------------>|
    |(9) 200            |
    |<------------------|
```
例3：
SIPp作为SIP 服务器进行处理。
```
Remote              SIPp UAS
    |(1) INVITE         |
    |----------------->|
    |(2) 180             |
    |<-----------------|
    |(3) 200             |
    |<-----------------|
    |(4) ACK             |
    |----------------->|
    |                      |
    |(5) PAUSE           |
    |                      |
    |(6) BYE              |
    |------------------>|
    |(7) 200              |
|<------------------－－|
```
第一章例4：
典型的SIP register成功后、然后invite到AST，AST回了100与180或者403 forbidden消息，SIPp发送ACK，延迟5000ms后，SIPp发送Bye，AST回200 OK
```
REGISTER ----------――>
         200 <----------
         200 <----------
      INVITE ---------->
         100 <----------
         180 <----------
         403 <----------
         200 <----------
         ACK ---------->
             [  5000 ms]
         BYE ---------->
         200 <----------－-
```
第二章如何编写场景xml文件
在编写场景xml文件之前，首先要懂得SIP得整个消息流程。SIPp是通过xml场景文件读取后，根据xml定义得流程去分析下一步要进行得动作。因此，熟悉了xml文件得定义内容、同时也要了解整个SIP消息发送流程，那么就很容易编写xml文档了。
例1：
模拟若干个注册包到AST，AST回 407 authentication，SIPp发送invite带auth验证消息到AST，AST回100 tring 和200 ok。SIPp回Bye消息，AST回200 ok。
```
Register­-----------àAST
401ß----------------AST
Register-----------àAST
100ß----------------AST
200ß----------------AST
```
编写该SIP得发包流程得场景xml文件内容如下：
```
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE scenario SYSTEM "sipp.dtd">
 
<!-- This program is free software; you can redistribute it and/or      -->
<!-- modify it under the terms of the GNU General Public License as     -->
<!-- published by the Free Software Foundation; either version 2 of the -->
<!-- License, or (at your option) any later version.                    -->
<!--                                                                    -->
<!-- This program is distributed in the hope that it will be useful,    -->
<!-- but WITHOUT ANY WARRANTY; without even the implied warranty of     -->
<!-- MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the      -->
<!-- GNU General Public License for more details.                       -->
<!--                                                                    -->
<!-- You should have received a copy of the GNU General Public License  -->
<!-- along with this program; if not, write to the                      -->
<!-- Free Software Foundation, Inc.,                                    -->
<!-- 59 Temple Place, Suite 330, Boston, MA  02111-1307 USA             -->
<!--                                                                    -->
<!--                 Sipp default 'branchc' scenario.                   -->
<!--                                                                    -->
 
<!— 首先发送SIP注册消息，Register。里面的From与To是注册的号码       à
<scenario name="branch_client">
  <send retrans="500">
    <![CDATA[
 
      REGISTER sip:[remote_ip] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field0] <sip:[field0]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 1 REGISTER
      Contact: sip:[field0]@[local_ip]:[local_port]
      Content-Length: 0
      Expires: 300
 
    ]]>
   </send>
 
  <!--  SIPp会收到来自AST要求验证的401 消息体，Recv意思为Receive，接收到来自AST的401要求验证的消息，Next为如果收到401，那么转至Label为1的地方进行操作           à     
  <recv response="401" auth="true" next="1">
  </recv>
   
  <!--  send invite with authentication messages -->
  <!—  开始发送Register消息，里面将把验证的密码消息发送给对方，在消息体里面是抓不到密码消息的，而且已经被md5方式加密过。因此通过ethereal是无法抓到的。à
<label id="1"/>
  <send retrans="500">
  <![CDATA[
 
      REGISTER sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field0] <sip:[field0]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 2 REGISTER
      Contact: sip:[field0]@[local_ip]:[local_port]
      [field1]
      Content-Length: [len]
      Expires: 3600
    ]]>
  </send>
 
  <!--   收到来自AST的200 ACK消息后，系统转至等待1000ms，或者可以直接去掉该设置 à
  <recv response="200"  next="2">
  </recv>
 
  <label id="2"/>
  <pause milliseconds="1000"/>
  
  
  <!-- definition of the response time repartition table (unit is ms)   -->
  <ResponseTimeRepartition value="10, 20, 30, 40, 50, 100, 150, 200"/>
 
  <!-- definition of the call length repartition table (unit is ms)     -->
  <CallLengthRepartition value="10, 50, 100, 500, 1000, 5000, 10000"/>
 
</scenario>
```
例2：
模拟多个封包发送Invite消息到AST，AST回407要求验证，SIPp发送invite消息带407 请求验证的消息到AST，AST返回200 ok。SIPp收到200 ok，延迟1000ms后发送bye消息到AST，AST返回200 ok。
```
Invite ―――――――――>AST
401 ß-----------------AST
Invite with auth--------àAST
200 ok <------------------AST
Pause 1000
Bye---------------------àAST
200 ok ß-----------------AST
XML文件内容如下：
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE scenario SYSTEM "sipp.dtd">
 
<!-- This program is free software; you can redistribute it and/or      -->
<!-- modify it under the terms of the GNU General Public License as     -->
<!-- published by the Free Software Foundation; either version 2 of the -->
<!-- License, or (at your option) any later version.                    -->
<!--                                                                    -->
<!-- This program is distributed in the hope that it will be useful,    -->
<!-- but WITHOUT ANY WARRANTY; without even the implied warranty of     -->
<!-- MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the      -->
<!-- GNU General Public License for more details.                       -->
<!--                                                                    -->
<!-- You should have received a copy of the GNU General Public License  -->
<!-- along with this program; if not, write to the                      -->
<!-- Free Software Foundation, Inc.,                                    -->
<!-- 59 Temple Place, Suite 330, Boston, MA  02111-1307 USA             -->
<!--                                                                    -->
<!--                 Sipp default 'uac' scenario.                       -->
<!--                                                                    -->
 
<!--  发送Invite消息到asterisk，from为主叫号码，to为被叫号码         -à
<scenario name="Basic Sipstone UAC">
  <!-- In client mode (sipp placing calls), the Call-ID MUST be         -->
  <!-- generated by sipp. To do so, use [call_id] keyword.              -->
  <send retrans="500">
    <![CDATA[
 
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 1 INVITE
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Type: application/sdp
      Content-Length: [len]
 
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      c=IN IP[media_ip_type] [media_ip]
      t=0 0
      m=audio [media_port] RTP/AVP 0
      a=rtpmap:0 PCMU/8000
 
    ]]>
  </send>
<!--  收到来自AST的407要求验证消息                                   -à
  <recv response="407" auth="true">
  </recv>
<!--  首先发送200 ACK消息到AST                                        --à
  <!-- Packet lost can be simulated in any send/recv message by         -->
  <!-- by adding the 'lost = "10"'. Value can be [1-100] percent.       -->
  <send>
    <![CDATA[
 
      ACK sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 1 ACK
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Length: 0
 
    ]]>
<!---  接着发送invite带407请求验证消息                                ―――>
  </send>
 
  <!-- This delay can be customized by the -d command-line option       -->
  <!-- or by adding a 'milliseconds = "value"' option here.             -->
 
  <!-- Send 407 Authentication messages                                 -->
  <send retrans="500">
  <![CDATA[
 
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 2 INVITE
      Contact: sip:[field0]@[local_ip]:[local_port]
      [field2]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Type: application/sdp
      Content-Length: [len]
 
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      t=0 0
      c=IN IP[media_ip_type] [media_ip]
      m=audio [media_port] RTP/AVP 0
      a=rtpmap:0 PCMU/8000
    ]]>
  </send>
  <!---    SIPp收到来自AST的100 trying消息                         ――>
  <recv response="100" >
  </recv>
  <!--  延迟5000ms后，发送Bye消息到AST                              ――>
  <pause milliseconds="5000"/>
  <!-- The 'crlf' option inserts a blank line in the statistics report. -->
  <send retrans="500">
    <![CDATA[
 
      BYE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 2 BYE
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Length: 0
 
    ]]>
  </send>
  <!-- SIPp收到200 OK消息                                            ――>
  <recv response="200" crlf="true">
  </recv>
 
  <!-- definition of the response time repartition table (unit is ms)   -->
  <ResponseTimeRepartition value="10, 20, 30, 40, 50, 100, 150, 200"/>
 
  <!-- definition of the call length repartition table (unit is ms)     -->
  <CallLengthRepartition value="10, 50, 100, 500, 1000, 5000, 10000"/>
 
</scenario>
```
例3：
SIPp发送Invite消息到AST，AST回407要求密码验证，SIPp返回200 ok，并发送invite带密码消息到AST，AST返回200 OK，接着返回180，SIPp此时开始传输RTP到AST，延迟5000ms发送DTMF=1的号码，然后SIPp发送Bye消息到AST，AST返回200 ok给SIPp。
```
Invite－―――――---――>AST
407 <-------------------->AST
Invite with auth--------àAST
200 ok<------------------>AST
180 ring<---------------->AST
    RTP==================>AST
    DTMF digits 1========>AST
    BYE------------------>AST
200 OK<-------------------AST
XML文档描述如下：
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE scenario SYSTEM "sipp.dtd">
 
<!-- This program is free software; you can redistribute it and/or      -->
<!-- modify it under the terms of the GNU General Public License as     -->
<!-- published by the Free Software Foundation; either version 2 of the -->
<!-- License, or (at your option) any later version.                    -->
<!--                                                                    -->
<!-- This program is distributed in the hope that it will be useful,    -->
<!-- but WITHOUT ANY WARRANTY; without even the implied warranty of     -->
<!-- MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the      -->
<!-- GNU General Public License for more details.                       -->
<!--                                                                    -->
<!-- You should have received a copy of the GNU General Public License  -->
<!-- along with this program; if not, write to the                      -->
<!-- Free Software Foundation, Inc.,                                    -->
<!-- 59 Temple Place, Suite 330, Boston, MA  02111-1307 USA             -->
<!--                                                                    -->
<!--                 Sipp 'uac' scenario with pcap (rtp) play           -->
<!--                                                                    -->
 
<scenario name="UAC with media">
  <!-- In client mode (sipp placing calls), the Call-ID MUST be         -->
  <!-- generated by sipp. To do so, use [call_id] keyword.                -->
  <send retrans="500">
    <![CDATA[
 
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 1 INVITE
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Type: application/sdp
      Content-Length: [len]
 
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      c=IN IP[local_ip_type] [local_ip]
      t=0 0
      m=audio [auto_media_port] RTP/AVP 8
      a=rtpmap:8 PCMA/8000
      a=rtpmap:101 telephone-event/8000
      a=fmtp:101 0-11,16
 
    ]]>
  </send>
 
  <recv response="407" auth="true">
  </recv>
 
  <!-- By adding rrs="true" (Record Route Sets), the route sets         -->
  <!-- are saved and used for following messages sent. Useful to test   -->
  <!-- against stateful SIP proxies/B2BUAs.                             -->
  <!-- Packet lost can be simulated in any send/recv message by         -->
  <!-- by adding the 'lost = "10"'. Value can be [1-100] percent.       -->
  <send>
    <![CDATA[
 
      ACK sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 1 ACK
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Length: 0
 
    ]]>
  </send>
 
  <send retrans="500">
  <![CDATA[
 
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 2 INVITE
      Contact: sip:[field0]@[local_ip]:[local_port]
      [field2]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Type: application/sdp
      Content-Length: [len]
 
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      t=0 0
      c=IN IP[media_ip_type] [media_ip]
      m=audio [auto_media_port] RTP/AVP 0
      a=rtpmap:0 PCMU/8000
    ]]>
 </send>
 
 <recv response="100" optional="true">
 </recv>
 
 <recv response="180" optional="true">
 </recv>
 
 <recv response="200" rtd="true" crlf="true">
 </recv>
  <!-- Play a pre-recorded PCAP file (RTP stream)                       -->
  <nop>
    <action>
      <exec play_pcap_audio="pcap/g711a.pcap"/>
    </action>
  </nop>
 
  <!-- Pause 8 seconds, which is approximately the duration of the      -->
  <!-- PCAP file                                                        -->
  <pause milliseconds="5000"/>
 
  <!-- Play an out of band DTMF '1'                                     -->
  <nop>
    <action>
      <exec play_pcap_audio="pcap/dtmf_2833_1.pcap"/>
    </action>
  </nop>
 
  <pause milliseconds="5000"/>
 
  <!-- The 'crlf' option inserts a blank line in the statistics report. -->
  <send retrans="500">
    <![CDATA[
 
      BYE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 2 BYE
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Length: 0
 
    ]]>
  </send>
 
  <recv response="200" crlf="true">
  </recv>
 
  <!-- definition of the response time repartition table (unit is ms)   -->
  <ResponseTimeRepartition value="10, 20, 30, 40, 50, 100, 150, 200"/>
 
  <!-- definition of the call length repartition table (unit is ms)     -->
  <CallLengthRepartition value="10, 50, 100, 500, 1000, 5000, 10000"/>
 
</scenario>
```
例3：
SIPp发送Invite消息到AST，AST回407要求密码验证，SIPp返回200 ok，并发送invite带密码消息到AST，AST返回200 OK，接着返回180，SIPp此时开始传输RTP到AST，延迟5000ms发送DTMF=1的号码，然后SIPp发送Bye消息到AST，AST返回200 ok给SIPp。
```
Invite－―――――---――>AST
407 <-------------------->AST
Invite with auth--------àAST
200 ok<------------------>AST
180 ring<---------------->AST
    RTP==================>AST
    DTMF digits 1========>AST
    BYE------------------>AST
200 OK<-------------------AST
XML文档描述如下：
<?xml version="1.0" encoding="ISO-8859-1" ?>
<!DOCTYPE scenario SYSTEM "sipp.dtd">
 
<!-- This program is free software; you can redistribute it and/or      -->
<!-- modify it under the terms of the GNU General Public License as     -->
<!-- published by the Free Software Foundation; either version 2 of the -->
<!-- License, or (at your option) any later version.                    -->
<!--                                                                    -->
<!-- This program is distributed in the hope that it will be useful,    -->
<!-- but WITHOUT ANY WARRANTY; without even the implied warranty of     -->
<!-- MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the      -->
<!-- GNU General Public License for more details.                       -->
<!--                                                                    -->
<!-- You should have received a copy of the GNU General Public License  -->
<!-- along with this program; if not, write to the                      -->
<!-- Free Software Foundation, Inc.,                                    -->
<!-- 59 Temple Place, Suite 330, Boston, MA  02111-1307 USA             -->
<!--                                                                    -->
<!--                 Sipp 'uac' scenario with pcap (rtp) play           -->
<!--                                                                    -->
 
<scenario name="UAC with media">
  <!-- In client mode (sipp placing calls), the Call-ID MUST be         -->
  <!-- generated by sipp. To do so, use [call_id] keyword.                -->
  <send retrans="500">
    <![CDATA[
 
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 1 INVITE
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Type: application/sdp
      Content-Length: [len]
 
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      c=IN IP[local_ip_type] [local_ip]
      t=0 0
      m=audio [auto_media_port] RTP/AVP 8
      a=rtpmap:8 PCMA/8000
      a=rtpmap:101 telephone-event/8000
      a=fmtp:101 0-11,16
 
    ]]>
  </send>
 
  <recv response="407" auth="true">
  </recv>
 
  <!-- By adding rrs="true" (Record Route Sets), the route sets         -->
  <!-- are saved and used for following messages sent. Useful to test   -->
  <!-- against stateful SIP proxies/B2BUAs.                             -->
  <!-- Packet lost can be simulated in any send/recv message by         -->
  <!-- by adding the 'lost = "10"'. Value can be [1-100] percent.       -->
  <send>
    <![CDATA[
 
      ACK sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 1 ACK
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Length: 0
 
    ]]>
  </send>
 
  <send retrans="500">
  <![CDATA[
 
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 2 INVITE
      Contact: sip:[field0]@[local_ip]:[local_port]
      [field2]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Type: application/sdp
      Content-Length: [len]
 
      v=0
      o=user1 53655765 2353687637 IN IP[local_ip_type] [local_ip]
      s=-
      t=0 0
      c=IN IP[media_ip_type] [media_ip]
      m=audio [auto_media_port] RTP/AVP 0
      a=rtpmap:0 PCMU/8000
    ]]>
 </send>
 
 <recv response="100" optional="true">
 </recv>
 
 <recv response="180" optional="true">
 </recv>
 
 <recv response="200" rtd="true" crlf="true">
 </recv>
  <!-- Play a pre-recorded PCAP file (RTP stream)                       -->
  <nop>
    <action>
      <exec play_pcap_audio="pcap/g711a.pcap"/>
    </action>
  </nop>
 
  <!-- Pause 8 seconds, which is approximately the duration of the      -->
  <!-- PCAP file                                                        -->
  <pause milliseconds="5000"/>
 
  <!-- Play an out of band DTMF '1'                                     -->
  <nop>
    <action>
      <exec play_pcap_audio="pcap/dtmf_2833_1.pcap"/>
    </action>
  </nop>
 
  <pause milliseconds="5000"/>
 
  <!-- The 'crlf' option inserts a blank line in the statistics report. -->
  <send retrans="500">
    <![CDATA[
 
      BYE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port];branch=[branch]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>[peer_tag_param]
      Call-ID: [call_id]
      CSeq: 2 BYE
      Contact: sip:[field0]@[local_ip]:[local_port]
      Max-Forwards: 70
      Subject: Performance Test
      Content-Length: 0
 
    ]]>
  </send>
 
  <recv response="200" crlf="true">
  </recv>
 
  <!-- definition of the response time repartition table (unit is ms)   -->
  <ResponseTimeRepartition value="10, 20, 30, 40, 50, 100, 150, 200"/>
 
  <!-- definition of the call length repartition table (unit is ms)     -->
  <CallLengthRepartition value="10, 50, 100, 500, 1000, 5000, 10000"/>
 
</scenario>
```
第三章 SIPP压力测试最好的工具
如何使用SIPp进行压力测试
1)  编辑好xml场景文件；
2)  编辑csv文件；
a)  Csv文件为sipp要压力测试的读取的变量文件，也就是说从csv文件一个个去读取，然后填写到xml的field变量中，从而实现压力测试。举例说明：
b)  Xml文件的分析；
```
      INVITE sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field1] <sip:[field1]@[remote_ip]:[remote_port]>
         这里的field0，field1为csv文件每个字段的内容。
c)       再来看csv文件的分析；
3000;3001
 3001;3002
 3003;3004
```
 这里的filed0，field1所指的就是3000，3001，以此类推，因此你可以建立自己大批量内容的csv文件。
d) 如何编写401 407的验证的csv文件；
 当AST发回要求验证的401或407验证的时候，SIPp只要发回验证密码的请求内容即可。注意401是register请求验证消息，407是invite请求验证消息，这点不能搞混。举例说明：
```
  <recv response="401" auth="true" next="1">
  </recv>
  <send retrans="500">
  <![CDATA[
 
      REGISTER sip:[field0]@[remote_ip]:[remote_port] SIP/2.0
      Via: SIP/2.0/[transport] [local_ip]:[local_port]
      From: [field0] <sip:[field0]@[local_ip]:[local_port]>;tag=[call_number]
      To: [field0] <sip:[field0]@[remote_ip]:[remote_port]>
      Call-ID: [call_id]
      CSeq: 2 REGISTER
      Contact: sip:[field0]@[local_ip]:[local_port]
      [field1]   这里就是发送的验证消息，【field1】
      Content-Length: [len]
      Expires: 3600
    ]]>
  </send>
         那么我们如何编写csv文件呢？如下：
         3000;[authentication username=3000 password=3000]
         3001;[authentication username=3001 password=3001] (以此类推)
         说明：这里的field0为3000，filed1为[authentication username=3000 password=3000]
```
3)  使用方法：
```
     ./sipp -sf reg.xml -inf reg.csv -p 6077 -i <local-ip> -m 4 <ip address of registrar>:5060
     -sf  读取场景文件
     -inf 读取csv文件
     -p  本机采用端口
     -i  本机IP
     -m  要进行压力测试的次数
     Ip address of registrar  要进行压力测试的IP地址
```
4)  使用Sipp后的界面显示：

 
这里是启动后的界面，可以看到运行的整个过程。简单介绍下：
[＋－*/]，简单来讲就是加减乘除，是指运行同一秒中要处理的并发数。*是之前的并发数成倍的增长，＋是指并发数从10－30个逐步增加。－与/同理。
```
【q】:Soft exit 软件退出
【p】:Pause traffic 软件暂停运行
这里我们可以观察到
Call rate(length)      port     Total-time   Total-calls       Remote host
14.0(0 ms)/1.000s      6077      307179.49s     4120556      192.168.0.194:5060
14 new calls during 1.002 s period    每秒处理并发为14个call
374 calls (limit 420)                 374个call等待处理，最多420个call
非常详细的记录
测试者可以按下1、2、3、4、5按键来看更为详细的记录，下图为按键2后的图面
 
```
这里我们可以看到每秒处理的成功的call为13.417个，总共处理成功的call为412952个。Failed call为被丢弃失败的call为15588个。
5)       如何编写csv文件，这里有个bash小脚本程序，可以帮助实现：
```
#!/bin/bash
i=3000
while [ $i != 3100 ]
do
  i=$(($i+1))
#  j=$(($i+1))
echo “” >test.csv
echo "$i;[authentication username=$i password=$i]" >>test.csv
done
```
这里生成文件csv文件为
```
3000;[authentication username=3000 password=3000]
3001;[authentication username=3001 password=3001]
           .
           .
           .
3100;[authentication username=3100 password=3100]
```
6) 要从3000到3100都要注册，因此在AST系统要实现建立好3000到3100号码分配，而且密码必须要与号码一致。譬如3000的号码密码必须为3000，这样才能与csv文件配置一致。这里就不在烦述了。
7) 灵活的配置xml、csv文件可以实现各种消息的混合一起实现，譬如可以实现register、invite带407、最后发送bye消息，也可以实现共同invite到一个meeting，然后发送rtp方式，滞留10分钟后，发送bye退出meeting等等。
当然必须要有扎实的对SIP精通的消息流程，这里介绍的方法为通过eyebeam软终端注册后将整个包抓下来分析，然后根据分析的包内容修改模版即可实现模拟过程。

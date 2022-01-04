# FreeSwitch中的会议功能
 FreeSwitch 默认支持会议功能，有如下特点：
          （1） 不需要创建一个会议室的操作，只需要通过 conference 拨码计划就可以实现;
          （2） 会议室不真正存在， 直到有人呼入为止；
          （3） 会议功能很强大，能实现灵活控制。
 这样讲太学术化，来点直观的，步骤如下：
          （1） 运行 FREESWITCH 服务器程序；
          （2） 注册 1000、1001、1002三部IP话机；
          （3） 通过 1000 呼叫 3000，通话建立后， 1000 将听到一段保持音乐；
          （4） 通过 1001 呼叫 3000，通话建立后， 1001将能听到1000的声音，1000也能听到1001的声音；
          （5） 通过 1002 呼叫 3000，通话建立后，  1002将能听到 1000 和 1001的声音，1001能听到1000和1002的声音，1000也能听到 1001 和 1002 的声音。              
            那 3000 这个号码是怎么来的？ 请看
```            
 \conf\dialplan\default.xml 中的内容，如下所示：
             \<!--
                   start a dynamic conference with the settings of the "default" conference profile in conference.conf.xml
                   使用 conference.conf.xml 中定义的属性进行如下会议
             -->                                                                                                                                                       
            \<extension name="nb_conferences">
                \<condition field="destination_number" expression="^(30\d{2})$">
                   \<action application="answer"/>
                   \<action application="conference" data="$1-${domain_name}@default"/>
                \</condition>
             \</extension>
            ......
```
 FreeSwitch 提供了一些控制会议成员行为的方法，罗列如下：
 ```
          （1）Talk volume: The volume of the audio the caller sends (that is, gain control).
                   与会成员讲话的音量控制；
          （2）Listen volume: The volume of the audio the caller hears.
                   与会成员收听语音的音量控制；
          （3）Energy threshold: The minimum energy level of the audio from the caller to be considered talking. Raising the energy level will cut down on background noise when
                   a participant is in a noisy environment.
                   语音门限控制 。
 ```
            具体用法，可以查看：
```            
\conf\autoload_configs\console.conf.xml ，内容如下：
             <caller-controls>
                 <group name="default">
                   <control action="mute" digits="0"/>             // 静音
                   <control action="deaf mute" digits="*"/>      // 解除静音
                   <control action="energy up" digits="9"/>     // 增加门限
                   <control action="energy equ" digits="8"/>   // 
                   <control action="energy dn" digits="7"/>     // 降低门限
                   <control action="vol talk up" digits="3"/>     // 提高讲话音量 
                   <control action="vol talk zero" digits="2"/>  // 讲话音量设置为0
                   <control action="vol talk dn" digits="1"/>     // 降低讲话音量
                   <control action="vol listen up" digits="6"/>  // 提高收听音量
                   <control action="vol listen zero" digits="5"/> // 收听音量设置为0
                   <control action="vol listen dn" digits="4"/>    // 降低收听音量 
                   <control action="hangup" digits="#"/>           // 退出会议
               </group>
            </caller-controls>
```
 FreeSwitch中可以设置主持人以及会议密码。设置了主持人后，可以影响会议的开展；设置了会议密码后，与会成员必须输入正确密码才能入会。
        主持人对会议的影响主要体现在以下两个方面：
      （1）直到主持人入会后，会议才开始；
      （2）主持人退出会议后，会议才结束。
```
        那怎么设置主持人？方法如下：
        <action application="conference" data="$1@default"/> // 未设置主持人
        <action application="conference" data="$1@default+flags{moderator}"/> // 设置了主持人
 
        如何设置会议密码？方法如下：
        <action application="conference" data="$1@default+1234"/> // 设置入会密码为 1234
 
        如何既设置主持人，又设置会议密码？方法如下：
       <action application="conference" data="$1@default+1234+flags{moderator}"/>
```
       
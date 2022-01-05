# mjSip学习笔记
1.URL:http://www.mjsip.org/
2.它也是用JAVA编写的,唯一要求的外部库是客户端使用的JMF(只要安装了JMF就可以了,编译和执行都不用特殊处理).它比SUN的JAIN的STACK简单,而且新,上次RELEASE估计在2005年十月份,所对应的SIP功能也比JAIN多,包括支持REFER等消息格式.它也提供源程序,源程序结构比JAIN简单多了,三个部分:包括SIP的STACK和SIP的SERVER和SIP的客户端.它也有BIN的下载,服务器和客户端配置都很简单,基本上看着配置文件内部说明就可以了.从我看到该网站到学习下栽编译测试和分析LOG基本上不用一天工作时间就完成了.
3.不过它的MAKE文件是GNU的不是ANT的,对NETBEANS来说使用不便,不过由于它结构简单,我很方便的用NETBEANS5.0生成了三个PRJ(STACK,SERVER,UA),再从源程序中拷贝JAVA文件和配置文件到这三个PRJ中,很方便地就再编译和执行成功了.
这三个NETBEANS5.0的PRJ源程序和项目文件在这里:

MjSIP.rar(只用于学习,版权问题盖不负责,请去http://www.mjsip.org/查询).
4.它的SERVER缺省运行在STATELESS中,当然也可是STATEFUL的.
5.和JAIN的SAMPLE一样,大概是JMF的缘故它的UA在通话中也有很大的延迟,大约一秒多吧.
6.可惜没有商业开发的免费许可(要购买).
7.测试了下,一次INITE对话大概要用0.2秒多点时间,其他简单的TRANSACTION只要用0.1妙左右.
8.2006年4月13日,发现各致命的错误:
   当INVITE或REGISTER等消息因为认证(407,401)而被打回后,再加入了认证HEADER再送时,消息的BRANCH值仍然和第一次的一样.这样一来就和第一次同一个TRANSACTION了(尽管程序是NEW了个新TRANSACTION,但它是用上次消息经过加工后NEW的,所以BRANCH值仍然是上次旧的),而RFC要求是两个不同的TRANSACTION,这样一来服务器就可能因为新消息仍然属于旧TRANSACTION而拒绝响应.
   我在ExtendedInviteDialog.onTransFailureResponse中把旧消息的BRANCH值更新了下(在BaseMessage.refreshBranch()中增加了更新函数),测试了下INVITE或REGISTER的情况,正常.

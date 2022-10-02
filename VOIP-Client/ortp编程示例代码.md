# ortp编程示例代码
   鉴于很多网友找我要ortp的示例代码，因此，今天抽空把相关资料整理了一下，写了一个windows版的ortp示例程序，发布在这里供网友们参考吧。
    编译及运行环境：VS2008，windows
    编程语言：c/c++，ortp库为c语言封装，我用c++对其进行了进一步封装，如果需要c语言的封装接口，只需要把类中相关函数提取出来即可使用。
    ortp库：ortp-0.9.1（由于是以前写的代码，故用的ortp库比较老，但不影响使用和学习，我附件中的工程中已经把ortp-0.9.1库文件添加进去了）
    整个测试代码在工程的附件中，大家下载后直接编译后，在Debug目录下打开2个本程序，一个选择Client，一个选择Server，即可看到测试效果。
    下面，我的相关代码发布如下（附件中有完整的工程）。
## 一、ORTP接收端封装类
```
////////////////////////////////////////////////////////////////////////// 
//  COPYRIGHT NOTICE 
//  Copyright (c) 2011, 华中科技大学 卢俊（版权声明） 
//  All rights reserved. 
//  
/// @file    CortpClient.h   
/// @brief   ortp客户端类声明文件 
/// 
/// 实现和提供ortp的客户端应用接口 
/// 
/// @version 1.0    
/// @author  卢俊  
/// @date    2011/11/03 
// 
// 
//  修订说明： 
////////////////////////////////////////////////////////////////////////// 
#ifndef CORTPCLIENT_H_ 
#define CORTPCLIENT_H_ 
#include  
#include  
/** 
 *  COrtpClient ortp客户端管理类  
 *    
 *  负责封装和提供ortp相关接口 
 */
class COrtpClient  
{  
public:   
/**  构造函数/析构函数 
     *   
     *  在创建/销毁该类对象时自动调用 
     */
    COrtpClient();  
    ~COrtpClient();    
/** ORTP模块的初始化 
     * 
     *  在整个系统最开始调用，负责ORTP库的初始化 
     *  @return: bool  是否成功 
     *  @note:    
     *  @see:     
     */
staticbool init();  
/** ORTP模块的逆初始化 
     * 
     *  在系统退出前调用，负责ORTP库的释放 
     *  @return: bool  是否成功 
     *  @note:    
     *  @see:     
     */
staticbool deInit();  
/** 创建RTP接收会话 
     * 
     *  负责产生RTP接收端会话，监听服务器端的数据 
     *  @param:  const char * localip 本地ip地址 
     *  @param:  int localport  本地监听端口 
     *  @return: bool  是否成功 
     *  @note:    
     *  @see:     
     */
bool create(constchar * localip, int localport );  
/** 获取接收到的rtp包 
     * 
     *  将接收到的rtp数据包取出 
     *  @param:  char * pBuffer 
     *  @param:  int & len 
     *  @return: bool  是否成功 
     *  @note:    
     *  @see:     
     */
bool get_recv_data( char *pBuffer, int &len );  
private:  
    RtpSession *m_pSession;     /** rtp会话句柄 */
long        m_curTimeStamp; /** 当前时间戳 */
int         m_timeStampInc; /** 时间戳增量 */
};  
#endif // CortpClient_H_ 
////////////////////////////////////////////////////////////////////////// 
//  COPYRIGHT NOTICE 
//  Copyright (c) 2011, 华中科技大学 卢俊（版权声明） 
//  All rights reserved. 
//  
/// @file    CortpClient.cpp   
/// @brief   ortp客户端类实现文件 
/// 
/// 实现和提供ortp的客户端应用接口 
/// 
/// @version 1.0    
/// @author  lujun  
/// @date    2011/11/03 
// 
// 
//  修订说明： 
////////////////////////////////////////////////////////////////////////// 
#include "CortpClient.h" 
/* the payload type define */
#define PAYLOAD_TYPE_VIDEO 34 
/* RTP video Send time stamp increase */
#define VIDEO_TIME_STAMP_INC  3600 
/** 从rtp接收缓冲区一次读取的字节数 */
#define READ_RECV_PER_TIME    1024 
COrtpClient::COrtpClient()  
{  
    m_pSession = NULL;  
    m_timeStampInc = 0;  
    m_curTimeStamp = 0;  
}  
COrtpClient::~COrtpClient()  
{  
if (!m_pSession)  
    {  
        rtp_session_destroy(m_pSession);  
    }  
}  
bool COrtpClient::init()  
{  
int ret;  
    WSADATA wsaData;  
/** 初始化winsocket */
if ( WSAStartup(MAKEWORD(2,2), &wsaData) != 0)  
    {  
returnfalse;  
    }  
    ortp_init();  
    ortp_scheduler_init();  
returntrue;  
}  
bool COrtpClient::deInit()  
{  
    ortp_exit();  
if (WSACleanup() == SOCKET_ERROR)  
    {  
returnfalse;  
    }  
returntrue;  
}  
bool COrtpClient::create( constchar * localip, int localport )  
{  
if ( m_pSession != NULL)  
    {  
returnfalse;  
    }  
/** 创建新会话 */
    m_pSession = rtp_session_new(RTP_SESSION_RECVONLY);  
if ( !m_pSession)  
    {  
returnfalse;  
    }  
/** 配置相关参数 */
    rtp_session_set_scheduling_mode(m_pSession,1);  
    rtp_session_set_blocking_mode(m_pSession,1);  
    rtp_session_set_local_addr(m_pSession,localip,localport);  
    rtp_session_enable_adaptive_jitter_compensation(m_pSession,1);  
    rtp_session_set_jitter_compensation(m_pSession,40);  
    rtp_session_set_payload_type(m_pSession,PAYLOAD_TYPE_VIDEO);  
    m_timeStampInc = VIDEO_TIME_STAMP_INC;  
returntrue;  
}  
bool COrtpClient::get_recv_data( char *pBuffer, int &len )  
{  
int recvBytes  = 0;  
int totalBytes = 0;  
int have_more = 1;  
while(have_more)  
    {  
if ( totalBytes+READ_RECV_PER_TIME < len )  
        {  
/** 缓冲区大小不够 */
returnfalse;  
        }  
        recvBytes = rtp_session_recv_with_ts(m_pSession,pBuffer+totalBytes,READ_RECV_PER_TIME,m_curTimeStamp,&have_more);  
if (recvBytes <  span>
        {  
break;  
        }  
        totalBytes += recvBytes;  
    }  
/** 判断是否读取到数据 */
if (totalBytes == 0)  
    {  
returnfalse;  
    }  
/** 记录有效字节数 */
    len = totalBytes;  
/** 时间戳增加 */
    m_curTimeStamp += m_timeStampInc;  
returntrue;  
}  
```
## 二、ORTP发送端封装类
```
////////////////////////////////////////////////////////////////////////// 
//  COPYRIGHT NOTICE 
//  Copyright (c) 2011, 华中科技大学 卢俊（版权声明） 
//  All rights reserved. 
//  
/// @file    CortpServer.h 
/// @brief   ortp服务器类声明文件 
/// 
/// 实现和提供ortp的服务器端应用接口 
/// 
/// @version 1.0    
/// @author  卢俊 
/// @date    2011/11/03 
// 
// 
//  修订说明： 
////////////////////////////////////////////////////////////////////////// 
#ifndef CORTPSERVER_H_ 
#define CORTPSERVER_H_ 
#include  
/** 
 *  COrtpServer RTP发送类  
 *    
 *  负责使用RTP协议进行数据的发送 
 */
class COrtpServer  
{  
public:   
/**  构造函数 
     *   
     *  该函数为该类的构造函数，在创建该类对象时自动调用 
     */
    COrtpServer();  
/** 析构函数 
     * 
     * 该函数执行析构操作，由系统自动调用 
     */
    ~COrtpServer();    
/** ORTP模块的初始化 
    * 
    *  在整个系统最开始调用，负责ORTP库的初始化 
    *  @return: bool  是否成功 
    *  @note:    
    *  @see:     
    */
staticbool init();  
/** ORTP模块的逆初始化 
    * 
    *  在系统退出前调用，负责ORTP库的释放 
    *  @return: bool  是否成功 
    *  @note:    
    *  @see:     
    */
staticbool deInit();  
/** 创建RTP接收会话 
    * 
    *  负责产生RTP接收端会话，监听服务器端的数据 
    *  @param:  const char * destIP 目的地址的IP 
    *  @param:  int destport 目的地址的监听端口号 
    *  @return: bool  是否成功 
    *  @note:    
    *  @see:     
    */
bool create(constchar * destIP, int destport );  
/** 发送RTP数据 
     * 
     *  将指定的buffer中的数据发送到客户端 
     *  @param:  unsigned char * buffer 需要发送的数据 
     *  @param:  int len 有效字节数 
     *  @return: int 实际发送的字节数 
     *  @note:   
     *  @see:    
     */
int send_data( unsigned char *buffer, int len );  
private:  
    RtpSession *m_pSession;     /** rtp会话句柄 */
long        m_curTimeStamp; /** 当前时间戳 */
int         m_timeStampInc; /** 时间戳增量 */
char       *m_ssrc;         /** 数据源标识 */
};  
#endif // COrtpServer_H_ 
////////////////////////////////////////////////////////////////////////// 
//  COPYRIGHT NOTICE 
//  Copyright (c) 2011, 华中科技大学 卢俊（版权声明） 
//  All rights reserved. 
//  
/// @file    CortpServer.cpp 
/// @brief   ortp服务器类实现文件 
/// 
/// 实现和提供ortp的服务器端应用接口 
/// 
/// @version 1.0    
/// @author  lujun  
/// @date    2011/11/03 
// 
// 
//  修订说明： 
////////////////////////////////////////////////////////////////////////// 
#include "COrtpServer.h" 
/* the payload type define */
#define PAYLOAD_TYPE_VIDEO 34 
/* RTP video Send time stamp increase */
#define VIDEO_TIME_STAMP_INC  3600 
COrtpServer::COrtpServer()  
{  
    m_ssrc     = NULL;  
    m_pSession = NULL;  
    m_timeStampInc = 0;  
    m_curTimeStamp = 0;  
}  
COrtpServer::~COrtpServer()  
{  
if (!m_pSession)  
    {  
        rtp_session_destroy(m_pSession);  
    }  
}  
bool COrtpServer::init()  
{  
int ret;  
    WSADATA wsaData;  
/** 初始化winsocket */
if ( WSAStartup(MAKEWORD(2,2), &wsaData) != 0)  
    {  
returnfalse;  
    }  
    ortp_init();  
    ortp_scheduler_init();  
returntrue;  
}  
bool COrtpServer::deInit()  
{  
    ortp_exit();  
if (WSACleanup() == SOCKET_ERROR)  
    {  
returnfalse;  
    }  
returntrue;  
}  
bool COrtpServer::create( constchar * destIP, int destport )  
{  
    m_ssrc = getenv("SSRC");  
    m_pSession = rtp_session_new(RTP_SESSION_SENDONLY);   
    rtp_session_set_scheduling_mode(m_pSession,1);  
    rtp_session_set_blocking_mode(m_pSession,1);  
    rtp_session_set_remote_addr(m_pSession,destIP,destport);  
if(m_ssrc != NULL)  
    {  
        rtp_session_set_ssrc(m_pSession,atoi(m_ssrc));  
    }  
    rtp_session_set_payload_type(m_pSession,PAYLOAD_TYPE_VIDEO);  
    m_timeStampInc = VIDEO_TIME_STAMP_INC;  
returntrue;  
}  
int COrtpServer::send_data( unsigned char *buffer, int len )  
{  
int sendBytes = 0;  
/** 强转 */
constchar *sendBuffer = (constchar*)buffer;  
    sendBytes = rtp_session_send_with_ts(m_pSession,sendBuffer,len,m_curTimeStamp);  
if ( sendBytes < 0)  
    {  
        m_curTimeStamp += m_timeStampInc; /** 增加时间戳 */
    }     
return sendBytes;  
} 
```
## 三、测试程序
```
////////////////////////////////////////////////////////////////////////// 
//  COPYRIGHT NOTICE 
//  Copyright (c) 2011, 华中科技大学 卢俊 （版权声明） 
//  All rights reserved. 
// 
/// @file    main.cpp  
/// @brief   ortp测试文件 
/// 
/// 测试ortp发送结构体 
/// 
/// @version 1.0   
/// @author  卢俊 
/// @e-mail  lujun.hust@gmail.com 
/// @date    2011/10/19 
// 
// 
//  修订说明： 
////////////////////////////////////////////////////////////////////////// 
#include  
#include "COrtpClient.h" 
#include "COrtpServer.h" 
/** 本地IP地址 */
constchar * LOCAL_IP_ADDR = "127.0.0.1";  
/** 本地监听端口 */
constint LOCAL_RTP_PORT = 8000;  
/** 目的监听端口 */
constint DEST_RTP_PORT = 8000;  
/** 目的IP地址 */
constchar * DEST_IP_ADDR  = "127.0.0.1";  
/** 一次发送的数据长度 */
constint SEND_LEN_PER_TIME =  8*1024;  
/** 接收缓冲区的总大小 */
constint RECV_BUFFER_LEN   = 10*1024;  
/** 一次接收的数据长度 */
constint RECV_LEN_PER_TIME = 1024;  
bool ortpServer()  
{  
    COrtpServer ortpServer;  
    COrtpServer::init();  
if (!ortpServer.create(DEST_IP_ADDR,DEST_RTP_PORT))  
    {  
        std::cout < span>"ortpServer.create fail!\n";  
        getchar();  
        getchar();  
returnfalse;  
    }  
    unsigned char * buffer = new unsigned char[SEND_LEN_PER_TIME];  
while (1)  
    {  
if ( ortpServer.send_data(buffer,SEND_LEN_PER_TIME) <  span>
        {  
            std::cout < span>"send fail!\n";  
        }  
        Sleep(100);  
        std::cout < span>"send bytes\n";  
    }  
delete [] buffer;  
    COrtpClient::deInit();  
returntrue;  
}  
bool ortpClient()  
{  
    COrtpClient ortpClient;  
    COrtpClient::init();      
if (!ortpClient.create(LOCAL_IP_ADDR,LOCAL_RTP_PORT))  
    {  
        std::cout < span>"ortpClient.create fail!\n";  
        getchar();  
        getchar();  
returnfalse;  
    }  
char *buffer = newchar[RECV_BUFFER_LEN];  
while(1)  
    {  
int len = RECV_BUFFER_LEN;  
if (!ortpClient.get_recv_data(buffer,len))  
        {  
            Sleep(10);  
continue;  
        }  
        std::cout < span>"successful recv,data len =" < lenstdendl span>
    }  
    COrtpClient::deInit();  
delete [] buffer;  
returntrue;  
}  
void main()  
{  
    std::cout < span>"enter num,1 -<client, 2-<server! \n";  
int num;  
    std::cin << num;  
while(1)  
    {  
if (num == 1)  
        {  
            ortpClient();  
break;  
        }  
elseif (num == 2)  
        {  
            ortpServer();  
break;  
        }  
else
        {  
            std::cout < span>"please input 1 or 2 !\n";  
        }  
    }  
    getchar();  
    getchar();  
} 
```
   有关ORTP的介绍、RTP的介绍、RTP的负载类型和时间戳的含义等理论性的东西，都可以在我博客中的其他文章中找到，以上就是整个工程的代码，注释不是很多，因为有些地方我也不是特别清楚，比如jitter、scheduling什么的，如果有什么其他疑问欢迎留言或者E-mail来信交流。

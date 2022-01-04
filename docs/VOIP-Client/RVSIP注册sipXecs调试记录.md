# RVSIP注册sipXecs调试记录
1.首先，为了方便调试，安装了VC6.0。RVSIP属于早期的产品，需要使用VC6.0
2.开始调试
  下面的代码编译后，是可以注册的
```
/*-----------------------------------------------------------------------*/
/*                           DEFINITIONS                                 */
/*-----------------------------------------------------------------------*/
/*Define the registrar address as string. To run the simpleRegistration
 application you need to change this string to the address of your registrar*/
//#define REGISTRAR  "sip:192.168.0.200"
#define REGISTRAR  "sip:211.155.87.141"

/*Define the To and From as strings. Note that the host in the To header is the
 registrars address. Please update this host with the address of your
 registrar. The From header should contain your local SIP-Url. Please update
 it with your local address.*/
#define FROM "From:sip:5003@211.155.87.141:5060"
#define TO   "To:sip:5003@211.155.87.141:5060"
//#define FROM "From:sip:5001@192.168.0.200:5060"
//#define TO   "To:sip:5001@192.168.0.200:5060"

/*Define the Expires header value in delta-seconds*/
#define EXPIRES "Expires:100"

/*Define the Contact headers as strings. Note that CONTACT1 should contain your
 local SIP-Url. Please update it with your local address.*/
//#define CONTACT1 "Contact:sip:5001@211.155.87.141:5060"
#define CONTACT1 "Contact:sip:tpxip@192.168.100.234:5060"//192.168.0.101:5060"
//#define CONTACT2 "Contact:sip:me@another.com;expires=200"
#define CONTACT2 "Contact:sip:tpxip@192.168.100.234;expires=200"

/* Define username and password for the authentication proccess */
#define USERNAME "9902"
#define PASSWORD "aaaaaa"
```
  在sipXecs中，影响注册的是红色的字体部分。按照下面的参数填写，用VC6.0编译出来，似乎是可以注册的。从初步的擦拭结果看，使用RVSIP的SimpleRegister模式的时候，影响注册参数主要是1)FROM 2)TO以及认证中使用的PASSWORD，而作为Digest username的参数，似乎并没有使用，但是，作为参数，也发送给了sipXecs.这里的注册，应该主要考虑的是参数的问题。
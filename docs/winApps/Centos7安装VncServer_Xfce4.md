VncServer安装
首先执行:
\#yum install vnc-server
\#yum install tigervnc-server
Xfce4安装
网上摘抄的，这个已经写烂了，只适合CentOS 6
\#wget http://dl.Fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
\#rpm -ivh epel-release-6-8.noarch.rpm
\#yum search xfce
\#yum groupinfo xfce
\#yum groupinstall xfce
错误信息
1.Error: Cannot retrieve metalink for repository: epel. Please verify its path and try again
    今天在测试环境使用yum安装，遇到一个问题：
    Error: Cannot retrieve metalink for repository: epel. Please verify its path and try again
    处理很简单，修改文件“/etc/yum.repos.d/epel.repo”， 将baseurl的注释取消， mirrorlist注释掉。即可。

# Ubuntu单网卡设置安装双interface和Stun server的方法
Ubuntu单网卡下如何设置双interface和Stun server的安装呢？下文给出了详细的描述，具体内容如下所述。
因为stun server需要同时监听两个entry(即两对IP和PORT）进行对NAT的检测，所以需要server需要两个interface. 首先在Ubuntu上设置两个虚拟的interface,因为只有一个物理的interface,所以只能在这一个上进行虚拟。编辑网络interface文件/etc/network/interfaces， 如下： 
***
```

 # more interfaces  
auto lo  
iface lo inet loopback  
auto eth0:0                                            /*启用虚拟iface0:0*/  
iface eth0:0 inet static                              /*设置此iface为静态IP*/  
address 172.25.27.218  
netmask 255.255.255.0  
gateway 172.25.27.254 auto eth0:1                                         
iface eth:1 inet dhcp                               /*设置此iface为动态获取IP* 
```
***
然后重启网络配置
```
sudo /etc/init.d/networking restart. 
```
下载stun server的软件stund_0.96_Aug13.gz，然后解压。进入stun安装目录,然后make就ok了。 
***
```
:~
# cd stund  
all   
# ./server -h  
STUN server version 0.96  
Usage:  
./server [-v] [-h] [-h IP_Address] [-a IP_Address] [-p port] [-o port] [-m mediaport] If the IP addresses of your NIC are 10.0.1.150 and 10.0.1.151, run this program with  
    ./server -v -h 10.0.1.150 -a 10.0.1.151  
STUN servers need two IP addresses and two ports, these can be specified with:  
-h sets the primary IP  
-a sets the secondary IP  
-p sets the primary port and defaults to 3478  
-o sets the secondary port and defaults to 3479  
-b makes the program run in the backgroud  
-m sets up a STERN server starting at port m  
-v runs in verbose mode # 
```
***
启动就简单了，有-h提示，一目了然。
```
-v -h 172.25.27.126 -a 172.25.27.218  
```
### 总结：
希望本文介绍的Ubuntu单网卡设置安装双interface和Stun server的方法能够对读者有所帮助，更多有关linux系统的知识还有待于读者去探索和学习。
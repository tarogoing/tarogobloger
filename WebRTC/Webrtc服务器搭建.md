# Webrtc服务器搭建
Webrtc服务器包括：房间服务器（Room Server）、信令服务器（Signaling Server）、防火墙打洞服务器（STUN/TURN/ICE Server）
一、房间服务器搭建
1、代码下载
服务器项目地址https://github.com/webrtc/apprtc
```
git clone https://github.com/webrtc/apprtc
```
2、配置依赖环境
```
sudo apt-get install npm
npm -g install grunt-cli
```
Google App Engine SDK for Python：
可选择使用sudo apt-get install python-webtest直接安装python，然后下载Google App Engine SDK for Python并解压，编辑/etc/profile
```
export PATH=$PATH:$HOME/google_appengine/
```
保存退出并执行source /etc/profile。
node直接使用sudo apt-get install nodejs-legacy安装版本过低，使用下面方法安装：
wget https://nodejs.org/dist/v7.7.0/node-v7.7.0-linux-x64.tar.gz
tar xvf node-v7.7.0-linux-x64.tar.gz

配置环境，编辑/etc/profile
```
export PATH=$PATH:$HOME/node-v5.9.0-sunos-x64/bin
```
保存退出执行source /etc/profile
3、安装apprtc代码中的grunt依赖
```
cd apprtc
npm install
grunt build //编译
```
4、修改配置文件
主要是src/app_engine目录下的apprtc.py和constants.py文件。对于src/app_engine目录下的文件每次修改后需执行命令grunt build重新编译，也可以直接编辑out/app_engine目录下的apprtc.py和constants.py避免重新编译。
```
constants.py
#TURN_BASE_URL = 'https://computeengineondemand.appspot.com'
TURN_BASE_URL = 'http://192.168.2.128:3487'
#TURN_URL_TEMPLATE = '%s/turn?username=%s&key=%s'
TURN_URL_TEMPLATE = '%s/turn.php?username=%s&key=%s'
#CEOD_KEY = '4080218913'
CEOD_KEY = '1234' 

WSS_INSTANCES = [{
    #WSS_INSTANCE_HOST_KEY: 'apprtc-ws.webrtc.org:443',
    WSS_INSTANCE_HOST_KEY: '192.168.2.128:443',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-a'
}, {
    #WSS_INSTANCE_HOST_KEY: 'apprtc-ws-2.webrtc.org:443',
    WSS_INSTANCE_HOST_KEY: '192.168.2.128:443',
    WSS_INSTANCE_NAME_KEY: 'wsserver-std-2',
    WSS_INSTANCE_ZONE_KEY: 'us-central1-f'
}]

apprtc.py
 if wss_tls and wss_tls == 'false':
    wss_url = 'ws://' + wss_host_port_pair + '/ws'
    wss_post_url = 'http://' + wss_host_port_pair
  else:
    #wss_url = 'wss://' + wss_host_port_pair + '/ws'
    wss_url = 'ws://' + wss_host_port_pair + '/ws'
    #wss_post_url = 'https://' + wss_host_port_pair
    wss_post_url = 'http://' + wss_host_port_pair

def make_pc_config(ice_transports):
  config = {
  #'iceServers': [],
  'iceServers': [{"urls":"stun:192.168.2.128"},{"urls":"turn:lin@192.168.2.128","credential":"1234"}],
  'bundlePolicy': 'max-bundle',
  'rtcpMuxPolicy': 'require'
  };

  if ice_transports:
    config['iceTransports'] = ice_transports
  return config
```
把原来的wss和https的scheme都改为ws和http，不要让客户端或者浏览器去使用SSL链接。若有第三方根证书的签名机构颁发的证书可忽略。
修改完后重新执行grunt build。
5、启动房间服务器
dev_appserver.py --host=0.0.0.0 ./apprtc/out/app_engine

二、信令服务器搭建
1、安装GO环境
直接使用命令sudo apt-get install golang-go安装的版本太低，后面执行go get collidermain会报错，所以采用下面一种方法：
下载GO安装包并解压
```
wget https://storage.googleapis.com/golang/go1.6.3.linux-amd64.tar.gz
tar xvf go1.6.3.linux-amd64.tar.gz
```
编辑打开文件/etc/profile（也可根据自己需求选择其他环境配置文件编辑），在文件末尾添加两行
```
export GOROOT=$HOME/go 
export PATH=$PATH:$GOROOT/bin
```
保存退出执行source /etc/profile。
2、配置信令服务器
新建目录(collider_root)用于存放Collider的go代码程序。
```
mkdir -p ~/collider_root
mkdir ~/collider_root/src
```
同设置GOROOT设置，在/etc/profile中添加
```
export GOPATH=$HOME/collider_root
export PATH=$PATH:$GOPATH/bin
```
建立链接（也可以直接将~/apprtc/src/collider/目录中的collider、collidermain、collidertest直接拷贝到~/collider_root/src目录下）
```
ln -sf ~/apprtc/src/collider/collider $GOPATH/src/
ln -sf ~/apprtc/src/collider/collidermain $GOPATH/src/
ln -sf ~/apprtc/src/collider/collidertest $GOPATH/src/
```
编辑$GOPATH/collidermain/main.go,修改房间服务器为我们前面的房间服务器：
```
//var roomSrv = flag.String("room-server", "https://appr.tc", "The origin of the room server")
var roomSrv = flag.String("room-server", "http://192.168.2.128:8080", "The origin of the room server")
```
3、安装信令服务器依赖和collidermain
```
go get collidermain
go install collidermain
```
若go get collidermain命令运行失败,那么则用下面这个麻烦的方法:
```
cd $GOPATH/src
wget http://www.golangtc.com/static/download/packages/golang.org.x.net.tar.gz
tar xvf golang.org.x.net.tar.gz
go install golang.org/x/net/websocket/
```
4、运行
```
 $GOPATH/bin/collidermain -port=8089 -tls=false 
```
5、测试
```
go test collider
```
三、STUN/TURN/ICE服务器的搭建
1、下载并安装（详细阅读安装手册 INSTALL）
wget http://turnserver.open-sys.org/downloads/v4.4.1.2/turnserver-4.4.1.2-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz
tar xvfz turnserver-4.4.1.2-debian-wheezy-ubuntu-mint-x86-64bits.tar.gz

sudo apt-get update
sudo apt-get install gdebi-core
sudo gdebi coturn*.deb

2、编辑配置文件
编辑配置文件,打开系统默认启动配置:
$ vim /etc/default/coturn
把上面打开编辑的文件中的这一行TURNSERVER_ENABLED=1去掉注释,保存退出。然后编辑/etc/turnserver.conf
```
listening-device=eth0
relay-device=eth1
Verbose
fingerprint
lt-cred-mech
use-auth-secret
static-auth-secret=1234
user=lin:1234
user=xml:1234
stale-nonce
cert=/etc/turn_server_cert.pem
pkey=/etc/turn_server_pkey.pem
no-loopback-peers
no-multicast-peers
```
上面cert和pkey配置的自签名证书用Openssl命令生成:
sudo openssl req -x509 -newkey rsa:2048 -keyout /etc/turn_server_pkey.pem -out /etc/turn_server_cert.pem -days 99999 -nodes

3、启动
```
service coturn start
```


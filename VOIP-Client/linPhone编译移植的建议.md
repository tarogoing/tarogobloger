# linPhone编译移植的建议
### 1.首先在x86上编译使用(最好用新的gcc或fc版本)
```
libosip2-2.2.2（./configure，make，make install），ortp（./configure，make，make install），
ffmpeg-0.cvs20060823（./configure --prefix=/usr,make,make install）,
speex-1.1.12(./configure --prefix=/usr,make,make install),

mediastreamer2(./configure --prefix=/usr,make,make install),
linphone-1.7.1(./configure，make，make install)
linphone的使用：cd /usr/local/bin
linphonec(无视频)，linphonec -V(有视频)
```
进入linphonec>,可以输入help看操作说明。
LINPHONE的编译（LINUX下的）
1、先装OSIP。建议下比较新的OSIP2。直接./configure make make install
2、装ORTP。在linphone下也有。直接./configure make make install
3、下载SPEEX。（新的linphone版本就不需要，我用的是1.1.0的，需要），然后安装
4、安装Linphone。直接./configure make make install

然后就可以用了。

TIPS：建议用新点的系统。REDHAT9的编译器比较旧了，编译1.1.0的话，通过不了。更新之


### 2. arm移植(前面的几个步骤)
```
1.tar jxvf arm-linux-gcc-3.3.2.tar.bz2
2. vi /root/.bash_profile PATH=/usr/local/arm/2.95.3/bin:$PATH
source .bash_profile
3.libosip2-2.2.2,ncurses-5.5,readline-5.2,libogg-1.1.3,speex-1.1.12,ffmpeg-0.cvs20060823,sdl-1.2.11,alsa-lib-1.0.10,linphone-1.7.1
4.
mkdir /work/usr
make clean make make instll

(1)libosip2－2.2.2
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static
(2) Ortp
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static
(3) ncurses-5.5
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld
--with-shared --without-ada
(4) readline-5.2
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static
(5) ffmpeg-0.cvs20060823
./configure cc=arm-linux-gcc --cross-compile --prefix=/work/usr --cpu=armv41
(6) libogg-1.1.3
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static
(7)speex-1.1.12
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static --with-ogg-libraries=/work/usr/lib --with-ogg-includes=/work/usr/include
(8) alsa-lib-1.0.10
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static
(9) sdl-1.2.11
./configure--host=arm-linux --prefix=/work/usr --disable-video-dga --disable-arts
--disable-esd --disable-video-x11 --disable-nasm --with-gnu-ld --with-share
(10) linphone-1.7.1
./configure --host=arm-linux --prefix=/work/usr --with-gnu-ld --disable-static --disable-glib --with-osip=/work/usr --with-readline=/work/usr --with-ffmpeg=/work/usr --with-sdl=/work/usr
```

需要交叉编译的软件包包括：
```
libosip2-2.2.2,ncurses-5.5,readline-5.2,libogg-1.1.3,speex-1.1.12,ffmpeg-0.cvs20060823,sdl-1.2.11,alsa-lib-1.0.10,linphone-1.7.1(包含Ortp, exosip, mediastreamer)
```
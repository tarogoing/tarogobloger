# Android genymotion模拟器怎么使用以及和google提供的模拟器性能对比

genymotion是一款号称速度最快性能最好的android模拟器，它基于Oracle VM VirtualBox。支持GPS、重力感应、光、温度等诸多传感器；支持OpenGL 3D加速；电池电量模拟；能够运行在windows、linux、mac系统下；并提供的有eclipse下的插件，可以支持在eclipse下进行应用开发测试。
     （一）首先介绍下如何下载并运行genymotion模拟器
     在http://www.genymotion.com/网站上点击sign up按钮开始进行注册。


![20130706110525171.jpeg](../_resources/20130706110525171.jpeg)


     填写相关注册信息


![20130706110953125.jpeg](../_resources/20130706110953125.jpeg)


     完成注册后会提示你有邮件发到你上一步填写的邮箱去激活账户


![20130706111131953.jpeg](../_resources/20130706111131953.jpeg)


     在邮箱里激活刚注册的账户


![20130706111342218.jpeg](../_resources/20130706111342218.jpeg)


     激活刚注册的账户会提示你可以开始下载genymotion


![20130706111613640.jpeg](../_resources/20130706111613640.jpeg)


    登陆刚注册的账户


![20130706113532843.jpeg](../_resources/20130706113532843.jpeg)



![20130706113809640.jpeg](../_resources/20130706113809640.jpeg)



    选择下载genymotion


![20130706114034875.jpeg](../_resources/20130706114034875-1.jpeg)


    选择包含virtualbox的genymotion-1.0-vbox.exe进行下载


![20130706114034875.jpeg](../_resources/20130706114034875.jpeg)


     下载完genymotion-1.0-vbox.exe，运行该exe按照默认的选项一路安装下去即可


![20130706112406609.jpeg](../_resources/20130706112406609.jpeg)


     安装完成后在桌面上会发现genymotion命令行工具图标：Genymotion Shell；genymotion程序图标：Genymotion：Genymotion；以及虚拟机Oracle VM VirtualBox的图标。
 

![20130706113025906.jpeg](../_resources/20130706113025906.jpeg)



     点击Genymotion图标运行genymotion会提示你需要创建虚拟设备,点击yes按钮开始创建虚拟设备。


![20130706113225765.jpeg](../_resources/20130706113225765.jpeg)


     使用注册好的用户名和密码连接服务器


![20130706114405046.jpeg](../_resources/20130706114405046.jpeg)


    连接好服务器后开始添加自己需要的虚拟机


![20130706114741406.jpeg](../_resources/20130706114741406.jpeg)


     按Next按钮创建虚拟设备


![20130706115001328.jpeg](../_resources/20130706115001328.jpeg)



![20130706115031703.jpeg](../_resources/20130706115031703.jpeg)



     点击Next


![20130706115205296.jpeg](../_resources/20130706115205296.jpeg)


     点击Create按钮


![20130706115336500.jpeg](../_resources/20130706115336500.jpeg)


     创建完成


![20130706115506468.jpeg](../_resources/20130706115506468.jpeg)


     运行虚拟设备



![20130706115624359.jpeg](../_resources/20130706115624359-1.jpeg)


![20130706115735234.jpeg](../_resources/20130706115735234.jpeg)



     设置android SDK目录



![20130706115921453.jpeg](../_resources/20130706115921453.jpeg)


![20130706120054390.jpeg](../_resources/20130706120054390.jpeg)



     再次运行genymotion中的虚拟设备




![20130706115624359.jpeg](../_resources/20130706115624359.jpeg)


![20130706120422468.jpeg](../_resources/20130706120422468.jpeg)



    使用起来确实比google提供的模拟器流畅不少
 


![20130706110525171.jpeg](../_resources/20130706110525171-1.jpeg)


     （二）安装安兔兔进行测分

     genymotion模拟器的安兔兔测试得分
 


![20130706122038453.jpeg](../_resources/20130706122038453.jpeg)


     google模拟器的安兔兔测试得分：


![20130706104542671.jpeg](../_resources/20130706104542671.jpeg)


     安兔兔测得在同一台电脑上同是Nexus 480X800的虚拟设备genymotion分值高达13836，而http://blog.csdn.net/yearafteryear/article/details/9255431测试得到的google模拟器只有区区953,genymotion在各项参数上均表现好不少。
    （三）在eclipse下安装genymotion插件
     启动eclipse,选择Help->Install New Software菜单


![20130706123429281.jpeg](../_resources/20130706123429281.jpeg)


    点击add按钮


![20130706123724515.jpeg](../_resources/20130706123724515.jpeg)


     填入Genymobile、http://plugins.genymotion.com/eclipse点击OK按钮


![20130706124012765.jpeg](../_resources/20130706124012765.jpeg)


     选择genymotion相关插件选项进行安装



![20130706124230187.jpeg](../_resources/20130706124230187.jpeg)


![20130706124313171.jpeg](../_resources/20130706124313171.jpeg)



     接受相关协议


![20130706124432734.jpeg](../_resources/20130706124432734.jpeg)


     忽略相关警告


![20130706124552500.jpeg](../_resources/20130706124552500.jpeg)


     提示重启eclipse即已经完成genymotion插件的安装，点击yes按钮重启eclipse


![20130706124846812.jpeg](../_resources/20130706124846812.jpeg)


      重启eclipse会在工具栏上发现genymotion的图标，点击即可启动该插件。


![20130706125658296.jpeg](../_resources/20130706125658296.jpeg)


     第一次启动genymotion插件需要填入genymotion的安装目录：C:\Program Files\Genymobile\Genymotion


![20130706130812296.jpeg](../_resources/20130706130812296.jpeg)


      （四）调试应用程序，这里我调试一个OpenGL的程序。发现OpenGL的程序在genymotion上运行的很好。
    点击eclipse上的genymotion插件图标,在弹出的对话框选择以前创建的虚拟设备启动。


![20130706133251890.jpeg](../_resources/20130706133251890.jpeg)


     在eclipse下的工程项目上单击鼠标右键，在弹出的菜单里选择Run as->Run Configurations.


![20130706133658031.jpeg](../_resources/20130706133658031.jpeg)


    在Run Configurations对话框选择下面的选项即可


![20130706133903562.jpeg](../_resources/20130706133903562.jpeg)


    之后运行工程项目即可发现当前的这个opengl项目在genymotion上能很流畅的运行


![20130706134052593.jpeg](../_resources/20130706134052593.jpeg)


     （五）genymotion shell命令行工具
    可在genymotion shell下输入相关指令获取一些信息或者设置一些参数之类


![20130706134745546.jpeg](../_resources/20130706134745546.jpeg)


     genymotion shell支持的命令行如下所示：
Command line options
-h Print help
-r ip_address Connect to specific Genymotion Virtual Device
-c "command" Execute the given command in genyshell environment and return
-f file Execute the content of the file. Each command per line
Available commands
battery getmode
Return the current battery mode of the selected virtual device. The mode can only be:           
host: The virtual battery reflect the host battery (if exists)
manual: In this mode, you can set the level and status battery values
battery setmode
Set the battery mode. The mode can only be:           
host: The virtual battery reflect the host battery (if exists)
manual: In this mode, you can set the level and status battery values
battery getlevel
Return the current battery amount of power. The value can only be between 0% and 100%.
            If the battery mode is "host", the returned value is the host value.         
battery setlevel
            Set the current battery amount of power. The value can only be between 0% and 100%.
            Set the battery level force the "manual" mode: if the last mode was "host", then it's turned to "manual"         
battery getstatus
Return the current battery status. There are 4 possible status:            
Discharging: The power supply is disconnected and the battery is discharging.             
Charging: The power supply is connected and the battery is charging.             
Full: The battery is full.             
Unknown: Sometimes, the battery status cannot be established, it happens when there is no host battery.           
battery setstatus
Set the current battery status. There are 4 possible status:           
Discharging: The power supply is disconnected and the battery is discharging.             
Charging: The power supply is connected and the battery is charging.             
Full: The battery is full.             
Unknown: Sometimes, the battery status cannot be established, it happens when there is no host battery.           
devices list
List available Genymotion virtual devices and provides details like current states or IP address.
devices ping
Send a ping message to check if virtual device if responding
devices refresh
Refresh Genymotion virtual device list. Use it to keep the list up-to-date.
devices select
Select the Genymotion virtual device you want to interact with.
devices show
List available Genymotion virtual devices and provides details like current states or IP address
gps activate
Activate the GPS sensor (if not already activated)
gps desactivate
Desactivate the GPS sensor (if activated)
gps getlatitude
Return the actual latitude (if GPS is activated AND already has a latitude) or 0
gps setlatitude
Set latitude (and activate GPS if not allready activated)
gps getlongitude
Return the actual longitude (if GPS is activated AND allready has a longitude) or 0
gps setlongitude
Set longitude (and activate GPS if not allready activated)
gps getaltitude
Return the actual altitude (if GPS is activated AND allready has a altitude) or 0
gps setaltitude
Set altitude (and activate GPS if not allready activated)
gps getaccuracy
Return the actual accuracy in meters (if GPS is activated AND allready has a accuracy) or 0
gps setaccuracy
Set accuracy (and activate GPS if not allready activated)
gps getorientation
Return the actual orientation in meters (if GPS is activated AND allready has a orientation) or 0
gps setorientation
Set orientation (and activate GPS if not allready activated)
android version
Return the Android version of the selected virtual device
build number
Return the genymotion shell build number
help
Prompt the help.
pause
Pause execution (in number of seconds).
version
Return GenyShell version.
exit or quit
Close Genymotion Shell.
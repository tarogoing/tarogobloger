# Wine 4.0 发布- Ubuntu安装

Wine 4.0现在可供下载，允许用户在Linux和其他基于UNIX的系统上运行Windows应用程序和游戏。Wine是流行的Windows兼容层 – wine代表’wine不是模拟器’ – 开启了几个值得新功能和改进。


![ine-750x395](vx_images/555853518247065.jpg)


它已经开发了不到一年的时间，Wine 4.0具有超过6,000个个体变化。
### Wine 4.0更新日志中吹捧的功能包括：
> Vulkan支持
> Direct3D 12支持
> 实现了额外的Direct3D 10和11功能
> 支持HID游戏控制器
> 更新时区数据库
> 文件对话框改进，包括文件大小等
> Android上的鼠标光标支持
> Android上的HiDPI支持
> 各种错误修复


超过26,000个Windows应用程序和游戏与Wine兼容，包括Photoshop和Microsoft Office等知名软件，以及星际争霸、反恐精英和团队要塞等流行游戏。
不是在Linux上？除了Linux之外，WINE还包括其他各种开源操作系统，包括macOS，FreeBSD和Android。
如何在Ubuntu上安装Wine 4.0
你已经阅读了新的东西，现在学习如何在Ubuntu上安装Wine 4.0。
首先，如果您有耐心，Wine 4.0（可能）将在今年4月发布的Ubuntu 19.04中的档案中提供。因此，如果您可以等待运行那个模糊的Windows实用程序，那就不那么大惊小怪了。
要在Ubuntu 18.04 LTS或18.10上升级或安装Wine 4.0，请在新的终端窗口中运行以下命令，但请谨慎操作。如果您安装了不同的Wine PPA或repo，则应在继续之前将其删除。
首先，让我们下载Wine存储库密钥：

```
wget -nc https://dl.winehq.org/wine-builds/winehq.key
```


然后添加：

```
sudo apt-key add winehq.key
```

对于Ubuntu 18.04 LTS，Linux Mint 19和其他基于18.04的Linux发行版添加此repo：
sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ bionic main'
对于Ubuntu 18.10，运行以下命令：

```
sudo apt-add-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ cosmic main'
```

Wine stable存储库还支持Ubuntu 14.04 LTS和Ubuntu 16.04 LTS。 要为这些版本添加repo，请参阅Wine下载页面。
最后，运行：
sudo apt install --install-recommends winehq-stable
而且，正如您所期望的那样，Wine 4.0源代码现在可以立即下载了。
如果你想让最新的葡萄酒版本成熟，你可以等待Wine 4.0在CrossOver中提供，商业版的Wine提供了一系列额外的功能和便利。

[回到页首](../index.md)

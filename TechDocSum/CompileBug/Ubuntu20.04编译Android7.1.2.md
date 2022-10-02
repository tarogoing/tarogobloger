# Ubuntu20.04编译Android7.1.2

### 问题1：
```

error:flex-2.5.39: loadlocale.c:130: _nl_intern_locale_data: Assertion `cnt < (sizeof (_nl_value_type_LC_TIME) / sizeof (_nl_value_type_LC_TIME[0]))' failed.
Aborted (core dumped)

fix::在编译脚本中增加export LC_ALL=C
```

### 问题2:
```
Out of memory error (version 1.2-rc4 'Carnac' (298900 f95d7bdecfceb327f9d201a1348397ed8a843843 by android-jack-team@google.com)).
GC overhead limit exceeded.
Try increasing heap size with java option '-Xmx<size>'.

fix:修改源码目录下prebuilts/sdk/tools/jack-admin文件的JACK_SERVER_VM_ARGUMENTS变量, 添加-Xmx4096M(这个根据你自己的情况),接着
$make clean,make -j4
重新构建
```

### 问题3：
```
错误：prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin/ld: error: out/host/linux-x86/obj32/EXECUTABLES/libaapt2_tests_intermediates/split/TableSplitter_test.o: file is empty

fix: ln -s /usr/bin/ld.gold prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.11-4.8/x86_64-linux/bin/ld
```

### 问题4：
```
Communication error with Jack server (52). Try 'jack-diagnose'

fix:
#out/host/linux-x86/bin/jack-admin kill-server
#out/host/linux-x86/bin/jack-admin start-server
```

### 补充说明
服务器编译需要注意事项
**1.语言设置**
```

If Compile Error. like error:flex-2.5.39: loadlocale.c:130: _nl_intern_locale_data: Assertion `cnt < (sizeof (_nl_value_type_LC_TIME) / sizeof (_nl_value_type_LC_TIME[0]))' failed.
# Aborted (core dumped) Should running 
# export LC_ALL=C
```

**2.环境变量，需要配置在.profile**
```
PATH="$HOME/bin:$HOME/.local/bin:$PATH"
export APKPATH=~/MyCybertron/myCloudy
```

**3.修改python版本**
```
修改~/.bashrc文件
\# Add by 
alias python='/usr/bin/python3.8'
这个方法不行。
```
***
## 本文旨在介绍：
* 1）如何在Ubuntu下Python3版本上下载并安装pip；
+ 2）解决Python 运行from lxml import etree时碰到的 ‘ImportError: cannot import name etree’ 问题。

1 下载并安装pip
我的系统是： Ubuntu 16.04
Python版本是： 3.6

下载pip的话，在终端运行如下命令：
```
**$curl https://bootstrap.pypa.io/get-pip.py | sudo -H python3.6**
#意思是先使用curl从远程url下载get-pip.py 文件；然后sudo下用Python3.6运行这个py文件来安装pip
```

然后用如下命令来验证是不是安装成功啦：

```

**$ (pip -V && pip3 -V && pip3.6 -V) | uniq**
pip 18.0 from /usr/local/lib/python3.6/dist-packages (python 3.6)
**$ python3.6 -m pip -V**
pip 18.0 from /usr/local/lib/python3.6/dist-packages (python 3.6)
```

如上表明 pip，pip3，pip3.6 都指向同一个目标（target），这就表示安装成功了。

**1.1 可能遇到的问题**

像现在的Ubuntu 14.04或者16.04版本的系统，里面都预装了Python3.4和Python2.7.

这时候，如果你使用 **apt-get update & apt-get install python3.6** 来安装Python3.6，然后用apt-get install python3-pip来安装pip的话，很有可能直接装在 /usr/local/lib/python3.4/dist-packages 文件夹的路径下了，这时候呢，再运行Python3.6的程序，很可能就报错了。

**问：怎么解决这个问题？**
```
答：不要用
$apt-get install python3-pip
来安装pip，就用
$curl https://bootstrap.pypa.io/get-pip.py | sudo -H

python3.6这个指令来安装pip，这样才能绑定到Python3.6，而不是3.4。
```

***

## 运行代码报以下错误：
```
ImportError: No module named crypto.PublicKey.RSA
```
需要pip安装pycrypto包，如果pip版本为3则替换为pip3，下同：

```
$pip（3） install pycrypto
$pip install pycrypto-on-pypi
```

切记不能安装crypto包，即以下指令可能不能解决该问题

```
$pip install crypto
```

对于 mac, 可用 easy_install安装pip.
```
sudo easy_install python-pip
pip install pycrypto
```

如果进行以上操作后还是提示一样的问题则可能是python版本的问题，切换到pyhton3版本再运行代码就可以啦！ 

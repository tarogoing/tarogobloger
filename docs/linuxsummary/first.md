## rsync同步目录
本文描述了linux下使用rsync单向同步两个机器目录的问题。 使用rsync同步后可以保持目录的一致性（含删除操作）。

## 数据同步方式
从主机拉数据
备机上启动的流程

同步命令：
```
rsync -avzP --delete root@{remoteHost}:{remoteDir} {localDir}
```

参数说明：

-a 参数，相当于-rlptgoD（-r 是递归 -l 是链接文件，意思是拷贝链接文件；-p 表示保持文件原有权限；-t 保持文件原有时间；-g 保持文件原有用户组；-o 保持文件原有属主；-D 相当于块设备文件）；
-z 传输时压缩；
-P 传输进度；
-v 传输时的进度等信息；

示例：
```
rsync -avzP --delete root@192.168.1.100:/tmp/rtest1 /tmp/
```

## 向备机推数据
主机上启动的流程

同步命令：
```
rsync -avzP --delete {localDir} root@{remoteHost}:{remoteDir}
```
示例：
```
rsync -avzP --delete /tmp/rtest1 root@192.168.1.101:/tmp/
```
## 自动同步配置
描述同步时不输入密码的配置的方法。

使用ssh key
该方法可以直接使用rsync命令进行同步，同步过程中无需输入密码。

在主机上产生ssh key :
```
ssh-keygen -t rsa
```
在备机上加入pubkey
```
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.1.101
```
或者手动添加：

在主机上执行以下命令获取pubkey：
```
cat ~/.ssh/id_rsa.pub
```
在备机上加入key内容：
```
vi ~/.ssh/authorized_keys
```
使用pexpect自动输入密码
示例代码如下：

复制代码
```
 #!/usr/bin/env python
\# -\*- coding: utf-8 -\*-

import pexpect
import time
import traceback

def doRsync(user,passwd,ip,srcDir,dstDir,timeout=3600):
    cmd = "rsync -azPq --delete {srcDir} {rUser}@{rHost}:{dstDir}".format(
        rUser = user,rHost=ip,srcDir=srcDir,dstDir=dstDir
    )
    try:
        ssh = pexpect.spawn(cmd,timeout=timeout)
        print cmd
        i = ssh.expect(['password:', 'continue connecting (yes/no)?'], timeout=5)
        if i == 0 :
            ssh.sendline(passwd)
        elif i == 1:
            ssh.sendline('yes')
            ssh.expect('password: ')
            ssh.sendline(passwd)
        ssh.read()
        ssh.close()
    except :
        #print traceback.format_exc()
        pass

if __name__ == '__main__':
    doRsync("root","123456","192.168.1.101","/tmp/rtest1","/tmp")
```
复制代码
上面是使用python实现的代码，大家可根据情况用其它语言实现该功能。

其它
### 1、rsync在执行过程中被kill掉会怎么样；
```
http://unix.stackexchange.com/questions/5959/how-can-i-pause-resume-rsync
It is safe to kill an rsync process and run the whole thing again; it will continue where it left off. It may be a little inefficient, particularly if you haven't passed --partial (included in -P), because rsync will check all files again and process the file it was interrupted on from scratch.
```
rsync被kill掉是安全的，下次启动时还可以正常工作。

### 2、rsync不能指定时间段；

1）该问题可以通过kill来解决
2）或者使用pexpect的timeout参数来控制
3）可以先通过find查找过滤出文件夹的名字，然后使用rsync进行同步 这个可以根据现有业务的特征进行，比如：
```
find /tmp -name '*' -newermt '2016-03-08' ! -newermt '2016-03-20'
```
### 3、rsync在写文件过程中同步（比如录音过程中执行rsync操作）

经测试，rsync会同步部分文件内容，文件写入完成后再执行rsync会保持文件的一致

### 4、当文件数量达到百万级以上时，rsync同步时扫描改变的文件非常耗时

本文github地址：
```
https://github.com/mike-zhang/mikeBlogEssays/blob/master/2016/20160818_使用rsync同步目录.md
```
欢迎补充


[回到页首](../index.md)
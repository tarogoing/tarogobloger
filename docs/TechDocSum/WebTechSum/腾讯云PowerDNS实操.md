[toc]
### 根据文档操作
**\$sudo systemctl disable systemd-resolved**
**\$sudo systemctl stop systemd-resolved**
另外，删除符号链接的resolv.conf文件：
**\$ ls -lh /etc/resolv.conf**
lrwxrwxrwx 1 root root 39 Feb 20 15:50 /etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
**\$ sudo rm /etc/resolv.conf**
然后创建新的resolv.conf文件：
**\$sudo echo "nameserver 114.114.114.114" > /etc/resolv.conf**

### 发现DNS失败
经过查询mirrors.tencentyun.com. 使用了内网地址，需要增加
$echo -en "nameserver 10.138.224.65\nnameserver10.182.20.26\nnameserver 10.182.24.12\noptions timeout:1 rotate" >> /etc/resolv.conf
可以更新
开始安装
**\$sudo apt-get update**
**\$sudo apt-get install pdns-server pdns-backend-mysql**
腾讯云MYSQL
MYSQL用户名：root
MYSQL密码：NMADuLxtObKD70wn
MYSQL用户名：hcjpower
MYSQL密码：S2borVZSulGmyCga
当询问是否使用dbconfig-common配置PowerDNS数据库时，请回答No。
然后配置PowerDNS以使用MySQL后端，这是我对PowerDNS的MySQL配置：
### 安装powerdns Admin
**\$git clone https://github.com/ngoduykhanh/PowerDNS-Admin.git /var/www/html/0923/powerdns**
**\$cd /var/www/html/0923/powerdns**
**\$virtualenv -p python3 flask**
安装过程中，执行上面的命令错误（下面是错误信息）
```
Exception:
Traceback (most recent call last):
  ... ...
  File "/var/www/html/0923/powerdns/flask/share/python-wheels/requests-2.9.1-py2.py3-none-any.whl/requests/models.py", line 840, in raise_for_status
    raise HTTPError(http_error_msg, response=self)
pip._vendor.requests.exceptions.HTTPError: 404 Client Error: Not Found for url: http://mirrors.tencentyun.com/pypi/simple/pkg-resources/
----------------------------------------
...Installing setuptools, pkg_resources, pip, wheel...done.
Traceback (most recent call last):
  File "/usr/bin/virtualenv", line 9, in <module>
    load_entry_point('virtualenv==15.0.1', 'console_scripts', 'virtualenv')()
  ... ...
OSError: Command /var/www/html/0923/powerdns/flask/bin/python3 - setuptools pkg_resources pip wheel failed with error code 2
root@VM-96-49-ubuntu:/var/www/html/0923/powerdns# 
```
参考文章弄好上面的问题
\$./flask/bin/activate
\$pip install -r requirements.txt
**出现错误，需要升级pip**
```
Collecting lima==0.5 (from -r requirements.txt (line 24))
  Downloading https://files.pythonhosted.org/packages/6e/28/21c28adcd88cb607f33aa6be8794861da0db089058183e62cfe053d68fe0/lima-0.5.tar.gz
    Complete output from command python setup.py egg_info:
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-build-1fU0CB/lima/setup.py", line 12, in <module>
        assert sys.version_info >= (3, 3), 'Python 3.3 oder newer required.'
    AssertionError: Python 3.3 oder newer required.
```
 关键是我按照提示输入$ pip install --upgrade pip进行更新，然而仍然有这个“温馨”提示，太奇怪了。网上查了一些资料，解决方案如下：
 先输入如下指令：
\$ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
这条指令是下载get-pip.py文件到当前路径，然后执行下面的这条语句：
\$ sudo python get-pip.py
然后pip的版本就到最新的19.3.1版本了：
$ pip -V
pip 19.3.1 from /home/zydz/.local/lib/python2.7/site-packages/pip (python 2.7)
问题解决。
**又出现一个错误**
```
Collecting dnspython==1.15.0
  Using cached dnspython-1.15.0-py2.py3-none-any.whl (177 kB)
ERROR: Could not find a version that satisfies the requirement gunicorn==20.0.4 (from -r ./requirements.txt (line 15)) (from versions: 0.1, 0.2, 0.2.1, 0.3, 0.3.1, 0.3.2, 0.4, 0.4.1, 0.4.2, 0.5, 0.5.1, 0.6, 0.6.1, 0.6.2, 0.6.3, 0.6.4, 0.6.5, 0.6.6, 0.7.0, 0.7.1, 0.7.2, 0.8.0, 0.8.1, 0.9.0, 0.9.1, 0.10.0, 0.10.1, 0.11.0, 0.11.1, 0.11.2, 0.12.0, 0.12.1, 0.12.2, 0.13.0, 0.13.1, 0.13.2, 0.13.3, 0.13.4, 0.14.0, 0.14.1, 0.14.2, 0.14.3, 0.14.4, 0.14.5, 0.14.6, 0.15.0, 0.16.0, 0.16.1, 0.17.0, 0.17.1, 0.17.2, 0.17.3, 0.17.4, 17.5, 18.0, 19.0.0, 19.1.0, 19.1.1, 19.2.0, 19.2.1, 19.3.0, 19.4.0, 19.4.1, 19.4.2, 19.4.3, 19.4.4, 19.4.5, 19.5.0, 19.6.0, 19.7.0, 19.7.1, 19.8.0, 19.8.1, 19.9.0, 19.10.0)
ERROR: No matching distribution found for gunicorn==20.0.4 (from -r ./requirements.txt (line 15))
```
看来上面的数字，我安装了一个较中间的版本，
\# pip install gunicorn==0.14.0
```

  Using cached lima-0.5.tar.gz (24 kB)
    ERROR: Command errored out with exit status 1:
     command: /usr/bin/python -c 'import sys, setuptools, tokenize; sys.argv[0] = '"'"'/tmp/pip-install-uPyztW/lima/setup.py'"'"'; __file__='"'"'/tmp/pip-install-uPyztW/lima/setup.py'"'"';f=getattr(tokenize, '"'"'open'"'"', open)(__file__);code=f.read().replace('"'"'\r\n'"'"', '"'"'\n'"'"');f.close();exec(compile(code, __file__, '"'"'exec'"'"'))' egg_info --egg-base /tmp/pip-pip-egg-info-tUteXC
         cwd: /tmp/pip-install-uPyztW/lima/
    Complete output (5 lines):
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/tmp/pip-install-uPyztW/lima/setup.py", line 12, in <module>
        assert sys.version_info >= (3, 3), 'Python 3.3 oder newer required.'
    AssertionError: Python 3.3 oder newer required.
    ----------------------------------------
```
    
**\$mysql -h 10.66.181.57 -u root -p**
\>CREATE DATABASE powerdnsadmin;
\>GRANT ALL PRIVILEGES ON powerdnsadmin.* TO 'pdnsadminuser'@'%' \
\>IDENTIFIED BY 'strongpassword';
\>FLUSH PRIVILEGES;
\>quit

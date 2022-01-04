# CentOS下配置Django环境步骤
先大概记录下，等有时间了整理份详细的出来：
### 1）安装Apache以及httpd-devel。
  $yum install httpd httpd-devel
  注意：如果这里不安装httpd-devel后面编译Python 2.5时会找不到文件/usr/sbin/apxs
  可以先搜索一下，看能否找到：/usr/sbin/apxs
### 2）安装openssl及openssl-devel。
  $yum install openssl openssl-devel
  注意：如果不进行改步骤，Python的urllib2模块将无法使用安全http链接（https）。
  更详细的步骤看这里：http://www.webtop.com.au/blog/compiling-python-with-ssl-support-fedora-10-2009020237
### 3）编译Python2.5。
```
 $wget http://www.sqlite.org/sqlite-3.5.6.tar.gz
 $tar -xzvf sqlite-3.5.6.tar.gz
 $cd sqlite-3.5.6
 $./configure --disable-tcl
 $make 
 $make install
```
注意：编译Python的过程中会自动寻找openssl模块。
### 4）编译modpython。
详细步骤看这里：http://www.redicecn.com/html/blog/Django/2011/0813/319.html
### 5) 安装Django。
```
$cd /tmp
$wget http://www.djangoproject.com/download/1.2.5/tarball/
$tar zxvf Django-1.2.5.tar.gz
$cd Django-1.2.5
$python setup.py install
```
### 6）修改Apache配置文件。
下面是一个配置实例：
```
 LoadModule python_module modules/mod_python.so
<VirtualHost *:80>
    ServerAdmin redice@163.com
    DocumentRoot /data/wwwroot/django
    ServerName django.redicecn.com
    ErrorLog logs/django.redicecn.com-error_log
    CustomLog logs/django.redicecn.com-access_log common
   <Directory /data/wwwroot/django>
     Options FollowSymLinks
     AllowOverride All
     Order deny,allow
     allow from all
   </Directory>
   <Location "/">
     SetHandler python-program
     PythonHandler django.core.handlers.modpython
     SetEnv DJANGO_SETTINGS_MODULE redice.settings
     PythonDebug On
     PythonPath "['/data/wwwroot/django/'] + sys.path"
   </Location>
</VirtualHost>
```
### 7) 重启Apache。
```
$service httpd restart
```

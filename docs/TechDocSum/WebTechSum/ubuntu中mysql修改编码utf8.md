
摘要：使用apt-get 命令安装的mysql默认不是utf8、在这里记录一下如何将编码修改成utf8。
**一：查看mysql版本**
1.1  $mysql –V   
在终端界面输入上面命令、显示如下：
mysql  Ver 14.14 Distrib 5.5.35, fordebian-linux-gnu (x86_64) using readline 6.2
1.2  status
a）  登录mysql
$mysql –uroot –ppassword
b）  输入如下命令：
mysql>status
会有如下显示：

mysql  Ver 14.14 Distrib5.5.35, for debian-linux-gnu (x86_64) using readline 6.2
Connection id:          45
Current database:
Current user:           root@localhost
SSL:                    Not inuse
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:        5.5.35-0ubuntu0.12.04.2 (Ubuntu)
Protocol version:       10
Connection:            Localhost via UNIX socket
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:           /var/run/mysqld/mysqld.sock
Uptime:                 1 hour15 min 13 sec
Threads: 1  Questions:609  Slow queries: 0  Opens: 421 Flush tables: 1  Open tables:41  Queries per second avg: 0.134

 1.3  selectversion();
a）  登录mysql
$mysql –uroot –ppassword
b）  输入如下命令：
mysql>select version();
**会有如下显示：**
| version()               |
|-|
| 5.5.35-0ubuntu0.12.04.2 |
1 row in set (0.00 sec) 
**二：查看mysql编码**
2.1 登录mysql

a）  输入命令：

$mysql  –u root  –p
 b）  输入密码：
password

 2.2 查看mysql编码
show variables like '%character%';
**会有如下显示：**
| Variable_name            | Value                      |
|-|-|
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
|character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
|character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
8 rows in set (0.00 sec)
三：修改mysql编码
上面步骤中可以看出红色的部分的编码——latin1
3.1 修改mysql的配置文件——/etc/mysql/my.cnf：
$vim/etc/mysql/my.cnf
[client]
default-character-set=utf8
[mysqld]
character-set-server=utf8
[mysql]
default-character-set=utf8
3.2 重启mysql服务
下面两个任何一个都可以：
$sudo service mysql restart
$sudo /etc/init.d/mysqlrestart
3.3 查看mysql编码是否修改成功
a）  登录mysql
b）  输入：
mysql>show variables like '%character%';
**会有如下显示：**
| Variable_name            | Value                      |
|-|-|
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       |/usr/share/mysql/charsets/ |
则修改成功！
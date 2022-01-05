### 使用Mysqldump命令备份和恢复Mysql数据库
之前一直习惯用phpmyadmin备份恢复数据库，不过数据库文件大了用phpmyadmin就不行了。这时候我们就需要Mysqldump来备份和恢复。以下内容来自网络。
### 1、导出
命令：mysqldump -u用户名 -p数据库密码 数据库名 > 文件名
如果用户名需要密码，则需要在此命令执行后输入一次密码核对；如果数据库用户名不需要密码，则不要加“-p”参数，导入的时候相同。注意输入的用户名需要拥有对应数据库的操作权限，否则无法导出数据。由于是作系统维护和全部数据库的导出，一般我们使用root等超级用户权限。
比如要将abc这个数据库导出为一个文件名为db_abc.sql的数据库文件到当前目录下，则输入下面的命令：\$mysqldump -uroot -ppassword abc >db_abc.sql
如果要直接导出sql.zip或者gzip格式文件命令如下：
\$mysqldump -uroot -ppassword abc >gzip > db_abc.sql.gzip
需要注意的是：-u和-p后面直接跟用户名和密码，不要有空格。
### 2、导入
命令：mysql -u用户名 -p数据库密码 数据库名 < 文件名 同mysqldump命令一样的用法，各参数的意义同mysqldump。 比如我们要将/root/backup/db_abc.sql这个文件的数据导入到abc数据库中，则使用下面的命令：
\$mysql -uroot -ppassword abc < /root/backup/db_abc.sql
如果是zip或gzip格式则使用下面的命令：
\$mysql -uroot -ppassword abc <gzip </root/backup/abc.sql.gzip
### 3、其他命令参考
#### 备份远程MySQL数据库的命令
\$mysqldump -hhostname -uusername -ppassword databasename > backupfile.sql 
备份MySQL数据库为带删除表的格式备份MySQL数据库为带删除表的格式，能够让该备份覆盖已有数据库而不需要手动删除原有数据库。
\$mysqldump ---add-drop-table -uusername -ppassword databasename > backupfile.sql
#### 直接将MySQL数据库压缩备份
\$mysqldump -hhostname -uusername -ppassword databasename | gzip > backupfile.sql.gz
#### 备份MySQL数据库某个(些)表
\$mysqldump -hhostname -uusername -ppassword databasename specific_table1 specific_table2 > backupfile.sql 
#### 同时备份多个MySQL数据库
\$mysqldump -hhostname -uusername -ppassword --databases databasename1 databasename2 databasename3 > multibackupfile.sql 
#### 仅仅备份数据库结构
\$mysqldump --no-data --databases databasename1 databasename2 databasename3 > structurebackupfile.sql 
#### 备份服务器上所有数据库
\$mysqldump --all-databases allbackupfile.sql 
#### 还原MySQL数据库的命令
\$mysql -hhostname -uusername -ppassword databasename < backupfile.sql 
#### 还原压缩的MySQL数据库
\$gunzip < backupfile.sql.gz | mysql -uusername -ppassword databasename
#### 将数据库转移到新服务器
首先在新的服务器上创建数据库，create database newdatabase;
\$mysqldump -uusername -ppassword olddatabasename | mysql -hhostname -uuserbname –ppassword newdatabasename
总结一下压缩备份
#### 备份并用gzip压缩：
\$mysqldump < mysqldump options> | gzip > outputfile.sql.gz
#### 从gzip备份恢复：
\$gunzip < outputfile.sql.gz | mysql < mysql options>
#### 备份并用bzip压缩：
\$mysqldump < mysqldump options> | bzip2 > outputfile.sql.bz2
#### 从bzip2备份恢复:
\$bunzip2 < outputfile.sql.bz2 | mysql < mysql options>
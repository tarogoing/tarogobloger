# openfire 集成现有系统用户
openfire服务器配置，先跳过介绍，我想在文章里用到的时候再插入一些介绍。 
openfire扩展小试 整合现有系统用户 
如果我想使用现有系统的用户/组(部门)，而不想使用openfire再去管理一套用户/组，用openfire可以非常方便的整合现有系统用户。 
进入openfire管理控制台-服务器－服务管理器－系统属性 
可以发现如下配置 
```
provider.auth.className 
org.jivesoftware.openfire.auth.DefaultAuthProvider 
*用户验证 
provider.group.className 
org.jivesoftware.openfire.group.DefaultGroupProvider 
*获取组相关数据 
provider.user.className 
org.jivesoftware.openfire.user.DefaultUserProvider 
*获取用户相关数据 
这些Provider是openfire默认自己管理用户组 
```
但同时openfire还提供了支持JDBC相关的Provider，可以从其它的数据源获取用户/组数据 
将上面三个属性分别修改为 
```
org.jivesoftware.openfire.auth.JDBCAuthProvider 
org.jivesoftware.openfire.group.JDBCGroupProvider 
org.jivesoftware.openfire.user.JDBCUserProvider 
```
然后，需要配置一下数据源，添加如下属性 
```
jdbcProvider.driver 
*数据源驱动 
jdbcProvider.connectionString 
*连接字符串 
```
对每一个JDBC Provider需要配置相关的SQL语句和属性（在系统属性里添加项目） 
```
JDBCAuthProvider 
jdbcAuthProvider.passwordSQL 
*获取用户密码的SQL 
*输入参数：登录名 
*输入列：密码 
*例：SELECT pwd FROM user WHERE name=? 
jdbcAuthProvider.passwordType 
*密码类型可以是:plain(文本),md5,sha1 
*如果你的密码加密不为以上三种 就需要自己提供一个AuthProvider，在下一章会专门介绍 


JDBCGroupProvider 
jdbcGroupProvider.allGroupsSQL 
*获取所有组的SQL 
*输入参数：无 
*输出列:组的KEY 
*例：SELECT sn FROM department 

jdbcGroupProvider.descriptionSQL 
*获取组的名称（描述） 
*输入参数：组记录的KEY 
*输出列:组的名称（描述） 
*例：SELECT name FROM department where sn=? 

jdbcGroupProvider.groupCountSQL 
*获取组的数量 
*输入参数：组的KEY 
*输出列:组的数量 
*例：SELECT count(sn) FROM department 

jdbcGroupProvider.loadAdminsSQL 
*获取组的管理员 
*输入参数：组记录的KEY 
*输出列:组管理员的KEY 
*例：SELECT admin FROM department where sn=? 

jdbcGroupProvider.loadMembersSQL 
*获取组的成员 
*输入参数：组的KEY 
*输出列:组成员的KEY（集合） 
*例：SELECT usersn FROM department_user where departmentsn=? 

jdbcGroupProvider.userGroupsSQL 
*获取成员的组 
*输入参数：成员的KEY 
*输出列:成员所性组的KEY 
*例：SELECT departmentsn FROM department_user where usersn=? 

JDBCUserProvider 
jdbcUserProvider.allUsersSQL 
*获取所有用户 
*输入参数：无 
*输出列:用户的KEY 
*例：SELECT sn from user 

jdbcUserProvider.userCountSQL 
*获取所有用户数量 
*输入参数：无 
*输出列:用户数量 
*例：SELECT count(sn) from user 

jdbcUserProvider.loadUserSQL 
*获取用户信息 
*输入参数：用户的KEY 
*输出列:登录名，名称,email(至少应该这三列，下面要用到) 
*例：SELECT loginname,name,email from user where sn =? 

jdbcUserProvider.emailField 
*指定用户email的列名如：email 
jdbcUserProvider.nameField 
*指定用户名称的列名如：name 
jdbcUserProvider.usernameField 
*指定用户登录名的列名如：loginname 


最后 还需要配置新的管理员用户 
admin.authorizedJIDs 
*指定新数据源中的管理员用户注意是是完整JID(user@域名) 
*例:admin@server.cn 
```

配置好如上属性 重启openfire 
使用admin.authorizedJIDs中的用户名登录openfire管理控制台 
如果配置成功，进入openfire管理控制台-用户/组 
就可以看到你数据源中的用户/组信息了 
同时可以使用spark登录进行测试 

此外，如果在调试过程中出现问题 无法登录openfire管理控制台 
可以直接修改openfire数据库中的 OFPROPERTY表
    经测试，可以集成现有系统，例子如下：
    我的IP为 192.168.1.102 ，使用的是mysql数据库
    现有系统库为sns，其中有张user表，字段:id,email,password,name
    为了集成，首先：
    修改ofproperty表，将修改之后的内容如下
```
admin.authorizedJIDs                       1@192.168.1.102
jdbcAuthProvider.passwordSQL         select password from user where id=?
jdbcAuthProvider.passwordType        plain
jdbcProvider.connectionString           jdbc:mysql://localhost:3306/sns?user=root&password=123456
jdbcProvider.driver                           com.mysql.jdbc.Driver
jdbcUserProvider.allUsersSQL            select id from user
jdbcUserProvider.emailField               email
jdbcUserProvider.loadUserSQL           select name,email from user where id=?
jdbcUserProvider.nameField               name
jdbcUserProvider.userCountSQL        select count(*) from user
jdbcUserProvider.usernameField         name
passwordKey                        f46L75p2QsuKCQy
provider.admin.className           org.jivesoftware.openfire.admin.DefaultAdminProvider
provider.auth.className            org.jivesoftware.openfire.auth.JDBCAuthProvider
provider.group.className           org.jivesoftware.openfire.group.DefaultGroupProvider
provider.lockout.className         org.jivesoftware.openfire.lockout.DefaultLockOutProvider
provider.securityAudit.className   org.jivesoftware.openfire.security.DefaultSecurityAuditProvider
provider.user.className            org.jivesoftware.openfire.user.JDBCUserProvider
provider.vcard.className           org.jivesoftware.openfire.vcard.DefaultVCardProvider
update.lastCheck                   1262616901497
xmpp.auth.anonymous                true
xmpp.domain                        192.168.1.102
xmpp.session.conflict-limit        0
xmpp.socket.ssl.active             true
```
注：红色为修改的内容
重启openfire服务器即可.
以后登陆需要输入id,password.有人会问了，为什么用id而不用email呢？
这是因为email中带有'@'符号，而在openfire中会用于记录服务器的域名。所以会有冲突。
况且，openfire作为IM集成到现有系统中时，走的登陆是隐式登陆，即嵌入到原有系统的登陆方式中。
对用户而言透明。
问题：
但是，需要解决的问题在客户端显示时，现有spark是显示的id，jwchat估计也是，需要修改一下，使其对用户而言只显示用户名！如果能够解决，那么对于现有系统而言，基本无任何改动。
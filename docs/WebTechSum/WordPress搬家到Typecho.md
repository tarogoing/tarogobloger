[toc]
# **博客从WordPress搬家到Typecho**

WordPress虽然功能强大，但是排版真的太繁琐了，每次写文章花在排版上的时间比写文章的时间还多，导致博客两年多了才40几篇文章(当然，更主要的原因还是懒^^)。

某日偶然看到Typecho的介绍：新版支持Markdown语法。在查阅了一些相关的资料后，发现这种以文章内容为中心的博客平台才是我需要的啊！主题、插件、排版那些都不用折腾了！在估计了一些搬家成本之后果断决定将博客从WordPress搬到Typecho。

博客搬家其实也就是转移数据库和文件了，Typecho有专门的WordPress导入插件，所以搬家过程不会很繁琐。

## 数据库导入

Typecho的WordPress导入插件可以将WordPress数据库的文章、评论、独立页面等内容导入，插件需要直接连接WordPress数据库，如果Typecho所在的VPS和WordPress不在同一个网段(Typecho不能直接访问WordPress数据库)，则需要将WordPress的数据库备份后在导入到Typecho所在VPS的一个临时数据库里再导入。

###备份WordPress数据库
在VPS可以使用mysqldump命令备份数据库：
基本用法是：

|     |     |
| --- | --- |
| 1<br>2 | # 备份数据<br>mysqldump -hhostname -Pport -uusername -puserpass dbname > filename |

当然，如果没有VPS的登陆权限，也可以使用phpmyadmin等工具备份。
###导入WordPress数据库到临时中转数据库
切换到mysql命令行，在Typecho所在的VPS新建一个数据库，稍后Typecho导入工具将从这个数据库里导入数据，比如为importdb：

|     |     |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7 | create  database  importdb;<br>//切换到新建的数据库:<br>use importdb;<br>// 用source命令将上面备份的数据库导入到这个临时数据库里：<br>source dbfilename |

### 将WordPress数据导入Typecho

下载并启用插件[(L)](http://docs.typecho.org/plugins/wordpress-to-typecho)[WordpressToTypecho](http://docs.typecho.org/plugins/wordpress-to-typecho)，填写临时数据库的连接参数就可以导入了。

提示：  *该插件v1.0.3 Beta版本的介绍说只能导入WordPress 2.7的数据库，高版本需要转换成2.7的才能导入，但是我直接导入WordPress 3.4的数据库也没有遇到任何问题。*

###优化数据库
导入完成之后就可以将临时数据库删除了：

|     |     |
| --- | --- |
| 1<br>2 | // 删除数据库<br>drop  database  importdb; |

最后别忘了优化一下数据库:

|     |     |
| --- | --- |
| 1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18 | OPTIMIZE  TABLE  typecho_comments;<br>OPTIMIZE  TABLE  typecho_contents;<br>OPTIMIZE  TABLE  typecho_fields;<br>OPTIMIZE  TABLE  typecho_metas;<br>OPTIMIZE  TABLE  typecho_options;<br>OPTIMIZE  TABLE  typecho_relationships;<br>OPTIMIZE  TABLE  typecho_users;<br>```<br>如果以前的草稿不需要的话也可以删除:<br>```sql<br>//<br>delete  from  typecho_contents  where  type='post_draft'<br>```<br>## 附件导入<br>将WordPress的附件文件打包，解压上传到Typecho的附件目录就可以了，因为Typecho的图片不会生成缩略图，所以可以把WordPress的缩略图删除以节省空间：<br>```shell<br>find -name  *-150x*.png \| xargs rm -f<br>find -name  *-150x*.jpg \| xargs rm -f |

上面的命令不一定能将所有尺寸的缩略图删除，所以查找一下其它尺寸的缩略图，然后手动删除：

|     |     |
| --- | --- |
| 1<br>2 | find -name *-*x*.jpg<br>find -name *-*x*.png |

## 导入的文章转成Markdown格式

由于WordPress的旧文章仍然保存为HTML格式，如果文章不多的话，可以手动将文章转成markdown格式以便统一。

推荐一个在线的HTML转markdown工具：[(L)](http://www.atool.org/html2markdown.php)[Html2MarkDown](http://www.atool.org/html2markdown.php)，可以将HTML代码转换成markdown格式。

注意：  *Typeche的文章首行有一个**<!--markdown-->**标记，只有有这个标记的文章才会使用markdown语法渲染，所以以前的文章转换之后需要在首行加上**<!--markdown-->**标记。*

## 插件安装

和WordPress热闹的社区相比，Typecho就显得冷清多了，插件也是屈指可数的，而且还不好找。
推荐几个我在用的插件(插件好久都没更新，但还是可以兼容)：

- [Highlight Js(智能实现代码高亮)](http://70.io/2013/12/highlight-js-typecho-plugin)
- [Sitemap(Google Sitemap 生成器)](http://plugins.typecho.me/plugins/google-sitemap.html)
- [Captcha(Typecho评论中文验证码Captcha插件)](http://www.ccvita.com/499.html)
- [Duoshuo(多说实时同步插件)](https://github.com/rakiy/DSync_Typecho)

## Typecho新手需要注意的问题

- Typecho会吞掉php代码错误。有一次调试发现footer.php中产生一个错误之后页面正常输出，但是服务器响应代码为500。这对搜索引擎是非常不友好的，所以如果修改了代码之后最后检查一下服务器响应是否正确。

## 总结

Typecho拥有轻巧的架构、简洁的后台、简单的文章编写和展示界面，就像AI Writer等做减法的写作工具，它就是用来写作的。作为一个小众的博客平台，特别适合最求轻便和简单的用户。

本文作者：Lance Liao
本文链接：https://www.shuyz.com/posts/wordpress-to-typecho/
版权声明：Copyright © 2012-2019 树叶的BLOG, 未经作者同意禁止转载!
在Telegram上讨论此文：https://t.me/joinchat/EwhMZRVB7xI4OpuXLXukkw

[(L)](https://www.shuyz.com/tag/wordpress/)[\f0c6](https://www.shuyz.com/tag/wordpress/)WordPress[(L)](https://www.shuyz.com/tag/typecho/)[\f0c6](https://www.shuyz.com/tag/typecho/)Typecho

[(L)](https://www.shuyz.com/posts/fix-openwrt-connection-with-putty/)[\f0d9](https://www.shuyz.com/posts/fix-openwrt-connection-with-putty/)使用串口修复OpenWrt路由器[多看 —— 一个优雅的Kindle系统软件](https://www.shuyz.com/posts/duokan-an-alternative-kindle-system/)[\f0da](https://www.shuyz.com/posts/duokan-an-alternative-kindle-system/)
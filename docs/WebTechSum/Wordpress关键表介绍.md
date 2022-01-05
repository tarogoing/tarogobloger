**Wordpress关键表介绍**

wp_posts：**文章表**，**文章ID**(ID)、标题(post_title)、发布时间(post_time)、别名(post_name)(文章固定链接使用的URL)、状态(post_status)、类型(post_type)等。

wp_links：**友情链接表**，友情链接等链接信息。
wp_options：**博客主题表**，博客题目、描述、站点URL等信息。
wp_terms：**标签表**，标签编号、名称、别名；**分类目录名称**、别名。

wp_term_relationships：对象ID(object_id，**部分**对应**文章ID**)，对象**分类ID**(term_taxonomy_id，部分对应**分类目录**编号)

wp_term_taxonomy：**分类目录表，**词项分类标号(term_taxonomy_id)、词项编号(term_id)、类型（taxonomy，**目录**、**标签**、**导航菜单**）、文章数（count）

wp_users：用户表，所有后台登录帐号
wp_usermeta：用户信息表，用户姓名、邮箱、描述等。
wp_conmments：评论表
wp_commentmeta：不清楚其功能
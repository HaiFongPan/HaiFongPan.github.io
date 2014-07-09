---
author: leopure
comments: true
date: 2012-02-28 07:37:09+00:00
layout: post
slug: '解决java读取mysql数据后的乱码问题'
title: 解决Java读取Mysql数据后的乱码问题
wordpress_id: 364
categories:
- Study Note
---

JAVAEE非常让人蛋疼菊紧，好好的一本书，前八章的内容竟然是web编程技术中闪过的，真心不太明白学院老师是如何选教材的。果然还不如去借一本书看，买的好后悔。

言归正传，以前用的mysql是大神同学给配置好的，所有乱码问题都解决了，这次轮到自己了，其实在mysql的客户端中中文显示是正常的，但是java一读出来就不对了，很显然，应该是mysql编码问题，默认貌似是latin1，虽然我不曾认识这个编码，但是从现在开始获取我要开始憎恨它了。

从网上找了一些资料，发现主要问题是my.ini这个配置文件，如何找到这个目录呢，我是在C:\Program Files\MySQL\MySQL Server 5.5 目录下面
关键要修改几个地方：\n\n
[client]加入
[cc lang="perl]default-character-set=gbk[/cc]
[mysql]也是

[cc lang="perl]default-character-set=gbk[/cc]
并把
[mysqld]中的
[cc lang="perl]character-set-server=utf8 #gbk应该也可以[/cc]

然后保存文件
打开控制台
[cc lang="perl]net stop mysql //停止mysql服务
net start mysql //开启mysql服务[/cc]


然后进入mysql

<code>
mysql> show variables like 'character%';
| Variable_name            | Value
| character_set_client     | gbk
| character_set_connection | gbk
| character_set_database   | utf8
| character_set_filesystem | binary
| character_set_results    | gbk
| character_set_server     | utf8
| character_set_system     | utf8
| character_sets_dir       | C:\Program Files\MySQL\MySQL Server 5.5\share\charsets\ |
</code>

如果结果如上所示，应该就不是乱码了。


如果还是不行。亲参考[这里](http://developer.51cto.com/art/200906/130425.htm)

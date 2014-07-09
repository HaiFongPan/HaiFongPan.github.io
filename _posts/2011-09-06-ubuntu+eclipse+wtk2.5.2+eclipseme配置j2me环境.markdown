---
author: leopure
comments: true
date: 2011-09-06 00:43:16+00:00
layout: post
slug: ubuntu+eclipse+wtk2.5.2+eclipseme配置j2me环境
title: ubuntu+eclipse+wtk2.5.2+eclipseme配置j2me环境
wordpress_id: 95
categories:
- J2ME
- Study Note
---

这学期开始上J2ME了，老师说起来很简单，但是还是必须认真对待，之前没有试过J2ME的东西。
将博客作为记录学习笔记的地方还是不错的选择。windows下面的安装比较简单，ubuntu下面的配置稍微有点点不一样，无非就是安装wtk的时候用到了指令，在这里也记录下。
当然事先还是靠了谷歌搜索下了，知识是前人留下的产物，理解了就是自己的。

前提1：JDK
前提2：eclipse
前提3：[eclipseme](http://sourceforge.net/projects/eclipseme/files/eclipseme/)（我的理解是链接eclipse和模拟器的一个枢纽吧）
前提4：wtk（Wireless Toolkit，这里我又不太理解了，有一个是名字带ml版本的，比较大，另一个没有ml，不知道啥区别？？？我下了第二个，程序还是跑起来，就先不管了）
[wtk下载](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javame-419430.html#sun_java_wireless_toolkit-2.5.2_01b-oth-JPR)

JDK安装：略过
eclipse安装：直接解压
wtk的安装：
首先官网上似乎说了要先下载几个包才能正确安装wtk:
前提：这里用到了su的权限，
执行下列指令即可[cc lang="perl"]$sudo apt-get install libxpm-dev libxt-dev libx11-dev libice-dev libsm-dev libc6-dev libstdc++6-4.4-dev[/cc]
接着是wtk的安装,这里有人说要取得权限先:
[cc lang="perl"]$sudo chmod +x sun_java_wireless_toolkit-2.5.2_01-linuxi486.bin
./sun_java_wireless_toolkit-2.5.2_01-linuxi486.bin#开始安装，不行的话加sudo
[/cc]
\n\n
安装的时候开始会有条例让你同意，这里我是按了很久回车建才按到底的，中间过程很简单，无非要你选择个jdk的路径
如果不知到java路径在哪里的：
[cc lang="perl"]leopure@ubuntu:~$ which java
/usr/bin/java#这是我的，所以jdk目录：/usr/bin
[/cc]
之后wtk就安装完成。

接着安装的是插件eclipseme：首先是打开是eclipse-->help-->install new software

[![](http://www.leopan.me/wp-content/uploads/2011/09/1.png)](http://www.leopan.me/wp-content/uploads/2011/09/1.png)
点击add
[![](http://www.leopan.me/wp-content/uploads/2011/09/2.png)](http://www.leopan.me/wp-content/uploads/2011/09/2.png)
填入信息。选择自己的插件
[![](http://www.leopan.me/wp-content/uploads/2011/09/3.png)](http://www.leopan.me/wp-content/uploads/2011/09/3.png)

装完重启eclipse

这样插件就装好了，然后在windows-->preference中如果又J2ME选项就表示成功了,但是这里还得配置下:
选择好wtkroot的目录.
[![](http://www.leopan.me/wp-content/uploads/2011/09/5.png)](http://www.leopan.me/wp-content/uploads/2011/09/5.png)[![](http://www.leopan.me/wp-content/uploads/2011/09/6.png)](http://www.leopan.me/wp-content/uploads/2011/09/6.png)全文完，转载说明出处。本文非全部原创。

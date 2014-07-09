---
author: leopure
comments: true
date: 2011-08-21 15:10:31+00:00
layout: post
slug: ubuntu+wordpress+dotcloud搭建属于自己的博客
title: ubuntu+wordpress+dotcloud搭建属于自己的博客
wordpress_id: 41
categories:
- dotcloud
- ubuntu
tags:
- blog
- dotcloud
- free
- ubuntu
- wordpress
- 免费
---

最近是流行搭博客么？表示没有什么水平，没钱买空间和域名，就用了免费的tk域名和免费的dotcloud，步骤还算是比较简单的，所以拿出来分享下，也由于博客没有人气，所以算是自己的日志或者是记录怎么搭建。
首先当然是准备工作，安装Dotcloud CLI(Command Line Interface)，由于通过easy_install安装所以
首先安装easy_install： 
[cc lang="Perl"]  $wget http://pypi.python.org/packages/source/s/setuptools/setuptools-0.6c11.tar.gz 
   $tar -xf setuptools-0.6c11.tar.gz 
   $cd setuptools-0.6c11 
   $sudo python2.7 setup.py install   //其中的python2.7大家换成自己机器上的
[/cc]\n\n
  接下来才是安装Dotcloud CLI: 
[cc lang="Perl"]$sudo easy_install pip && sudo pip install dotcloud   //安装python-pip以安装  [/cc] 
此时dotcloud CLI应该装完,可以用最简单的命令测试:
[cc lang="perl"] $dotcloud[/cc] 
此时键入API KEY，API Key可以登录dotcloud网站在 setting里找到。 

此时可以创建你的blog所在目录了，例如我的myapp/blog/[wordpress](http://cn.wordpress.org/)，[wordpress](http://cn.wordpress.org/)在官方下载。中英随意。
将wordpress解压到你的目录下面。

接着键入代码：
[cc lang="perl"]cd myapp/blog/wordpress
git clone https://github.com/qpleple/wordpress-on-dotcloud #此处如果没有安装git则sudo apt-get install git
mv wordpress-on-dotcloud/* .
dotcloud create myblog #你app自己的名字
dotcloud push myblog
dotcloud push myblog 
dotcloud push myblog #push 2-3次工程进入提示网址即可5分钟注册wordpress
[/cc]


最后可以绑定你的域名：
[cc lang="perl"]$ dotcloud alias add myblog.www www.example.com[/cc]
根据提示更改DNS。

这样自己的blog就搭建完了，当然push的功能非常不完美，容易坏事儿。推荐使用ssh。
推荐mckelvin的简易教程：http://www.mckelvin.tk/?p=740


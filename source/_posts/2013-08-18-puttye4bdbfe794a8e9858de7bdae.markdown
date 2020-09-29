---
author: admin
comments: true
date: 2013-08-18 10:47:48+00:00
layout: post
slug: putty%e4%bd%bf%e7%94%a8%e9%85%8d%e7%bd%ae
title: Putty使用配置
wordpress_id: 704
categories:
- IT
- linux
---

Putty是一款不错的，在windows下可以用来进行ssh和telnet远程连接的小工具，绿色纯净。之所以说它绿色纯净，是它就纯粹只是一个不到1M大小的exe文件。但是刚上手的时候，由于懒得配置，就会觉得用起来非常不舒服，跟linux中原生态终端有相当程度的山寨痕迹。但是今天下定决心好好配置一番，发现最终效果还不错。





[![微博桌面截图_20130818182114](https://wonderflow.info/images/2013-08-18-puttye4bdbfe794a8e9858de7bdae/微博桌面截图_20130818182114.jpg)](https://wonderflow.info/images/2013-08-18-puttye4bdbfe794a8e9858de7bdae/微博桌面截图_20130818182114.jpg)





1、登陆，在hostname处填写相应IP，点击Open按钮，即可连接过去。





2、修改字体，对着上边栏右键->Change Settings->Window->Appearance，看到右边有Font Settings，点击Change按钮，即可改变字体和按钮，推荐Consolas，size选择11。





3、修改背景色，在同一Window菜单栏下有个Colors，右边有个default background，随便改成你喜欢的。





4、修改能显示的最大行数，直接点Window那一栏，右边有个Lines of scrollback，默认是200行，实在太少了，直接加个0，给到2000。





5、解决中文字体乱码问题，选择Wondows->Translation，右边有个Remote charactor set。Putty会根据远程端使用的字体转义，所以你只要告诉他你所连的远程机器使用的字符集即可。如何查看？




{% codeblock %}
$ echo $LANG $LANGUAGE
{% endcodeblock %}




据你远程机器使用的字符集在下拉菜单中选择对应字符集，如果没有的话，就选择最下面的use font encoding。





6、保存配置，OK这一切都设置好了，点击apply，应用你所做的配置，但是如果你把putty关了，下一次开的时候，你会发现，之前所有配置都没有了，怎么办？没关系，putty虽然程序只有几百k大小，但是它也给你提供了保存这些配置的功能。还是老样子change settings->Session右边可以自己选中Default Settings点击保存，也可以新建一个保存。





基本上，最让我觉得山寨痕迹严重的几个点都可以通过上面的说明解决了。其他需求，请自行多看看putty给出的change settings里面的选择吧。





PS： 登陆后使用




{% codeblock %}
$ source ~/.bashrc
{% endcodeblock %}




配置应用上去，包括一些alias。





使用screen命令可以运行程序在后台执行，哪怕putty关掉也没关系。




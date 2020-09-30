+++
author = "admin"
date = "2013-08-02T11:50:12Z"
slug = "2013/08/02/ubuntu12-04-install-record"
title = "Ubuntu12.04LTS Install Record"
Categories = ["IT", "linux"]
Tags = ["linux", "ubuntu"]
+++

一、软件源更新




{% codeblock %}

$vi /etc/apt/sources.list

{% endcodeblock %}




二、定制命令提示符：




{% codeblock %}

$vi ~/.bashrc $PS1="\[\e]0;\u@\h: \w\a\e[32;1m\]\W\$\[\e[0m\] " #效果:仅高亮显示短路径和$符号

{% endcodeblock %}




三、安装chrome 登录，同步相应配置，这样浏览器方面的各种配置就齐全了





四、github相关 
安装git:




{% codeblock %}

$ sudo apt-get install git

{% endcodeblock %}




github：[配置SSH](https://help.github.com/articles/generating-ssh-keys)





五、中文输入法[ibus](http://wiki.ubuntu.org.cn/IBus) 
找到language support这个应用，然后找到keyboard input method system设置项（默认的是none），选择ibus项就可以了。





六、vim (配置ruby插件，拿的别人的方案)




{% codeblock %}

$ sudo apt-get install ctags

{% endcodeblock %}




start from:[git clone](https://github.com/wonderflow/My-Vim-Rails)





七、字体设置：





download consolas.tty




{% codeblock %}

$ sudo mkdir -p /usr/share/fonts/vista 
$ sudo cp Yahei.ttf /usr/share/fonts/vista/ 
#改变权限： 
$sudo chmod 644 /usr/share/fonts/vista/*.ttf 
#安装： 
$cd /usr/share/fonts/vista/ 
$sudo mkfontscale 
$sudo mkfontdir 
$sudo fc-cache -fv

{% endcodeblock %}




八、安装ruby [wiki on ruby-china](http://ruby-china.org/wiki)





九、安装[WPS.deb](http://linux.wps.cn)





从命令行出现bug：
 /opt/kingsoft/wps-office/office6/wps: error while loading shared libraries: libgthread-2.0.so.0: cannot open shared object file: No such file or directo





解决方法：
 64位的系统，wps是32位编译的，所以少了一些库。




{% codeblock %}
$ sudo apt-get install ia32-libs
{% endcodeblock %}




下载[字体文件](http://bbs.wps.cn/thread-22355435-1-1.html)




{% codeblock %}
$cp [downloaded files] /usr/share/fonts/wps-office
{% endcodeblock %}




再次打开成功。





十、安装vmware虚拟机





download:
vmware_player.sh;
win7.iso and anything you need~




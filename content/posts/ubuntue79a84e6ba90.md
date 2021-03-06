+++
author = "admin"
date = "2013-10-18T08:18:21Z"
slug = "2013/10/18/ubuntue79a84e6ba90"
title = "ubuntu源的本地化"
Categories = ["linux", "shell"]
Tags = ["linux", "source", "ubuntu"]
+++

最近，为了装个runit，装了2天，各种问题都出现了，始终无法解决，都快装哭了。





最后用排除法，一个个排除问题，发现是使用的网易ubuntu源的问题。 哎，看来从研究设计到工程细节，还是有很大的技术距离的。





静默安装方法




{% codeblock %}
 
sudo DEBIAN_FRONTEND=noninteractive apt-get install -f -y --force-yes --no-install-recommends runit  

{% endcodeblock %}




最后为了安全稳定，决定自己建立ubuntu的局域网源，这样后面安装软件也会更快。





1、安装依赖软件




{% codeblock %}

sudo apt-get install dpkg-dev

{% endcodeblock %}




2、将 /var/cache/apt/archives/下的所有deb文件拷到(cp命令，图形界面都没问题) 你想要的地方，拷贝前建议执行一下




{% codeblock %}
sudo apt-get autoclean
{% endcodeblock %}




比方说拷至home下新建的deb文件夹




{% codeblock %}
sudo cp /var/cache/apt/archives/*.deb /var/www/deb/
{% endcodeblock %}




3、拷贝完成后到/var/www/deb/目录下执行命令，生成Packages.gz让告诉ubuntu源的内容。




{% codeblock %}
sudo dpkg-scanpackages . /dev/null | gzip >Packages.gz
{% endcodeblock %}




4、安装apache2使你的部分源对外开放




{% codeblock %}
sudo apt-get install apache2
{% endcodeblock %}




5、在apache默认的/var/www目录下建立一个链接，链接到我们的安装包存放的目录。




{% codeblock %}
sudo ln -s /var/www/deb/ /var/www/ubuntu-local
{% endcodeblock %}




此时可以用浏览器访问一下 http://127.0.0.1/ubuntu-local/ 看看能不能访问到，如果访问不到，就要改一下文件的权限，文件需要可读可执行权限。 
6、更改/etc/apt/sources.list文件




{% codeblock %}
sudo vi /etc/apt/sources.list
{% endcodeblock %}




把 deb http://127.0.0.1/ubuntu-local / 加入到sources.list即可。
 **注意空格后有一个斜杠** 
网上其它电脑修改源时，要把127.0.0.1改为你的ip地址或域名





最后，除了ubuntu源的本地化以外，包括github等也都可以本地化，使自动化部署的进程加快。





被163的源坑了两天，从此除了cn.ubuntu的国内源，我再也不相信别的源了。



<!-- more -->


{% codeblock %}

# deb cdrom:[Ubuntu 12.04.2 LTS _Precise Pangolin_ - Release amd64 (20130213)]/ dists/precise/main/binary-i386/

# deb cdrom:[Ubuntu 12.04.2 LTS _Precise Pangolin_ - Release amd64 (20130213)]/ dists/precise/restricted/binary-i386/
# deb cdrom:[Ubuntu 12.04.2 LTS _Precise Pangolin_ - Release amd64 (20130213)]/ precise main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ precise main restricted
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://cn.archive.ubuntu.com/ubuntu/ precise-updates main restricted
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://cn.archive.ubuntu.com/ubuntu/ precise universe
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise universe
deb http://cn.archive.ubuntu.com/ubuntu/ precise-updates universe
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://cn.archive.ubuntu.com/ubuntu/ precise multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ precise-updates multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://cn.archive.ubuntu.com/ubuntu/ precise-backports main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ precise-backports main restricted universe multiverse

deb http://security.ubuntu.com/ubuntu precise-security main restricted
deb-src http://security.ubuntu.com/ubuntu precise-security main restricted
deb http://security.ubuntu.com/ubuntu precise-security universe
deb-src http://security.ubuntu.com/ubuntu precise-security universe
deb http://security.ubuntu.com/ubuntu precise-security multiverse
deb-src http://security.ubuntu.com/ubuntu precise-security multiverse

## Uncomment the following two lines to add software from Canonical’s
## ‘partner’ repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu precise partner
# deb-src http://archive.canonical.com/ubuntu precise partner

## This software is not part of Ubuntu, but is offered by third-party
## developers who want to ship their latest software.
deb http://extras.ubuntu.com/ubuntu precise main
deb-src http://extras.ubuntu.com/ubuntu precise main

{% endcodeblock %}



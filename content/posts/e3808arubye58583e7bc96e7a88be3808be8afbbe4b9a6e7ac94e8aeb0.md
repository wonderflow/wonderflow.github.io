+++
author = "admin"
date = "2013-11-21T16:27:21Z"
slug = "2013/11/21/e3808arubye58583e7bc96e7a88be3808be8afbbe4b9a6e7ac94e8aeb0"
title = "《ruby元编程》读书笔记"
Categories = ["IT", "Reading Notes", "Ruby"]
Tags = ["ruby"]
+++

总的来说，《ruby元编程》是一本好书。 当我拿到这本书的时候，第一反应是什么叫元编程？书上的定义是这样的：**“元编程是写出编写代码的代码”**。 而我认为，元编程是ruby语言的一些高级特性，是属于ruby的**奇技淫巧**。





有意思的是，这本书不是枯燥的一章一章给你灌输这些知识，而是描述公司里一个老程序员一对一带一个年轻程序员的故事。就是所谓的mentor/buddy制度，大部分IT公司都有，就是不知道贯彻的怎么样。不管书中描述的这样的工作情况是不是真的，都很令人神往。





我觉得一个IT行业的大公司就该有这样的气度，把新员工当朋友、兄弟一样培养，毕竟IT行业中，人才是最大的财富。这样带起来的团队，相比凝聚力也是极强的。





书中的内容也根据工作日的划分成了周一到周五5个部分，分别以“对象模型”，“方法”，“代码块”，“类定义”以及“编写代码的代码”这5个点作为话题描述。





整本书都值得一看，书中很多技术给了我不少启发，挑几个有意思的记一记。







  * ruby的class关键字更像是一个作用域操作符而不是类型声明语句。它的确可以创建一个还不存在的类，不过也可以把这看成是一种副作用。对于class关键字，其核心任务是把你带到类的上下文中，让你可以在其中定义方法。



  * 打开类这样的技术也有隐患：如果你粗心地为某个类添加了某些功能，就可能把类本身的一些方法替换到，造成bug，人们给这张方式起了一个不太好听的名字：**猴子补丁（Monkeypatch）**，这就是著名的猴子补丁的由来了。当然，有时候一些临时性的猴子补丁可是有救场的奇效的。



  * 如何以大写字母开头的引用（包括类名和模块名），都是常量。



  * **动态派发**：当你调用一个方法时，实际上是给一个对象发送了一条消息。ruby用自身的语法完美的解释了这个关于类执行方法的概念。





{% codeblock %}
class MyClass
   def my_method(my_arg)
     my_arg * 2
   end
 end
 obj = MyClass.new
 obj.my_method(3) # => 6
 obj.send(:my_method, 3) # => 6

{% endcodeblock %}


<!-- more -->



  * 关于符号，symbol，是一个很不错的定义，它跟string的用法有相似性，但是它不是string。



  * **幽灵方法**，在ruby中，编译器并不强制方法调用时的行为，这意味着你可以调用一个并不存在的方法。而接收这个不存在方法的函数，就是method_missing方法，你可以覆写这个方法，并使用动态派发技术，来实现幽灵方法。 7. yield关键字可以回调一个你定义的块，相当好用。





{% codeblock %}
def a_method
  return yield if block_given? 'no block'
end
a_method # => "no block" 
a_method { "here's a block!" } # => "here's a block!"

{% endcodeblock %}




  * **闭包**：当定义一个块时，它会获取当时环境中的绑定，并且把它传给一个方法时，它会带着这些绑定一起进入该方法。



  * **变量的作用域**：ruby是不存在嵌套作用域的。class、module、def三个关键字把ruby的作用域截然分开，哪怕他们是嵌套的。在某个class里def一个方法，这里面不同层次的变量是互不相通的。



  * 但是ruby中由扁平化作用域的技术。定义class、方法可以这样写：





{% codeblock %}
my_var = "success" 
MyClass = Class.new do 
  puts "#{my_var} in the class definition!" 
  define_method :my_method do
    puts "#{my_var} in the method!" 
  end
end 
MyClass.new.my_method
 => Success in the class definition!
    Success in the method!

{% endcodeblock %}




  * **上下文探针技术**：instance_eval()方法，它能让你通过对象进入到类的环境中执行代码。



  * 块不是对象，但是可以通过lambda()、proc()和Proc.new()这三个方法转化为对象进行存储，然后通过Proc#call()方法来执行这个块。



  * **&操作符**：将块作为参数传递，附加到一个绑定上，这个参数必须是参数列表中的最后一个，且以&符号开头。



  * **类实例变量**：仅能被类本身所访问，写在类定义里面，方法定义外面的@var。



  * **单件方法**：只属于某个实例对象的方法。





{% codeblock %}
str = "just a regular string" 
def str.title?
  self.upcase == self
end 
str.title? # => false 
str.methods.grep(/title?/) # => ["title?"]
str.singleton_methods # => ["title?"]

{% endcodeblock %}




而关于类方法的真相其实就是：类只是对象，而类名只是常量，类方法其实就是最基础那个类的单件方法。多么巧妙的设计。





**类宏**：类似attr_accessor()方法这样，所有的attr_*()方法都定义在Module类中，Module包含在Object中，因此不管self是类还是模块，都可以使用它们。虽然它们看起来很像关键字，但其实它们只是普通方法，只是可以用在类定义中。







  * **eigenclass**：是一个对象单件方法的存活之所，它是一个特殊的类。ruby中还有一种特殊的语法可以让你进入某个类的eigenclass。




{% codeblock %}
class << an_object
# 这里是你自己的代码
end 

{% endcodeblock %}




所以ruby中方法查找的过程是先进入eigenclass，再进入该类本身，再进入他的一个个supercalss。







  * 环绕别名：使用alias可以给ruby方法取别名。语法是第一个出现的是方法的新名字，第二个出现的是原始的名字。alias是一个关键字而非一个方法，所以两个方法名之间没有逗号。 




{% codeblock %}
#一个简单的用法示例 
class String
 alias :real_length :length
 def length
   real_length > 5 ? 'long' : 'short'
 end 
end 
"War and Peace".length # => "long" 
"War and Peace".real_length # => 13

{% endcodeblock %}




环绕别名的两个注意点： 1）环绕别名是一个猴子补丁，他可能会破坏已有代码。 2）永远不要把一个环绕别名加载两次。







  * **here文档（here document）**：用以描述一个占多行的代码字符串。它以一个双小于号（<<）打头，后面紧跟“结束序列（terminination sequence）”的字符组。在此之后可以定义字符串的内容，直到碰到第一个只包含结束序列的行时，字符串定义才结束。 




{% codeblock %}
s = << END This is a "muti-line" string wishing you a great #{Time.now.year + 1}
 END 
puts s This is a "muti-line" string wishing you a great 2014

{% endcodeblock %}




  * eRB标准库是ruby默认的模板系统，他是一个代码处理器，使用它你可以把ruby代码嵌入到如何文件中，加载的时候，ruby会把<%=...>这个特殊符号中嵌入的ruby代码进行执行。 




{% codeblock %}
require 'erb' 
erb = ERB.new(File.read('xx.file')) # xx.file 存放了包含<%=...>嵌入ruby代码的文档内容 
erb.run #然后执行结果会自动把xx.file中<%=...>符号里嵌入的ruby代码执行。

{% endcodeblock %}




**最后，ruby是一个极具表现力的语言，相当灵活多变。关键还是要多写多看。**





以上。




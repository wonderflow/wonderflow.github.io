+++
author = "admin"
date = "2013-09-15T16:31:10Z"
slug = "2013/09/15/javascripte4b8ade79a84e997ade58c85efbc88javascripte69d83e5a881e68c87e58d97e8afbbe4b9a6e7ac94e8aeb0efbc89"
title = "JavaScript中的闭包（JavaScript权威指南读书笔记）"
Categories = ["IT", "JavaScript", "Reading Notes"]
+++

最近在看javascript的书，稍微学习一些前端的东西，以便做些web应用的方便。





以前看到闭包这个概念一直不太懂，今天下定决心好好研读了一下，终于明白了一二。





像绝大多数高级编程语言，javascript也有其词法的作用域。意思是说，当函数被执行的时候，起作用的不是被调用时的变量作用域，而是在函数被定义的时候声明的变量域。所以为了实现这个词法作用域的功能，javascript必须在中间过程中不仅保存函数的代码，还需要保存当时所引用的作用域链。[这个函数对象和函数变量保存起来的作用域的组合在计算机科学里的称呼就叫做闭包。](http://en.wikipedia.org/wiki/Closure_(computer_science))





从技术角度来说，javascript里的函数都是闭包：它们都是对象，并且它们有一个与之关联的作用域链。大多数函数被调用的时候，影响它们的作用域与定义它们的作用域是相同的，但是这并不影响它们是闭包。只有当函数被调用的时候，影响它们的作用域与定义它们时候的不相同的时候，闭包的概念才会变得有趣起来。**这个通常发生在函数的返回值本身就是一个内嵌函数的时候。**



<!-- more -->



要理解闭包的概念，我们先来看一段内嵌函数返回相关的代码。




{% codeblock %}

var scope = "global scope"; // A global variable
function checkscope() {
var scope = "local scope"; // A local variable
function f() { return scope; } // Return the value in scope here
return f();
}
checkscope() // => "local scope"

{% endcodeblock %}




checkscope()函数在内部定义了函数f，并在返回的时候执行了f函数，所以我们得到了结果 “local scope”。





再来看看下面这段：




{% codeblock %}

var scope = "global scope"; // A global variable
function checkscope() {
var scope = "local scope"; // A local variable
function f() { return scope; } // Return the value in scope here
return f;
}
checkscope()() // What does this return?

{% endcodeblock %}




这个函数定义了f函数以后，把这个函数对象作为返回值了，同时我们在外面执行了这个刚返回的函数本身。我们得到的结果是什么呢？大家不妨用浏览器自带的控制台试试，结果还是”local scope”。





所以请记住词法作用域的这一条基本规则：**javascript执行函数的时候使用的作用域环境是定义它们时候的那样。**





但是我们忍不住会想，这个也太神奇了把，按照我们传统C或C++语言的思想，这个临时变量在函数执行完后就应该被销毁了，怎么此时还利用着临时变量的值呢？这就是我们想要说的，javascript是动态语言，是一种高级语言。如果你用一些底层语言的思想去局限它，那你就陷入了误区。javascript得知你在后面需要引用到这个局部变量，或者更准确的说，这个函数的闭包的时候，它会很神奇的帮你保存好当时的变量环境，即保存好函数的闭包。等你彻底不用的时候，javascript里面的垃圾收集器才会去销毁它。





让我们再来看一段有趣的代码：




{% codeblock %}

var uniqueInteger = (function() { // Define and invoke
var counter = 0; // Private state of function below
return function() { return counter++; };
}());

{% endcodeblock %}




我们仔细的看过这段代码后，发现它是执行了一个函数，函数内部定义了一个初始值为一的自增函数，并把这个函数返回。
也就是说，最后uniqueInteger这个对象其实是一个自增函数。
读者不妨玩一玩这个uniqueInteger，根据闭包被存储的原则，当你不断的调用uniqueInteger()得到的是不同的值。并且这里有个额外的优点就是，一旦这个uniqueInteger被定义完后，就再也没有途径去修改counter值了。





再来看一段代码：




{% codeblock %}

function counter() {
var n = 0;
return {
count: function() { return n++; },
reset: function() { n = 0; }
};
}
var c = counter(), d = counter(); // Create two counters
c.count() // => 0
d.count() // => 0: they count independently
c.reset() // reset() and count() methods share state
c.count() // => 0: because we reset c
d.count() // => 1: d was not reset

{% endcodeblock %}




这里值得说明的是亮点：1、c和d同时共享着私有变量n。2、每次我们调用counter()的时候，都生成了新的私有变量和作用域链。
所以虽然c和d通向n的途径是一样的，但是它们执行count或reset函数的时候，互相不影响n的值。





最后有意思的是这段代码：




{% codeblock %}

// This function adds property accessor methods for a property with
// the specified name to the object o. The methods are named get
// and set. If a predicate function is supplied, the setter
// method uses it to test its argument for validity before storing it.
// If the predicate returns false, the setter method throws an exception.
//
// The unusual thing about this function is that the property value
// that is manipulated by the getter and setter methods is not stored in
// the object o. Instead, the value is stored only in a local variable
// in this function. The getter and setter methods are also defined
// locally to this function and therefore have access to this local variable.
// This means that the value is private to the two accessor methods, and it
// cannot be set or modified except through the setter method.
function addPrivateProperty(o, name, predicate) {
var value; // This is the property value

// The getter method simply returns the value.
o["get" + name] = function() { return value; };

// The setter method stores the value or throws an exception if
// the predicate rejects the value.
o["set" + name] = function(v) {
if (predicate && !predicate(v))
throw Error("set" + name + ": invalid value " + v);
else
value = v;
};
}

// The following code demonstrates the addPrivateProperty() method.
var o = {}; // Here is an empty object

// Add property accessor methods getName and setName()
// Ensure that only string values are allowed
addPrivateProperty(o, "Name", function(x) { return typeof x == "string"; });

o.setName("Frank"); // Set the property value
console.log(o.getName()); // Get the property value
o.setName(0); // Try to set a value of the wrong type

{% endcodeblock %}




一旦这个addPrivateProperty函数执行完毕后，这个private的效果就彻底成功了，除了setName和getName以外，你就再也没有别的途径来搞到value的值了。





最后再来看一下下面这两段代码的区别：




{% codeblock %}

// This function returns a function that always returns v
function constfunc(v) { return function() { return v; }; }

// Create an array of constant functions:
var funcs = [];
for(var i = 0; i < 10; i++) funcs[i] = constfunc(i);

// The function at array element 5 returns the value 5.
funcs[5]() // => 5

{% endcodeblock %}




以及下面这段代码：




{% codeblock %}

// Return an array of functions that return the values 0-9
function constfuncs() {
var funcs = [];
for(var i = 0; i < 10; i++)
funcs[i] = function() { return i; };
return funcs;
}

var funcs = constfuncs();
funcs[5]() // What does this return?

{% endcodeblock %}




如果你运行了这两段代码以后看了结果，就会明白，在第二段代码中，当时影响它的作用域变量i其实在执行完这个函数后还在变化。所以最后所有funcs里的函数里的i都是最后的10。





最后要说的是，this是一个关键词，而不是一个局部变量，所以你要用var self = this来保存这个值，与此相同的是arguments变量。




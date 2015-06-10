---
author: admin
comments: true
date: 2012-08-10 12:41:30+00:00
layout: post
slug: stl-containers-%e4%b9%8b-vector
title: STL Containers 之 vector
wordpress_id: 428
categories:
- C++
- IT
tags:
- C++
---

我觉得，vector是STL中最常用的容器，没有之一。





因为它简单，安全，也容易理解。实际上它就是一个刚学C++时我们梦寐以求的动态数组，大小不需要一开始的时候定，而又不像链表操作起来那么麻烦。





先说一下最简单的几个用法吧：





比如说你要创建一个一维数组，数组元素是一个结构体。（为什么用结构体？因为结构体都会了的话，int之类的，就简单了。）





在示例代码中用注释的方式讲解一下各种vector函数的用法。



```

# include 
# include 

using namespace std;


struct Node{
int x,y;
Node(){x = 30;}
Node(int x,int y):x(x),y(y){}//构造函数
};





int main(){
vectormyvector;   //定义一个vector数组
for(int i=0;i<9;i++){
myvector.push_back(Node(i,1+i));    //插入一个元素
}
myvector.pop_back();//与之对应的pop_back是从末尾踢掉一个元素





cout<<"first out:";
for(int i=0;i<myvector.size();i++){ //vector元素的遍历，几乎跟平时数组一样的写法
cout<<" "<<myvector[i].x;   //vector元素的输出
}
cout<<endl;





//说一下迭代器，迭代器可以理解为指针，目的是为了迭代容器中的元素
//迭代器的定义跟我们声明的容器类型是相同的，在这里是
vector::iterator it;
it = myvector.begin()+3;    //begin()就是开始的位置，加几就是指向了数组中的第几个





myvector.erase(it); //删除数组第一个元素
//erase函数就是起到删除的作用，但是参数是迭代器
//也可以成段删除，给的两个参数都是迭代器，分别是成段删除的开头位置和末尾位置
myvector.erase(myvector.begin(),myvector.begin()+3);//这里的加操作符是迭代器元素个数的加
//删除了这么多，我们重新输出看看
//此处我们用的迭代器的方法遍历vector数组
//值得一提的是，如果在遍历的过程中删除，那么vector元素在删除操作的同时
//会把被删除元素后面的元素往前移动，有删除的那次指针不加
cout<<"second out:";
for(it = myvector.begin();it<myvector.end();it++){
cout<<" "<x;
}
cout<<endl;





//一次性全部删除如何操作？
myvector.clear();//对，调用一次clear就可以了。





//重新分配给vector一些值，下面我们用到assign函数，分配内容。
const Node a(100,0);
myvector.assign(4,a);//可以直接分配，第一个是次数，第二个参数是数值(元素)
//这样以后，myvector中就是4个{100,0}
vector vt;
//再用迭代器来范围性的赋值，两个参数分别是开始元素和结尾元素
//同时要说明的是，assign函数调用以后，先前的内容会被清除
vt.assign(myvector.begin()+1,myvector.end()-1);
cout<<"third out:";
for(it = vt.begin();it<vt.end();it++){
cout<<" "<x;
}
cout<<endl;
Node myints[5];
for(int i=0;i<5;i++)
myints[i] = Node(i,i+1);
myvector.assign(myints,myints+5);//最后，用数组assign赋值





//反向迭代器，跟迭代器是一样的，就是以数组的末尾为开头，就是定义的方法有所不同
vector::reverse_iterator rit;
//相应的，要从末尾开始，我们要用vector中的rbgin()和rend()函数，意思自然是反向开头和反向末尾
//反向迭代器的用法和迭代器其实是一样的，就是为了统一才出现了rbegin,rend;
cout<<"fourth out:";
for(rit=myvector.rbegin();rit<myvector.rend();rit++){
cout<<" "<x;
}
cout<<endl;





//然后我们说一下用下标定位数组的元素,用at函数和用中括号是一样的效果
cout<<"fifth out:";
cout<<myvector.at(2).x<<" "<<myvector[2].x<<" ";
//front函数和back函数是一对，分别指向数组第一个元素和最后一个元素的值
cout<<myvector.front().x<<" "<<myvector.back().x<<endl;





cout<<"is empty? : "<<myvector.empty()<<endl;//vector是否为空
cout<<"size: "<<myvector.size()<<endl;//vector中的元素个数
//说到size，就顺便说一下resize，resize函数可以重新改变vector的大小
myvector.resize(3);
//这个就把size变成了3，如果原来的大小超过了3，就把超过的地方删去，
//如果原来不足3，就自动调用缺省构造函数
myvector.resize(6,Node(1,1));
//这个就是把大小变成6，原本有的不动，扩展出来的用（0,0）填
myvector.resize(10);





cout<<"sixth out:";
for(int i=0;i<myvector.size();i++){
cout<<" "<<myvector[i].x;
}
cout<<endl;





//最后说一下swap
//vector这里的swap和algorithm里面的swap是大不一样的，
//这里的swap是交换两个vector容器
vt.swap(myvector);//进行了这步以后，vt就和myvector交换了
cout<<"seventh out:";
for(int i=0;i<myvector.size();i++){
cout<<" "<<myvector[i].x;
}
cout<<endl;





//另外还有一种添加元素的方法是insert，但是不常用，这里就不介绍了
//还有一些其他的非常不常用，我就不介绍了。





}
/*
输出：
first out: 0 1 2 3 4 5 6 7
second out: 4 5 6 7
third out: 100 100
fourth out: 4 3 2 1 0
fifth out:2 2 0 4
is empty? : 0
size: 5
sixth out: 0 1 2 1 1 1 30 30 30 30
seventh out: 100 100
*/





```





基本上我平时常用的我均已经在代码中提到了，其实vector本身独立使用并不多见，更多的是vector跟其他stl库结合使用，尤其是algorithm库里的泛型算法。





深入阅读：[C++官方网站上的描述](http://www.cplusplus.com/reference/stl/vector/)




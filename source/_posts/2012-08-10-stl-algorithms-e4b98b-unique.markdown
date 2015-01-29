---
author: admin
comments: true
date: 2012-08-10 10:40:13+00:00
layout: post
slug: stl-algorithms-%e4%b9%8b-unique
title: STL Algorithms 之 unique
wordpress_id: 424
categories:
- C++
- IT
tags:
- C++
---

C++的文档中说，STL中的unique是类似于这样实现的：
[cpp collapse="false"]
template <class ForwardIterator>
  ForwardIterator unique ( ForwardIterator first, ForwardIterator last )
{
  ForwardIterator result=first;
  while (++first != last)
  {
    if (!(*result == *first))  // or: if (!pred(*result,*first)) for the pred version
      *(++result)=*first;
  }
  return ++result;
}
n
仔细一看就知道，它并不是帮你直接把一个数组中所有重复的元素除去，而是对数组扫描一次，只看当前元素和前面一个元素，如果当前值和前面的值相等，那么跳过，否则就把这个值算上，迭代器递增，最后返回给你一个位置，表示我扫描到多少个当前值与其前面一个元素值不同的元素。

所以，要真正利用好unique，我们必须先对我们所需要进行unique操作的数组排序，然后再使用unique。

这样以后其实还是不满足我们的要求的，因为实际上unique函数实现的只是把不同的元素“unique”放到数组的前面，而数组的后面还有一段重复的没有去掉。这个时候就可以利用到unique函数的返回值啦，它返回的就是重复元素出现的第一个位置。

另外，unique函数可以接受两个参数（数组的开头，数组的末尾），也可以接受三个参数（数组的开头，数组的末尾，两个元素的比较（即定义怎样算元素相等））

看一下实例吧：
[cpp collapse="false"]
// resizing vector
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

bool myfunction (int i, int j) {
  return (i==j);
}

int main () {
  int myints[] = {10,20,20,20,30,30,20,20,10};    // 10 20 20 20 30 30 20 20 10
  vector<int> myvector (myints,myints+9);
  vector<int>::iterator it;

  
  it = unique (myvector.begin(), myvector.end()); // 10 20 30 20 10 ?  ?  ?  ?
  //尖所指即it的位置                                //                ^            
												  
  myvector.erase( it , myvector.end() );       // 第一种去掉末尾的方法

  cout << "first: myvector contains:";
  for (int i=0;i<myvector.size();i++)
    cout << " " << myvector[i];
  cout<<endl;

  sort(myvector.begin(),myvector.end());		//先排序

  it = unique (myvector.begin(), myvector.end(), myfunction);   
  // 使用比较函数，但此处是跟缺省的比较一样的。

  myvector.resize( it - myvector.begin() );       // 10 20 30 

  cout << "second: myvector contains:";
  for (it=myvector.begin(); it!=myvector.end(); ++it)
    cout << " " << *it;
  cout << endl;

  return 0;
}

/*
输出：
first: myvector contains: 10 20 30 20 10
second: myvector contains: 10 20 30
*/
n

深入阅读：[C++官方网站上的描述](http://www.cplusplus.com/reference/algorithm/unique/)。

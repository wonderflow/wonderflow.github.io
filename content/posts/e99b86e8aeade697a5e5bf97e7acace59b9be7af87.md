+++
author = "admin"
date = "2012-07-10T16:46:07Z"
slug = "2012/07/10/e99b86e8aeade697a5e5bf97e7acace59b9be7af87"
title = "集训日志第四篇"
Categories = ["Life", "njust"]
Tags = ["rank", "小结", "暑假集训", "比赛", "集训日志"]
+++

今天鱼头组织了第一次组队赛，用的是浙大2010年九月份的月赛。题目质量不错，发现有一份watashi的[解题报告](http://blog.watashi.ws/1492/zojmonthly1009/)，大家可以过去围观，上面还有各题的代码，对于想要赛后AK的队员非常有帮助。

鱼头让我们对照着比赛各个阶段的rank来做下总结，那么就对着rank说吧。
<!-- more -->
[![](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/1.jpg)](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/1.jpg)

第一个小时的时候，节奏不错，10分钟左右的时候，我和老高讨论出A的水题本质，25分钟的时候敲完A，wa了。中间嘉炜把G题的思路搞了出来同时在我wa了以后开始敲，在47~48分钟，嘉炜AC了G，我和老高也讨论出了A的错误，ans每次要初始化为INF，忘了初始化。

[![](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/2.jpg)](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/2.jpg)

看图片也能发现，第二个小时的时候有点茫然，快速把水题切完后，大家花了一个小时看了一下所有的题目，然后讨论了几道题目的想法。然后突然watermelon 看似很轻松的AC了B，让我们的节奏乱了。就想着怎么去暴力搞B了，竟然没有把嘉炜打断一下问问数学方法。

[![](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/3.jpg)](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/3.jpg)

在这整整一个小时里，我和老高都在想怎么搞B比较好，同时在努力看懂F题的意思。这时HJWAJ在敲java的I题，感觉是高精度~！然后情节曲折的发现java超时了！再然后发现一个C++的优化，可以把精度保证在int内，然后果断换成C++ AC了。这中间我抽空开始敲B的暴力搜索解法了！

[![](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/4.jpg)](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/4.jpg)

执迷不误的我们终于发现了B题的搜索不可能实现的本质，可怜我还用各种STL库写了一个巨长的搜索方法。唉，就这样我写了一个多小时毫无用处的代码。伤不起。。。
有兴趣的朋友可以看下。我觉得有必要再多看看STL的东西，在近日发一篇STL常用库函数合集文章。

```
#include<iostream>
#include<set>
#include<vector>
#include<utility>
#include<algorithm>
#include<queue>
#include<stack>
#include<cstdio>
using namespace std;
typedef vector<stack<double> > VSD;
typedef stack<double> SD;

VSD des,src;
double chi[21],math[21],com[21];
int ai,hi,oi;
double fchi[21],fmath[21],fcom[21];
int fai,fhi,foi;
//vector里面【0】是computer
//【1】是Chinese
//【2】是math

void init()
{
	sort(com,com+oi);
	sort(chi,chi+hi);
	sort(math,math+ai);
	sort(fcom,fcom+foi);
	sort(fchi,fchi+fhi);
	sort(fmath,fmath+fai);
	des.clear();
	src.clear();
	SD tp;
	while(!tp.empty())tp.pop();
	for(int i=0;i<oi;i++)
		tp.push(com[i]);
	src.push_back(tp);
	while(!tp.empty())tp.pop();
	for(int i=0;i<hi;i++)
		tp.push(chi[i]);
	src.push_back(tp);
	while(!tp.empty())tp.pop();
	for(int i=0;i<ai;i++)
		tp.push(math[i]);
	src.push_back(tp);

	while(!tp.empty())tp.pop();
	for(int i=0;i<foi;i++)
		tp.push(fcom[i]);
	des.push_back(tp);
	while(!tp.empty())tp.pop();
	for(int i=0;i<fhi;i++)
		tp.push(fchi[i]);
	des.push_back(tp);
	while(!tp.empty())tp.pop();
	for(int i=0;i<fai;i++)
		tp.push(fmath[i]);
	des.push_back(tp);
}

void print()
{
	cout<<"src:"<<endl;
	for(int i=0;i<src.size();i++)
	{
		SD tp = src[i];
		while(!tp.empty())
		{
			cout<<tp.top()<<" ";
			tp.pop();
		}
		cout<<endl;
	}
	cout<<"des:"<<endl;
	for(int i=0;i<des.size();i++)
	{
		SD tp = des[i];
		while(!tp.empty())
		{
			cout<<tp.top()<<" ";
			tp.pop();
		}
		cout<<endl;
	}
}

bool check(VSD s)
{
	VSD tp = des;
	if(tp.size()!=s.size())return false;
	for(int i=0;i<tp.size();i++)
	{
		if(s[i].size()!=tp[i].size())return false;
		while(!s[i].empty())
		{
			if(tp[i].empty())return false;
			if(s[i].top()!=tp[i].top())return false;
			s[i].pop();
			tp[i].pop();
		}
		if(!tp[i].empty())return false;
	}
	return true;
}



int bfs()
{
	queue<pair<VSD,int> >myque;
	set<VSD>myset;
	myset.clear();
	while(!myque.empty())myque.pop();
	myset.insert(src);
	myque.push(make_pair(src,0));
	pair<VSD,int>ind;
	VSD temp;
	while(!myque.empty())
	{
		ind = myque.front();
		myque.pop();
		if(check(ind.first))return ind.second;
		//0->1
		temp = ind.first;
		if(!temp[0].empty())
		{
			if(temp[1].empty()||temp[0].top()>temp[1].top())
			{
				temp[1].push(temp[0].top());
				temp[0].pop();
				if(myset.count(temp)<1)
				{
					myset.insert(temp);
					myque.push(make_pair(temp,ind.second+1));
				}
			}
		}
		//1->0
		temp = ind.first;
		if(!temp[1].empty())
		{
			if(temp[0].empty()||temp[1].top()>temp[0].top())
			{
				temp[0].push(temp[1].top());
				temp[1].pop();
				if(myset.count(temp)<1)
				{
					myset.insert(temp);
					myque.push(make_pair(temp,ind.second+1));
				}
			}
		}
		//1->2
		temp = ind.first;
		if(!temp[1].empty())
		{
			if(temp[2].empty()||temp[1].top()>temp[2].top())
			{
				temp[2].push(temp[1].top());
				temp[1].pop();
				if(myset.count(temp)<1)
				{
					myset.insert(temp);
					myque.push(make_pair(temp,ind.second+1));
				}
			}
		}

		//2->1
		temp = ind.first;
		if(!temp[2].empty())
		{
			if(temp[1].empty()||temp[2].top()>temp[1].top())
			{
				temp[1].push(temp[2].top());
				temp[2].pop();
				if(myset.count(temp)<1)
				{
					myset.insert(temp);
					myque.push(make_pair(temp,ind.second+1));
				}
			}
		}
	}
}

int main()
{
	int n;
	double gpa;
	char str1[100],str2[100];
	while(scanf("%d",&n)!=EOF)
	{
		ai = hi = oi = 0;
		fai = foi = fhi = 0;
		for(int i=0;i<n;i++)
		{
			scanf("%s%s%lf",str1,str2,&gpa);
			switch(str1[1])
			{
				case 'o'://Computer
					com[oi++] = gpa;
					break;
				case 'h'://Chinense
					chi[hi++] = gpa;
					break;
				case 'a':
					math[ai++] = gpa;
					break;
			}
			switch(str2[1])
			{
				case 'o':
					fcom[foi++] = gpa;
					break;
				case 'h':
					fchi[fhi++] = gpa;
					break;
				case 'a':
					fmath[fai++] = gpa;
					break;
			}
		}
		init();
		//print();
		printf("%d\n",bfs());


	}
	return 0;
}
```

[![](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/5.jpg)](https://wonderflow.info/images/2012-07-10-e99b86e8aeade697a5e5bf97e7acace59b9be7af87/5.jpg)

最后一个小时，HJWAJ独当一面，把一道简单的模拟题搞出来了，至此，HJWAJ在本次比赛独立搞出3题，成为最大的功臣~呵呵。我和老高略打酱油了。
其实B应该是可以搞出来的，最后就快想出来了，可惜时间不够了。没办法，在B搜索暴力解法上的时间花的太多了。再有就是最后一个DP题了，可惜了，没能搞出来。
老高说：“我们队的DP和计算几何有待加强。DP是基础，应该每个人都牢牢掌握！”我觉得老高说的非常有道理。
总的来说，这场比赛有所失误，但是庆幸的是，我们都没有卡题，没过也都是因为算法错误。基本上当时有想法的题都过了。


文章的末尾标注一下自己接下来所要做的事情：
1、写个STL常用库函数合集。
2、研究DP专题，写下自己研究DP的一些总结。
3、狗狗40题还得按老样子继续刷！

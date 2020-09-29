---
author: admin
comments: true
date: 2012-07-13 09:16:05+00:00
layout: post
slug: zoj1389-fill-the-cisterns-zoj1425-crossed-matchings
title: 'ZOJ1389 Fill the Cisterns! && ZOJ1425 Crossed Matchings '
wordpress_id: 218
categories:
- ACM
- Zoj
tags:
- ACM
- DP
- Zoj
- 二分
- 狗狗四十题
- 解题报告
---

“Fill the Cisterns!”这个题就是给你很多个相连通的水箱，高度体积各不相同，然后给你一定体积的水，问你最后达到的水箱高度是多少。

做法很简单，就是二分。每次枚举每个水箱，算出可以达到的容量跟标准容量比较，然后调整二分的值。注意精度即可。这个算是《狗狗四十题》里面相对简单的题了。

```
#include<iostream>
#include<cstdio>
using namespace std;
#define EPS 1e-6
struct Node
{
	int b,h,w,d;
}node[50100];
int water;
int n;

bool calc(double m)
{
	double sum = 0;
	for(int i=0;i<n;i++)
	{
		if(node[i].b<m)
		{
			if(m>node[i].b+node[i].h)
				sum+=node[i].h*node[i].w*node[i].d;
			else sum+=node[i].w*node[i].d*(m-node[i].b);
		}
	}
	if(sum>=water)return true;
	return false;
}

int main()
{
	int T;
	scanf("%d",&T);
	while(T--)
	{
		scanf("%d",&n);
		double total=0;
		for(int i=0;i<n;i++)
		{
			scanf("%d%d%d%d",&node[i].b,&node[i].h
					,&node[i].w,&node[i].d);
			total+=node[i].h*node[i].w*node[i].d;
		}
		scanf("%d",&water);
		double l=0,r=10e9;
		double mid;
		if(water>total)
		{
			printf("OVERFLOW\n");
			continue;
		}
		while(l<r)
		{
			mid = (l+r)/2;
			if(calc(mid))r = mid-0.001;
			else l = mid+0.001;
		}
		printf("%.2lf\n",l);
	}

	return 0;
}
```


“Crossed Matchings”是个有意思的题目，给你上下各一列数，让你进行匹配。规则是数字相同的才可以匹配，且每个数字只可以被匹配一次，并且匹配的时候一定要是两个匹配以线段相交的形式出现，且每组匹配只能有一个交点。再然后，还有一个限制条件，就是每组两个匹配的四个字符不能都是一样的。

搞图论这么久，谁一眼看到这个题，会想到是个DP呢？然后题目里还有一句很重要的话：“all numbers are positive integers less than 100”告诉你数据范围的。看下图就应该明白了。

[![](https://wonderflow.info/images/2012-07-13-zoj1389-fill-the-cisterns-zoj1425-crossed-matchings/zoj1425.png)](https://wonderflow.info/images/2012-07-13-zoj1389-fill-the-cisterns-zoj1425-crossed-matchings/zoj1425.png)

```
//all numbers are positive integers less than 100
// so algorithm of O(n^3) is OK
#include<iostream>
#include<memory.h>
#include<cstdio>
using namespace std;
int n,m;
int up[120],down[120];
int dp[120][120];//dp[i][j] means the most matches that up[i] to down[j] can get
//dp[i][j] comes with max(dp[match[j]-1][match[i]-1],dp[i-1][j],dp[i][j-1]);
int findu[120];
int findd[120];


int main()
{
	int T;
	scanf("%d",&T);
	while(T--)
	{
		scanf("%d%d",&n,&m);
		for(int i=1;i<=n;i++)
			scanf("%d",&up[i]);
		for(int j=1;j<=m;j++)
			scanf("%d",&down[j]);
		memset(dp,0,sizeof(dp));
		init();
		for(int i=1;i<=n;i++)
		{
			for(int j=1;j<=m;j++)
			{
				dp[i][j] = max(dp[i][j],dp[i][j-1]);
				dp[i][j] = max(dp[i][j],dp[i-1][j]);
				int fa,fb;
				if(up[i]==down[j])continue;
				for(fa=j-1;fa>0;fa--)
					if(up[i]==down[fa])break;
				for(fb=i-1;fb>0;fb--)
					if(down[j]==up[fb])break;
				if(fa<1||fb<1)continue;
				dp[i][j] = max(dp[i][j],dp[fb-1][fa-1]+1);
				//if(dp[i][j])cout<<i<<" "<<j<<" "<<endl;
			}
		}
		printf("%d\n",dp[n][m]*2);

	}
}
```
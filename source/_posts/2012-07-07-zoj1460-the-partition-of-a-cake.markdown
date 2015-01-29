---
author: admin
comments: true
date: 2012-07-07 04:19:58+00:00
layout: post
slug: zoj1460-the-partition-of-a-cake
title: 'ZOJ1460 The Partition of a Cake '
wordpress_id: 124
categories:
- ACM
- Zoj
tags:
- ACM
- Zoj
- 狗狗四十题
- 解题报告
- 计算几何
---

这真是过的艰辛的一题啊。搞ACM搞到了最后一年，就决定碰到的每道题都不能错过，尤其是好题目。以前总是想着，放着吧，以后研究到的时候总归会做到的。熟不知以后也会遇到新的题目，一直这样推给以后，其实只是欺骗自己而已。所以，哪怕明知其难，也要迎难而上。

果然不出所料，对于我这个计算几何小白，果然搞了很久才搞定。一开始是想着自己找规律的，但是自己找到的规律也没有自己证明，其实就是YY的，果然不出所料，WA了。然后问HJWAJ怎么做，才知道了方法。就是对于一个空间，每切一刀增加的空间数，就等于点数加一。

关于这一个规律的证明，其实想象一下就可以了。如果没有交点，那么显然是一刀把原来切的哪个平面分成两个。如果出现一个交点，就说明这一刀经过了两个平面，它把经过的平面都分成了两个，依次类推就是。

针对这题还有几个注意点：
1、除掉切割在蛋糕边沿的那些线。
2、去除重复的切割线。
3、求两直线交点的时候要先判其是否平行。然后再看交点是否在范围内。
4、一刀切下去，跟自己这一刀切出来的重复点才算重复点。以前就出现的重点的也要计算。

学到的一些知识点：
1、直线求交点
2、两线段求叉积（就是把一条线段平移到跟另一条有公共点)这个可以用来判平行
3、计算几何里面的EPS，是我以前就知道的，但是也在这里写一下作为注意点吧。


n
#include<iostream>
#include<math.h>
#include<algorithm>
#include<cstdio>
using namespace std;
#define EPS 10e-3
struct Point 
{
	double x,y;
};
struct Line
{
	Point a,b;
}line[10];

Point intersect(Line u,Line v)//两线交点
{
	Point ret = u.a;
	double t=((u.a.x-v.a.x)*(v.a.y-v.b.y)-(u.a.y-v.a.y)*(v.a.x-v.b.x))
		/((u.a.x-u.b.x)*(v.a.y-v.b.y)-(u.a.y-u.b.y)*(v.a.x-v.b.x));
	ret.x+=(u.b.x-u.a.x)*t;
	ret.y+=(u.b.y-u.a.y)*t;
	return ret;
}

Point ans[10000];


bool check(Line s)//去除边上cut
{
	if(s.a.x==0&&s.a.y==0)
	{
		if(s.b.x==1000&&s.b.y==0)return true;
		if(s.b.x==0&&s.b.y==1000)return true;
	}
	if(s.b.x==1000&&s.b.y==1000)
	{
		if(s.a.x==0&&s.a.y==1000)return true;
		if(s.a.x==1000&&s.a.y==0)return true;
	}
	if(s.a.x==1000&&s.a.y==0)
	{
		if(s.b.x==1000&&s.b.y==1000)return true;
		if(s.b.x==0&&s.b.y==0)return true;
	}
	if(s.b.x==0&&s.b.y==1000)
	{
		if(s.a.x==1000&&s.a.y==1000)return true;
		if(s.a.x==0&&s.a.y==0)return true;
	}
	return false;
}

bool check2(Line l,Line r)//去重边
{
	if(l.a.x==r.a.x&&l.a.y==r.a.y&&l.b.x==r.b.x&&l.b.y==r.b.y)return true;
	if(l.a.x==r.b.x&&l.a.y==r.b.y&&l.b.x==r.a.x&&l.b.y==r.a.y)return true;
	return false;

}

bool parallel(Line l,Line r)//判平行
{
	if(l.a.x==l.b.x&&r.a.x==r.b.x)return true;
	double k1 = double(l.b.y-l.a.y)/double(l.b.x-l.a.x);
	double k2 = double(r.b.y-r.a.y)/double(r.b.x-r.a.x);
	if(fabs(k1-k2)<EPS)return true;
	return false;
}

int main()
{
	int n;
	while(scanf("%d",&n)&&n)
	{
		for(int i=0;i<n;i++)
		{
			scanf("%lf%lf%lf%lf",&line[i].a.x,&line[i].a.y,&line[i].b.x,&line[i].b.y);
		}
		int ap = 0;
		bool sm ;
		for(int i=0;i<n;i++)
		{
			sm = false;
			if(check(line[i]))continue;
			for(int j=0;j<i;j++)
			{
				if(check2(line[i],line[j]))
				{
					sm = true;
					break;
				}
			}
			if(sm)continue;
			line[ap++] = line[i];
		}
		n = ap;
		int num = 0;
		int cnt=1;
		for(int i=0;i<n;i++)
		{
			int tmp = 0;
			int ssf = -1;
			for(int j=0;j<i;j++)
			{
				if(parallel(line[i],line[j]))continue;//平行则跳过
				ans[num]= intersect(line[i],line[j]);
				if(ans[num].x>0&&ans[num].x<1000&&ans[num].y>0&&ans[num].y<1000);//在界内才算
				else continue;
				bool fs = false;
				if(ssf==-1)ssf = num;
				else 
					for(int k=ssf;k<num;k++)
					{
						if(fabs(ans[k].x-ans[num].x)<EPS
								&&fabs(ans[k].y-ans[num].y)<EPS)
						{
							fs = true;break;
						}
					}
				if(fs)continue;
				num++;
				tmp++;
			}
			cnt+=tmp+1;
		}
		printf("%d\n",cnt);
	}
}

n

---
author: admin
comments: true
date: 2012-07-03 14:33:25+00:00
layout: post
slug: usaco-riding-the-fences
title: USACO Riding the Fences
wordpress_id: 71
categories:
- ACM
- usaco
tags:
- ACM
- usaco
---

本身这个就是一个无向图欧拉回路的题。
欧拉回路的两个特性一是连通，二是点的度数要么都为偶数，要么有且仅有两个奇数
然后写的时候，有个递归，递归的退出的时候才存结果，而不是递归一开始的时候存。
为什么呢？
可能存在这样的情况
1->2->3->4
然后存在3->5->6->3这样一个分支。
先输出的话，按照字典续小的，4就直接被输出了，显然这样就不行了。但是递归的最后才记录的话，说明已经确定了，那就是稳定的情况了。
n
/*
TASK: fence 
LANG: C++
NAME: BOBO
*/
#include<iostream>
#include<vector>
#include<cstdio>
#include<memory.h>
using namespace std;

int f,cnt;
bool vis[1500];
int edge[600][600];
int deg[600];
int ans[600];
int ns;
void dfs(int index)
{
	for(int i=1;i<=cnt;i++)
	{
		if(edge[index][i])
		{
			edge[index][i] = --edge[i][index];
			dfs(i);
		}
	}
	ans[ns++] = index;
}
int main()
{
	int a,b;
	//freopen("fence.in","r",stdin);
	//freopen("fence.out","w",stdout);
	while(scanf("%d",&f)!=EOF)
	{
		ns = cnt = 0;
		memset(edge,0,sizeof(edge));
		memset(deg,0,sizeof(deg));
		for(int i=0;i<f;i++)
		{
			scanf("%d%d",&a,&b);
			edge[a][b] = ++edge[b][a];
			cnt = max(a,cnt);
			cnt = max(b,cnt);
			deg[a]++;
			deg[b]++;
		}
		int num;
		for(num=1;num<=cnt;num++)if(deg[num]%2)break;

			if(num==cnt+1)dfs(1);
			else dfs(num);
		for(int i=ns-1;i>=0;i--)
		{
			printf("%d\n",ans[i]);
		}
	}
}
n      

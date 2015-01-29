---
author: admin
comments: true
date: 2012-07-13 08:31:13+00:00
layout: post
slug: zoj1426-counting-rectangles-zoj1387-decoding-morse-sequences
title: ZOJ1426 Counting Rectangles && ZOJ1387 Decoding Morse Sequences
wordpress_id: 216
categories:
- ACM
- Zoj
tags:
- AC自动机
- DP
- Zoj
- 字符串
- 暴力
- 狗狗四十题
- 解题报告
---

Counting Rectangles 这题一开始都没人做，其实就是个水题。因为题目中说的很清楚，只会有垂直和水平的线。这个时候，我们可以枚举任意两条横线，对于每条竖线，看有多少竖线跟这两条横线相交，设相交数为tmp。最后对于这两条横线和所有的竖线构成的矩形个数就是(temp-1)*temp/2（一个框构成的矩形有temp-1个，两个框构成的矩形有temp-2个。。以此类推至1个）。这样就能在N^3的复杂度内算出所有矩形个数了。
n
/*
枚举任意两条横线，对于每条竖线，看有多少竖线跟这两条横线相交，设相交数为tmp
最后对于这两条横线和所有的竖线构成的矩形个数就是(temp-1)*temp/2
这样就能在N^3的复杂度内算出所有矩形个数了。
 */
#include<iostream>
#include<cstdio>
#include<algorithm>
#include<vector>
#include<memory.h>
using namespace std;
struct Line
{
	int x1,y1,x2,y2;
	Line(){}
	Line(int a,int b,int c,int d)
	{
		if(a>c)swap(a,c);
		if(b>d)swap(b,d);
		x1 = a;
		y1 = b;
		x2 = c;
		y2 = d;
	}
};
vector<Line>h,v;
bool intersect[120][120];

int main()
{
	int T;
	int s;
	int x1,y1,x2,y2;
	scanf("%d",&T);
	while(T--)
	{
		scanf("%d",&s);
		h.clear();
		v.clear();
		memset(intersect,0,sizeof(intersect));
		for(int i=0;i<s;i++)
		{
			scanf("%d%d%d%d",&x1,&y1,&x2,&y2);
			if(x1==x2)v.push_back(Line(x1,y1,x2,y2));
			else h.push_back(Line(x1,y1,x2,y2));
		}
		for(int i=0;i<v.size();i++)
		{
			for(int j=0;j<h.size();j++)
			{
				if(v[i].x1>=h[j].x1&&v[i].x1<=h[j].x2
						&&h[j].y1>=v[i].y1&&h[j].y1<=v[i].y2)
					intersect[i][j]=true;
			}
		}
		int ans,tmp;
		ans = 0;
		for(int i=0;i<v.size();i++)
		{
			for(int j=i+1;j<v.size();j++)
			{
				tmp = 0;
				for(int k=0;k<h.size();k++)
				{
					if(intersect[i][k]&&intersect[j][k])tmp++;
				}
				ans+=(tmp-1)*tmp/2;
			}
		}
		printf("%d\n",ans);

	}
	return 0;
}
n

Decoding Morse Sequences是个字符串的题，要求在一篇莫斯电码构成的文章中，找出给定字典所能构成的文章内容数有多少。
看到10000个字符和10000个单词，当时就虚了，根本不敢做。
结果没想到数据量比较小，暴力匹配加上DP就AC了。
做法就是开一个dp[10000]的数组，表示当前位置为止能用单词组合构成多少篇文章，然后对于dp[i]，遍历字典中的所有单词，看能否跟当前dp[i]的最后一部分匹配。若匹配成功，自然dp[i]+=dp[i-len[j]]。然后这样一直枚举每个位置到底部。

其实要是这种方法没有AC的话，还有两种思路可以优化。
1、在匹配的过程中使用字典树，在匹配的过程中降低O（word_length*wordnum）的复杂度至O（max_word_length)。
2、在匹配的过程中用随机数取代逐个匹配，随机一定次数，简称瞎搞。。。
3、使用AC自动机构成一个字典，然后对全文匹配的过程中调用DP过程（不过这个方法比较虚，让zyz搞了一个下午，各种TLE，MLE，看来对AC自动机的理解还是不够深刻，有待以后再来做这个题）

[cpp title="my bruteforce dp code"]
#include<iostream>
#include<cstring>
#include<memory.h>
#include<cstdio>
#include<string>
using namespace std;
char str[10100];
int dp[10100];
char mos[26][8] = {".-","-...","-.-.","-.."//A,B,C,D
				,".","..-.","--.","...."  //E,F,G,H
				,"..",".---","-.-",".-.." //I,J,K,L
				,"--","-.","---",".--."   //M,N,O,P
				,"--.-",".-.","...","-"   //Q,R,S,T
				,"..-","...-",".--","-..-"//U,V,W,X
				,"-.--","--.."};			  //Y,Z

char word[10010][100];
int len[10010];
char oneword[30];

bool match(int src,int n,int pi)
{
	for(int j=0;j<n;j++)
	{
		if(str[src+j]!=word[pi][j])return false;
	}
	return true;
}

int main()
{
	int T,n;
	scanf("%d",&T);
	while(T--)
	{
		scanf("%s",str);
		scanf("%d",&n);
		memset(len,0,sizeof(len));
		for(int i=0;i<n;i++)
		{
			scanf("%s",oneword);
			word[i][0] = '\0';
			int j=0;
			while(oneword[j])
			{
				strcat(word[i],mos[oneword[j]-'A']);
				len[i]+=strlen(mos[oneword[j]-'A']);
				j++;
			}
		}
		memset(dp,0,sizeof(dp));
		dp[0] = 1;
		int i;
		for(i=1;str[i-1];i++)
		{
			for(int j=0;j<n;j++)
			{
				if(len[j]>i)continue;
				if(match(i-len[j],len[j],j))
				{
					dp[i]+=dp[i-len[j]];
				}
			}
		}
		printf("%d\n",dp[i-1]);

	}
}
n

后来ZYZ终于用AC自动机的方法AC了！
[cpp title="zyz's code"]
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
const char mos[26][5]={".-","-...","-.-.","-..",".","..-.","--.","....","..",".---","-.-",".-..","--","-.","---",".--.","--.-",".-.","...","-","..-","...-",".--","-..-","-.--","--.."};
const int Delta='A';
const int N=2;
const int M=550000;
struct node
{
	int next[N];
	int fail;
	int len;
	//int used;
	int cnt;
}tree[M];
int tp;
void init()
{
	memset(tree,0,sizeof(tree));
	tp=1;
}
void insert(char*key)
{
	int p=0,len=0,index,ind,i,j;
	for(i=0;key[i];i++)
	{
		index=key[i]-Delta;
		len+=strlen(mos[index]);
		for(j=0;mos[index][j];j++)
		{
			ind=mos[index][j]-'-';
			if(tree[p].next[ind]==0)
			{
				tree[p].next[ind]=tp++;
			}
			p=tree[p].next[ind];
		}
	}
	tree[p].len=len;
	//tree[p].used=1;
	tree[p].cnt++;
}
int que[M];
void build()
{
	int i,p=0,q=0,u,v;
	for(i=0;i<N;i++)
	{
		if(tree[0].next[i])
		{
			que[p++]=tree[0].next[i];
		}
	}
	while(q<p)
	{
		u=que[q++];
		for(i=0;i<N;i++)
		{
			v=tree[u].next[i];
			if(v)
			{
				que[p++]=v;
				tree[v].fail=tree[tree[u].fail].next[i];
				//tree[v].used|=tree[tree[v].fail].used;
			}
			else
			{
				tree[u].next[i]=tree[tree[u].fail].next[i];
			}
		}
	}
}
int dp[10010];
char text[10010];
int solve()
{
	int i,q=0,t,ind;
	dp[0]=1;
	for(i=1;text[i-1];i++)
	{	 
		dp[i]=0;
		ind=text[i-1]-'-';
		q=tree[q].next[ind];
		t=q;
		while(t)//&&tree[t].used)
		{
			if(tree[t].len&&i>=tree[t].len)
			{
				dp[i]+=dp[i-tree[t].len]*tree[t].cnt;
			}
			t=tree[t].fail;
		}
	}
	return dp[i-1];
}
int main()
{
	int cas,n;
	char key[25];
	scanf("%d",&cas);
	getchar();
	while(cas--)
	{
		scanf("%s",text);
		init();
		scanf("%d",&n);
		getchar();
		while(n--)
		{
			scanf("%s",key);
			insert(key);
		}
		build();
		printf("%d\n",solve());
	}
	return 0;
}
n

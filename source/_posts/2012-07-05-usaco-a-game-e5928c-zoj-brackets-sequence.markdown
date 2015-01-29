---
author: admin
comments: true
date: 2012-07-05 15:49:27+00:00
layout: post
slug: usaco-a-game-%e5%92%8c-zoj-brackets-sequence
title: USACO A GAME 和 ZOJ1463 Brackets Sequence
wordpress_id: 92
categories:
- ACM
- usaco
tags:
- ACM
- usaco
- Zoj
- 狗狗四十题
- 解题报告
---

今天正好做了两道DP题，所以一起贴了出来，本身这两道题的关联度不大。

[USACO](http://ace.delos.com/usacogate)上这题是个博弈，两个人，轮流取数，规则是只能取一个队列两端的数，然后先手和后手都要使用最优策略，使得最后得到的数之和最大。
问先手和后手最后各得多少分。

一开始怎么也想不出来。后来跟方易凡讨论了一下，发现了他常用的一种思考问题的方式。就是先考虑小的情况。
如果队列中只有一个数字的话，显然，先取的人拿走，后手为0.
有两个数字的话，先手拿较大的那个，后手拿较小的那个。
此时，规模扩大到三个。这下怎么办呢？
其实按照正常的思维方式，就是左右都试试，那么，我取了队列左边的数，队列第二个开始，一直到最右边的数留到了第二轮，在第二轮中，原来的先手变成了后手。也就是说，我当前取得的数值最优策略，其实是我这次取到的数和下次作为后手的最优策略取到的数之和。然后在所有情况中取较大的值。
这个时候，最优子问题也就出来了。

可见，在思考问题的时候，尤其是DP这种，本身的思路就是把大规模的问题转换成子问题的算法上，就要尽可能从小数据上面去思考、去找规律。

第二题是一个经典的括号匹配的DP。
这个题在浙大上是有trick的，就是不能用scanf，只能用gets读入一行，具体为什么，我也很费解。
说下解法。
就是设置一个dp[i][j]数组，表示字符串i到j之间匹配完成时，最少要添加的括号数。
然后开始的时候，dp[i][i]显然是1,当i>j的时候，dp[i][j]显然是0。从一个点向两边扩散开去。
如果str[i]==str[j],dp[i][j] = max(dp[i][k]+dp[k+1][j],dp[i+1][j-1]) 
否则的话，dp[i][j] = max(dp[i][k]+dp[k+1][j])
O（N^3）的复杂度，N<=100，所以可以解决。

然后考虑输出的问题，在POJ上也有一道几乎一样的题目，那个题，我用STL里的string ans[i][j] 直接在每个结点把答案算了一遍，最后直接输出就行了。但是这个方法在zoj上是行不通的，因为会出现[segmentation fault](http://acm.zju.edu.cn/onlinejudge/faq.do)。
然后就要递归找答案了。这个时候我们用一个 int ans[i][j]来记录答案，如果在i和j正好匹配的时候，dp值是最优的，那么ans[i][j]=-1 
否则我们就让ans[i][j] = k。k就是上面那个让我们得到最优解的枚举值。在输出的时候递归一下，就可以解决了，还算是一目了然的。

PS：好久没这么耐心的写解题报告了啊，哈哈~

n
//USACO A GAME
/*
ID:bobo
PROG:game1
LANG:C++
*/
#include<iostream>
#include<cstdio>
#include<memory.h>
using namespace std;
int rec[120];
int n;
int dp[120][120][2];
int dfs(int a,int b,bool f)
{
	if(dp[a][b][1]!=-1&&!f)return dp[a][b][1];
	if(dp[a][b][0]!=-1&&f)return dp[a][b][0];
	int t1 = dfs(a,b-1,0)+rec[b];
	int t2 = dfs(a+1,b,0)+rec[a];
	if(t1>t2)
	{
		dp[a][b][0] = t1;
		dp[a][b][1] = dfs(a,b-1,1);
	}
	else
	{
		dp[a][b][0] = t2;
		dp[a][b][1] = dfs(a+1,b,1);
	}
	if(f)return dp[a][b][0];
	return dp[a][b][1];
}
int main()
{
//	freopen("game1.in","r",stdin);
//	freopen("game1.out","w",stdout);
	while(scanf("%d",&n)!=EOF)
	{
		for(int i=1;i<=n;i++)
			scanf("%d",&rec[i]);
		memset(dp,-1,sizeof(dp));
		for(int i=1;i<=n;i++)
		{
			dp[i][i][0] = rec[i];
			dp[i][i][1] = 0;
		}
		dfs(1,n,1);
		printf("%d %d\n",dp[1][n][0],dp[1][n][1]);

	}

}
n

n
//ZOJ Brackets Sequence
#include<iostream>
#include<cstring>
#include<string>
#include<cstdio>
using namespace std;
#define INF (1<<30)
int dp[110][110];
int ans[110][110];
char str[110];
void print(int a,int b)
{
	if(a>b)return;
	if(a==b)
	{
		if(str[a]=='('||str[a]==')')
			printf("()");
		else printf("[]");
		return;
	}
	if(ans[a][b]==-1)
	{
		if(str[a]=='('||str[a]==')')
		{
			printf("(");
			print(a+1,b-1);
			printf(")");
		}
		else
		{
			printf("[");
			print(a+1,b-1);
			printf("]");
		}
	}
	else
	{
		print(a,ans[a][b]);
		print(ans[a][b]+1,b);
	}
}
int main()
{
	int n;
	scanf("%d",&n);
	gets(str);
	for(int icase=0;icase<n;icase++)
	{
		if(icase)
			printf("\n");
		gets(str);
		gets(str);
		int len = strlen(str);
		memset(dp,0,sizeof(dp));
		for(int i=0;i<len;i++)
		{
			dp[i][i] = 1;
			ans[i][i] = -1;
			for(int j=i+1;j<len;j++)
			{
				dp[i][j] = INF;
			}
		}
		for(int i=len-1;i>=0;i--)
		{
			for(int j=i+1;j<len;j++)
			{
				if(str[i]=='('&&str[j]==')'&&dp[i][j]>dp[i+1][j-1])
				{
					dp[i][j] = dp[i+1][j-1];
					ans[i][j] = -1; 
				}
				if(str[i]=='['&&str[j]==']'&&dp[i][j]>dp[i+1][j-1])
				{
					dp[i][j] = dp[i+1][j-1];
					ans[i][j] = -1;
				}
				for(int k=i;k<j;k++)
				{
					if(dp[i][j]>dp[i][k]+dp[k+1][j])
					{
						dp[i][j] = dp[i][k]+dp[k+1][j];
						ans[i][j] = k;
					}
				}
			}
		}
		print(0,len-1);
		printf("\n");

	}


	return 0;
}

n


---
author: admin
comments: true
date: 2012-07-12 11:56:25+00:00
layout: post
slug: zoj1462-team-them-upzoj1066-square-ice
title: ZOJ1462 Team Them Up! && ZOJ1066 Square Ice
wordpress_id: 204
categories:
- ACM
- Zoj
tags:
- ACM
- C++
- DP
- 反图
- 染色
- 模拟
- 狗狗四十题
- 解题报告
---

今天似乎是被《team them up》虐了一天啊。昨天就知道了解法，就是一直wa。中间还去把当年正常比赛官方数据下载下来对比了一下。结果发现zoj和poj的数据竟然是加强了的。。。。

说下题意：一堆人要分班级，人与人之间有的互相认识，有的不认识。现在只有两个班，但是我分班级有限制条件。
1、一个班级的同学都要互相认识。
2、一个班级里一定要至少有一个人。
3、两个班级里的人数要尽可能靠近

看到这个题可能会没有什么想法。比如我就是这样。然后根据以前的经验，从小数据开始，如果1个人的话怎么分，2个人的话怎么分等等。后来发现这个不存在一个递推关系，因为人之间还有图的关系。
然后就问了一下薛斌怎么想的，发现他的思路很好。
一开始人与人之间互相认识，而且只有认识的才能在一个班。那么不认识的必然不能再一个班。这样我们把互相之间不认识的人连到一起，最后又变成了一个跟之前做过的一道[《this sentence is false》](http://wonderflow.info/archives/131)有些类似了。做法也是类似。就是把连在一起的互相不认识的人染色。直接相连的人都是必须要0~1染不同的颜色的。如果发现有一个点要染成0，同时它已经被染成了1，或者相反。那么就是矛盾出现了。
最后我们就得到了一个个被染成了0~1的块。
问题也转换成了：现在有很多个物品，每个物品有两个部件组成，而这两个部件要分别放到不同的袋子里。最后还要求这两个袋子的重量尽可能相近。
很容易想到这是一个经典的DP问题。区别于传统的01背包。现在只是把原来的01背包里的0换掉。改成了A、B背包。做法就是两个数组，互相交替着存一轮一轮可以达到的DP值。
[![](http://wonderflow.info/wp-content/uploads/2012/07/11.png)](http://wonderflow.info/wp-content/uploads/2012/07/11.png)
最后从n/2处开始往前找能达到的DP值。
关于怎么把答案输出，我是用的vector，在求DP的时候顺便把答案也一起都存下来了。

然后就要说说困扰了我一下午的原因了：就是dp[0]=true的情况，用过一次后就该刷新的，谁知我手贱，写for循环的时候从1开始，这下就给跪了。后来薛斌来了一组神数据，瞬间就让我给过了。附在代码下面。

n
/*
 * step 1: use reverse edge to make strong connected graph
 * step 2: use 0 or 1 to color every strong connected components  
  			and count the number of them . name them as con[i][2];
 * step 3: for every components do a DP problem to decide which class to put
 *
 * there comes the question: how to solve the DP problem?
 * make an n sized array as dp[N]; 
 * for every con[i],try to put the number.
 */
#include<iostream>
#include<cstdio>
#include<memory.h>
#include<vector>
#include<algorithm>
#define MAXN 128
using namespace std;
int n;
bool edge[MAXN][MAXN];
int color[MAXN];
int con[MAXN];
bool vis[MAXN];
vector<int> rec[MAXN][2];
bool over;
bool dp[MAXN];
vector<int>ans[MAXN];
vector<int>ansp[MAXN];
bool tdp[MAXN];
//make reverse edge to get strong connected graph
void dfs(int v,int num)
{
	con[v] = num;
//	cout<<"ver"<<v<<" "<<num<<endl;
	for(int i=1;i<=n;i++)
	{
		if((!edge[v][i]||!edge[i][v])&&!con[i])
			dfs(i,num);
	}
}

//paint every connected graph
void paint(int v,int c)
{
	rec[con[v]][c].push_back(v);
	color[v] = c;
	//cout<<v<<" "<<c<<endl;
	if(over)return;
	for(int i=1;i<=n;i++)
	{
		if(i==v)continue;
		if(con[v]==con[i]&&(!edge[v][i]||!edge[i][v]))
		{
			if(color[i] == -1)paint(i,1^c);
			else if(color[i]==c)
			{
				over = true;
			//	cout<<"over; "<<i<<endl;
				return;
			}
		}
	}
}

int main()
{
	int T;
//	freopen("xb.in","r",stdin);
	scanf("%d",&T);
	int icase=0;
	while(T--)
	{
		if(icase)printf("\n");
		memset(edge,0,sizeof(edge));
		memset(con,0,sizeof(con));
		memset(color,-1,sizeof(color));
		icase++;
		scanf("%d",&n);
		int t,m;
		for(int i=1;i<=n;i++)
		{
			while(scanf("%d",&t)&&t)
				edge[i][t] = true;
		}
		int num = 1;
		for(int i=1;i<=n;i++)
		{
			if(!con[i])dfs(i,num++);
		}
		over = false;
		for(int i=0;i<=n;i++)
		{
			rec[i][0].clear();
			rec[i][1].clear();
			ansp[i].clear();
			ans[i].clear();
		}
		for(int i=1;i<=n;i++)
		{
			if(color[i]==-1)
				paint(i,0);
		}
		if(over)
		{
			printf("No solution\n");
			continue;
		}
		else
		{
			memset(tdp,0,sizeof(tdp));
			memset(dp,0,sizeof(dp));
			tdp[0] = true;
			for(int i=1;i<num;i++)
			{
				for(int j=n;j>0;j--)
				{
					if(rec[i][0].size()<=j&&tdp[j-rec[i][0].size()])
					{
						dp[j] = true;
						int t = j-rec[i][0].size();
					ans[j].clear();
						for(int k=0;k<ansp[t].size();k++)ans[j].push_back(ansp[t][k]);
						for(int k=0;k<rec[i][0].size();k++)
							ans[j].push_back(rec[i][0][k]);
					}
					if(rec[i][1].size()<=j&&tdp[j-rec[i][1].size()])
					{
						dp[j] = true;
					ans[j].clear();
						int t = j-rec[i][1].size();
						for(int k=0;k<ansp[t].size();k++)ans[j].push_back(ansp[t][k]);
						for(int k=0;k<rec[i][1].size();k++)ans[j].push_back(rec[i][1][k]);
					}
				}
				for(int j=0;j<=n;j++)
				{
					ansp[j].clear();
					for(int k=0;k<ans[j].size();k++)
						ansp[j].push_back(ans[j][k]);
					tdp[j] = dp[j];
				}
				for(int j=0;j<=n;j++)
				{
					dp[j] = 0;
					ans[j].clear();
				}
			}
			memset(vis,0,sizeof(vis));
			for(int i=(n+1)/2;i>0;i--)
			{
				if(tdp[i])
				{
					sort(ansp[i].begin(),ansp[i].end());
					printf("%d",ansp[i].size());
					for(int k=0;k<ansp[i].size();k++)
					{
						printf(" %d",ansp[i][k]);
						vis[ansp[i][k]] = true;
					}
					printf("\n");
					printf("%d",n-ansp[i].size());
					for(int k=1;k<=n;k++)
						if(!vis[k])
							printf(" %d",k);
					printf("\n");
					break;
				}
			}
		}

	}
}

/*
1
9
2 5 6 7 8 9 0
1 5 6 7 8 9 0
4 5 6 7 8 9 0
3 5 6 7 8 9 0
1 2 3 4 0
1 2 3 4 5 7 8 9 0
1 2 3 4 5 6 8 9 0
1 2 3 4 5 6 7 9 0
1 2 3 4 5 6 7 8 0
*/
n

zoj1066是个简单的题。怎么搞都可以搞出来。但是如果注意读题的话，或许解出此题的速度会加快很多。
做法：-1和1的情况都是很好处理的，注意到他说每行、每列最后的和是1，那么在一行的和成为1之前，0的一个拐角都是拐向左边的，成为1之后，就是拐向右边，一列的情况则是成为1之前拐向上面，成为1之后拐向下面。
当然，不管这个，凭感觉去写，先把-1和1的情况处理好，然后0的情况则是四边去尝试，最后也是能过的，不过就是考虑的情况多了点，会觉得比较烦。。。我就是这么过的。。。呵呵~~

n
#include<iostream>
#include<memory.h>
#include<cstdio>
using namespace std;
int rec[20][20];
bool hh[100][100];
bool edge[100][100];



int main()
{
	int n;
	int icase = 0;
	while(scanf("%d",&n)&&n)
	{
		if(icase)puts("");
		icase++;
		for(int i=0;i<n;i++)
		{
			for(int j=0;j<n;j++)
			{
				scanf("%d",&rec[i][j]);
			}
		}
		memset(edge,0,sizeof(edge));
		memset(hh,0,sizeof(edge));
		for(int i=0;i<4*n-1;i++)
		{
			for(int j=0;j<4*n+3;j++)
			{
				if((i-1)%4==0&&(j-1)%4==2)
				{
					int x = (i-1)/4;
					int y=  (j-1)/4;
				//	cout<<rec[x][y]<<" "<<x<<" "<<y<<endl;
					switch(rec[x][y])
					{
						case 1:
							hh[i][j-1] = hh[i][j+1] = true;
							hh[i][j-2] = hh[i][j+2] = true;
							break;
						case -1:
							hh[i-1][j] = hh[i+1][j] = true;
							hh[i-2][j] = hh[i+2][j] = true;
							break;
					}
				}
			}
		}
		for(int i=0;i<4*n-1;i++)
		{
			for(int j=0;j<4*n+3;j++)
			{
				if((i-1)%4==0&&(j-1)%4==2)
				{
					int x = (i-1)/4;
					int y=  (j-1)/4;
					
					if(!rec[x][y])
					{
						if(!hh[i][j-2])
						{
							if(i-2>0&&!hh[i-2][j])
							{
								hh[i][j-1] = hh[i-1][j] = true;
								hh[i][j-2] = hh[i-2][j] = true;
								continue;
							}
							if(i+2<4*n-1&&!hh[i+2][j])
							{
								hh[i][j-1] = hh[i+1][j] = true;
								hh[i][j-2] = hh[i+2][j] = true;
								continue;
							}
						}
						if(!hh[i][j+2]) 
						{

							if(i-2>0&&!hh[i-2][j])
							{
								hh[i][j+1] = hh[i-1][j] = true;
								hh[i][j+2] = hh[i-2][j] = true;
								continue;
							}
							if(i+2<4*n-1&&!hh[i+2][j])
							{
								hh[i][j+1] = hh[i+1][j] = true;
								hh[i][j+2] = hh[i+2][j] = true;
								continue;
							}
						}

					}
				}
			}
		}


		printf("Case %d:\n\n",icase);
		for(int i=0;i<4*n-1;i++)
		{
			for(int j=0;j<4*n+3;j++)
			{
				if(!i||i==4*n-2||!j||j==4*n+2)
				{
					printf("*");
					continue;
				}
				if(((i-1)%4==0&&(j-1)%4==0)
					||((i-1)%4==2&&(j-1)%4==2))
				{
					printf("H");
					continue;
				}
				if((i-1)%4==0&&(j-1)%4==2)
				{
					printf("O");
					continue;
				}
				if((i-1)%4==0&&(j-1)%2&&hh[i][j])
				{
					printf("-");
					continue;
				}
				if((i-1)%2&&(j-1)%4==2&&hh[i][j])
				{
					printf("|");continue;
				}
				printf(" ");

			}
			printf("\n");
		}

	}
}
n



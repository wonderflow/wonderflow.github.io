+++
author = "admin"
date = "2012-07-14T16:33:22Z"
slug = "2012/07/14/zoj3475-the-great-wall-ie69c80e5b08fe589b2efbc88dinicefbc89"
title = "ZOJ3475 The Great Wall I(最小割)（Dinic）"
Categories = ["ACM", "Zoj"]
Tags = ["ACM", "Dinic", "wonderflow", "最小割", "网络流", "解题报告"]
+++

今天第二场组队赛，就搞出了这个题。说来去年搞了一个暑假图论网络流神马的，在自己的博客前面这么多题目解题报告里面竟然没有出现过，实在不应该。
题意：
对于王国X，存在敌对国E和附属国A，我们要建造围墙把敌对国隔开，附属国会给钱你，如果你建造的城墙把它围进去的话。当然，X城和附属国不一定要连通。这个就是和这一套题里的神题《The Great Wall II》唯一的区别了。另外题目里还说，地图以外的区域也要建造围墙隔起来。

看到我的做法和watashi发的[解题报告](http://blog.watashi.ws/1877/zojmonthly1102/)里面的方法有所区别，觉得还是有必要写一下题解。

思路：看到这个题，基本就觉得是最小割了。为什么？感觉啊~！因为要把两个区域分开，然后分开还需要费用，很容易想到是网络流。然后剩下的问题就是怎么建边了。因为地图以外和敌对国家都是必须要隔开的，所以可以把地图以为的那个点设置为汇点，所有敌对国家向汇点连无穷大的边。然后源点自然是X国，边就是按照输入给的边来建，按照无向图建边。最后要解决的问题就是附属国的问题了。
当时跟老高稍微想了一下，就想到，最后，附属国肯定是属于X国部分或者敌国部分，要是一开始我就把附属国给的钱全算上，然后如果附属国被分割到敌国部分去，就再花费那个附属国贡献的钱就可以了。这样就很容易想到，只要源点和附属国再连一条边，流量为附属国贡献的钱就可以了。
至此，建边的过程就完成了。用我的dinic敲了一下，就过了。第一次wa是因为太兴奋，忘了把freopen注释掉就交了。。

另外，我的dinic模板经过一年多的检验，发现一直不错，效率和准确性等都有保证，大家可以放心使用。

```
//WonderfloW's dinic Template
#include<iostream>
#include<memory.h>
#include<cstdio>
using namespace std;
#define Node 20100 //点数
#define Edge 60400
#define INF ((1<<30)-1)
#define min(x,y) ((x)<(y)?(x):(y))
#define flowtype int

struct Graph
{
	struct G_Edge
	{
		int to,next;
		flowtype val;
	}edge[Edge];
	int head[Node];
	int level[Node];
	int que[Node];
	int vis[Node];
	int p,N;
	void init()
	{
		p=0;
		for(int i=0;i<=N+1;i++)head[i] = -1;
	}
	void insert(int a,int b,flowtype c,flowtype c_rev)
	{
		edge[p].next = head[a];
		edge[p].to = b;
		edge[p].val = c;
		head[a] = p++;
		edge[p].next = head[b];
		edge[p].to = a;
		edge[p].val = c_rev;
		head[b] = p++;
	}
	bool bfs_level(int src,int des)
	{
		memset(level,-1,sizeof(level));
		int q,p,index,i;
		p = q = 0;
		que[q++] = src;
		level[src] = 0;
		while(p!=q)
		{
			index = que[p++];
			for(i=head[index];i!=-1;i=edge[i].next)
			{
				int v= edge[i].to;
				if(edge[i].val>0&&level[v]==-1)
				{
					level[v] = level[index]+1;
					que[q++] = v;
				}
			}
		}
		return level[des]!=-1;
	}
	flowtype dinic(int cur,int des,flowtype lim)
	{
		if(cur==des)return lim;
		int index,i;
		flowtype mul_flow = 0,one_flow;
		for(int i=head[cur];i!=-1&&lim-mul_flow>0;i=edge[i].next)
		{
			index = edge[i].to;
			if(edge[i].val>0&&level[cur] == level[index]-1)
			{
				one_flow = dinic(index,des,min(edge[i].val,lim-mul_flow));
				if(!one_flow)level[index] = -1;
				else
				{
					mul_flow+=one_flow;
					edge[i].val-=one_flow;
					edge[i^1].val+=one_flow;
				}
			}
		}
		return mul_flow;
	}
	flowtype max_flow(int src,int des)
	{
		flowtype ans = 0;
		while(bfs_level(src,des))ans+=dinic(src,des,INF);
		return ans;
	}
};
Graph g;
struct NN
{
	int a,b,c;
}adn[10];
int main()
{
	int n,m;
	int k;
	int a,b,c;
//	freopen("in.txt","r",stdin);
	while(scanf("%d%d",&n,&m)!=EOF)
	{
		g.N = n*m+3;
		g.init();
		for(int i=0;i<=2*n;i++)
		{
			for(int j=0;j<m+i%2;j++)
			{
				scanf("%d",&c);
				int x = i/2;
				int y;
				if(i%2==0)y=j;
				else y=j-1;
				if(x<0||x>=n||y<0||y>=m)a=n*m;
				else a = x*m+y;
				if(i%2==0)x--;
				else y++;
				if(x<0||y>=m)b=n*m;
				else b = x*m+y;
				g.insert(a,b,c,c);
			//	cout<<a<<" "<<b<<" "<<c<<endl;
			}
		}
		scanf("%d",&k);
		int s;
		for(int i=0;i<k;i++)
		{
			scanf("%d%d%d",&adn[i].a,&adn[i].b,&adn[i].c);
			if(!adn[i].a)
			{
				s = adn[i].b*m+adn[i].c;
				continue;
			}
			if(adn[i].a<0)
			{
				g.insert(adn[i].b*m+adn[i].c,m*n+1,INF,INF);
			//	cout<<"d: "<<adn[i].b*m+adn[i].c<<endl;
			}
		}

		//cout<<"s:"<<s<<endl;
		int sum = 0;
		for(int i=0;i<k;i++)
		{
			if(adn[i].a>0){
				sum+=adn[i].a;
				g.insert(s,adn[i].b*m+adn[i].c,adn[i].a,0);
			//	cout<<"a: "<<adn[i].b*m+adn[i].c<<endl;
			}
		}
		g.insert(m*n,m*n+1,INF,0);
		int maxflow = g.max_flow(s,m*n+1);
	//	cout<<maxflow<<endl;
	//	cout<<sum<<endl;
		printf("%d\n",maxflow-sum);


	}
}
```


顺带总结下今天的比赛：
前两个半小时节奏不错，过了四个题，中间一个小时卡思路非常严重，前面3个人都有题目敲，甚至有点抢键盘的感觉，但是后来分配的还是很合理的。不过中间的这一个小时基本就是大家都对题目没有什么想法。然后后面想出来的两个题，敲完了都没过，不过老高还是很给力的最后把G题的公式给推了出来，犀利啊！H和C没过确实非常可惜。加油吧，总体发挥的还算稳定。

后来去看了下当年星姐他们做的情况，可能是因为当时是寒假的原因吧。。做的真是寂寞如雪啊。。。呵呵~

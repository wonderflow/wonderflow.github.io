+++
author = "admin"
date = "2012-08-14T02:35:52Z"
slug = "2012/08/14/hdu4337-king-arthurs-knightsefbc88e6b182e4b880e69da1e59388e5af86e5b094e9a1bfe59b9ee8b7afefbc89"
title = "HDU4337 King Arthur's Knights（求一条哈密尔顿回路）"
Categories = ["ACM"]
Tags = ["解题报告"]
+++

这是2012多校第四场的一道题目，觉得蛮有实际价值的。

题意：很多骑士一起聚餐，互相是朋友的骑士想要坐在一起，保证每个骑士跟全场一半以上的人是朋友，要你输出一种座位方式，让每个骑士左右两边都是朋友。

分析：每个骑士就是一个点，左右两边的人分别就是出边和入边，也就是给你一张图，让你求出一条哈密尔顿回路。而一张图存在哈密尔顿回路的充要条件就是任意两个点的度数之和大于等于总点数。而题目中描述的每个骑士跟全场至少一半的人是朋友，任意两个骑士肯定就跟全场的人都是朋友咯。所以肯定存在一条哈密尔顿回路。

哈密尔顿回路的求法：

1、首先要证明图G是连通的。用反证法，若G不连通，则G至少分成两部分，一部分顶点集为V1，另外一部分顶点集为V2，令|V1|=n1，|V2|=n2，则n1+n2=n。在V1中取一个顶点v1(属于V1集合),d(v1)<=n1-1,在V2中取一个顶点v2（属于V2集合）,d(v2)<=n2-1,所以d(v1)+d(v2)<=n1+n2-2<n-1，与已知的矛盾。（d(v)表示顶点v的度数）

2、然后开始找哈密尔顿回路：设(v1,v2,...vp)是G中的一条连通的路，且v1与vp就是通路中两边的端点，这样的通路只要从1号点开始深搜即可，肯定能找到。下面证明这条通路可以构成一条回路，若v1和vp本来就是相连的，那么问题解决了。否则在v1到vp的这条路径上找一个点vi，这个点本身和v1存在路径(即存在（v1,vi）这条边)，而这个点的前面一个点vi-1和vp存在一条路径(存在（vi-1,vp）这条边)，那么去掉原来(vi-1,vi)这条边，现在的这个图就是一个回路。（这样的点一定是存在的，因为每两个点都跟整张图上的每个点都有边）
[![](https://wonderflow.info/images/2012-08-14-hdu4337-king-arthurs-knightsefbc88e6b182e4b880e69da1e59388e5af86e5b094e9a1bfe59b9ee8b7afefbc89/无标题.png)](https://wonderflow.info/images/2012-08-14-hdu4337-king-arthurs-knightsefbc88e6b182e4b880e69da1e59388e5af86e5b094e9a1bfe59b9ee8b7afefbc89/无标题.png)

3、如果p点就是n点，就是把所有的点都已经包括到我们找到的圈里面了，那么算法结束。否则取与圈中某点相连，但是不在圈中的点，加入到圈里面，由于图G是连通的，所以如果p<n，这样的点一定存在，设为vx。然后以vx为起点，再找一条简单通路，这条通路商的点是全都没有加入到我们上面的那个圈里面的。然后把vx和我们圈中那个与vx想连的点连起来，原来的圈拆开，就又出现了一条更长的通路，用第二步的方法把这条路变成环就可以了。循环第2、3两步直到结束。


```
#include<iostream>
#include<time.h>
#include<cstring>
#include<cstdio>
#include<vector>
#include<assert.h>
using namespace std;
int n,m;
bool edge[256][256];

struct Node{
	int v,pre,next;
}node[1024];
int _p;
bool vis[256];

int getroad(int front){
	int index = front;
	int tp;
	while(true){
		tp = node[index].v;
		vis[tp] = true;
		bool fd = false;
		for(int i=1;i<=n;i++){
			if(edge[tp][i]&&!vis[i]){
				node[index].next = _p;
				node[_p].pre = index;
				node[_p].v = i;
				index = _p++;
				fd = true;
				break;
			}
		}
		if(!fd)break;
	}
	return index;
}

int main(){
	int a,b;
	while(scanf("%d%d",&n,&m)!=EOF){
		memset(edge,0,sizeof(edge));
		for(int i=0;i<m;i++){
			scanf("%d%d",&a,&b);
			edge[a][b] = true;
			edge[b][a] = true;
		}
		memset(vis,0,sizeof(vis));
		memset(node,0,sizeof(node));
		_p = 0;
		int index = 0;
		int front = index;
		int t1=0,t2;
		node[_p++].v = 1;
		int num = 0;
		while(true){
			num++;
			if(num>n)break;
			t2 = getroad(t1);
			if(t1==front){
				index = t2;
			}else{
				for(int i=front;i!=index;i=node[i].next){
					if(edge[node[i].v][node[t1].v]){
						front = node[i].next;
						node[i].next = t1;
						node[t1].pre = i;
						index = t2;
						break;
					}
				}
			}
		//	cout<<"front: "<<node[front].v<<" "<<node[index].v<<endl;
			if(edge[node[front].v][node[index].v]){
				node[front].pre = index;
				node[index].next = front;
			}else for(int i=node[front].next;i!=index;i=node[i].next){
				if(edge[node[front].v][node[i].v]&&edge[node[node[i].pre].v][node[index].v]){
					int ms = node[i].pre;
					node[front].pre = i;
					node[i].pre = node[i].next;
					node[i].next = front;
					node[ms].next = index;
					node[index].next = node[index].pre;
					for(int p=node[index].pre;p!=i;){
						int t = node[p].pre;
						node[p].pre = node[p].next;
						node[p].next = t;
						p = t;
					}
					node[index].pre = ms;
					break;
				}
			}
			bool ov = true;
			for(int i=1;i<=n;i++){
				if(!vis[i]){
					node[_p].v = i;
					t1 = _p++;
					ov = false;
					break;
				}
			}
			if(ov)break;
		}
		printf("%d",node[0].v);
		for(int i=node[0].next;i;i=node[i].next){
			printf(" %d",node[i].v);
		}
		printf("\n");
	}
	return 0;
}
```

附一个类似的证明方法的定理：

G（V，E）是一个简单图，若对于V中任意两个顶点v1,v2属于V，d(v1)+d(v2)>=n-1,其中|V|=n，则G中有哈密尔顿通路。

---
author: admin
comments: true
date: 2012-07-21 16:34:28+00:00
layout: post
slug: zoj1103-zoj1116-a-well-formed-problem-zoj1197-zoj1321-parallelepiped-walk
title: ZOJ1103 && ZOJ1116 A Well-Formed Problem && ZOJ1197 && ZOJ1321 Parallelepiped
  Walk
wordpress_id: 270
categories:
- ACM
- Zoj
tags:
- ACM
- 大自然题
- 拓扑排序
- 搜索
- 模拟
- 狗狗四十题
- 解题报告
- 计算几何
---

zoj1103 《Hike on a Graph》就是一个宽搜，问你多少步能让3个棋子都走到一起，三个棋子走的时候，不用轮流走，但是走的边的颜色一定要是对手两个棋子所连边的颜色相同的。当然，它给你的是一个矩阵，两两点之间的颜色都是给你的，还是很简单的题。

n
#include<iostream>
#include<cstring>
#include<cstdio>
#include<queue>
using namespace std;
int n;
char map[60][60];
struct State
{
	int x,y,z;
	int val;
}src,asdx,temp;
queue<State>myque;
char tp[10];
bool vis[60][60][60];

int bfs()
{
	while(!myque.empty())myque.pop();
	memset(vis,0,sizeof(vis));
	src.val = 0;
	myque.push(src);
	vis[src.x][src.y][src.z] = true;
	while(!myque.empty())
	{
		asdx = myque.front();
		myque.pop();
		if(asdx.x==asdx.y&&asdx.x==asdx.z)return asdx.val;
		for(int i=0;i<n;i++)
		{
			if(map[asdx.x][i]==map[asdx.y][asdx.z]&&!vis[i][asdx.y][asdx.z])
			{
				temp.x = i;
				temp.y = asdx.y;
				temp.z = asdx.z;
				vis[temp.x][temp.y][temp.z] = true;
				temp.val = asdx.val+1;
				myque.push(temp);
			}
			if(map[asdx.y][i]==map[asdx.x][asdx.z]&&!vis[asdx.x][i][asdx.z])
			{
				temp.x = asdx.x;
				temp.y = i;
				temp.z = asdx.z;
				vis[temp.x][temp.y][temp.z] = true;
				temp.val = asdx.val+1;
				myque.push(temp);
			}
			if(map[asdx.z][i]==map[asdx.x][asdx.y]&&!vis[asdx.x][asdx.y][i])
			{
				temp.x = asdx.x;
				temp.y = asdx.y;
				temp.z = i;
				vis[temp.x][temp.y][temp.z] = true;
				temp.val = asdx.val+1;
				myque.push(temp);
			}
		}
	}
	return -1;
}

int main()
{
	while(scanf("%d",&n)&&n)
	{
		scanf("%d%d%d",&src.x,&src.y,&src.z);
		src.x--;
		src.y--;
		src.z--;
		for(int i=0;i<n;i++)
		{
			for(int j=0;j<n;j++)
			{
				scanf("%s",tp);
				map[i][j] = tp[0];
			}
		}
		int ans = bfs();
		if(ans!=-1)printf("%d\n",ans);
		else printf("impossible\n");

	}
	return 0;
}
n

zoj1116 《A Well-Formed Problem》又是一个恶心的模拟题，描述的是分析XML格式是否符合要求，关于是否符合要求，就是他给你的5条，逐个看就行了。写这个一定要细心，就像写编译原理课设似的，处理好每一步的情况就行。设置一个栈存放属性的开始和闭合，设置一个set，检查是否唯一。总之很恶心。看着watashi的代码敲的，学到了好多。比如宏定义一些小函数，用起来很方便。比如定义一个类，每次在主函数里简单的调用几个成员函数即可。
n
#include<iostream>
#include<cstring>
#include<stack>
#include<set>
#include<string>
#include<cctype>
#include<cstdio>
using namespace std;
#define SKIP(p) while(isspace(*p))p++
#define GET(p) while(isalnum(*p)||*p=='-')p++
#define ERROR error=true;return NULL
#define EAT(p) while(*p!='"'&&*p!='\0')p++
struct XML
{
	bool empty,error;
	set<string>st;
	stack<string>sk;

	XML():empty(true),error(false)
	{st.clear();while(!sk.empty())sk.pop();}

	void push(string tag)
	{
		if(st.count(tag)>0)error = true;
		else{
			empty = false;
			sk.push(tag);
			st.insert(tag);
		}
	}

	void pop(string tag)
	{
		if(sk.empty()||tag!=sk.top()){
			error = true;
		}else{
			sk.pop();
			st.erase(tag);
		}
	}

	char* solve(char * str)
	{
		char *p = str;
		bool end = false;
		set<string>prop;
		prop.clear();
		SKIP(p);
		if(*p=='/')
		{
			end = true;
			p++;
			SKIP(p);
		}
		char *q = p;
		GET(p);
		string sts = string(q,p);
		if(end)pop(sts);
		else push(sts);
		if(error)return NULL;
		SKIP(p);
		while(*p!='\0'&&*p!='/'&&*p!='>')
		{
			q = p;
			GET(p);
			sts = string(q,p);
			if(prop.count(sts)>0){
				ERROR;
			}else{
				prop.insert(sts);
			}
			SKIP(p);
			if(*p!='='){
				ERROR;
			}
			p++;
			SKIP(p);
			if(*p!='"'){
				ERROR;
			}
			p++;
			EAT(p);
			if(*p=='\0'){
				ERROR;
			}
			p++;
			SKIP(p);
		}
		if(*p=='/')
		{
			if(sk.empty()){
				ERROR;
			}
			sts = sk.top();
			st.erase(sts);
			sk.pop();
			p++;
			SKIP(p);
		}
		if(*p!='>'){
			ERROR;
		}
		return p+1;
	}

	void feed(char*p){
		while(!error&&*p!='\0'){
			if(*p=='<'){
				p=solve(p+1);
			}else{
				p++;
			}
		}
	}
};

int main()
{
	char buf[10000];
	gets(buf);
	do{
		XML xml;
		while(gets(buf)
				&&strcmp(buf,"<?xml version=\"1.0\"?>")
				&&strcmp(buf,"<?end?>"))
		{
			xml.feed(buf);
		}
		if(xml.error||xml.empty||!xml.sk.empty())puts("non well-formed");
		else puts("well-formed");
	}while(strcmp(buf,"<?end?>"));
	return 0;
}
n

ZOJ1197 《Sorting Slides》是一个匹配的题目，很多个重合的矩形框以及很多个点，问你能否把这些点和矩形框一一对应。比如如果某个点只在一个矩形框内，就表示这个矩形框对应这个点，如果某个矩形框去掉了很多个已经被匹配走的点后只剩下一个点了，那么这个点就属于这个矩形框。
最后让你输出的就是匹配的情况。
用匈牙利算法做很多遍匹配，每次去掉一条边看是不是能让匹配数变少，如果让匹配数变少了，那么自然这就是我们要的边，然后不断的做下去，看有没有矛盾或者匹配不到。

n
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <algorithm>
using namespace std;
#define maxn 30

bool MaxMatchDFS(bool g[][maxn], int m, int a, int y[], bool u[])
{
    for (int i=0; i<m; i++)
        if (!u[i] && g[a][i])
        {
            int t=y[i];
            u[i]=true; y[i]=a;
            if (t==-1 || MaxMatchDFS(g, m, t, y, u)) return true;
            y[i]=t;
        }
    return false;
}
int MaxMatch(bool g[][maxn], int n, int m) //v1[y[i]] matches v2[i]
{
    int y[maxn];
    memset(y, 255, sizeof(y));
    int c=0;
    for (int i=0; i<n; i++)
    {
        bool u[maxn]={0};
        if (MaxMatchDFS(g, m, i, y, u)) c++;
    }
    return c;
}

bool init(bool g[][maxn], int &n)
{
    int xmin[maxn], xmax[maxn], ymin[maxn], ymax[maxn];
    int x[maxn], y[maxn];
    scanf("%d", &n);
    if (n==0) return false;
    for (int i=0; i<n; i++)
        scanf("%d%d%d%d", &xmin[i], &xmax[i], &ymin[i], &ymax[i]);
    for (int i=0; i<n; i++)
        scanf("%d%d", &x[i], &y[i]);
    memset(g, false, sizeof(*g)*maxn);
    for (int i=0; i<n; i++)
        for (int j=0; j<n; j++)
        {
            if (xmin[i]<x[j] && xmax[i]>x[j]
                && ymin[i]<y[j] && ymax[i]>y[j])
            g[i][j]=1;
        }
    return true;
}

int main()
{
    int cs=0;
    int n, unique, match; 
    bool g[maxn][maxn];
    bool found;
    while (init(g, n))
    {
        found=false;
        printf("Heap %d\n", ++cs);
        for (int i=0; i<n; i++)
        {
            unique=-1;          //0:false 1:true -1:not sure
            match=-1;
            for (int j=0; j<n; j++)
                if (g[i][j])
                {
                    g[i][j]=false;
					//cut an edge and try whether it's a right match
                    if (MaxMatch(g, n, n)==n-1)
                    {
                        if (unique==-1){unique=1;match=j;}
                        else {unique=0; break;} //如果当前有多个这样的边，显然也是矛盾的
                    }
                    g[i][j]=true;
                }
            if (unique==1)
            {
                if (found==false) found=true;
                else printf(" ");
                printf("(%c,%d)", i+'A', match+1);
            }
        }
        if (found==false) printf("none\n\n");
        else printf("\n\n");
    }
    return 0;
}
n

zoj1321 《Parallelepiped Walk》果断是个很恶心的计算几何题，需要强大的空间想象力。问你一个长方体表面上的任意两点之间的距离最小是多少，当然，只能走长方体的表面。
如果一个个推情况的话，一定会恶心到吐血。那么怎么做会简单一点呢？先把一个点平移到地面，然后固定住这个点，把整个长方体进行反转，把另外一个点转到地面来计算它们的长度，选取一个最小值。其实就是递归各种情况。需要想象力，当时我想了好久。。。还好工图及格了！



n
#include<iostream>
#include<algorithm>
#include<cstdio>
using namespace std;
#define INF ((1<<30)-1)
int ans;
//将一个点定在平面上，然后把立方体向各个面滚动

int square(int x)
{
	return x*x;
}

void walk(int i,int j,int x0,int y0,int xx,int yy,int zz,int x,int y,int z)
{
	if(zz)
	{
		if(i<=0&&i>-2)
			walk(i-1,j,x0+z,y0,z-zz,yy,xx,z,y,x);
		if(i>=0&&i<2)
			walk(i+1,j,x0-x,y0,zz,yy,x-xx,z,y,x);
		if(j<=0&&j>-2)
			walk(i,j-1,x0,y0+z,xx,z-zz,yy,x,z,y);
		if(j>=0&&j<2)
			walk(i,j+1,x0,y0-y,xx,zz,y-yy,x,z,y);
	}
	else
	{
		ans = min(ans,square(x0-xx)+square(y0-yy));
	}
}

int main()
{
	int x,y,z,x0,y0,z0,xx,yy,zz;
	while(scanf("%d%d%d %d%d%d %d%d%d",&x,&y,&z
				,&x0,&y0,&z0,&xx,&yy,&zz)!=EOF)
	{
		if(z0!=0&&z0!=z)
		{
			if(y0==0||y0==y)
			{
				swap(y,z);
				swap(y0,z0);
				swap(yy,zz);
			}
			else
			{
				swap(x,z);
				swap(x0,z0);
				swap(xx,zz);
			}
		}
		if(z0!=0)zz = z-zz;
		ans = INF;
		walk(0,0,x0,y0,xx,yy,zz,x,y,z);
		printf("%d\n",ans);
	}


	return 0;
}
/*
 * 网上摘录的一段理解:
 *
第一步：

         将A的旋转到底面。

1、  判断A是否在下底面或者上底面，如果不在上下底面，通过交换坐标使其在上底面或者下底面。

2、  如果在上底面，可以通过对对称中心（l/2,w/2,h/2）,做对称变换，最终使A在下底面上。

第二步：

         walk(i,j,x1,y1,x2,y2,z2,l,w,h);

         i+1 表示底面向右下翻，长方体顺时针旋转；

         i-1 表示底面向左下翻，长方体逆时针旋转；

         j+1 表示底面向里下翻，长方体向外旋转；

         j-1 表示底面向外下翻，长方体向里旋转；

         可以证明一下结论：

1、  沿每个方向最多翻转两次就可以得到正确答案；

2、  即使两个点都旋转到底面时，他们直接的连线不完全在面上。那么他们之间的距离一定不是最短。

 * 
 * */
n

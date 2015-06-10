---
author: admin
comments: true
date: 2012-07-14 16:02:48+00:00
layout: post
slug: zoj1391-horizontally-visible-segments-zoj1413-2d-nim
title: ZOJ1391 Horizontally Visible Segments && ZOJ1413 2D Nim
wordpress_id: 232
categories:
- ACM
- Zoj
tags:
- ACM
- C++
- Zoj
- 暴力
- 模拟
- 狗狗四十题
- 线段树
- 解题报告
---

《Horizontally Visible Segments》是一个线段树的题，给你n条垂直的线，问你三条两两互不挡住但是能用横线连起来的直线对有多少。

做法就是用线段树，首先对于x坐标排序，然后对一长条染色，被覆盖住的线段部分显然不可能再被后面的线段所看见。所以只要记录当前直线被哪些线段（id）覆盖过就可以了。最后对于已经标记好颜色的标号集合，枚举任意两个，然后再枚举其中一个的标号集合里面有没有元素的标号集合包含另外一个没使用的。简单来说，就是暴力查看存不存在三个标号两两可以看见。
做法基本就是这样了。
注意点：
1、在标号颜色的时候，要用set判重。
2、写线段树的时候加个延迟标记，表示连续的、同时也是被延迟更新的。
3、为什么点数由8000变成了16000？因为把（1，1），（2，2）这样的看成一条线段，需要拆下点。

其他：代码都是跟老高学的，对线段树不熟练，有时间还是要把notonlysuccesss的[线段树合集](http://www.notonlysuccess.com/index.php/segment-tree-complete/)刷一下啊。

```
#include<iostream>
#include<set>
#include<cstdio>
#include<vector>
#include<memory.h>
#include<algorithm>
using namespace std;
#define MAXN 16100
#define LL(a) (a<<1)
#define RR(a) (a<<1|1)
int n;
struct Node
{
	int x,y1,y2;
	Node(int xx,int y11,int y22)
	{
		if(y11>y22)swap(y11,y22);
		x = xx;y1 = y11;y2=y22;
	}
	Node(){}
	bool operator < (Node a)const
	{
		return x<a.x;
	}
};
vector<Node>vt;
set<int>s[MAXN<<1];
int tree[MAXN<<2];
int cover[MAXN<<2];

void push_down(int root)
{
	if(cover[root])
	{
		cover[LL(root)] = cover[RR(root)] = -1;
		tree[LL(root)] = tree[RR(root)] = tree[root];
		cover[root] = 0;
	}
}

void insert(int root,int l,int r,int a,int b,int id)
{
	if(l>=a&&r<=b)
	{
		tree[root] = id;
		cover[root] = -1;
		return;
	}
	push_down(root);
	int mid = ((l+r)>>1);
	if(a<=mid)insert(LL(root),l,mid,a,b,id);
	if(b>mid)insert(RR(root),mid+1,r,a,b,id);
}

void query(int root,int l,int r,int a,int b,int id)
{
	if(cover[root])
	{
		if(!tree[root])return;
		s[tree[root]].insert(id);
		return;
	}
	//push_down(root);
	int mid = (l+r)>>1;
	if(a<=mid)query(LL(root),l,mid,a,b,id);
	if(b>mid)query(RR(root),mid+1,r,a,b,id);

}

int main()
{
	int T;
	scanf("%d",&T);
	int y1,y2,x;
	while(T--)
	{
		scanf("%d",&n);
		vt.clear();
		for(int i=0;i<n;i++)
		{
			scanf("%d%d%d",&y1,&y2,&x);
			vt.push_back(Node(x,y1,y2));
			s[i].clear();
		}
s[n].clear();
		sort(vt.begin(),vt.end());
		memset(tree,0,sizeof(tree));
		memset(cover,-1,sizeof(cover));
		for(int i=0;i<vt.size();i++)
		{
			y1 = vt[i].y1;
			y2 = vt[i].y2;
			query(1,0,MAXN,y1*2,y2*2,i+1);
			insert(1,0,MAXN,y1*2,y2*2,i+1);
		}
		int ans = 0;
		for(int i=1;i<=n;i++)
		{
			for(set<int>::iterator j=s[i].begin();j!=s[i].end();j++)
			{
				int a = *j;
				for(set<int>::iterator k=s[a].begin();k!=s[a].end();k++)
				{
					for(set<int>::iterator p=s[i].begin();p!=s[i].end();p++)
					{
						if(*k==*p)ans++;
					}
				}
			}
		}
		printf("%d\n",ans);


	}
	return 0;
}
```


对于《2D Nim》这题，样例没过哥就AC了。。数据实在太弱了。后来HJWAJ跟我说了一种，统计横向和纵向连通块的长度以及其相应个数，然后两张图进行对比，感觉没也错。反正都AC了，实在想不出什么反例，POJ和ZOJ的数据都是一样的，无论怎么瞎搞都能过。我当时就是求了个连通块个数，然后比较连通块个数是否相等就过了。我的代码就不贴了。
在网上看到有个人写的解题报告代码写的不错,使用了各种STL，就是思路写的不够详细。大致就是把两个图进行平移和旋转各种变化，然后检验两个图形集合里的所有变化出来的集合是不是一样的，如果一样，显然能证明局势相等，否则就是不相同的局势。
不过看他的代码也基本能明白做法了。[原文网址](http://blog.csdn.net/program_shun/article/details/6596471)

说白了，就是个很恶心的模拟题。就是数据出弱了，随便怎么搞都能过，就显得没那么恶心了。o(∩_∩)o ~

```
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;
#define maxn 110

typedef vector<pair<int,int> > PIECE;       //包含联通片和联通片里的点坐标 
typedef vector<PIECE> GRAPH;    //（联通片，第几片） 

const int dx[]={0,0,1,-1};
const int dy[]={1,-1,0,0};

int w, h;
int a[maxn][maxn], b[maxn][maxn];

void init()
{
    int n, x, y;
    scanf("%d%d%d", &w, &h, &n);
    memset(a, 0, sizeof(a));
    memset(b, 0, sizeof(b));
    for (int i=0; i<n; i++)
    {
        scanf("%d%d", &x, &y);
        a[x][y]=1;
    }
    for (int i=0; i<n; i++)
    {
        scanf("%d%d", &x, &y);
        b[x][y]=1;
    }
}

void dfs(int x, int y, int aa[][maxn], int &xmin, int &xmax, int &ymin, int &ymax, PIECE &p)
{
    if (x<0 || y<0 || x>=w || y>=h) return;
    if (aa[x][y]==1){
        aa[x][y]=0;
        p.push_back(make_pair(x, y));
        xmin = min(xmin, x);
        xmax = max(xmax, x);
        ymin = min(ymin, y);
        ymax = max(ymax, y);
        for (int i=0; i<4; i++)
            dfs(x+dx[i], y+dy[i], aa, xmin, xmax, ymin, ymax, p);
    }
}

PIECE mirror(const PIECE p, int xmin, int xmax, int ymin, int ymax)
{
    PIECE ret;
    for (PIECE::const_iterator j=p.begin(); j!=p.end(); j++)
        ret.push_back(make_pair(j->first, ymax-(j->second-ymin)));
    return ret;
}

PIECE rot0(const PIECE p, int xmin, int xmax, int ymin, int ymax)       //0 deg
{
    PIECE ret;
    for (PIECE::const_iterator j=p.begin(); j!=p.end(); j++)
        ret.push_back(make_pair( j->first-xmin, j->second-ymin ));
    sort(ret.begin(), ret.end());
    return ret;
}

PIECE rot90(const PIECE p, int xmin, int xmax, int ymin, int ymax)       //90 deg
{
    PIECE ret;
    for (PIECE::const_iterator j=p.begin(); j!=p.end(); j++)
        ret.push_back(make_pair( ymax-j->second , j->first-xmin));
    sort(ret.begin(), ret.end());
    return ret;
}

PIECE rot180(const PIECE p, int xmin, int xmax, int ymin, int ymax)       //180 deg
{
    PIECE ret;
    for (PIECE::const_iterator j=p.begin(); j!=p.end(); j++)
        ret.push_back(make_pair( xmax-j->first, ymax-j->second));
    sort(ret.begin(), ret.end());
    return ret;
}

PIECE rot270(const PIECE p, int xmin, int xmax, int ymin, int ymax)       //270 deg
{
    PIECE ret;
    for (PIECE::const_iterator j=p.begin(); j!=p.end(); j++)
        ret.push_back(make_pair(j->second-ymin, xmax-j->first));
    sort(ret.begin(), ret.end());
    return ret;
}

bool process()
{
    GRAPH g1, g2;
    PIECE p0, p1, p;
    int id1=0, id2=0;
    int xmin, xmax, ymin, ymax;
    
    g1.clear();
    for (int x=0; x<w; x++)
        for (int y=0; y<h; y++)
        {
            if (a[x][y])
            {
                p0.clear(); xmin=ymin=maxn; xmax=ymax=-1;
                dfs(x, y, a, xmin, xmax, ymin, ymax, p0);
                p1=mirror(p0, xmin, xmax, ymin, ymax);
                p=rot0(p0, xmin, xmax, ymin, ymax);    g1.push_back(p); 
                p=rot90(p0, xmin, xmax, ymin, ymax);   g1.push_back(p); 
                p=rot180(p0, xmin, xmax, ymin, ymax);  g1.push_back(p);
                p=rot270(p0, xmin, xmax, ymin, ymax);  g1.push_back(p);
                p=rot0(p1, xmin, xmax, ymin, ymax);    g1.push_back(p); 
                p=rot90(p1, xmin, xmax, ymin, ymax);   g1.push_back(p);
                p=rot180(p1, xmin, xmax, ymin, ymax);  g1.push_back(p); 
                p=rot270(p1, xmin, xmax, ymin, ymax);  g1.push_back(p); 
                id1++;
            }
        }
            
    g2.clear();
    for (int x=0; x<w; x++)
        for (int y=0; y<h; y++)
            if (b[x][y])
            {
                p0.clear(); xmin=ymin=maxn; xmax=ymax=-1;
                dfs(x, y, b, xmin, xmax, ymin, ymax, p0);
                p=rot0(p0, xmin, xmax, ymin, ymax);    g2.push_back(p);
                id2++;
            }
    
    if (id1!=id2) return false;             //联通片个数不同 
    
    sort(g1.begin(), g1.end());
    sort(g2.begin(), g2.end());
    
    GRAPH::iterator i=g1.begin();
    GRAPH::iterator j=g2.begin();
    while (i!=g1.end() && j!=g2.end())
    {
        if (*i!=*j) i++;
        if (*i==*j)
        {
            j++;
            if (j==g2.end()) return true;
        }
        if (i==g1.end()) return false;
    }
    return false;
}

int main()
{
    int cs;
    scanf("%d", &cs);
    while (cs--)
    {
        init();
        if (process()) printf("YES\n");
        else printf("NO\n");
    }
    return 0;
}
```

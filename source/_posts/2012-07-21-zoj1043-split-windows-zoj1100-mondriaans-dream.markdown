---
author: admin
comments: true
date: 2012-07-21 16:13:14+00:00
layout: post
slug: zoj1043-split-windows-zoj1100-mondriaans-dream
title: ZOJ1043 Split Windows && ZOJ1100 Mondriaan's Dream
wordpress_id: 267
categories:
- ACM
- Zoj
tags:
- ACM
- DP
- Zoj
- 狗狗四十题
- 解题报告
---

ZOJ1043《Split Windows》是一个题目描述很长很长的题目，以至于我花了一个小时才看懂，估计也是因为看的时候不断在分心的原因吧。
题目描述的意思是，给你一颗描述矩形框如何分割的树，树的叶子结点都用字母表示，每个子树部分都用‘-’或‘|’表示如何分割。然后让你把符合该描述的最小的一种矩形分割画出来。然后还花了很多的笔墨教你如何左右大小调整，其实意思就是如果大小不一致了，按比例放大，遇到不是整数的情况，就左子树向上取整。

其实这道题目很有点当年那个画N倍长度的“X”题的翻版的感觉，就是一个递归画图。但是要先把每个子树的高度调整好。
做法：
1、建树，求每个子树的树高。
2、调整树高，遇到‘-’,横向分割的，就调整上下两部分与根结点，高度放大，上面部分向上取整；遇到‘|’，就调整左右两部分高度与根节点相同，宽度放大，左边向上取整。
3、调整树高的同时把矩形画出来。每次递归的时候传左上角那个点的坐标。
4、一定要注意细节。如果过了样例还WA，那么就倒霉了，这种题目不好debug啊，实际上没什么trick，就是烦。。

```
#include<iostream>
#include<cmath>
#include<cstdio>
#include<cstring>
using namespace std;
struct RecTree
{
	char *raw;
	char buf[1024][1024];
	char r[1024];
	int h[1024],w[1024];//每个节点的高度和宽度
	int lson[1024],rson[1024];//左子树，右子树节点
	int _n;
	RecTree(char *p){
		raw = p;
		_n = 0;
	}

	int makeTree()//得到高度、宽度、左右子树
	{
		int n = _n++;
		r[n] = *raw++;
		switch(r[n])
		{
			case '|':case '-':
				lson[n] = makeTree();
				rson[n] = makeTree();
				if(r[n]=='|'){
					h[n] = max(h[lson[n]],h[rson[n]]);
					w[n] = w[lson[n]]+w[rson[n]];
				}else{
					h[n] = h[lson[n]]+h[rson[n]];
					w[n] = max(w[lson[n]],w[rson[n]]);
				}
				break;
			default:
				lson[n] = rson[n] = -1;
				h[n] = 2;
				w[n] = 2;
				break;
		}
		return n;
	}

	void preDraw(int root,int height,int width)
	{
		if(lson[root]==-1){
			buf[height][width] = r[root];
			for(int i=1;i<w[root];i++)
				if(buf[height][width+i]!='*')
					buf[height][width+i]='-';
			for(int i=1;i<h[root];i++)
				if(buf[height+i][width]!='*')
					buf[height+i][width]='|';
			buf[height+h[root]][width+w[root]] = '*';
			buf[height][width+w[root]] = '*';
			buf[height+h[root]][width] = '*';
			for(int i=1;i<w[root];i++)
				if(buf[height+h[root]][width+i]!='*')
					buf[height+h[root]][width+i]='-';
			for(int i=1;i<h[root];i++)
				if(buf[height+i][width+w[root]]!='*')
					buf[height+i][width+w[root]]='|';
			return;
		}
		if(r[root]=='|'){
			int ww = w[lson[root]]+w[rson[root]];
			w[lson[root]]= (w[root]*w[lson[root]]+ww-1)/ww;
			w[rson[root]]= (w[root]*w[rson[root]])/ww;
			h[lson[root]] =	h[rson[root]] = h[root];
		}else{
			int hh = h[lson[root]]+h[rson[root]];
			h[lson[root]]=(h[root]*h[lson[root]]+hh-1)/hh;
			h[rson[root]]=(h[root]*h[rson[root]])/hh;
			w[lson[root]] =	w[rson[root]] = w[root];
		}
		/*draw edge*/
		for(int i=1;i<w[root];i++)
			if(buf[height+h[root]][width+i]!='*')
				buf[height+h[root]][width+i]='-';
		for(int i=1;i<h[root];i++)
			if(buf[height+i][width+w[root]]!='*')
				buf[height+i][width+w[root]]='|';
		buf[height+h[root]][width+w[root]] = '*';
		buf[height+h[root]][width] = '*';
		buf[height][width+w[root]] = '*';
		/*draw edge*/

		preDraw(lson[root],height,width);
		if(r[root]=='|')preDraw(rson[root],height,width+w[lson[root]]);
		else preDraw(rson[root],height+h[lson[root]],width);


	}
	void draw()
	{
		memset(buf,' ',sizeof(buf));
		preDraw(0,0,0);
		for(int i=0;i<=h[0];i++){
			for(int j=0;j<=w[0];j++){
				printf("%c",buf[i][j]);
			}
			printf("\n");
		}
	}

};

int main()
{
	int n;
	char buf[5024];
	scanf("%d",&n);
	for(int icase=1;icase<=n;icase++)
	{
		scanf("%s",buf);
		printf("%d\n",icase);
		RecTree rt = RecTree(buf);
		rt.makeTree();
		rt.draw();
	}


	return 0;
}
```

ZOJ1100 《Mondriaan's Dream》是一个非常好的状态压缩DP。题目的意思就是给你一块大小为n*m的矩形，问你能有多少种分割成一个个2*1小矩形的方案。
阿森说，这是轮廓线DP，每次拿一个int存储当前行被分割后的状态。int的前十位，每一位，1表示当前的这个格子是被占用的，0表示这个各自是空的。
[![](https://wonderflow.info/images/2012-07-21-zoj1043-split-windows-zoj1100-mondriaans-dream/dp1100.png)](https://wonderflow.info/images/2012-07-21-zoj1043-split-windows-zoj1100-mondriaans-dream/dp1100.png)
有了这张图片，应该会好理解一些。想清楚了转移，那么DP的代码非常简洁，尤其是位操作的压缩DP，各种优美。

```
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
int n,m;
int dp[12][12][1<<12];

int dfs(int x,int y,int s)
{
	if(y>=m)return dfs(x+1,0,s);
	if(x==n){
		if(!s)return 1;
		return 0;
	}
	if(dp[x][y][s]!=-1)return dp[x][y][s];
	if(s&(1<<y))return dp[x][y][s] = dfs(x,y+1,s^(1<<y));
	int ret = dfs(x,y+1,s^(1<<y));
	if(y<m-1&&!(s&(1<<(y+1))))ret+=dfs(x,y+2,s);
	return dp[x][y][s] = ret;
}

int main()
{
	while(scanf("%d%d",&n,&m)!=EOF)
	{
		if(m+n==0)break;
		memset(dp,-1,sizeof(dp));
		printf("%d\n",dfs(0,0,0));
	}
	return 0;
}
```

+++
author = "admin"
date = "2012-07-08T15:57:51Z"
slug = "2012/07/08/zoj1506-left-labyrinths-zoj1518-this-sentence-is-false"
title = "ZOJ1506 Left labyrinths && ZOJ1518 This Sentence is False"
Categories = ["ACM", "Zoj"]
Tags = ["ACM", "Zoj", "狗狗四十题", "解题报告"]
+++

今天把第一场定时训练训练的C和E都过了，都是不错的题目。

Left labyrinths讲的是一个迷宫，你进入迷宫后要遵守规则，优先靠右走。然后问你最后能否到中央大厅。
我感觉有这样几个注意点：
1、迷宫的入口如何搜寻：就是左右是‘#’但是上下是‘.’的点或者反过来。
2、迷宫的大厅什么时候算走到了？找到田字格的‘.’块，注意2*2就算了，我一开始想象着要3*3才算，果断WA了。
3、规则说靠右走，但是没说只能靠右走，其实必要的时候还是要往别的方向走的。
4、每个格子不是只能走一次，而是每个格子的每个方向只能走一次。

有了这几个注意点，基本上AC就没问题了。这题主要是题目理解上的问题了吧，一开始我无论怎样都WA，后来去[watashi](http://watashi.ws/)的blog上一看[他的代码](http://blog.watashi.ws/991/gougou40/attachment/1038/)，才知道了这些个注意点。然后发现大神的代码写得各种简洁，巧妙的利用了memset对字符串的块刷新和用位运算来表示一个点的四个方向有没有访问过，让我看了各种佩服。不愧是曾经的final冠军~

```
#include<iostream>
#include<memory.h>
#include<cstdio>
using namespace std;
int n,m;
char map[120][120];
bool vis[120][120][5];
int dir[4][2] = {-1,0,0,1,1,0,0,-1};//上右下左
int fdir[8][2] = {0,1,0,-1,1,0,-1,0,1,1,1,-1,-1,1,-1,-1};
int sa,sb,sd;

bool check(int a,int b)
{
	int num1 = 0,num2=0;
	int ta,tb;
	for(int i=0;i<4;i++)
	{
		ta = a+dir[i][0];
		tb = b+dir[i][1];
		if(ta>=0&&ta<n&&tb>=0&&tb<m&&map[ta][tb]=='#')
		{
			if(i%2)num1++;
			else num2++;
		}
	}
	if((num1==2&&num2==0)||(num1==0&&num2==2))
	{
	//	cout<<num1<<" "<<num2<<" "<<a<<" "<<b<<endl;
		return true;
	}
	return false;
}

bool finalfind(int a,int b)
{
	return (a>0&&b>0&&map[a - 1][b - 1] != '#' && map[a - 1][b] != '#' && map[a][b - 1] != '#')
        || (a>0&&b<m-1&&map[a - 1][b + 1] != '#' && map[a - 1][b] != '#' && map[a][b + 1] != '#')
        || (a<n-1&&b>0&&map[a + 1][b - 1] != '#' && map[a + 1][b] != '#' && map[a][b - 1] != '#')
        || (a<n-1&&b<m-1&&map[a + 1][b + 1] != '#' && map[a + 1][b] != '#' && map[a][b + 1] != '#');
	
}

void dfs(int a,int b,int dirs)
{
	vis[a][b][4]=true;
	if(check(a,b))
	{
		sa = a;
		sb = b;
		sd = dirs;
	//	cout<<a<<" "<<b<<" "<<sd<<endl;
		return ;
	}
	int ta,tb;
	for(int i=0;i<4;i++)
	{
		ta = a+dir[i][0];
		tb = b+dir[i][1];
		if(ta>=0&&ta<n&&tb>=0&&tb<m&&!vis[ta][tb][4]&&map[ta][tb]=='.')
		{
			dfs(ta,tb,i);
		}
	}
}

bool formal_dfs(int a,int b,int dirs)
{
	vis[a][b][dirs] = true;
	if(finalfind(a,b))
	{
		//cout<<a<<" "<<b<<endl;
		return true;
	}
	int ta,tb;
	for(int i=-1;i<3;i++)
	{
		ta = a+dir[(4+i+dirs)%4][0];
		tb = b+dir[(4+i+dirs)%4][1];
		if(ta>=0&&ta<n&&tb>=0&&tb<m&&!vis[ta][tb][4]&&
				!vis[ta][tb][(4+i+dirs)%4]&&map[ta][tb]=='.')
		{
			return formal_dfs(ta,tb,(4+dirs+i)%4);
		}
	}
	return false;
}

int main()
{
	while(scanf("%d%d",&n,&m)!=EOF)
	{
		for(int i=0;i<n;i++)
			scanf("%s",map[i]);
		memset(vis,0,sizeof(vis));
		dfs(0,0,0);
		//cout<<sa<<" "<<sb<<" "<<sd<<endl;
		vis[sa][sb][4] = true;
		if(formal_dfs(sa,sb,sd))printf("YES\n");
		else printf("NO\n");
	}

	return 0;
}
```

This Sentence is False这题是个图论题，阿森跟我说了一种方法，老高跟我说了一种方法。
我自己根据理解写了一下，最后写成了老高的方法。
题目就是让你找出现在的话里面有没有自相矛盾的地方，如果暂时没有自相矛盾的，那么问你最多有几句话是真的。

做法有两种，建立的都是无向图，建边就是建的双向的，下面就不赘述了，先写阿森的方法：
基于图顶点的染色，首先每句话都是没有染色的点，如果一句话说另外一句话是真的，那么这句话到另外一句话连一条边，权值为0，如果这句话说另外一句话是假的，那么也连一条边，权值为1。最后我其实就是要给这张图的每个点染色。边的权值为0的两个端点颜色必须相同，权值不为0的两个点，颜色必须不同。然后在染色的过程中，可以发现如果颜色出现冲突，那么明显是出现了自相矛盾的情况了。再然后对于每个被染色的连通块，其实起点只要随便选择一种染颜色就可以了。对于每个连通块，统计两种颜色的个数，最后的答案加上每个连通块颜色数多的各种颜色的个数。就是真话最多的可能情况。

然后说下我的，也就是老高一开始想到的解法：
对于每个句子，拆分成两个结点，一个是代表这句话是真的，一个代表这句话是假的。然后对于每个句子所代表的情况，如果本句话说另外一个句子是真的，那么代表本句话真的结点和代表另外一句话是真的结点连一条无向边，同时，代表本句话是假的的结点和代表另外一句话是假的的结点也连一条无向边。为什么呢？因为一句话说另外一句话是对的，如果这句话是对的，那么同时也证明了另外一句话是对的；如果这句话是假的，那么也间接的说明了另外一句话是假的。同样的情况对于另外一种“本句话说另外一句话是假的”也是适用的，就是代表本句话是真的的结点和代表另外一句话是假的的结点连边，然后对称的再互相连边。总之最后构成的是一个对称的图。
怎么判自相矛盾呢？我是另外写了个并查集，连边的时候不断的在合并点，如果一个结点的真和假出现在了同一个集合内，那么必然是自相矛盾的。当然，如果不用并查集其实也可以的，就是如果一个点的真和假出现在了同一个连通块内，代表是自相矛盾的。
最后剩下的就是怎么算真话的个数了。很简单，就是对于每个连通块，算代表真话成立的结点和代表假话的结点各有多少个，取多的那个。注意，每个点只访问一次！访问一种情况的时候，同时把另外一种情况的点也标记。因为两种是对称的，所以也变相的证明了为什么选择代表假话的结点多的时候也可以用。
这题AC的还算顺利。

```
#include<iostream>
#include<memory.h>
#include<cstdio>
#include<vector>
using namespace std;
#define NODE 2010

int n;
char str1[100],str2[10],str3[100];
bool vis[NODE];
int num;

int father[NODE];//奇数表示该点正确，偶数表示该点错误
vector<int>edge[NODE];

void init()
{
	for(int i=1;i<=2*n;i++)
	{
		father[i] = i;
		edge[i].clear();
	}
}

int find_set(int x)
{
	if(x!=father[x])father[x] = find_set(father[x]);
	return father[x];
}

void union_set(int x,int y)
{
	if(find_set(x)!=find_set(y))
		father[x] = father[y];
}

void insert(int a,int b)
{
	edge[a].push_back(b);
	edge[b].push_back(a);
}

int odd,even;

void dfs(int index)
{
	vis[index] = true;
	if(index%2)vis[index+1] = true;
	else vis[index-1] = true;
	if(index%2)odd++;
	else even++;
	//cout<<index<<" ";
	for(int i=0;i<edge[index].size();i++)
	{
		if(!vis[edge[index][i]])dfs(edge[index][i]);
	}
}

int main()
{
	while(scanf("%d",&n)&&n)
	{
		init();
		for(int i=1;i<=n;i++)
		{
			scanf("%s%d%s%s",str1,&num,str2,str3);
			switch(str3[0])
			{
				case 'f':
					union_set(i*2-1,2*num);
					union_set(i*2,2*num-1);
					insert(i*2-1,2*num);
					insert(i*2,2*num-1);
					break;
				default:
					union_set(i*2-1,2*num-1);
					union_set(i*2-1,2*num-1);
					insert(i*2-1,2*num-1);
					insert(i*2,2*num);
					break;
			}
		}
		bool over = false;
		for(int i=1;i<=n;i++)
		{
			if(find_set(2*i-1)==find_set(2*i))
			{
				over = true;
				break;
			}
		}
		if(over)
		{
			printf("Inconsistent\n");
			continue;
		}
		memset(vis,0,sizeof(vis));
		int ans = 0;
		for(int i=1;i<=2*n;i++)
		{
			if(!vis[i])
			{
				odd = even = 0;
				dfs(i);
				ans+=max(odd,even);
			}
		}
		printf("%d\n",ans);
	}

}
```


在最后，祝薛斌今天生日快乐~今天还为他唱歌了呢。算是够哥们了吧~

+++
author = "admin"
date = "2012-07-06T04:19:18Z"
slug = "2012/07/06/usaco-american-heritage-e6a091e79a84e9818de58e86"
title = "USACO American Heritage 树的遍历"
Categories = ["ACM", "usaco"]
Tags = ["ACM", "usaco"]
+++

[题目](http://ace.delos.com/usacoprob2?a=Nymi3zD4Dh8&S=heritage)给出树的前序和中序遍历，让你给出树的后续遍历。
我记得正好是两年前，第一次新生赛的时候，就做到了这个题目，那个时候觉得好难啊，也是，连递归都没有理解的人，来做这样的题，确实有点难了。
如输入：
ABEDFCHG
CBADEFGH
输出：
AEFDBHGC

做这题的窍门就是找规律了啊，可以看见，前序遍历的最开始，都是各个子树的根节点。而中序遍历则是可以用来帮助确定子树的范围。
理解了树的结构和递归以后，就很容易了。
此时做起来倒是格外的亲切了。然后写了两个，一个用动态指针存树，一个用静态数组存树。



```
//动态存树
/*
TASK:heritage
LANG:C++
USER:BOBO
*/

#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
char str1[200],str2[200];

struct Node
{
	char c;
	Node *l,*r;
}*tree;
int num;

void build(Node * root,int l,int r)
{
	int k = -1;
	for(int i=l;i<=r;i++)
	{
		if(str1[i] == root->c)
		{
			k = i;break;
		}
	}
	if(k-l>0)
	{
		root->l = new Node();
		root->l->c = str2[num++];
		build(root->l,l,k-1);
	}
	else root->l = NULL;
	if(r-k>0)
	{
		root->r = new Node();
		root->r->c = str2[num++];
		build(root->r,k+1,r);
	}
	else root->r = NULL;
}

void print(Node * root)
{
	if(root->l)print(root->l);
	if(root->r)print(root->r);
	printf("%c",root->c);
}

void del(Node * root)
{
	if(root->l)del(root->l);
	if(root->r)del(root->r);
	delete root;
}

int main()
{
	//freopen("heritage.in","r",stdin);
	//freopen("heritage.out","w",stdout);
	while(gets(str1))
	{
		gets(str2);
		int len = strlen(str1);
		tree = new Node();
		num = 0;
		tree->c = str2[num++];
		build(tree,0,len-1);
		print(tree);
		printf("\n");
		del(tree);
	}
}
```




```
//静态存树
/*
TASK:heritage
LANG:C++
USER:BOBO
*/

#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
char str1[200],str2[200];

struct Node
{
	char c;
	int l,r;
}node[10000];
int con;
int num,tree;

void build(int root,int l,int r)
{
	int k = -1;
	for(int i=l;i<=r;i++)
	{
		if(str1[i] == node[root].c)
		{
			k = i;break;
		}
	}
	if(k-l>0)
	{
		node[root].l =con++;
		node[node[root].l].c = str2[num++];
		build(node[root].l,l,k-1);
	}
	else node[root].l = -1;
	if(r-k>0)
	{
		node[root].r = con++;
		node[node[root].r].c = str2[num++];
		build(node[root].r,k+1,r);
	}
	else node[root].r = -1;
}

void print(int root)
{
	if(node[root].l!=-1)print(node[root].l);
	if(node[root].r!=-1)print(node[root].r);
	printf("%c",node[root].c);
}
int main()
{
	//freopen("heritage.in","r",stdin);
	//freopen("heritage.out","w",stdout);
	while(gets(str1))
	{
		gets(str2);
		int len = strlen(str1);
		con = 0;
		tree = con++;
		num = 0;
		node[tree].c = str2[num++];
		build(tree,0,len-1);
		print(tree);
		printf("\n");
	}
}
```

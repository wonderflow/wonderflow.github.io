+++
author = "admin"
date = "2012-07-06T06:35:07Z"
slug = "2012/07/06/zoj1504-slots-of-fun-zoj1423-yourtermproject"
title = "ZOJ1504 Slots of Fun && ZOJ1423 (Your)((Term)((Project)))"
Categories = ["ACM", "Zoj"]
Tags = ["ACM", "Zoj", "狗狗四十题", "解题报告"]
+++

最近教练出的每日早晚一小时定时训练，出的都是狗狗40的套题里面的题目，觉得很好，有必要都AC一遍，慢慢来吧。

Slots of Fun 这题，一开始想了40分钟，不知道怎么快速计算每个点的坐标，结果最后才想到，只要扫的时候用两重循环扫替代线性的扫描，就可以快速定位一个点的坐标了。（果然智商是硬伤啊！！）
然后常规的方法就是用sqrt（3）这样的把每个点相对于起始点的坐标计算出来，然后用eps比较是否相等。
后来小翟告诉我一个方法，非常简单，就是使用余弦定理，如果一个点的坐标(x,y)表示的是第几排，第几个数字的话，那么A,B两点之间的距离其实就是dis^2 = (xa-xb)^2+(ya-yb)^2+2*|xa-xb|*|ya-yb|cos(α) ,而本身我们的图像可以知道α是个60°角，那么答案就很明显啦。

```
#include<stdio.h>
#include<cstring>
#include<string>
#include<map>
#include<algorithm>
using namespace std;
string ans;
char s[200];
struct Node
{
	int x,y;
	char c;
}node[200];
map<char,bool>mymap;
int dis(int x1,int y1,int x2,int y2)
{
	int y=y2-y1,x=x2-x1;
	return x*x+y*y-x*y;
}
int main()
{
	int n,i,j,i1,i2,j1,j2;
	while(scanf("%d",&n) && n)
	{
		mymap.clear();
		scanf("%s",s);
		ans = "";
		int num = 0;
		for(int i=0;i<n;i++)
		{
			for(int j=0;j<=i;j++)
			{
				node[num].x = i;
				node[num].y = j;
				node[num].c = s[num++];
			}
		}
		int a1,a2,a3;
		for(int i=0;i<num;i++)
		{
			if(mymap[node[i].c])continue;
			a1 = a2 = a3 = -1;
			mymap[node[i].c] = true;
			a1 = i;
			for(int j=a1+1;j<num;j++)
				if(node[j].c==node[i].c)
				{
					a2 = j;
					break;
				}
			for(int k=a2+1;k<num;k++)
				if(node[k].c == node[i].c)
				{
					a3 = k;
					break;
				}
			if(a1==-1||a2==-1||a3==-1)continue;
			int d1 = dis(node[a1].x,node[a1].y,node[a2].x,node[a2].y);
			int d2 = dis(node[a1].x,node[a1].y,node[a3].x,node[a3].y);
			int d3 = dis(node[a2].x,node[a2].y,node[a3].x,node[a3].y);
			if(d1&&d1==d2&&d2==d3)ans+=node[a1].c;
		}

		if(ans=="")
			printf("LOOOOOOOOSER!\n");
		else
		{
			sort(ans.begin(),ans.end());
			printf("%s\n",ans.c_str());
		}
	}
	return 0;
}
```



(Your)((Term)((Project)))这题是要求去掉括号，本身这个题目很简单。但是有一个小trick。
如样例 A-((A+B))
我一开始就是被这个样例搞死了。
其实只要想清楚3点就可以了。就是去掉括号的三个条件，满足一个就可以把左右两个括号去掉。
1、括号在最外面
2、括号左边不是‘-’
3、括号中间是单个字符
然后控制好保存括号匹配的栈，就OK了。

```
#include<iostream>
#include<cstdio>
#include<cstring>
using namespace std;
char str[300];
char ans[300];
bool que[300];
int main()
{
	int n;
	scanf("%d",&n);
	gets(str);
	while(n--)
	{
		gets(str);
		int len = strlen(str);
		int num = 0;
		for(int i=0;i<len;i++)
		{
			if(str[i]!=' ')str[num++] = str[i];
		}
		len = num;
		str[len] = 0;
		num = 0;
		memset(que,0,sizeof(que));
		bool f = true;
		int an = 0;
		char c;

		for(int i=0;i<len;i++)
		{
			c = str[i];
			if(c>='A'&&c<='Z')
			{
				while(str[i+1]==')'&&que[num-1])
				{
					i++;
					num--;
				}
				while(ans[an-1] == '('&&str[i+1]==')')
				{
					an--;
					i++;
					num--;
				}
				ans[an++] = c;
				f = true;
				continue;
			}
			if(c=='-')
			{
				ans[an++] = c;
				f = false;
				continue;
			}
			if(c=='+')
			{
				ans[an++] = c;
				f = true;
				continue;
			}
			if(c == '(')
			{
				if(f)
				{
					que[num++] = true;
				}
				else
				{
					que[num++] = false;
					ans[an++] = c;
				}
				f = true;
				continue;
			}
			if(c == ')')
			{
				if(que[num-1]==false)
				{
					ans[an++] = c;
				}
				num--;
				f = true;
				continue;
			}
		}
		for(int i=0;i<an;i++)
			printf("%c",ans[i]);
		printf("\n");
	}
}
```

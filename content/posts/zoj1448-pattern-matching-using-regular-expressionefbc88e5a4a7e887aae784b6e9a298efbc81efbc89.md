+++
author = "admin"
date = "2012-07-10T16:59:50Z"
slug = "2012/07/10/zoj1448-pattern-matching-using-regular-expressionefbc88e5a4a7e887aae784b6e9a298efbc81efbc89"
title = "ZOJ1448 Pattern Matching Using Regular Expression（大自然题！）"
Categories = ["ACM", "Zoj"]
Tags = ["ACM", "C++", "DP", "大自然题", "模拟", "正则表达式", "狗狗四十题", "解题报告"]
+++

狗狗四十题里面的这个题非常有意思，号称“大自然”题。其实就是模拟简单正则匹配的规则，录入模式串和匹配串，把最左最长的匹配子串输出出来。

一开始没有什么思路，后来看了watashi的代码，才发现了做法。话说watashi的那个[《狗狗40题搞完纪念》](http://blog.watashi.ws/991/gougou40/)绝对是个神贴啊。上面的代码资源，太可贵了！

具体做法是：
先扫模式串，把模式串的规则整理出来。把一些规则统一起来，如"."的规则，统一为字符0~256之间的任意字符，就是跟【1-9】这种一样咯。
然后用dp[i]表示从i开始作为匹配的头，最后面能匹配到的位置。

最后不得不说，这题还是相当恶心的，模式串从后往前开始对匹配串进行匹配是为了无后效性。碰到"*"即any==true的情况，就要遍历每个i为起点，找最后能匹配到的j的位置。总的来说，先把模式串统一整理一下的思想，是非常值得学习的。

```
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<vector>
using namespace std;
#define NODE 100

struct Node
{
	bool no,any;
	bool cmp;
	char from,to;
	bool match(char s)
	{
		if(!cmp)return true;
		if(!no&&from<=s&&s<=to)return true;
		if(no&&(s<from||s>to))return true;
		return false;
	}
};
vector<Node>regex;

int dp[NODE];

//对于一个串，构造一个分析串regex，
//有了分析串后，对于分析串里的每一个节点都要进行匹配
void getRegex(char * modes)
{
	regex.clear();
	int len = strlen(modes);
	for(int i=0;i<len;)
	{
		Node a;
		a.cmp = true;
		a.any = false;
		a.no = false;
		switch(modes[i++])
		{
			case '.':
				a.cmp = false;
				break;
			case '['://notice such case "[\*-\\]"
				if(modes[i]=='^')
				{
					a.no = true;
					i++;
				}
				if(modes[i]=='\\')
				{
					i++;
				}
					a.from = modes[i++];
					i++;
				if(modes[i]=='\\')
				{
					i++;
				}

				a.to = modes[i];
				i++;
				i++;
				if (a.from > a.to) { // between c1 and c2
					swap(a.from, a.to);
				}
				
				break;
			case '\\':
				i++;
				//no break;
			default:
				a.from = a.to = modes[i-1];
				break;
		}
		switch(modes[i++])
		{
			case '+':
				regex.push_back(a);
				//no break;
			case '*':
				a.any = true;
				regex.push_back(a);
				break;
			default:
				regex.push_back(a);
				i--;
				break;
		}
	}
}

//对于模式串，找能匹配的串
//如果匹配不到，那么dp[i]的值赋-1
//考虑，如果是一个*,可以不匹配，又要尽可能多的匹配
void solve(char *str)
{
	int n = strlen(str);
	for(int i=0;i<=n;i++)dp[i] = i;
	for(vector<Node>::reverse_iterator it = regex.rbegin();it!=regex.rend();it++)
	{
	//	 printf("{ %s, [%s(%d)-(%d)]}\n", it->any ? "y" : "n", it->no ? "^" : "", it->from, it->to);
		if(it->any)
		{
			for(int i=0;i<=n;i++)
			{
				for(int j=i;j<n;j++)
				{
					if(it->match(str[j]))dp[i] = max(dp[i],dp[j+1]);
					else break;
				}
			}
		}
		else
		{
			for(int i=0;i<n;i++)
			{
				dp[i] = it->match(str[i])?dp[i+1]:-1;
			}
			dp[n] = -1;
		}
	}
}

int main()
{
	char modes[100],str[100];
	while(gets(modes))
	{
		if(!strcmp(modes,"end"))break;
		gets(str);
		getRegex(modes);
		int len = strlen(str);
		solve(str);
		bool win = false;
		for(int i=0;i<len;i++)
		{
			if(dp[i]>i)
			{
				win = true;
				for(int k=0;k<i;k++)
					printf("%c",str[k]);
				printf("(");
				for(int k=i;k<dp[i];k++)
					printf("%c",str[k]);
				printf(")");
				for(int k=dp[i];k<len;k++)
					printf("%c",str[k]);
				break;
			}
		}
		if(!win)printf("%s",str);
		printf("\n");


	}
	
	return 0;
}
```

崔嵬的做法则是：比较正常的思路，也是我一开始想的思路（就是一开始的时候做着做着，碰到*就不知道怎么做下去了）
对于待匹配的串，枚举能匹配部分的起点。用一个match函数，返回所能正确匹配的最远终点（题目要求贪婪模式）。
碰到“[c1-c2]”、“[^c1-c2]”、“.”、“\”、都不难处理，而碰到“+”的时候，可以转移成一个前面的和一个“*”，也就是最麻烦的就是处理“*”的过程。
崔嵬用了一个递归的方法求的“*”，还是非常神奇的。

```
#include <stdio.h>
#include <string.h>
#include <math.h>
#include <algorithm>
#include <string>
#include <map>
#include <set>

using namespace std;

char **str, **pat;

int match() {
	int i=0,j=0,ans=-1,f,type,arg0,arg1,sj,sub;
	while ((*pat)[i]) {
		if ((*pat)[i]=='[') {
			type=((*pat)[i+1]!='^'),i+=((*pat)[i+1]!='^')?1:2;
			arg0=(((*pat)[i]=='\\')?(*pat)[++i]:(*pat)[i]),i+=2;
			arg1=(((*pat)[i]=='\\')?(*pat)[++i]:(*pat)[i]),i+=2;
			if (arg0>arg1) swap(arg0,arg1);
		} else if ((*pat)[i]=='.')
			type=2, ++i;
		else
			type=3, arg0=(((*pat)[i]=='\\')?(*pat)[++i]:(*pat)[i]), ++i;
		if ((*pat)[i]=='*' || (*pat)[i]=='+') {
			sj=j, ++i;
			while ((*str)[j] && !(type<2 && (((*str)[j]>=arg0&&(*str)[j]<=arg1)^type) || type==2 && !(*str)[j] || type==3 && (*str)[j]!=arg0))
				++j;
			for (f=sj+((*pat)[i-1]=='+');f<=j;f++) {
				(*str)+=f,(*pat)+=i;
				sub=match();
				(*str)-=f,(*pat)-=i;
				if (sub!=-1 && ans<sub+f)
					ans=sub+f;
			}
			return ans;
		}
		if ((type<2 && (((*str)[j]>=arg0&&(*str)[j]<=arg1)^type) || type==2 && !(*str)[j] || type==3 && (*str)[j]!=arg0))return -1;
		++j;
	}
	return j;
}


char spat[90],sstr[90];

int main() {
	str=new char*;
	pat=new char*;
	while (gets(spat),strcmp(spat,"end")) {
		gets(sstr);
		int l=strlen(sstr),su=-1,si;
		for (int i=0;i<=l;i++) {
			(*str)=sstr+i,(*pat)=spat;
			int u=match();
			if (u!=-1) {
				su=u,si=i;
				break;
			}
		}
		if (su>0) {
			for (int i=0;i<si;i++)
				putchar(sstr[i]);
			putchar('(');
			for (int i=0;i<su;i++)
				putchar(sstr[si+i]);
			putchar(')');
			for (int i=si+su;i<l;i++)
				putchar(sstr[i]);
			putchar('\n');
		} else
			puts(sstr);
	}
	return 0;
}

```




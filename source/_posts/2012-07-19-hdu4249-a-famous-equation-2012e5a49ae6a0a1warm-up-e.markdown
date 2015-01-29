---
author: admin
comments: true
date: 2012-07-19 12:23:06+00:00
layout: post
slug: hdu4249-a-famous-equation-2012%e5%a4%9a%e6%a0%a1warm-up-e
title: HDU4249 A Famous Equation 2012多校warm up E
wordpress_id: 265
categories:
- ACM
tags:
- ACM
- DP
- hdu
- 搜索
- 解题报告
---

今天下午做了场多校热身赛，题目都不难，做到一题有意思的DP，把它记录一下。

[题目描述](http://icpc.njust.edu.cn/Hdu/4249)：就是给你一个含问号的和式，问你有多少种情况满足题意。

一开始写了个搜索，暴力枚举每个问号位的情况，然后稍微减了下支，想想一年多前这样的搜索一定觉得很难写不想写，如今十来分钟就写好了，还挺得意，然后果断TLE了！
[cpp title="bruteforce search code"]
#include<iostream>
#include<cstdio>
#include<cstring>
#include<climits>
#include<stack>
#include<string>
#include<map>
using namespace std;

int rec[3][20],num[3];
int ask[3];
int ans;

int getz(int x)
{
	int ret = 1;
	for(int i=0;i<x;i++)
		ret*=10;
	return ret;
}

void dfs(int flag,int dex,int sum)
{
	if(sum<0)return;
	if(!flag&&dex==num[flag]){
		if(!sum)ans++;
		return;
	}
	if(dex==num[flag]){
		dex = 0;
		flag--;
	}
	if(rec[flag][dex]<0){
		for(int i=dex?0:1;i<10;i++){
			if(flag==2)dfs(flag,dex+1,sum+i*getz(num[flag]-dex-1));
			else dfs(flag,dex+1,sum-i*getz(num[flag]-dex-1));
		}
	}else{
		if(flag==2)dfs(flag,dex+1,sum+rec[flag][dex]*getz(num[flag]-dex-1));
		else dfs(flag,dex+1,sum-rec[flag][dex]*getz(num[flag]-dex-1));
	}


}

int main()
{
	int icase =0 ;
	char ss[100];
	int flag = 0;
	while(gets(ss))
	{
		icase++;
		memset(num,0,sizeof(num));
		memset(ask,0,sizeof(ask));
		flag = 0;
		printf("Case %d: ",icase);
		int siz = strlen(ss);
		for(int i=0;i<siz;i++){
			switch(ss[i]){
				case '+':case '=':
					flag++;break;
				case '?':
					rec[flag][num[flag]++] = -1;
					ask[flag]++;
					break;
				default:
					rec[flag][num[flag]++] = ss[i]-'0';
					break;
			}
		}
		if(ask[0]+ask[1]==0	|| ask[0]+ask[2]==0 || ask[1]+ask[2]==0){
			puts("1");continue;
		}
		ans = 0;
		dfs(2,0,0);
		printf("%d\n",ans);
	}
	return 0;
}
n

然后得到了阿森的提示，知道了要用DP的方法。就跟人在模拟这个过程类似，其实变化只有两种，一种是有进位，一种是无进位。用一个dp[i][0]表示第i位无进位时的情况总数，dp[i][1]表示第i位有进位时的情况总数。
然后用了各种三目运算符解决了不少if,else判断，再加上前置0的问题。哎，想清楚了DP过程后，前前后后debug了将近三个小时才AC。

n
#include<iostream>
#include<cstdio>
#include<cstring>
#include<climits>
#include<stack>
#include<string>
#include<map>
#include<algorithm>
using namespace std;

int rec[3][20],num[3];
__int64 dp[11][2];


int main()
{
	int icase =0 ;
	char ss[100];
	int flag = 0;
	while(gets(ss))
	{
		icase++;
		memset(num,0,sizeof(num));
		memset(dp,0,sizeof(dp));
		memset(rec,0,sizeof(rec));
		flag = 0;
		printf("Case %d: ",icase);
		int siz = strlen(ss);
		for(int i=0;i<siz;i++)
		{
			switch(ss[i])
			{
				case '+':case '=':
					flag++;
					break;
				case '?':
					rec[flag][num[flag]++] = -1;
					break;
				default:
					rec[flag][num[flag]++] = ss[i]-'0';
					break;
			}
		}
		for(int i=0;i<3;i++)
			reverse(rec[i],rec[i]+num[i]);
		if(num[2]<num[0]||num[2]<num[1])
		{
			puts("0");
			continue;
		}
		for(int i=0;i<num[2];i++)
		{
			for(int j=(rec[0][i]<0?((i<num[0]-1||num[0]==1)?0:1):rec[0][i]);j<=(rec[0][i]<0?9:rec[0][i]);j++)
			{
				for(int k=(rec[1][i]<0?((i<num[1]-1||num[1]==1)?0:1):rec[1][i]);k<=(rec[1][i]<0?9:rec[1][i]);k++)
				{
					if(((rec[2][i]<0&&i!=num[2]-1) || (i==num[2]-1&&(j+k)>0&&rec[2][i]<0) ||  (num[2]==1&&rec[2][i]<0)  )
							||(j+k)%10==rec[2][i])
					{
							__int64 rr = i>0?dp[i-1][0]:1;
							if(j+k>=10)dp[i][1] += rr; 
							else dp[i][0]+=rr;
					}
					if(rec[2][i]<0||(j+k+1)%10==rec[2][i])
					{
							__int64 rr = i>0?dp[i-1][1]:0;
							if(j+k+1>=10)dp[i][1] += rr;
							else dp[i][0]+=rr;
					}
				}
			}
		}

		printf("%I64d\n",dp[num[2]-1][0]);
	}
	return 0;
}
n

标程写的挺简洁了，值得学习：
[cpp title="standard program"]
#include<iostream>
#include<map>
#include<string>
#include<utility>
#include<cstdio>
#define ll "ll"
#define mem pair<pair<pair<string,string>,string>,int>
using namespace std;
map<mem,long long int> Map;
long long int dfs(string a,string b,string c,int carry,bool flag){
	int la=a.length(),lb=b.length(),lc=c.length();
	if(la==0 && lb==0 && lc==0)return carry==0?1:0;
	char ad=la?a[la-1]:'0',bd=lb?b[lb-1]:'0',cd=lc?c[lc-1]:'0';
	string newa=la?a.substr(0,la-1):"",
	       newb=lb?b.substr(0,lb-1):"",
	       newc=lc?c.substr(0,lc-1):"";
	mem tmp=make_pair(make_pair(make_pair(a,b),c),carry);
	if(Map.find(tmp)!=Map.end())return Map[tmp];
	long long int&res=Map[tmp];
	res=0;
	for(int i=0; i<10; i++)
		if(ad=='?' || ad-'0'==i)
		for(int j=0; j<10; j++)
			if(bd=='?' || bd-'0'==j){
				int k=i+j+carry,nc=0;
				if(k>=10)k-=10,nc++;
				if(cd=='?' || cd-'0'==k){
					if(!flag && (la==1 && i==0 || lb==1 && j==0 || lc==1 && k==0))continue;
					res+=dfs(newa,newb,newc,nc,false);
				}
			}
	return res;
}
int main(){
	string s;
	for(int t=1; cin>>s; t++){
		int pnt;
		string a="",b="",c="";
		for(pnt=0; s[pnt]!='+'; pnt++)
			a+=s[pnt];
		for(pnt++; s[pnt]!='='; pnt++)
			b+=s[pnt];
		for(pnt++; pnt<s.length(); pnt++)
			c+=s[pnt];
		Map.clear();
		printf("Case %d: %"ll"d\n",t,dfs(a,b,c,0,true));
	}
	return 0;
}
n


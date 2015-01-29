---
author: admin
comments: true
date: 2012-07-22 11:05:54+00:00
layout: post
slug: zoj1030-farmland-zoj1145-dreisam-equations
title: ZOJ1030 Farmland && ZOJ1145 Dreisam Equations
wordpress_id: 276
categories:
- ACM
- Zoj
tags:
- ACM
- Zoj
- 搜索
- 模拟
- 狗狗四十题
- 解题报告
- 计算几何
- 递归
---

zoj1030 《Farmland》绝对是恶心人的题，题目是要求找出边长为n的多边形。但是如果途中有其他的边或者点，就不算。
做法就是枚举每一条边，然后dfs，每次选择夹角最小的点。选择完以后要对所有的点枚举，查看点是否在多边形内。最后，因为每条边都按照正向走和反向走，可能出现重复，怎么解决重复呢，算面积，面积为负的舍弃。

代码写的比较烦躁：WA了十多次，搞了好久，果然我这样的人还是不适合做计算几何的、
n
#include<iostream>
#include<cstring>
#include<cstdio>
#include<vector>
#include<cmath>
#include<map>
#define infinity 1e10
#define EP 1e-8
using namespace std;
/*
 * 1.dfs出一个多边形(坐标变换，把一条边转成x轴正方向)
 * 2.判断多边形内是否有孤立的点，采用射线法，
 * 		(射线与多边形的交点是奇数个的时候，点在多边形内，在此之前要判断是否在线段上)
 * 3.判断多边形的面积,要一致，不然会出现重复算的情况
 * */
struct Lpoint
{
	double x,y;
}vt[220],node[220],p1,p2;;
vector<int>edge[220];
int id[220];
int inum,des,slen;
int vlen;
bool vis[220],use[220][220];
int ans;

double atan2(Lpoint a,Lpoint b)
{
	return atan2((a.x*b.y-b.x*a.y),(a.x*b.x+b.y*a.y));
}

double xmulti(Lpoint p1,Lpoint p2,Lpoint p0){
	return ((p1.x-p0.x)*(p2.y-p0.y)-(p2.x-p0.x)*(p1.y-p0.y));
}

//判断点是否相等
int Euqal_Point(Lpoint p1,Lpoint p2) {
	return((fabs(p1.x-p2.x)<EP)&&(fabs(p1.y-p2.y)<EP));
}

#define _sign(x) ((x)>0?1:((x)<0?2:0))
int inside_convex(Lpoint q,Lpoint * p)
{
	int i,s[3]={1,1,1};
	for (i=0;i<vlen&&s[1]|s[2];i++)
		s[_sign(xmulti(p[(i+1)%vlen],q,p[i]))]=0;
	return s[1]|s[2];
}

double area_polygon(Lpoint* p)
{
	double s1=0,s2=0;
	int i;
	for (i=0;i<vlen;i++)
		s1+=p[(i+1)%vlen].y*p[i].x,s2+=p[(i+1)%vlen].y*p[(i+2)%vlen].x;
	return (s1-s2)/2.0;
}

bool check(int n)
{
	for(int i=1;i<=n;i++){
		bool ss = false;
		for(int j=0;j<vlen;j++){
			if(Euqal_Point(node[i],vt[j])){
				ss = true;break;
			}
		}
		if(ss)continue;
		Lpoint q = node[i];
		if(inside_convex(q,vt))
			return false;
	}
	if(area_polygon(vt)<=0)return false;
	return true;
}

void dfs(int from,int to,int n)
{
	use[from][to] = true;
	if(des==to){
		if(vlen==slen&&check(n)){
			ans++;
		}
		return;
	}
	double ret = 1000000;
	int k=-1;
	for(int i=0;i<edge[to].size();i++)
	{
		int v = edge[to][i];
		if(v==from)continue;
		if(!use[to][v])
		{
			p1.x=node[to].x-node[from].x;
			p1.y=node[to].y-node[from].y;
			p2.x=node[v].x-node[to].x;
			p2.y=node[v].y-node[to].y;
			double tpp = atan2(p2,p1);
			if(tpp<ret)
			{
				ret = tpp;
				k = v;
			}
		}
	}
	if(k==-1)return;
	vt[vlen++] = node[k];
	dfs(to,k,n);
	vlen--;
}

int main()
{
	int T,temp,p,d;
	int n;
	int egnum;
	scanf("%d",&T);
	while(T--)
	{
		scanf("%d",&n);
		for(int i=0;i<=n;i++)edge[i].clear();
		for(int i=0;i<n;i++)
		{
			scanf("%d",&temp);
			p = temp;
			scanf("%lf%lf",&node[p].x,&node[p].y);
			scanf("%d",&egnum);
			for(int j=0;j<egnum;j++)
			{
				scanf("%d",&temp);
				d = temp;
				edge[p].push_back(d);
				edge[d].push_back(p);
			}
		}
		scanf("%d",&slen);
		memset(use,0,sizeof(use));
		vlen = 0;
		ans = 0;
		for(int i=1;i<=n;i++)
		{
			for(int j=0;j<edge[i].size();j++)
			{
				int v = edge[i][j];
				if(use[i][v])continue;
				vlen = 0;
				vt[vlen++] = node[v];
				des = i;
				dfs(i,v,n);
			}
		}
		printf("%d\n",ans);

	}
}
n

ZOJ1145 《Dreisam Equations》 是给你一堆没有写操作符的等式，让你把操作符填上。如：18 = 7 (5 3) 2 写：18=7+(5-3)*2
看起来比较好玩，写起来却实在麻烦，果然狗狗四十题没有省油的灯啊，还好这个题比较顺利，过了样例就AC了。
做法就是dfs尝试添加操作符。

n
#include<iostream>
#include<stack>
#include<string>
#include<cstring>
#include<cstdio>
using namespace std;

struct Calc
{
	char *buf;
	bool succeed;
	string ans;
	int pre[100],num;
	string ss[3];
	Calc(char *s){
		buf = s;
		succeed = false;
		ss[0] = "+";
		ss[1] = "-";
		ss[2] = "*";
	}
	void prepare()
	{
		num = 0;
		int sum = 0;
		while(*buf)
		{
			char c = *buf;
			if(c>='0'&&c<='9')sum = sum*10+c-'0';
			else if(c==' '){
				if(sum)pre[num++] = sum;
				sum = 0;
			}else if(c=='('){
				if(sum)pre[num++] = sum;
				pre[num++] = -1;
				sum = 0;
			}else if(c==')'){
				if(sum)pre[num++] = sum;
				pre[num++] = -2;
				sum = 0;
			}else if(c=='='){
				if(sum)pre[num++] = sum;
				pre[num++] = -3;
				sum = 0;
			}
			buf++;
		}
		if(sum)pre[num++] = sum;
	}

	void _run(stack<int>st,string ts,int dex)
	{
		if(dex==num){
			if(!st.empty()&&st.top()==pre[0]){
				succeed = true;
				ans = ts;
			}
			return;
		}
		if(pre[dex]==-1){
			if(st.empty()||st.top()==249){
				st.push(249);
				_run(st,ts+"(",dex+1);
			}else for(int i=250;i<253;i++){
				st.push(i);st.push(250-1);
				_run(st,ts+ss[i-250]+"(",dex+1);
				st.pop();st.pop();
			}
		}
		else if(pre[dex]==-2){
			int sum = 0;
			while(st.top()!=249){
				sum = st.top();
				st.pop();
			}
			st.pop();
			if(st.empty()||st.top()==249){
				st.push(sum);
				_run(st,ts+")",dex+1);
			}else{
				int op = st.top();st.pop();
				if(op==250){
					sum+=st.top();
				}else if(op==251){
					sum = st.top()-sum;
				}else{
					sum *= st.top();
				}
				st.pop();
				st.push(sum);
				_run(st,ts+")",dex+1);
			}
		}else {
			char str[10];
			sprintf(str,"%d",pre[dex]);
			if(st.empty()||st.top()==249){
				st.push(pre[dex]);
				ts+=str;
				_run(st,ts,dex+1);
			}else{
				int tp,sum;
				tp = st.top();
				st.pop();
				sum = tp+pre[dex];
				st.push(sum);
				_run(st,ts+"+"+str,dex+1);
				st.pop();
				sum = tp-pre[dex];
				st.push(sum);
				_run(st,ts+"-"+str,dex+1);
				st.pop();
				sum = tp*pre[dex];
				st.push(sum);
				_run(st,ts+"*"+str,dex+1);
			}
		}
	}

	void run()
	{
		stack<int>st;
		char ss[10];
		sprintf(ss,"%d=",pre[0]);
		string ts  = ss;
		_run(st,ts,2);
	}
};

int main()
{
	char buf[1024];
	int icase = 0;
	while(gets(buf)&&strcmp(buf,"0"))
	{
		icase++;
		Calc x(buf);
		x.prepare();
		x.run();
		printf("Equation #%d:\n",icase);
		if(x.succeed)printf("%s\n",x.ans.c_str());
		else printf("Impossible\n");
		puts("");
	}
	return 0;
}
n

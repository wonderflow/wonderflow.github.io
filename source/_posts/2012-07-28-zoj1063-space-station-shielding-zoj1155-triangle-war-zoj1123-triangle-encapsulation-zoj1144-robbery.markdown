---
author: admin
comments: true
date: 2012-07-28 14:48:41+00:00
layout: post
slug: zoj1063-space-station-shielding-zoj1155-triangle-war-zoj1123-triangle-encapsulation-zoj1144-robbery
title: ZOJ1063 Space Station Shielding && ZOJ1155 Triangle War && ZOJ1123 Triangle
  Encapsulation &&ZOJ1144 Robbery
wordpress_id: 348
categories:
- ACM
- Zoj
tags:
- ACM
- DP
- Zoj
- 模拟
- 状态压缩
- 狗狗四十题
- 解题报告
- 递推
---

ZOJ1063《Space Station Shielding》是个题意题，题意搞懂了，就是个bfs水题，先自己好好读读题意，实在看不懂的话，见下面代码里面的注释。 

```
/*
 * 此题难点在题意理解：
 * 要把整个矩形框内每个被称为“ACM”的格子周围涂上防护层
 *		当然相邻“ACM”格子之间不需要涂防护罩，从外层不可达的点也不需要涂防护层，
 *		这个不可达怎么理解？就是被ACM格子围住的点。
 * 问最后要涂的防护罩的个数。（每个小方格的一个面就需要一个防护罩。
 * 理解完题意以后就简单了，就是一个bfs。（最外层加一圈点）
 * */
#include<iostream>
#include<cstring>
#include<queue>
#include<cstdio>
using namespace std;
int n,m,k,p;
int dirx[6] = {1,-1,0,0,0,0};
int diry[6] = {0,0,1,-1,0,0};
int dirz[6] = {0,0,0,0,1,-1};
int acm[70][70][70];
struct Node
{
	int x,y,z;
	Node(){}
	Node(int x,int y,int z):x(x),y(y),z(z){}
};
queue<Node>myque;
int main()
{
	int tp,a,b,c;
	int ans;
	while(scanf("%d%d%d%d",&n,&m,&k,&p)!=EOF)
	{
		if(n+m+k+p==0)break;
		ans = 0;
		memset(acm,0,sizeof(acm));
		for(int i=0;i<p;i++){
			scanf("%d",&tp);
			c = tp%(n*m);
			a = c%n;
			b = c/n;
			c = tp/(n*m);
			acm[a+1][b+1][c+1] = 1;
		}
		acm[0][0][0] = -1;
		while(!myque.empty())myque.pop();
		myque.push(Node(0,0,0));
		while(!myque.empty()){
			int x = myque.front().x;
			int y = myque.front().y;
			int z = myque.front().z;
			myque.pop();
			for(int i=0;i<6;i++){
				int tx = x+dirx[i];
				int ty = y+diry[i];
				int tz = z+dirz[i];
				if(tx>=0&&tx<n+2&&ty>=0&&ty<m+2&&tz>=0&&tz<k+2){
					if(!acm[tx][ty][tz]){
						acm[tx][ty][tz] = -1;
						myque.push(Node(tx,ty,tz));
					}else if(acm[tx][ty][tz]==1){
						ans++;
					}
				}
			}
		}
		printf("The number of faces needing shielding is %d.\n",ans);
	}

}
```

ZOJ1155 《Triangle War》是一个状态压缩DP，状态自然是一个int的每一位存储每一条边(一共18条边)是否已经选择。一开始预处理好一个数组DP[1<<18]表示当前状态下先手的人一直到最后，还能多得的三角形数。从后往前推，最后面所有边都放满的情况，自然是不可能多得的，然后对于状态s，做一遍1～18的循环，看当前位是否还可以，如果可以填，那么填上看是否可以多得一个三角形，如果可以多得，那么就是说可以多走一步，把后面的那个状态的dp值和当前能得到的三角形数量加起来取个较大的。否则的话就是说下一个状态是对手多得的三角形个数，那么我本回合多得的三角形个数就是下一回合多得的三角形数的相反数（下一回合对方多得多少，我就比对方少得多少），这里也是取个较大的。
处理好了这个DP数组后，就是按照它给边的顺序一个个边填模拟它的游戏规则了。然后看最后的状态是谁先手，是第一个人先手，加上dp该状态的值，否则减去dp该状态的值，最后看得到的值是正还是负，正代表我比另外一个人多三角形，赢了，反之就是输了。

```
#include<iostream>
#include<assert.h>
#include<cstdio>
#include<cstring>
using namespace std;
#define MAXN 18

int tri[9][3] = { {1,2,3},{2,4,5},{2,3,5}
		    ,{3,5,6},{4,7,8},{4,5,8}
		    ,{5,8,9},{5,6,9},{6,9,10} };
int edge[11][11];
int fg[MAXN][2];
int predp[1<<MAXN];
int n;

void init(){
	memset(edge,-1,sizeof(edge));
	int num = 0;
	for(int i=0;i<9;i++)
	{
		int a = tri[i][0];
		int b = tri[i][1];
		int c = tri[i][2];
		if(edge[tri[i][0]][tri[i][1]]==-1){
			fg[num][0] = min(a,b);
			fg[num][1] = max(a,b);
			edge[a][b] = edge[b][a] = num++;
		}
		if(edge[a][c]==-1){
			fg[num][0] = min(a,c);
			fg[num][1] = max(a,c);
			edge[a][c] = edge[c][a] = num++;
		}
		if(edge[b][c]==-1){
			fg[num][0] = min(b,c);
			fg[num][1] = max(b,c);
			edge[b][c] = edge[c][b] = num++;
		}
	}
	//cout<<num<<endl;
}



int count_tri(int s)
{
	int ret = 0;
	for(int i=0;i<9;i++){
		int a = tri[i][0];
		int b = tri[i][1];
		int c = tri[i][2];
		if((s&(1<<edge[a][b]))
		 &&(s&(1<<edge[a][c]))
		 &&(s&(1<<edge[b][c]))){
			ret++;
		}
	}
	return ret;
}

void getdp()
{
	for(int i=0;i<(1<<MAXN);i++)predp[i] = -100000;
	predp[(1<<MAXN)-1] = 0;
	for(int i=(1<<MAXN)-1;i>=0;i--){
		for(int j=0;j<MAXN;j++){
			if(!(i&(1<<j))){
				int ad =  count_tri(i^(1<<j)) - count_tri(i);
				if(ad>0){
					predp[i] = max(predp[i],predp[i^(1<<j)]+ad);
				}else{
					predp[i] = max(predp[i],-predp[i^(1<<j)]);
				}
			}
		}
	}
}


int main()
{
	int T,T1;
	int a,b;
	bool c;
	scanf("%d",&T);
	init();
	getdp();
	int ss = 0;
	while(T--){
		if(ss)puts("");
		ss++;
		scanf("%d",&T1);
		for(int icase=1;icase<=T1;icase++){
			scanf("%d",&n);
			int s = 0,ts;
			int y = 0;
			c = 0;
			for(int i=0;i<n;i++){
				scanf("%d%d",&a,&b);
				ts = 1<<edge[a][b];
				int ad = count_tri(s|ts)-count_tri(s);
			//	cout<<edge[a][b]<<" "<<ad<<endl;
				y+=c?-ad:ad;
				if(!ad)c = !c;
				s|=ts;
			}
		//	cout<<y<<" ";
			y+=c?-predp[s]:predp[s];
			//cout<<predp[s]<<endl;
			printf("Game %d: %c wins.\n",icase,y>0?'A':'B');
		}

	}
}
```
 
ZOJ1123 《Triangle Encapsulation》搞了半天就是一个暴力，一共也就18*18个点，枚举一下每个点是否在三角形内就行了。这里用到了别人写的一个点在多边形内的模板。然后输出格式稍微要注意一点，具体难度不大。
```
//template from http://blog.csdn.net/shifuwawa/article/details/5747508
#include<iostream>
#include<cmath>
#include<cstdio>
#include<cstring>
using namespace std;
const int sup=250;
const int PI=acos(-1.0);
const int esp=1e-7;
int grid[50][50];//grid[i][j]=0表示不打印，-1表示打印空格,1表示打印坐标
struct dot//点数据结构：横纵坐标
{
	double x,y;
	dot(){}
	dot(double x,double y):x(x),y(y){}
};
struct line_seg//线段数据结构：起点终点
{
	dot st,ed;
};
dot zero;////////////圆心
inline double cros_mul(dot sp,dot ep,dot op)//得到(sp-op)*(ep-op)的叉积
{
	return (sp.x-op.x)*(ep.y-op.y)-(ep.x-op.x)*(sp.y-op.y); 
}
//判断线段a的两端是否在线段b的同侧,是判相交instersect的子函数
inline bool same_side(line_seg a,line_seg b) //**************************************
{ 
	dot tp1,tp2,zero;
	zero.x=zero.y=0;
	tp1.x=b.ed.x-b.st.x;
	tp1.y=b.ed.y-b.st.y;

	tp2.x=a.st.x-b.st.x;
	tp2.y=a.st.y-b.st.y;
	double ta=cros_mul(tp1,tp2,zero);
	tp2.x=a.ed.x-b.ed.x;
	tp2.y=a.ed.y-b.ed.y;
	double tb=cros_mul(tp1,tp2,zero);
	if( ta*tb>0)
		return true;//不相交
	return false;//相交
}
inline bool intersect(line_seg a,line_seg b)//***********************************
{
	if(!same_side(a,b) && !same_side(b,a))
		return true;
	return false;//不相交
}
inline bool online(dot Q,dot p1,dot p2)//判断Q点是否在以p1p2为端点的线段上，//*****************************************
{
	if( 0 == cros_mul(p1,Q,p2) )//(Q-p1)X(p2-p1)=0保证Q在p1p2所在的直线,P(x1,y1)XQ(x2,y2)=x1*y2-x2*y1
	{
		if(	Q.x<=max(p1.x,p2.x) && Q.x>= min(p1.x,p2.x) && Q.y<=max(p1.y,p2.y) && Q.y>=min(p1.y,p2.y))//保证Q不在线段的延长线上
			return true;
	}
	return false;
}
inline int InPolygon(dot polygon[],int n,dot p) //判断点p是否在多边形内（多边形以n个点的顶点集polygon表示） 用到online和intersect子函数//*******************************
{  
	int cnt=0,i;  
	line_seg line;  
	line.st=p;  
	line.ed.y=p.y;  
	line.ed.x=-1e9;  //以无限远处某点和p点构造一条平行x轴的射线
	for(i=0;i<n;i++)  
	{  
		//取得多边形的一条边  
		line_seg   side;  
		side.st=polygon[i];  
		side.ed=polygon[(i+1)%n];  
		if( online(p,side.st,side.ed) ) //假如点在多边形的某条边上,根据题目要求返回true或者false，这里返回false，认为不在多边形内
			return 0;   
		//   如果side平行x轴则不作考虑  
		if(side.st.y==side.ed.y)//////////if( fabs(side.st.y-side.ed.y)<esp )  
			continue;    
		if( online(side.st,line.st,line.ed) )//假如多边形的某条边的起点在构造的射线上  
		{
			if(side.st.y>side.ed.y)  //并且这条边不平行于x轴
				++cnt;  
		}  
		else if( online(side.ed,line.st,line.ed) ) //同上，不过是此边的终点
		{  
			if(side.ed.y>side.st.y)  
				++cnt;  
		}  
		else if( intersect(line,side) ) //另外就是普通的射线和多边形边是否相交的判断
			++cnt;    
	}  
	return cnt&1;  
} 

int main()
{
	dot po[3];
	dot vis[20][20];
	dot tmp;
	int icase = 0;
	while(scanf("%lf%lf",&po[0].x,&po[0].y)!=EOF){
		if(icase)puts("");
		else puts("Program 4 by team X");
		memset(vis,0,sizeof(vis));
		icase++;
		for(int i=1;i<3;i++)scanf("%lf%lf",&po[i].x,&po[i].y);
		for(int i=-9;i<=9;i++){
			for(int j=9;j>=-9;j--){
				tmp.x = i;
				tmp.y = j;
				if(InPolygon(po,3,tmp)){
					vis[i+9][9-j] = dot(double(i),double(j));
				}else{
					vis[i+9][9-j].x = -100;
				}
			}
		}
		for(int i=0;i<=18;i++){
			bool fa = 0,fb = 0;
			for(int j=0;j<=18;j++){
				if(vis[i][j].x!=-100&&vis[i][j].x!=100){
					fa = true;
				}
				if(vis[j][i].x!=-100&&vis[j][i].x!=100){
					fb = true;
				}
			}
			if(!fa){
				for(int j=0;j<=18;j++){
					vis[i][j].x = 100;
				}
			}
			if(!fb){
				for(int j=0;j<=18;j++){
					vis[j][i].x = 100;
				}
			}
		}
		for(int i=0;i<=18;i++){
			for(int j=18;j>=0;j--){
				if(vis[j][i].x==100||vis[j][i].x==-100)vis[j][i].x = 100;
				else break;
			}
		}
		for(int i=0;i<=18;i++){
			bool st = 0;
			for(int j=0;j<=18;j++){
				if(vis[j][i].x == 100)continue;
				if(st)printf(" ");
				st = true;
				if(vis[j][i].x == -100)printf("        ");
				else printf("(%2d,%3d)",(int)vis[j][i].x,(int)vis[j][i].y);
			}
			if(st)printf("\n");
		}
	}
	puts("\nEnd of program 4 by team X");

}
```

ZOJ1144 《Robbery》也是个比较简单的题目，就是递推。如果一个点没被探测，那么这个点就有可能有小偷，然后因为小偷每一步只能走一格，所以我两个相邻的时间间隔内的没被探测过的点要坐标距离在一以内才合法，所以我们就可用这样正着来一遍，然后根据最后一个时刻的点都是允许的，再反着来一遍，把之前多取的点筛选掉。最后再判断是否存在这样的点或者是否模棱两可。
```
#include <cstdio>
#include <cstring>
#include <iostream>
#include <algorithm>
#define MAXN 110
using namespace std;

char map[MAXN][MAXN][MAXN];//第一维时间
//正向扫一遍，把可能的点扫出来，此时的点肯定是变多的,
//		只有最后一个时刻的这样的点才算,用"."表示
//反向扫一遍，确认可能的点是否能从最后一个时间的点推导出来,用"*"

int n,m,k,p;
int dirx[5] = {0,1,-1,0,0};
int diry[5] = {0,0,0,1,-1};

int main() {
	int t,li,ti,ri,bi;
	int icase = 0;
	while(scanf("%d%d%d",&n,&m,&k)!=EOF){
		if(n+m+k==0)break;
		icase++;
		for(int p1=1;p1<=k;p1++){

			//****外面套一层表示不可达**********//
			for(int p2=0;p2<=n;p2++){
				map[p1][p2][0] = map[p1][p2][m+1] = '#';
			}
			for(int p2=0;p2<=m;p2++){
				map[p1][0][p2] = map[p1][n+1][p2] = '#';
			}
			//**************//
			
			for(int p2=1;p2<=n;p2++){
				for(int p3=1;p3<=m;p3++){
					map[p1][p2][p3] = (p1==1?(k==1?'*':'.'):' ');
					//如果只有一个人的话,第一个人也是最后一个人，
					//初始化没被扫过的点直接变*，否则是'.'代表需要验证，
				}
			}
		}
		scanf("%d",&p);
		for(int p1=0;p1<p;p1++){
			scanf("%d%d%d%d%d",&t,&li,&ti,&ri,&bi);
			for(int p2=li;p2<=ri;p2++){
				for(int p3=ti;p3<=bi;p3++){
					map[t][p2][p3] = '#';
				}
			}
		}

		for(int p1=1;p1<k;p1++){
			for(int p2=1;p2<=n;p2++){
				for(int p3=1;p3<=m;p3++){
					if(map[p1][p2][p3] != '.')continue;
					for(int d=0;d<5;d++){
						int tx = p2+dirx[d];
						int ty = p3+diry[d];
						if(map[p1+1][tx][ty]==' '){
							map[p1+1][tx][ty] = (p1+1==k)?'*':'.';
						}
					}
				}
			}
		}
		
		for(int p1=k;p1>1;p1--){
			for(int p2=1;p2<=n;p2++){
				for(int p3=1;p3<=m;p3++){
					if(map[p1][p2][p3]!='*')continue;
					for(int d=0;d<5;d++){
						int tx = p2+dirx[d];
						int ty = p3+diry[d];
						if(map[p1-1][tx][ty]=='.'){
							map[p1-1][tx][ty] = '*';
						}
					}
				}
			}
		}

		printf("Robbery #%d:\n",icase);
		bool flag = false;
		for(int p1=1;p1<=k;p1++){
			int x=-1,y;
			for(int p2=1;p2<=n;p2++){
				for(int p3=1;p3<=m;p3++){
					if(map[p1][p2][p3]=='*'){
						if(x==-1){
							x = p2;
							y = p3;
						}else{
							x = -2;
						}
					}
				}
			}
			if(x>0){
				flag = true;
				printf("Time step %d: The robber has been at %d,%d.\n",p1,x,y);
			}else if(x==-1&&p1==1){
				flag = true;
				puts("The robber has escaped.");
				break;
			}
		}
		if(!flag){
			puts("Nothing known.");
		}
		puts("");
	}
	return 0;
}
```


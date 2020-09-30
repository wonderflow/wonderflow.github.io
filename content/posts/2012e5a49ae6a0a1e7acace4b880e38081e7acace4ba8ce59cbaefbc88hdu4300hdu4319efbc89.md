+++
author = "admin"
date = "2012-07-27T12:59:12Z"
slug = "2012/07/27/2012e5a49ae6a0a1e7acace4b880e38081e7acace4ba8ce59cbaefbc88hdu4300hdu4319efbc89"
title = "2012多校第一、第二场（hdu4300~hdu4319）"
Categories = ["ACM"]
Tags = ["ACM", "Dinic", "DP", "hdu", "多校", "最小割", "网络流", "解题报告"]
+++

说一下自己搞出来的题目吧，感觉多校其实比较水的，题目也不难，模板题，陈题很多，不过两场我们队的名次都很烂诶，30开外了都，抗不住啊！出现的问题就是队里的模板太少、不全。不过也好，做一场补一场的模板。

hdu4300《Clairewd’s message》是个字符串，利用next数组求解，小trick就是next数组求出来的值要小于(len-1)/2才取。即：while(len-next[dx]<=(len-1)/2)dx = next[dx];

```
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;
int map[30];
int len;
int s[101000];
int next[101000];
char str[101000];
void get_next()
{
    int i,j;i=0;next[0]=-1;j=-1;
    while(i<len)
   {
      if(j==-1||map[s[j]]==s[i])
     {
          ++i;++j;
           next[i]=j;
      }
        else  j=next[j];
    }
}

int main()
{
	int T;
	scanf("%d",&T);
	while(T--){
		scanf("%s",str);
		len = strlen(str);
		for(int i=0;i<len;i++){
			map[str[i]-'a'] = i;
		}
		scanf("%s",str);
		len = strlen(str);
		for(int i=0;i<len;i++){
			s[i] = str[i] - 'a';
		}
		get_next();
	//	cout<<next[len]<<endl;
		int dx = len;
		while(len-next[dx]<=(len-1)/2)dx = next[dx];
	//	cout<<next[dx]<<endl;
		for(int i=0;i<len-next[dx];i++){
			printf("%c",s[i]+'a');
		}
		for(int i=0;i<len-next[dx];i++){
			printf("%c",map[s[i]]+'a');
		}
		printf("\n");
	}
	return 0;
}
```

hdu4307《Matrix》先搞出一个公式（HJWAJ）搞的，然后看见这个公式YY一下，网络流，最小割模型。
D = sigama(bij)-(sigama(bij)(ai=0,aj=0)+sigama(bij)(ai=1,aj=0)+sigama(bij)(ai=0,aj=1))
左边一个源点，右边一个汇点，中间1000个点，表示ai。源点表示ai=0，汇点表示ai=1，对于ai=0的情况，bij作为作为一条边连向源点（小优化：因为aj无论是0是1，只要ai为0，就要连源点，那么对于每个ai，直接求和100个j，变成一条边再连，就能少掉1000*999条边）；然后因为ai=1的时候，一定会取ci的值，那么ci作为一条边让ai连向汇点。
最后看ai=1，aj=0的情况，就是说，i处的值和j处的值分属两个不同的阵营的生活，要多割掉一条边权，那么直接在i和j之间再连一条bij的边权就过了。
想出建边方法后，这种数学外壳的网络流，估计还是他会先碰到，所以让HJWAJ练练手写了一下，贴下他的代码：

```
#include<iostream>
#include<fstream>
#include<iomanip>
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<cstdlib>
#include<cmath>
#include<set>
#include<map>
#include<queue>
#include<stack>
#include<string>
#include<vector>
#include<sstream>
using namespace std;
#define Node 2100  //点数
#define Edge 2400000  //边数
#define  INF ((1<<30)-1)  //无穷大,根据flowtype的不同，此值要修改大小
#define min(x,y)  ((x)<(y)?(x):(y))
#define  flowtype int
struct Graph
{
    struct G_Edge
    {
        int to,next;
        flowtype val;
    }edge[Edge];
    int head[Node];//表头
    int level[Node];//层次图
    int que[Node];//队列
    int vis[Node];//层次图记录是否访问过
    int p,N;//N为点数
    void init()
    {
        p=0;
        for(int i=0;i<=N+1;i++)head[i]=-1;
    }
    void insert(int a,int b,flowtype c,flowtype c_rev)
 //前向星构图，c是正向边流量，c_rev是反向边.
 //网络流的边都是成对构造,有性质:对于任意一对边，知道了一个，另一个只要异或1.
    {
        edge[p].next=head[a];
        edge[p].to=b;
        edge[p].val=c;
        head[a]=p++;
        edge[p].next=head[b];
        edge[p].to=a;
        edge[p].val=c_rev;
        head[b]=p++;
    }
    bool bfs_level(int src,int des)//构造层次图
    {
        memset(level,-1,sizeof(level));
        int q,p,index,i;
        q=p=0;
        que[q++]=src;
        level[src]=0;
        while(p!=q)
        {
            index=que[p++];
            for(i=head[index];i!=-1;i=edge[i].next)
            {
                int v=edge[i].to;
                if(edge[i].val>0&&level[v]==-1)
                {
                    level[v]=level[index]+1;
                    que[q++]=v;
                }
            }
        }
        return level[des]!=-1;
    }
    flowtype dinic(int cur,int des,flowtype lim)//多路增广dinic
    {
        if(cur==des)return lim;
        int index,i;
        flowtype mul_flow=0,one_flow;//多路增广用
        for(i=head[cur];i!=-1&&lim-mul_flow>0;i=edge[i].next)
        {
            index=edge[i].to;
            if(edge[i].val>0&&level[cur]==level[index]-1)
            {
                one_flow=dinic(index,des,min(edge[i].val,lim-mul_flow));
                if(!one_flow)level[index]=-1;
                else
                {
                    mul_flow+=one_flow;
                    edge[i].val-=one_flow;
                    edge[i^1].val+=one_flow;
                }
            }
        }
        return mul_flow;
    }
    flowtype max_flow(int src,int des)
    {
        flowtype ans=0;
        while(bfs_level(src,des))ans+=dinic(src,des,INF);
        return ans;
    }
};
Graph g;
int main()
{
    int n,a,i,j,t,sum,all;
    scanf("%d",&t);
    while(t--)
    {
        all=0;
        scanf("%d",&n);
        g.N=n+100;
        g.init();
        for(i=0; i<n; i++)
        {
            sum=0;
            for(j=0; j<n; j++)
            {
                scanf("%d",&a);
                sum+=a;
                if(i!=j)g.insert(i+1,j+1,a,0);
            }
            all+=sum;
            g.insert(0,i+1,sum,0);
        }
        for(i=0; i<n; i++)
        {
            scanf("%d",&a);
            g.insert(i+1,n+1,a,0);
        }
        //cout<<"ok"<<endl;
        printf("%d\n",all-g.max_flow(0,n+1));
    }
    return 0;
}
```

hdu4308《Saving Princess claire》就是个宽搜，把所有P的点看成一个点加入队列就可以了。

hdu4309《Seikimatsu Occult Tonneru》就是个裸的网络流，但是比赛的时候N多人没过，难点在哪？
因为给的模型如果数据量很大的话，就是一个很难的模型：问给你网络中n条边，有的有费用，有的没费用，有费用的点只要付走一次的钱，以后就变成免费的了。如果这题数据量大的话，还真的无解了，不会啊。（就是这个题吧数据量从12变成100，知道的怎么解决这样问题的朋友可以在下面留言大家一起探讨下啊~谢谢~）
但是坑爹的就是，这题有费用的边最多只有12条，暴力枚举每一条有费用的边用不用就行了。更坑爹的是，这题有费用的边还可以只走一次再不走它，然后这一次是不要钱的！！就是不用有费用的边的话，流量也要给1.
擦！题意啊，下次一定要好好读题。

hdu4310《Hero》状态压缩DP，现在发现状态压缩DP的题真多，这题只能算是最裸的。就是拿一个int的每一位是0是1表示一个敌人死没死，然后枚举。记忆化爆搜的思想啊。初学状态压缩的同学可以学习一下，我贴个代码方便下大家：

```
#include<iostream>
#include<cstdio>
using namespace std;
#define MAXN 20
#define bin(x) (1<<x)
#define LL long long
#define MIN(x,y) (x)<(y)?(x):(y)
#define INF 99999999999999LL
LL dp[1<<MAXN];
int hp[MAXN];
int hurt[MAXN];
int n;
LL calc(int s){
	LL dps = 0;
	for(int i=0;i<n;i++){
		if(s&bin(i))dps+=hurt[i];
	}
	return dps;
}
int main()
{
	while(scanf("%d",&n)!=EOF){
		for(int i=0;i<n;i++){
			scanf("%d%d",&hurt[i],&hp[i]);
		}
		for(int i=0;i<bin(n);i++)dp[i] = INF;
		dp[bin(n)-1] = 0;
		for(int i=bin(n)-2;i>=0;i--){
			for(int j=0;j<n;j++){
				if(!(i&bin(j)))dp[i] = MIN(dp[i],dp[i^bin(j)]+calc(i^bin(j))*hp[j]);
			}
		}
		printf("%lld\n",dp[0]);
	}
	return 0;
}
```

hdu4317《Unfair Nim》状态压缩DP，dp[i][j]表示到第i位进位情况为j的时候最少需要增加的石子个数。j是1<<10的大小，每一位的1、0表示当前堆石子前一位的进位情况。然后dp的转移又是2^10的，表示每一堆到底增加还是不增加。转移的时候用三个值(1、当前位原有值，2、当前得到的进位，3、当前准备增加的)异或得到一个新值，算最后的新值中1的个数，如果是偶数个的话就表示可以。用3个值两两异或然后与起来得到新的值的进位，然后用dp[i][j] = min(dp[i][j],dp[i+1][s]+cnt(k)*(1<<i));转移。
看代码更清楚点吧：不过阿森说我这个还可以优化，不知道那些0ms的人怎么过的，我7s过的，给了10s。（知道怎么优化的朋友在下面留言大家探讨下哈，谢谢~）

```
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;
#define INF ((1<<30)-1)
int rec[23];
int dp[23][1<<12];
int ca[23];
int maxx = 0;
int n;
int ans;
int count(int s){
	int ret = 0;
	while(s){
		s>>=1;
		ret++;
	}
	return ret;
}

inline int cnt(int s){
	int ret = 0;
	while(s){
		if(s&1)ret++;
		s>>=1;
	}
	return ret;
}

int main()
{
	while(scanf("%d",&n)!=EOF){
		maxx =  -1;
		for(int i=0;i<n;i++){
			scanf("%d",&rec[i]);
			maxx = max(maxx,count(rec[i]));
		}
		memset(dp,0,sizeof(dp));
		for(int i=0;i<=maxx+1;i++){
			for(int j=0;j<(1<<10);j++){
				dp[i][j] = INF;
			}
		}
		dp[maxx+1][0] = 0;
		for(int i=0;i<=maxx+1;i++){
			ca[i] = 0;
			for(int j=0;j<n;j++){
				ca[i]+=(1<<j)*((rec[j]&(1<<i))?1:0);
			}
		}
		for(int i=maxx;i>=0;i--){
			for(int j=0;j<(1<<n);j++){// 前位进位
				for(int k=0;k<(1<<n);k++){// 当前位增加值
					if((cnt(j^k^ca[i])&1)==0){//ca[]: 当前数字值
						int s = ((j&k)|(j&ca[i])|(k&ca[i]));
						dp[i][j] = min(dp[i][j],dp[i+1][s]+cnt(k)*(1<<i));
					//	if(dp[i][j]!=INF)cout<<j<<" "<<k<<" "<<ca[i]<<" "<<s<<endl;
					}
				}
			//	cout<<dp[i][j]<<endl;
			}
		}
//		cout<<"ans:"<<endl;
		if(dp[0][0]==INF)puts("impossible");
		else printf("%d\n",dp[0][0]);
		
	}

}
```

hdu4318《 Power transmission》最短路dijkstra算法，就是把加变成乘就行了。没什么trick。类似的以前有一个抓到小偷的概率的问题，每个路口通向不同的点抓到小偷的概率不同的，问你最大的一条路的抓到小偷的概率是多少。两个一样的题目，不过后者题面更好玩，更容易记住。

其他题目都是gaozhen和HJWAJ搞定的了，包括2~3个线段树，2个模板题，2~3个DP还有好多计算机何题。队里的同学有兴趣可以去询问他们哈。



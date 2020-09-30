+++
author = "admin"
date = "2012-08-02T05:37:32Z"
slug = "2012/08/02/usaco-raucous-rockers-beef-mcnuggets-fence-rails"
title = "USACO Raucous Rockers && Beef McNuggets && Fence Rails"
Categories = ["ACM", "usaco"]
Tags = ["ACM", "DP", "usaco", "搜索"]
+++

Raucous Rockers
题目大意：有N(<=20)首歌，打算放在M(<=20)张CD中，每张CD可存储T(<=20)分钟的音乐，给定每首歌的时长，问如何将歌曲按照日期（也就是输入）的顺序，存在这M张CD中，并且每首歌不可以分开存在多张CD上，使得存储的歌曲的数目最多。
解决思路：就是个01背包的变形吧？用滚动数组迭代。dp[m][t],表示我前m张cd用到第t分钟的时候，最多能放多少首歌。N首歌的那一个纬度可以省去，因为每次都只要用到当前纬度的状态。
与01背包的区别就是每次做完一个物品，我都要把影响扩展到后面的每一个容量，也就是说，我前面那些CD就能放下这么多歌了，那么我后面的CD也至少能放那么多歌。

```
/*
ID: bobo
PROG: rockers
LANG: C++
*/
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<algorithm>
#include<cmath>
using namespace std;
int dp[25][25];
int n,t,m;
int rec[25];
int main(){
//	freopen("rockers.in","r",stdin);
//	freopen("rockers.out","w",stdout);
	while(scanf("%d%d%d",&n,&t,&m)!=EOF){
		for(int i=0;i<n;i++)scanf("%d",&rec[i]);
		memset(dp,0,sizeof(dp));
		for(int i=0;i<n;i++){
			for(int j=m;j>=1;j--){
				for(int k=t;k>=rec[i];k--){
					dp[j][k] = max(dp[j][k],dp[j][k-rec[i]]+1);
					//cout<<dp[i][j][k]<<endl;
				}
			}
			for(int j=1;j<=m;j++){
				dp[j][0] = max(dp[j][0],dp[j-1][t]);
				for(int k=1;k<=t;k++){
					dp[j][k] = max(dp[j][k],dp[j][k-1]);
				}
			}
		}
		printf("%d\n",dp[m][t]);

	}
	return 0;
}
```

Beef McNuggets 
题目大意：有n（<=10）个整数a1, a2, …, an（<=256），问是否存在无法用a1～an的线性表达式（k1a1 + k2a2 + … + knan）组成的数，如果有，其中最大的数是多少？（这个数的上限为2,000,000,000！）

就是一个多重背包了，关键就是一点，两个互质的数a,b，他们不能构成的最大的数是a*b-a-b；（这个性质其实我还不太会证，AC的时候就是知道两个互质的数不能构成的数字不会很大，后来ac了看usaco的analysis才知道的。如果看到这里的同学知道怎么证的，麻烦留言互相学习啊~谢谢~）
至于背包的一些传统DP解法，有一篇文章[《背包九讲》](https://wonderflow.info/images/2012-08-02-usaco-raucous-rockers-beef-mcnuggets-fence-rails/背包九讲.pdf)，讲的非常好，如果之前没看过，大家可以点开来看看。
```
/*
ID: bobo
PROG: nuggets
LANG: C++
*/
//Given two relatively prime numbers N and M,
//the largest number that you cannot make is NM - M - N
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<algorithm>
#include<cmath>
using namespace std;
int n;
bool dp[1000000];
int rec[20];
int gcd(int a,int b){
	return b?gcd(b,a%b):a;
}
int main(){
//	freopen("nuggets.in","r",stdin);
//	freopen("nuggets.out","w",stdout);
	while(scanf("%d",&n)!=EOF){
		bool f = false;
		int minn = 256;
		for(int i=0;i<n;i++){
			scanf("%d",&rec[i]);
			minn = min(minn,rec[i]);
			if(rec[i] == 1)f = true;
		}
		if(n==1||f){
			puts("0");
			continue;
		}
		int ans = rec[0];
		for(int i=1;i<n;i++)ans = gcd(ans,rec[i]);
		if(ans!=1){
			puts("0");
			continue;
		}
		memset(dp,0,sizeof(dp));
		dp[0] = true;
		for(int i=0;i<n;i++){
			for(int j=0;j<1000000-rec[i];j++){
				if(dp[j])dp[j+rec[i]] = true;
			}
		}
		ans = -1;
		int num = 0;
		for(int i=1;i<1000000;i++){
			if(dp[i]){
				num++;
			}else{
				ans = i;
				num = 0;
			}
			if(num>=minn)break;
		}
		printf("%d\n",ans);
	}
}
```

Fence Rails 题意是：给你N(1 <= N <= 50)段长度不一的木材，另外有R(1 <= R <= 1023)条栏杆需要修复，1 <= ri <= 128。问以这些木材为原材料，能够做出最多多少块栏杆？（做出的栏杆需要是完整的，不能拼接）
关于这题，屈啸师兄的[题解](http://qxavier.info/2011/07/01/usaco-section-4-1-fence-rails/)非常给力啊，大家可以去那里围观，我就不赘述了。中心思想是搜索+剪枝。PS：实际上前面两题的题意我也是从屈啸师兄那粘贴过来的。:-D

```
/*
ID: bobo
PROG: fence8
LANG: C++
*/
/*
1、如果可以组成k块栏杆，那么最小的前k块栏杆一定是其中的一个解。
	因为就算存在不连续的解，我们也可以将较大的栏杆替换成较小的栏杆，
	最终转化为最小的k块栏杆。
2、根据R和ri ，可以看出栏杆的长度有很多都是重复的。
	将木材和栏杆排序，搜索时记录下当前的栏杆是选择哪块木材。
	搜索到当前栏杆i时，如果发现与上一个栏杆i+1的长度相同，
	则我们可以从i+1块栏杆所选择的那块木材开始搜。
3、假设前k块栏杆是题目的解，那么在这种情况下最大浪费掉的木材maxWaste就是sumBoard – sumRail[k]。
	在搜索过程中，如果发现某块木材的剩余量比最小的栏杆都小，就可以把他计入curWaste，
	如果发现curWaste > maxWaste，那么肯定无解。
4、我们还可以将R的范围减小。
	如果发现sumRail[k] <= sumBoard并且sumRail[k+1] > sumBoard，那么我们可以将R减小到k。
 * */
#include<iostream>
#include<cstdio>
#include<cstring>
#include<string>
#include<algorithm>
#include<cmath>
using namespace std;
int n,m;
int board[56],rail[1024];
int sumboard,sumrail,maxwaste,curwaste;
int vis[1024];

bool search(int r){
	if(r<0)return true;
	int i = (rail[r]==rail[r+1])?vis[r+1]:0;
	for(;i<n;i++){
		if(board[i]>=rail[r]){
			board[i]-=rail[r];
			if(board[i]<rail[0]){
				curwaste+=board[i];
				if(curwaste>maxwaste){
					curwaste-=board[i];
					board[i]+=rail[r];
					continue;
				}
			}
			vis[r] = i;
			if(search(r-1))return true;
			if(board[i]<rail[0])curwaste-=board[i];
			board[i]+=rail[r];
		}
	}
	return false;
}

void solve(){
	sort(rail,rail+m);
	sort(board,board+n);
	int p;
	for(p=0;p<m;p++){
		sumrail+=rail[p];
		if(sumrail>sumboard){
			sumrail-=rail[p];
			break;
		}
	}
	p--;
	maxwaste = sumboard-sumrail;
	while(p>=0){
		curwaste = 0;
		memset(vis,0,sizeof(vis));
		if(search(p)){
			printf("%d\n",p+1);
			return ;
		}
		maxwaste+=rail[p--];
	}
	puts("0");
}

int main(){
//	freopen("fence8.in","r",stdin);
//	freopen("fence8.out","w",stdout);
	while(scanf("%d",&n)!=EOF){
		sumboard = 0;
		sumrail = 0;
		for(int i=0;i<n;i++){
			scanf("%d",&board[i]);
			sumboard+=board[i];
		}
		scanf("%d",&m);
		for(int i=0;i<m;i++){
			scanf("%d",&rail[i]);
		}
		solve();
	}
}
```


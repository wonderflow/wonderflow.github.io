+++
author = "admin"
date = "2012-08-07T14:04:53Z"
slug = "2012/08/07/e5ad97e7aca6e4b8b2e79a84e7bc96e8be91e8b79de7a6bb"
title = "字符串的编辑距离"
Categories = ["Algorithm", "Poj"]
Tags = ["ACM", "algorithm", "DP", "poj", "字符串", "解题报告"]
+++

编辑距离，又称Levenshtein距离（也叫做Edit Distance），是指两个字串之间，由一个转成另一个所需的最少编辑操作次数。
许可的编辑操作包括1、将一个字符替换成另一个字符；2、插入一个字符3、删除一个字符。

根据《柔性字符串匹配》一书的介绍，一共有四种方法。
1、最古老，最灵活的动态规划解法。
2、基于自动机的公式法（常用语错误距离允许较小的情况）
3、用位并行来模拟其他方法。
4、先用简单的条件过滤掉文本中大段编辑距离超过范围的不相关文本，再用其他算法进行匹配。

先介绍一下最基本的动态规划解法。空间复杂度和时间复杂度均为O（n^2）。
正好[poj3356](http://poj.org/problem?id=3356)就是个这样的题目，有兴趣的可以做一下。

用一个数组dp[x][y]计算编辑距离，矩阵dp[i][j]表示：将x(1..i)变成y(1..j)所需的最少编辑操作次数。

那么我们很容易联想到这样的转移：

若x[i]与y[j]相等，那么dp[i][j] <- dp[i-1][j-1]
若不相等，则dp[i][j] <- dp[i-1][j-1]+1 （这个一步操作，就是把y[j]修改为x[i]的操作，使用了替换）
然后可以考虑删掉一个字符即 dp[i][j] <- dp[i][j-1]+1 ，这样就相当于是从x[i]和y[j-1]的情况转移过来，把y[j]直接删掉了。
还存在最后一种情况，就是增加一共字符 dp[i][j] <- dp[i-1][j]+1 ，如果从x[i-1],y[j]的情况转移过来，那么相当于x[i]是任意不用考虑的，即在y[j]后面添加一共与x[i]相同的字符与之达到匹配。

这样的话，我们就把有且仅有的3种状态转移讨论完了。

初始化：dp[0][0] = 0; dp[i][0] = dp[0][i] = i;

```
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;
#define INF ((1<<30)-1)
char str1[1024],str2[1024];
int dp[1024][1024];
int main(){
	int n,m;
	while(scanf("%d",&n)!=EOF){
		memset(str1,-1,sizeof(str1));
		memset(str2,-1,sizeof(str2));
		scanf("%s%d%s",str1,&m,str2);
		for(int i=0;i<=n;i++){
			for(int j=0;j<=m;j++){
				dp[i][j] = INF;
			}
		}
		int len = max(n,m);
		for(int i=0;i<=len;i++){
			dp[i][0] = i;
			dp[0][i] = i;
		}
		for(int i=1;i<=n;i++){
			for(int j=1;j<=m;j++){
				if(str1[i-1]==str2[j-1]){
					dp[i][j] = min(dp[i][j],dp[i-1][j-1]);//no change
					dp[i][j] = min(dp[i][j-1]+1,dp[i][j]);//delete one
					dp[i][j] = min(dp[i-1][j]+1,dp[i][j]);//add one
				}else {
					dp[i][j] = min(dp[i][j-1]+1,dp[i][j]);//delete one
					dp[i][j] = min(dp[i-1][j]+1,dp[i][j]);//add one
					dp[i][j] = min(dp[i-1][j-1]+1,dp[i][j]);//change one
				}
			}
		}
		printf("%d\n",dp[n][m]);
	}

	return 0;
}
```


然后再来看看这个题目： [POJ 1035 Spell Checker](http://poj.org/problem?id=1035)
题意就是要你在包含10000个单词的字典中找编辑距离为0或为1的匹配串，就是一个模糊匹配。

如果这个时候我们再用上面介绍的动态规划算法来做，那么显然时候超时的。

这个时候，我们就必须要灵活的采用下面介绍的几个方法了。

先看方法四，先用简单的方法过滤掉一匹不可能的匹配。在这个题目里，显然，我们要找的是编辑距离最大为1的，那么如果字典的串和我要匹配的串长度之差的绝对值超过了1，显然就可以过滤掉。

然后我们再用方法二，构造状态机。（其实按照我的理解，最朴素的状态机就是分情况讨论，在哪些情况下状态是怎么样的，就匹配，哪些情况下状态怎么样的是不匹配。再通俗一些，就是一大堆if..else..）

这里，因为编辑距离只有1，我们可以分情况讨论：（不同的状态嘛,状态机的思想我们平时就用了，就是分情况讨论，O(∩_∩)O哈哈~）。
1、模式串比匹配串短1，那么就是要添加一个字符，扫一遍就可以看出来了。
2、长度相等，那么就是某个字符要替换，扫一遍同样也看出来了（扫完仅一个字符不同）
3、匹配串比模式串长度短1，那么也是扫一遍，看是不是除了多一个字符以外就是完全匹配的。

这样一来，运行速度明显上升了。

网上找了一份写的比较好的代码拿过来给大家借鉴一下:原址为[鸵鸟](http://www.cnblogs.com/asuran/archive/2009/10/01/1577199.html)

```
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include <utility>
#include <functional>
using namespace std;

struct Word
{
    Word(int p, int l, const string& s):pos(p),len(l),word(s){};
    int pos;
    int len;
    string word;
};
class CheckRange
{
    int m_len;
public:
    CheckRange(int len):m_len(len){};
    bool operator()(const Word& w)
    {
        return m_len <= w.len;
    }
};
class LenLess{
public:
    bool operator()(const Word& w1, const Word& w2)
    {
        return w1.len < w2.len;
    };
};
class PosLess{
public:
    bool operator()(const Word& w1, const Word& w2)
    {
        return w1.pos < w2.pos;
    };
};
int checkString(const string& str1, const string& str2)
{
    int cnt = 0;
    if (str1.size() < str2.size())
    {
        int i = 0, j = 0;
        while(i < str1.size() && j < str2.size())
        {
            if (str1[i] != str2[j])++cnt,++j;
            else ++i,++j;
        }
        cnt += str1.size() - i + str2.size() - j;
    }
    else if (str1.size() == str2.size())
    {
        for (int i = 0; i < str1.size(); ++i)
            if (str1[i] != str2[i])++cnt;
    }
    else 
        cnt = checkString(str2, str1);
    return cnt;
};

int main(int argc, char* argv[])
{
    vector<Word> Dic;

    string w;
    int i = 0;
    while (cin >> w && w != "#")
    {
        Dic.push_back(Word(i,w.size(),w));
        ++i;
    }
    
    std::sort(Dic.begin(),Dic.end(),LenLess());

    vector<Word> v;
    vector<Word>::iterator iter, beg, end;
    vector<string> op;
    while (cin >> w && w != "#")
    {
        v.clear();
        beg = std::find_if(Dic.begin(), Dic.end(), CheckRange(w.size() - 1));
        end = std::find_if(Dic.begin(), Dic.end(), CheckRange(w.size() + 2));

        bool same = false;
        for (iter = beg; iter != end; ++iter)
        {
            int r = checkString(w, iter->word);
            if (r == 0)
            {
                v.clear();
                v.push_back(*iter);
                same = true;
                break;
            }
            else if (r == 1)
                v.push_back(*iter);
        };

        if (same == true) op.push_back(w + " is correct\n");
        else
        {
            std::sort(v.begin(),v.end(),PosLess());
            string tp = w + ":";
            for (int i = 0; i < v.size(); ++i)
                tp += " " + v[i].word;
            tp +="\n";
            op.push_back(tp);
        }
    }
    
    for (int i = 0; i < op.size(); ++i)
        cout << op[i];
    return 0;
}
```

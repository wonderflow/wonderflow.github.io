---
author: admin
comments: true
date: 2012-07-09 05:52:34+00:00
layout: post
slug: zoj-fans-and-gems%ef%bc%88%e6%90%9c%e7%b4%a2%ef%bc%9f%e6%a8%a1%e6%8b%9f%ef%bc%9f%e6%81%b6%e5%bf%83%e7%9a%84%e9%a2%98%e3%80%82%e3%80%82%ef%bc%89
title: ZOJ1237 Fans and Gems（搜索？模拟？恶心的题。。）
wordpress_id: 155
categories:
- ACM
- Zoj
tags:
- ACM
- C++
- Zoj
- 暴力
- 模拟
- 狗狗四十题
- 解题报告
---

此题的类型应该就是一个暴力搜索，但是搜索的变换过程非常之恶心。一旦处理不好，就是多写上百行代码的事情。
自己写了五个小时，写不下去了，因为实在恶心到了，处理的方法不好。最后一看[watashi](http://watashi.ws)的代码，羞愧的五体投地，就把自己的代码人道毁灭了！

那就学学watashi大牛的代码吧。也挺好。说明一下，代码都是watashi的，我自己只是做了一些适当的注释。其中用的什么样的方法，基本注释里面已经写的很清楚。看来暴力的题目，熟练使用STL才是王道啊。

再结合昨天看watashi代码的经验就是：
该暴力的题，全部使用STL，让代码清楚简洁干净利落到极致！
该高效的题，尽量使用位运算，不让任何多余的操作降低自己程序的效率！

恩，学到不少~~

```
#include <set>
#include <queue>
#include <cctype>
#include <cstdio>
#include <string>
#include <vector>
#include <utility>
#include <algorithm>

using namespace std;

typedef vector<string> VS;//这样的string数组，可以用来存图

//对于整张图遍历，检查是否为空，isdigit函数的功能是看当前字符是否是0~9
bool empty(const VS& vs) {
	for (VS::const_iterator i = vs.begin(); i != vs.end(); ++i) {
		for (string::const_iterator j = i->begin(); j != i->end(); ++j) {
			if (isdigit(*j)) {
				return false;
			}
		}
	}
	return true;
}

//调换这张图的顺序，准确的说是左右对称的这样调换
VS revit(const VS& vs) {
	VS ret(vs);
	for (VS::iterator i = ret.begin(); i != ret.end(); ++i) {
		reverse(i->begin(), i->end());
	}
	return ret;
}

//转置变换，x坐标和y坐标交换顺序
VS rotit(const VS& vs) {
	int n = vs.size(), m = vs[0].size();
	VS ret(m);
	for (int i = 0; i < n; ++i) {
		for (int j = 0; j < m; ++j) {
			ret[j] += vs[i][j];
		}
	}
	return ret;
}

//对于全图，进行LEFT操作，采用的方法是先挑gems和fly字符，最后碰到墙之前结算应该添加的空格数
//因为墙是不动的，肯定要填满那么多列的空格才能放墙，用string直接加在这里处理起来显得非常方便
string leftit(const string& s) {
	string ret;
	for (int i = 0; i < (int)s.size(); ++i) {
		switch (s[i]) {
		case '#':
			while ((int)ret.size() < i) {
				ret += ' ';
			}
			ret += '#';
			break;
		case '@':
		case '1':
		case '2':
		case '3':
			ret += s[i];
			break;
		}
	}
	return ret;
}

//很高明的函数类型定义，相当于定义了一个参数是const VS&，返回值是VS的函数类型
//类型名称是GAOIT
typedef VS GAOIT(const VS& vs);

//重载上面的leftit，改为对整张图的LEFT操作
VS leftit(const VS& vs) {
	VS ret;
	for (VS::const_iterator i = vs.begin(); i != vs.end(); ++i) {
		ret.push_back(leftit(*i));
	}
	return ret;
}

//对整张图进行RIGHT操作，方法就是先把整张图左右对称变换，再进行LEFT操作，
//再把整张图左右对称变换回来，watashi太有才了！
VS rightit(const VS& vs) {
	return revit(leftit(revit(vs)));
}

//对整张图的UP操作，变成先把整张图进行转置变换，再进行LEFT操作，再转置回来
VS upit(const VS& vs) {
	return rotit(leftit(rotit(vs)));
}

//向下操作就是先把整张图进行转置变换，再进行RIGHT操作，再转置变换回来，
//也正是看到了这四个函数，我瞬间觉得自己的几百行代码弱爆了！
VS downit(const VS& vs) {
	return rotit(rightit(rotit(vs)));
}

const int dx[] = {0, 1, 0, -1};
const int dy[] = {1, 0, -1, 0};

//DFS把相同的连着的gems消掉，如果没有相同的，在最后还原
int dfs(VS& vs, char ch, int x, int y) {
	if (vs[x][y] != ch) {
		return 0;
	} else {
		int ret = 1;
		vs[x][y] = ' ';
		for (int i = 0; i < 4; ++i) {
			ret += dfs(vs, ch, x + dx[i], y + dy[i]);
		}
		return ret;
	}
}

bool clearit(VS& vs) {
	bool ret = false;
	char ch;
	int n = vs.size(), m = vs[0].size();
	for (int i = 0; i < n; ++i) {
		for (int j = 0; j < m; ++j) {
			if (isdigit(vs[i][j])) {
				ch = vs[i][j];
				if (dfs(vs, ch, i, j) == 1) {
					vs[i][j] = ch;//dfs完发现就找到了自己，显然要把字符还原
				} else {
					ret = true;
				}
			}
		}
	}
	return ret;
}
/*
void dump(const VS& vs, const string& s = "") {
	puts(s.c_str());
	for (VS::const_iterator i = vs.begin(); i != vs.end(); ++i) {
		puts(i->c_str());
	}
}
*/
const int LIMIT = 18;
const char* dirs = "DLRU";
const GAOIT *gaoit[] = {downit, leftit, rightit, upit};	//函数指针这么赋值，干净利落，学到了~


//用set判重复，宽搜，对STL使用的淋漓尽致啊~
void gao(VS vs) {
	set<VS> s;
	queue<pair<VS, string> > q;//这个队列左边是图，右边是答案

	s.insert(vs);
	q.push(make_pair(vs, ""));
	while (!q.empty()) {
		// dump(q.front().first, q.front().second);
		for (int i = 0; i < 4; ++i) {
			vs = q.front().first;
			do {						//做向一个方向并拢的操作的同时，每次都刷新一下地图
				vs = gaoit[i](vs);
			//	dump(vs, q.front().second + (char)('0' + i));
			} while (clearit(vs));
			if (s.count(vs) > 0) {
				continue;
			}
			s.insert(vs);
			if ((int)q.front().second.length() == LIMIT) {
				throw string("-1");
			} else if (empty(vs)) {
				throw q.front().second + dirs[i];
			}
			q.push(make_pair(vs, q.front().second + dirs[i]));
		}
		q.pop();
	}
	throw string("-1");
}

int main() {
	int re, n, m;
	char buf[1024];
	VS vs;

	scanf("%d", &re);
	for (int ri = 1; ri <= re; ++ri) {
		scanf("%d%d", &n, &m);
		gets(buf);
		vs.clear();
		for (int i = 0; i < n; ++i) {
			gets(buf);
			vs.push_back(buf);
		}
		gets(buf);
		try {
			gao(vs);
		} catch (const string& e) {
			puts(e.c_str());
		}
	}

	return 0;
}

/*
值得注意的是这句话，“There is one line after each map, ignore them.”一定要用gets把它读掉，因为这行里是有字符的。
*/
```

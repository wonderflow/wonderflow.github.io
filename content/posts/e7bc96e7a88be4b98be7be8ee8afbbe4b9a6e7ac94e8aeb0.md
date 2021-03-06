+++
author = "admin"
date = "2012-08-28T12:31:10Z"
slug = "2012/08/28/e7bc96e7a88be4b98be7be8ee8afbbe4b9a6e7ac94e8aeb0"
title = "《编程之美》读书笔记"
Categories = ["Algorithm", "IT", "Reading Notes"]
Tags = ["书目推荐", "面试 算法"]
+++

总的来说，编程之美是一本讲面试算法的书，至于面试问算法到底问到的几率有多大，我不知道。不过这本书本身的很多问题是非常有意思的。

0.1、前言趣题：
	房间里有三盏灯，屋外有三个开关，分别控制三盏灯，进了房间才能知道哪个灯泡是亮的，问只进入房间一次，怎么区分哪个开关对应哪个灯？
这个题的有意思之处在于让你拓宽思维，不要被既定的线路束缚住。区别三个灯泡，一定需要两个条件，一个是灯泡亮否，还有一个条件哪里来呢？答案是温度，灯泡开了一会，表面温度自然就会上升。

0.2、前言感悟：
	面试如考试：基础很重要，但是考前看看面试题库也是个不错的选择。
	面试就是探讨：多把自己的想法告诉面试官，对问题的思考或许比问题的答案更重要。
	实践出来的才属于自己。

1.1、让CPU显示所需要的曲线？
	这个题的关键在于对系统函数的了解。适当比例使用sleep().awake()函数等即可。

1.2、按照中国象棋的规则摆放“将、帅”两个棋子的位置让他们不会死。
	这个题就是让我们预先编码“将、帅“所能走到的位置，然后输出合法的编码对。（规则就是编码对所在的x坐标不能相等，即将和帅不能在一条线上）
<!-- more -->
1.3、固定规则的排序问题。
	这个题所要表达的就是搜索加剪枝的思想。因为在工程中，最简单的优化就是把不可能发生的情况排除，这就是剪枝。	

1.4、买东西，单买和组合购买价格不同，找出最省钱的方案。
	贪心只能解决极少数情况，还是要用动态规划来解决。

1.5、在一个很大的文件里，每个ID都会出现两次，只有一个ID出现一次，怎么把这个ID找出来？
	一个很漂亮的解法就是把这些ID异或，最后得到的值就是答案。
       那么，如果现在每个ID出现N次，只有一个ID只出现N-1次，怎么找呢？
	解法是类似的，就是把ID化成二进制然后对应一个数组，每一位加到数组里，mod N加，最后除以那个单独出现的次数就可以了。

1.6、提出动态规划中备忘录的思想

1.7、一块由两条平行线构成的平面被切N刀（刀都是从一条平行线切到另一条），问最后被切成了几部分？
	这种题目有规律，按照规律推出公式。如果总共有N条直线，M个交点，那么区域的数目为N+M+1。现在的问题是交点数怎么求？如果把平行线上定上坐标的话，那么我们所需要求的其实就是左右两边的逆序数的个数。归并排序或者线段树都可以O（nlogn）求出来逆序数。

1.8、电梯调度算法，这个可以根据自己的想象力设计一个算法，由于电梯的层数和电梯运作的过程中可以载人，所以可以考虑贪心的解法。

1.9、面试时间安排算法，调度合理的面试时间。
	初步提出的是搜索，每次尝试一种安排的方法能否成功，然后再回溯。
	后面提出一个区间重复覆盖的问题。每次寻找一个空的区间覆盖，利用堆进行存储。

1.10、多线程编程，涉及到操作系统中一些PV操作的算法设计。

1.11、1.12、1.13：三种NIM游戏。
	1、一列石头，每次取一个，问谁能取到最后一个。利用对称性，除了最中间那个，每次对手取哪个，我总能找到与之对应的那一个。
	2、传统的异或求NIM积，思想还是对称性的思想。（如果异或出来是0，那么你哪堆石头的哪一位被取掉了，问就能再另外一堆里面的这一位取掉。直到最后只剩下最后一堆的最后一位。）
	3、POJ 1067那题的取石子游戏，可以用筛选法，把范围内的必输状态筛出来，也可以用公式。

1.14、1.15、1.17、连连看，数独，俄罗斯方块，扫雷的游戏设计
	如果自己要来设计这两个游戏，要解决哪些问题，首要思考的是哪些方面，怎么解决等等。
	因为这样类型的游戏数据范围都很小，只要想到解决方案就可以了，不用考虑复杂度。

1.16、给你一个表达式，计算其值。
	转化成逆波兰表达式，用堆栈求解。

2.1、二进制数中，1的个数。
```
 	int Count(BYTE v){
 		int num = 0;
 		while(v){
 		      v &= (v-1);
 		      num++;
 		}
 		return num;
 	}
```

2.2、求整数N的阶乘N！的末尾有多少个0？
 	算1～N中5的个数，有一个5就是最后乘积多一个0，因为2肯定比5多。
```
int Count(int v){
    int num = 0;
    while(v){
      num += v / 5;
      v /= 5;
    }
    return num;
}
```
	求N！的二进制表示中最低位1的位置？
	有了第一问的经验，就发现，这一问其实就是求2的因数个数。跟上面的做法是一样的。

2.3、在一堆数据中，有一个数据出现的次数超过了总次数的一半，怎么快速找出这个数据？
	这个可以用对消的思想，每出现两个不一样的ID就消去，最后留下的一个一定是那个出现一半以上的ID。

2.4、给定一个十进制正整数N，写下从1到N的所有整数，然后数出其中所有“1”的个数。
	这个题给我们的提示是一种解决问题的方法：从小的情况开始，考虑一位的情况，然后考虑两位、三位直到N位。（具体做法就是分析各位的情况，推算。）

2.5、在一列长度为N的数中，寻找最大的K个数。
	这个题有O（N）的算法，就是利用快速排序里面的那种思想，每次取一个数，大于这个数的方左边，小与这个数的放右边，然后根据左右边数字的数目选择左边或者右边递归。

2.6、2.7 都是GCD（最大公约数问题）。
	GCD最简单的写法就是一行：
```
	int gcd(int x,int y){
		return (!y)?x:gcd(y,x%y);
	}
```

	还有一种很犀利的做法，就是模拟辗转相除法（这样就避免了mod除巨大的复杂度了。同时，对x,y都是偶数的情况，就把2提出来，做到了log加速。)
```
	BigInt gcd(BigInt x, BigInt y){
		if(x<y)return gcd(y,x);
		if(y==0)return x;
		else{
			if(IsEven(x)){
				if(IsEven(y))return (gcd(x>>1,y>>1)<<1);
				else return gcd(x>>1,y);
			}else{
				if(IsEven(y)) return gcd(x,y>>1);
				else return gcd(y,x-y);
			}
		}
	}
```

2.8、任意给定一个正整数N，求一个最小的正整数M（M>1），使得N*M的十进制表示形式里只含有1和0.
	用搜索（BFS）的方法，假设我们得到了结果，每次添加一个1或者一个0，看能否整除N得到M，不行的话记录余数，再继续BFS下去。

2.9、求Fibonacci数列
	最简单的做法自然是写递归函数求了。
	高效一点的话，记忆化搜索。
	再高效一点的话，矩阵快速幂。

2.10、找最大，最小数。个人觉得正常O（N）扫一遍的方法很容易想到。

2.11、寻找平面上的最近点对。
	最土的方法自然是O（N^2）每个点两两之间试一下咯。
	可以利用分治的思想把复杂度降低到O（NlogN），把数组按x坐标排序，然后用数组中间值把数组分成left和right两部分，最小点对要么来自左半部分，要么来自右半部分，要么就是左半部分x最大数和右半部分最小数的差值。（这边难点在于怎么找左右跨区域的情况，总之就是去掉一些不必要的情况。）

2.12、	给你两个数的集合，让你从两个集合中各取一个数，让和为指定的数。
	O(N^2)的方法就是两个for循环枚举一下。
	还可以把一个数组O（NlogN）排序，然后二分查。

2.13，14，15、子数组的最大乘积，最大和，二维的情况。
	最大乘积比较简单，判断正、负和零的情况即可。最大和的算法在[以前的博文](http://wonderflow.info/archives/409)里面也介绍过了，在此不再赘述。
	二维的情况就是把二维的转化成一维的，相应的复杂度可以稍微少一点，变成O（N^3）。

2.16、最长递增子序列，还是那篇博文。

2.17、数组的循环移位
	用三次reverse函数解决的问题。
```
	Reverse(int* arr, int b, int e){
		for(;b<e;b++,e--){
			int t = arr[e];
			arr[e] = arr[b];
			arr[b] = t;
		}
	}
	RightShift (int* arr, int N, int k){
		K %= N;
		Reverse(arr,0,N-K-1);
		Reverse(arr,N-K,N-1);
		Reverse(arr,0,N-1);
	}
```

2.18、数组分割，使两个子数组的和最接近
	这也是一个经典的DP问题了，假设原数组的和为sum，那么我们就是要完成一个背包，使其得到一个小与等于sum/2的最大值。那么就是每一轮遍历一遍长度为sum的数组，然后对于存在的值加上当前物品的重量。然后再来第二轮这样依次算下去。

2.19、区间重合：赤裸裸的线段树我就不说了。

2.20、要有时间复杂度的概念！

3.1～3.2、字符串的一些常用操作函数，字典树。

3.3、[编辑距离](http://wonderflow.info/archives/400)。

3.4、无头单向链表中删除一个节点怎么删？
	把下一个节点的内容复制给我，删除下一个即可。

3.5、最短摘要生成算法：根据对于问题的思考，最后变成一个字符串匹配的问题。

3.6、判断两个单向链表是否相交
	做法就是把一个单向链表的尾指向另一个单向链表的头，看生成的新的单向链表有没有环。
        怎么判断一个单向链表有没有环？
	两个指针，一个每次走一步，一个每次走两步，如果有一天，两个指针相遇了，那么就是有环，
        怎么把环的起点找到？
	还是两个指针，一个从单向链表的头出发每次走一步，一个从上面所说的相遇点出发，也是每次走一步，当这两个指针再次相遇的时候，就是单向链表的环的入口点。

3.7、维护一个能方便取出最大值的队列容器。
	做法是队列用数组模拟，然后再维护一套指针，这套指针完成一个堆的指向。

3.8、二叉树中节点的最大距离。
	做法是动态规划，找左右子树中树高较大的那个返回。然后直到跟节点把左右子树的树高相加就可以了。

3.9、根据二叉树的前序遍历和中序遍历重新构建出二叉树，返回后序遍历。[传送门](http://wonderflow.info/archives/113)

3.10、分层遍历二叉树：BFS算法讲解。

3.11、程序改错。这个。。

4.1、概率题什么的实在不解释。

4.2、瓷砖覆盖：分横放竖放这样，DP一下就可以了。

4.3～over：不少智力题。。感觉编程之美越到后面越有点不想看了啊。。。。

大致就是这样了。正本书还是非常经典的，后来的面试过程中也确实碰到过非常类似的题目。总之，面试过程中可能面试到的大部分算法题，这里都基本涵盖了，可能有些大数据的题和一些开放性思维的题还有所欠缺。这本书还是很值的一读的。

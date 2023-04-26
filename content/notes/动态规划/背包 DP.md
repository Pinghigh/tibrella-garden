---
title: 背包DP
date: 2022-10-23 18:49:29
tags: [DP/动态规划, 背包问题]
categories: 动态规划
lastmod: 2023-01-16 09:01:48
keywords: 完全背包, 01背包, 动态规划
description: 目前记录了 01 背包和完全背包，其他的以后会补充的（
image: https://pic.imgdb.cn/item/6416687ca682492fccfb973a.jpg
---

第一次使用 $\LaTeX$

## 01背包

### 朴素
```C++
#include <iostream>
#include <algorithm>
#define MAXN 1010

using namespace std;

int n, m;
int v[MAXN], w[MAXN];
int f[MAXN][MAXN];

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    cin >> n >> m;

    for (int i = 1; i <= n; i ++) {
        cin >> v[i] >> w[i];
    }

    for (int i = 1; i <= n; i ++) {
        for (int j = 1; j <= m; j ++) {
            f[i][j] = f[i - 1][j];
            if (j >= v[i]) f[i][j] = max(f[i][j],f[i-1][j-v[i]] + w[i]);
        }
    }
    
    cout << f[n][m];  
    
    return 0;
}
```

### 一维优化

```C++
#include <iostream>
#define MAXN 1010

using namespace std;

int n,m;
int v[MAXN],w[MAXN];
int f[MAXN];

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        cin >> v[i] >> w[i];
    }

    for (int i = 1; i <= n; i ++) { // 按照物品从头到尾枚举
        for (int j = m; j >= v[i]; j --) { // 按照体积从最大体积到当前（第 i 个）物品枚举
            f[j] = max(
                f[j], // 此时未更新的 f[j] 是上一次（i-1）枚举的数据，即 f[j] 是不选第 i 个物品的 f[j]
                f[j - v[i]] + w[i] 
                );
        }
    }

    cout << f[m] << endl;

    return 0;
}
```

## 完全背包

### 朴素做法
即 3 维做法

* 状态表示 $f_{i,j}$  
    * 集合：所有只考虑前 $i$ 个物品，且总体积不大于 $j$ 的所有选法
    * 属性：$max$ （最大）
* 状态计算 -> 即集合的划分
    * 目前我们要求 $max f_{i,j}$，不容易直接求，就采用类似 01 背包问题的“曲线救国”方法
        1. 去掉 $k$ 个第 $i$ 个物品，变为 $max f_{i-1,j}$
        2. 去掉相应的 $k$ 个第 $i$ 个物品的空间，变为 $max f_{i-1,j-v_ik}$
        3. 求上面的 $max$
        4. 加回 $k$ 个第 $i$ 个物品的价值，变为 $max f_{i-1,j-v_ik} + w_ik$

```C++
int backpack() {
    #define MAXN 1010

    using namespace std;

    int n,m; //! 应当定义在函数外部，原因我不说，因为没人看
    int v[MAXN],w[MAXN];
    int f[MAXN][MAXN];

    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        cin >> v[i] >> w[i];
    }

    for (int i = 1; i <= n; i ++)
        for (int j = 0; j <= m; j ++) {
            for (int k = 0; k *v[i] <= j; k ++) 
                f[i][j] = max(f[i][j], f[i - 1][j - v[i] * k] + w[i] * k);
        }
    cout << f[n][m] << endl;
}
```

### 2 维优化

状态转移方程为 $f_{i,j} = max\{f_{i,j}, f_{i-1,j - v_ik} + w_ik\}$

因此可得
```C++
f[i,j] = max(f[i-1,j], f[i-1,j-v] + w, f[i-1,j-2*v] + 2*w, f[i-1,j-3*v] + 3*w, ...... ) 
f[i,j-v] = max(        f[i-1,j-v]    , f[i-1,j-2*v] +   w, f[i-1,j-3*v] + 2*w, ...... ) 
```

~~从我这拙劣的对齐~~ 可以看出他们的关系捏

也就是说 **第一行** 后面的一大坨 
```C++
f[i-1,j-v] + w, f[i-1,j-2*v] + 2*w, f[i-1,j-3*v] + 3*w, ......
```
等价于
```C++
f[i,j-v] + w
```

最终我们可以把该节第一行的状态转移方程优化为 $f_{i,j} = max(f_{i-1,j}, f_{i,j-v})$

代码如下
```C++
int full_backpack() {
    #define MAXN 1010

    using namespace std;

    int n,m; //! 应当定义在函数外部，原因我不说，因为没人看
    int v[MAXN],w[MAXN];
    int f[MAXN][MAXN];

    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        cin >> v[i] >> w[i];
    }

    for (int i = 1; i <= n; i ++)
        for (int j = 0; j <= m; j ++) {
        //  原做法
        //  for (int k = 0; k *v[i] <= j; k ++) 
        //      f[i][j] = max(f[i][j], f[i - 1][j - v[i] * k] + w[i] * k);
            f[i][j] = f[i - 1][j];
            if (j >= v[i]) f[i][j] = max(f[i][j], f[i][j-v[i]] + w[i]);
        }
    cout << f[n][m] << endl;
}
```

然后强删一维，变成一维DP数组捏

```C++
int backpack() {
    #define MAXN 1010

    using namespace std;

    int n,m; //! 应当定义在函数外部，原因我不说，因为没人看
    int v[MAXN],w[MAXN];
    int f[MAXN][MAXN];

    cin >> n >> m;
    for (int i = 1; i <= n; i ++) {
        cin >> v[i] >> w[i];
    }

    for (int i = 1; i <= n; i ++)
        for (int j = v[i]; j <= m; j ++) {
            // f[i][j] = f[i - 1][j]; 恒等式直接删除
            // if (j >= v[i]) 直接从 v[i] 开始循环就完事了
            f[j] = max(f[j], f[j-v[i]] + w[i]);
        }
    cout << f[m] << endl;
}
```


## 多重背包

### 朴素

$N$ 种物品，背包容量为 $V$，第 $i$ 件物品最多能选 $C_i$ 个，每件物品占空间 $v_i$，价值为 $w_i$  

第一种容易想到的解法是把完全背包的方法拿来用，因为多重背包只是加了 $k$ 的限制  
状态转移方程即 $f_{i,j} = \max\{f_{i,j}, f_{i-1,j - C_ik} + w_ik 0\} $

考虑多重背包和完全背包的区别可以发现，多重背包虽然每种物品不只一种，但是有确定的数量，而完全背包没有数量限制，则可以轻松地在忽略时间复杂度的情况下把多重背包**拆分**成 01 背包，即把 $M_i$ 件第 $i$ 件物品挨个算单个物品再按照 01 背包的方式处理。

```C++
for (int i = 1; i <= n; ++ i) {
    for (int j = 0; j <= m; ++ j) {
        for (int k = 0; k <= min(c[i],j/w[i])/* 数量和体积两个都限制数量 */; ++ k) {
            f[i][j] = max(f[i][j], f[i-1][j-k*v[i]] + k*w[i]);
        }
    }
}
```

### 二进制优化

这种暴力方式求解比较慢，容易发现，在求解过程中，每个**同种**物品是按照**不同种**物品进行处理的，换句话说，我选第一个 a 物品与选第二个 a 物品，实际上效果都是选 1 个 a 物品，但是这样的情况在上文解法中做了两次决策，显然有一种是多余的。  
也就是说，接下来的优化方向是**减少**每次决策所遇到的**同数量同种**物品，那会联想到什么呢？  
01 背包的决策是 **选** 与 **不选**，因此称为“01”背包，而 01 能够提醒我们利用**二进制**表示**不同的数**（不管想不想到反正这就是优化方法）  

易证，使用 $2^0, 2^1, 2^2, 2^3... 2^{k-1}$ 能够表示出 $0$ 到 $2^{k-1}$ 之间的所有的数，即二进制计数。也就是说，如果把物品的数量 $C_i$ 拆分成 $2^0, 2^1, 2^2...2^{k-1} + p,0 \leq p < 2^{k-1}$，由 $p$ 的范围可得，其中 $2^0+...2^{k-1}$ 能够表示出 $[0,p]$ 的所有整数，然后也能轻松得出 $2^0+...+2^{k-1}$ 选出若干与 $p$ 相加能表示出 $[p,p+2^k-1]$ 之间所有整数，两个合并即是 $[0,C_i]$  

因此可以将 $C_i$ 拆分为 $k+1$ 种新的物品，每种新物品的体积和价值分别为：
$$
2^0 \cdot w[i],2^1 \cdot w[i],2^2 \cdot w[i],...,2^{k-1} \cdot w[i],p \cdot w[i] \\\\
2^0 \cdot v[i],2^1 \cdot v[i],2^2 \cdot v[i],...,2^{k-1} \cdot v[i],p \cdot v[i]
$$
拆分完再进行 01 背包即可。

代码来自 [GrainRain 's Blog](https://grainrain.site/2022/10/13/2022.10.13%20-%20%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92/)
```C++
const int N = 25000;
// 一共有 1000 项，每项最多拆分成 log s 项  
const int M = 2010;

int n, m;
int v[N], w[N];
int f[N];

cin >> n >> m;

int cnt = 0;
for (int i = 1; i <= n; i ++) {
    int a, b, s;
    cin >> a >> b >> s;
    
    for (int k = 1; k <= s; k *= 2) { // 从2^0 开始枚举 2 的次幂
        s -= k;
        v[++ cnt] = a * k;
        w[cnt] = b * k;
    }
    if (s > 0) {
        v[++ cnt] = a * s;
        w[cnt] = b * s;
    }
}
n = cnt;
// 二进制拆分读入
    
for (int i = 1; i <= n; i ++)
    for(int j = m; j >= v[i]; j --)
        f[j] = max(f[j], f[j - v[i]] + w[i]);

cout << f[m] << endl;
// 01背包解题过程
```

## 分组背包

$N$ 组物品，第 $i$ 组有 $C_i$ 个物品，第 $i$ 组的第 $j$ 个物品的价值是 $w_{i,j}$，体积为 $v_{i,j}$，背包体积为 $M$，每组物品只能选择 $1$ 个

直接枚举组时挨个枚举物品就行了。  
按照组划分阶段，第 $i$ 组不选则 $f_{i,j} = f_{i-1,j}$，第 $i$ 组选第 $k$ 个则 $f_{i,j} = f_{i-1,j-v_{i,k}} + w_{i,k}$

这里直接给出数组删维优化过的代码
```C++
for (int i = 1; i <= n; ++ i) {
	for (int j = m; j >= m; -- j) {
		for (int k = 1; k <= c[i]; ++ k) {
			f[i][j] = max(f[j], f[j-v[i][k]] + w[i][k]);
		}
	}
}
```

## 依赖背包

~~我的 Windows 盘还剩 3 个 G，新买的东方冰之勇者记占 2G,LLVM 依赖于 MSVC 生成工具，MSVC 占用 2G 磁盘，LLVM 占用 2G 磁盘，问我应该怎么安装收益最大~~  
~~reboot to archlinux!~~

A 物品依赖于 B，选 B 物品之前需要先选 A 物品，问最大价值  

容易发现对于任意有依赖关系的物品 A B 来说，一共有以下三种可能的决策    
（A 依赖 B）
1. A B 都不选  
2. 选 B 不选 A  
3. A B 都选  

因为这三种情况只会出现一个，联想分组背包的特征，将这三种情况每个看成一个组里的物品，即转化为了分组背包。  

直接 dfs 就能解决依赖关系，但是这种情况怎么看都是要涉及到树形 DP 或者图论的） 
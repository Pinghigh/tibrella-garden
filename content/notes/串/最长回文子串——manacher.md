---
title: Manacher
date: 2023-01-31 20:27:12
tags: [算法, Manacher, 字符串]
categories: 算法
description: 求串的最长回文子串
image: https://pic.imgdb.cn/item/6416687ca682492fccfb973a.jpg
---

求一个字符串最长的回文子串。
字面意思，回文就是一个串正着读倒着读一样，子串就是一个连续的子序列。

## 预处理

不难发现对于一个回文字串，有两种情况：

1. `ABBA` 即长度为偶数，没有中心字符，半径为 2，长度为 4
2. `ABCBA` 即长度为奇数，有中心字符，半径为 3，长度为 5

懒得给他们处理两种情况，怎么办？只需要插入一些其他字符即可。  
比如说我们可以在头部插入一个 `?` 代表串开始，`^` 代表串结束，然后在新串的每一个字符之间插入一个 `#` 号，得到的新串就能达到简化情况的目的。    
按照上面的方法处理之后，原来的两个串会变为：  
（只观察回文子串）

1. `?#A#B#B#A#^` 长度变为奇数，有中心字符，半径为 5，长度为 9
2. `?#A#B#C#B#A#^` 长度为奇数不变，有中心字符，半径为 6，长度为 11

情况被简化成一种，即长度为奇数。与此同时我们发现，新串的回文子串的 **半径$-1$** 就是原串回文子串的 **长度**

与此同时，问题不变，还是求串的最长回文子串。

```C++
void init() {
    b += '$';
    b += '#';
    for (int i = 0; i < n; ++i) {
        b += a[i];
        b += '#';
    }
    b += '^';
    n=b.size();
}
```


## Manacher

### 基本思想

~~抽象来说~~就是对于字符串的每一个位置，维护以这个位置为中心的最长回文串长度同时算出这个回文串的右边界，再通过这个右边界来更新下一个位置的最长回文串长度与右边界。

具体来说，对于一个字符串 $S$，开一个数组 $P_i$ 记录以 $i$ 为中心的最长回文串半径（含 $S_i$），变量 $mid$ 为在 $i$ 之前边界最靠右的回文子串的中心，$mr$（也就是 $maxright$）记录这个回文子串的右边界，通过这些更新下一个位置的这些值。

为了方便说明，以 $i$ 为中心的最长回文子串称为 $T_i$

#### 继承对称点的数据

枚举到 $i$ 之前的状态：

![](https://s1.ax1x.com/2023/01/31/pS0hKFU.png)

枚举到 $i$ 后，对于 $mr$ 和 $i$ 来说有两种情况：$i$ 在左侧或 $mr$ 在左侧

* $i$ 在左侧
即 $i$ 被 $T_{mid}$ 包含。  
显然 $i$ 之前的 $P_k$ 都是已知的，同时 $i$ 在一个回文字串内，那么假设 $j$ 是 $i$ 关于 $mid$ 的对称点，则  
![](https://s1.ax1x.com/2023/01/31/pS0hKFU.png)  
那么 $P_j$ 已知，对于以 $j$ 为中心的最长回文子串 $T_j$ 来说有两种情况
  1. $T_j$ 被 $T_{mid}$ 完全包含，此时 $P_i$ 可以继承 $P_j$ （因为回文）（不用考虑边界问题，后续会处理）
  2. $T_j$ 未被 $T_{mid}$ 完全包含，此时 $T_i$ 一定被 $T_{mid}$ 完全包含。因为如果不被完全包含，则 $P_{mid}$ 的值仍可以增大，如下图  
   ![](https://s1.ax1x.com/2023/01/31/pS0hRk8.png)  
   图中蓝色块通过 $T_{mid}$ 串回文的性质已知相等，每个蓝色块均可以通过自身所在回文串推出其外侧橙色块也是相等的。  
   显然这种情况不会出现，因为这种情况下 $P_{mid}$ 是一个假值。


* $i$ 在右侧  
没有可以继承的值则可以直接重新推，不需要过多考虑。

```C++
if (i < mr) {
    p[i] = min(p[(mid << 1) - i], mr - i);
} else {
    p[i] = 1;
}
```

#### 更新 $P_i$

直接从当前记录的半径向外继续枚举，直到遇到两个不相同的字符为止（边界上有 `$` `^`，枚举到一定会停止，所以不用考虑边界问题）。

```C++
while (b[i - p[i]] == b[i + p[i]]) {
    ++ p[i];
}
```

#### 更新 $mr$ 和 $mid$

前文说过 $mr$ $mid$ 都是边界最靠右的回文子串的属性，那么只需要判断新找到的回文子串右边界是否大于 $mr$ 即可。

```C++
if (i + p[i] > mr) {
    mr = i + p[i];
    mid = i;
}
```

至此 Manacher 算法主要过程结束。

### 实现

```C++
#include <algorithm>
#include <cstring>
#include <iostream>
#include <string>

using std::cin;
using std::cout;
using std::endl;
using std::sort;
using std::string;

template <typename T>
T max(T a, T b) {
    return a>b?a:b;
}

template <typename T>
T min(T a, T b) {
    return a<b?a:b;
}

constexpr int N = 2e7 + 514;

int n;
string a, b;
int p[N];  // p[i] 表示以第 i 个字符为中心的最长回文子串半径
int res;

void init() {
    b += '$';
    b += '#';
    for (int i = 0; i < n; ++i) {
        b += a[i];
        b += '#';
    }
    b += '^';
    n=b.size();
}

void manacher() {
    int mr = 0, mid;
    for (int i = 1; i < n; ++i) {
        if (i < mr) {
            p[i] = min(p[(mid<<1)-i], mr - i);
        } else {
            p[i] = 1;
        }

        while (b[i-p[i]] == b[i+p[i]]) {
            ++ p[i];
        }

        if (i+p[i]>mr) {
            mr = i+p[i];
            mid = i;
        }
    }
}

int main() {
    std::ios::sync_with_stdio(0);
    cin.tie(0);
    cout.tie(0);

    a.reserve(N);
    b.reserve(N);

    cin >> a;
    n = a.size();

    init();
    manacher();

    for (int i = 0; i < n; ++i) { // 最后需要枚举一遍来找出最大值
        res = max(res, p[i]);
    }
    cout << res - 1 << endl;

    return 0;
}
```
### 时间复杂度[^1]

当 $P_j$ 左侧取到左边界及以外时，$P_i$ 才需要更新（其他情况都因为不满足条件而不用进循环），而右端点是单调递增且严格小于等于 $n$ 的，因此总时间复杂度为 $\operatorname{O}(n)$


[^1]: 参考[谷雨的笔记](https://www.acwing.com/file_system/file/content/whole/index/content/7964832/)
---
title: RMQ/ST表
date: 2023-01-27 14:22:33
tags: [RMQ, 数据结构]
categories: 数据结构
description: 用来解决区间最大最小值，O(1) 查询，比线段树快
image: https://pic.imgdb.cn/item/6416687ca682492fccfb973a.jpg
---

目标是解决 RMQ 问题，即对于一个大区间，短时间内查询一个小区间的最大最小值

下文以最大值为例说明

ST 表本质上是一个动态规划（倍增的）

先利用倍增（二进制拆分）预处理然后 $\operatorname{O}(1)$ 查询，比线段树快，但是只支持静态区间

## 原理

### 预处理

$f_{i,j}$ 表示从 $i$ 开始，长度为 $2^j$ 的区间中的最大值

预处理阶段，对于 $f_{i,j}$ 来说，直接
$$
    \max\{ f_{i,{j-1}},f_{i+2^{j-1},j-1} \}
$$
递推即可，非常容易想，时间复杂度为 $\operatorname{O}(n\log n)$

### 查询

对于一个区间 $[L,R]$ 来说，假设他的长度为 $len$ 

![](https://pic.imgdb.cn/item/63d4c4b0face21e9ef9f3cb9.jpg)

很容易能找到一个 $k$ 使得
$$
    \frac{len}{2} < 2^k \leqslant len
$$

![](https://pic.imgdb.cn/item/63d4c785face21e9efa61b8e.jpg)

显然 $[L,R]$ 被 $[L,2^k],[R-2^k-1,k]$ 严格包含，因此 ST 表中查询最大最小值的时间复杂度是常数级别的，而不是线段树的 $\log$ 级别  
查询结果就是 $\max
\{f_{L,2^k},f_{R-2^k-1k,k}\}$

至于 $k$，肉眼可见 $k=\lfloor\log _2 (len)\rfloor$  
此处可以直接使用 `cmath` 库中的 `log()` 函数，它是用来求以 10 为底的对数的
根据换底公式[^1]可知 $$k=\lfloor\log_2(len)\rfloor=\lfloor \frac{\lg(len)}{\lg2} \rfloor$$



[^1]: $ \log_xy=\frac{\log_cy}{\log_cx} $

## 实现

首先是初始化
```C++
for (int j = 0; j < M; ++j) {
    for (int i = 1; i + (1 << j) - 1 <= n; ++i) {
        if (!j) {
            f[i][j] = w[i];  // 从 i 开始长度为 2^0=1 的区间最大值为 i 本身
        } else {
            f[i][j] = max(f[i][j - 1], f[i + (1 << j - 1)][j - 1]);
        }
    }
}
```
然后是查询
```C++
int query(int l, int r) {
    int len = r - l + 1;
    int k = log(len) / log(2);

    return max(f[l][k], f[r - (1 << k) + 1][k]);
}
```

## 动态修改

这个标题比较诡异，因为我提到过

>只支持静态区间

但是实际上我们可以用一些小 trick 实现在 st 表的尾部增加值。（前提是题目要求比较少，比如目前这道题只需要在尾部增加数，同时只需要查询尾部某长度区间的最值）  
显然对于 st 表的每一个 $f_{i,j}$ 来说，他只会用到 $f_{k,j}$,$k>i$ 的值而不会用到前面的值，因此我们给他做一个倒转。  
换句话说，现在 $ f_{i,j} $ 表示从 $ i $ 开始，向前长度为 $ 2^j $ 的区间中的最大值。  
代码如下
```C++
void add(int a) {
    a += t;
    a %= d;
    ori[++tail] = a;
    for (int j = 0; tail - (1 << j) + 1 > 0; ++j) {
        if (!j) {
            f[tail][j] = a;
        } else {
            f[tail][j] = max(f[tail][j - 1], f[tail - (1 << j - 1)][j - 1]);
        }
    }
}

void query(int len) {
    int l = tail - len + 1;
    int k = log2(len);
    t =  max(f[tail][k], f[l + (1 << k) - 1][k]);
}
```
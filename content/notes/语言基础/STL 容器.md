---
title: STL
date: 2022-07-23 16:43:28
lastmod: 2022-10-23 18:53:11
tags: [STL]
categories: 语言基础
keywords: STL, vector
description: STL，尚未完工
image: https://pic.imgdb.cn/item/6416687ca682492fccfb973a.jpg
---

## STL

### vector 变长数组
需要引入头文件`vector`
#### 声明
```C++
vector<类型> 名称
```
多个vector组成的数组
```C++
vector<类型> 名称[some_num]
```
此处的类型也可以是自定义的结构体、类

示例
```C++
#include<vector>
vector<int> a;
vector<int> b[233];
```
#### [常用命令](##STL容器常用命令)
#### 迭代器
类似指针/数组下标之类的东西

```C++
#include <vector>
#include <iostream>
int main() {
    vector<int> a({"NVIDIA","FUCK","YOU","EOF"});

    std::cout << *a.begin() << std::endl; // a.begin() 指向 a 的第一个元素 此处相当于 a[0]
    std::cout << *(a.end()-1) << std::endl; // a.end() 指向 a 的最后一个元素的后一位，直接使用是越界访问，此处相当于 a[a.size()-1]

    vector<int>::iterator it = a.begin();
    vector<int>::iterator it_end = a.end(); // C++11 显然使用 auto it=a.begin(); 更好写
    std::cout << *it << std::endl; // 此时相当于 a[0] 与 *a.begin()
    // vector 的迭代器是一个随机访问迭代器，可以加减
    std::cout << *(it + 1) << std::endl; // 相当于 a[1]
    std::cout << *(it-it_end) << std::endl; // 求两个下标之间的距离
    return 0;
}
```

## STL容器常用命令
```C++
sth.size(); // 返回容器 sth 的实际长度（包含的元素个数）
sth.empty(); // 返回一个 bool 值，表示容器 sth 是否为空
sth.clear(); // 把容器 sth 清空
```
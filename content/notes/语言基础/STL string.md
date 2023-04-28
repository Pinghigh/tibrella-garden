---
title: STL string
date: 2022-07-18 20:24:48
tags: [字符串, string]
keywords: 字符串, size
categories: 语言基础
image: https://pic.imgdb.cn/item/6416687ca682492fccfb973a.jpg
---
可变长的字符序列，需要引入头文件 `<string>`

### 赋值
```C++
#include <string>
using namespace std;
int main(){
    string s1; // 默认空字符串
    string s2 = s1; // string 类型可以互相赋值，无需 memcpy() 或 strcpy() 。 s2 是 s1 的一个副本
    string s3 = "hiya"; // s3 是该字符串("hiya")字面值的一个副本（我也不知道是什么，总之 s3 的值是"hiya"
    string s4(5,'T'); // s4 的内容是“TTTTT” 

    return 0;
}
```

### 输入输出
```C++
#include <cstdio>
#include <iostream>
#include <string>
using namespace std;
int main(){
    string s1, s2;

//读入无空格字符串
    cin >> s1; // scanf() fgets() 不能读入 string 类型
//读入一行
    getline(cin,s2);
//输出
    cout << s1 << endl << s2 << endl; 
    /*
    printf() 不能直接输出string类型
    需要写成 printf("%s",s1.c_str())
    这里 s.c_str() 作用是把 string 类型转化为字符数组
    */
    // 当然也可以 puts(s1.c_str())

    return 0;
}
```

#### 后记
使用 `getline(cin,str)` 输入时需要注意回车符  
什么意思呢？举个例子
```C++
#include <iostream>
#include <string>

using namespace std;

int main(){
  string a,b;
  cin >> a;
  getline(cin, s);
  cout << a << endl << b << endl;
  return 0;
}
```
这时你输入
```TEXT
NVIDIA
FUCK U
```
程序会在你输入完第一行后直接输出一个`NVIDIA`与几个换行符    
原因是cin遇到回车停止，此时getline直接把`NVIDIA`后的部分(空)读入然后再次遇到回车符，停止输入。  
为了避免这种情况，我们需要忽略掉一个回车符，可以使用`getchar()`。  
```C++
#include <iostream>
#include <string>

using namespace std;

int main(){
  string a,b;
  cin >> a; // NVIDIA
  getchar(); // \n（跳过一个换行符）
  getline(cin, s); // FUCK U
  cout << a << endl << b << endl;
  return 0;
}
```
### 常用操作
+ `str.empty()`
  + 返回一个bool值。
  + 如果string为空返回1，非空返回0。
+ `str.size()` 返回string的长度，等价于 `s.length()`
  + `strlen()` 需要循环一遍数组，时间复杂度为O(n)。
  + `str.size()`  不需要循环，时间复杂度为O(1)。
+ 比较
  + string 型之间按字典序比较直接 `str1 < str2` 即可，`== != <= >= < >`都支持。
+ 相加
  + string 型之间相加直接 `str1 += str2`、`str3 = str1 + str2` 这样写即可。
  + 与其他类型（字符、字符串）相加时会先把这些类型的量转化为 string 对象，可以直接加这些类型的量，如 `str += 'a'` 、 `str1 = str2 + "abc"`
  + 相加运算时，必须保证等号两边都有 string 型，`str1 = str2 + "hello" + 'a' `、 `str1 += "hello"` 是可以的， 但是 `str1 = "hello" + 'a'` 会报错。
    + 注意运算顺序，如 `string s1 = s2 + "abc" + 'd'` 正确，因为会先将 s2 与 "abc" 相加得到另一个 string 对象，再继续运算。而 `string s1 = "abc" + 'd' + s2` 会报错，因为 "abc" 和 'd' 不能相加 （"abc"和"cde"也不行）
+ 操作单个字符
  + 可以当作字符数组处理，如 `str[0]` , `str[3]`
  + 独特的遍历方式
    ```C++
    #include <iostream>
    #include <string>
    using namespace std;
    int main(){
      cout << "So, NVIDIA," << endl;
      string s = "Fuck U";
      for (char c : s) { // char c可以写作 auto c
        cout << c << endl;
      }
      return 0;
    }
    ```
    输出：
    ```text
    So, NVIDIA,
    F
    u
    c
    k

    U
    ```
+ `str.pop_back()` 删除string型字符串的最后一个字符
+ `str.substr(int i,int len)`
  + 输出字符串str从i开始长度为len的字符串

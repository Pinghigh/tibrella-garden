---
title: "Hello World"
date: 1970-01-01T00:00:00+08:00
---
这是一个测试页

```cpp
#include <iostream>

using std::cout;

#define endl '\n'

int main() {
    std::ios::sync_with_stdio(0);
    cout.tie(0);

    cout << "Hello World!" << endl;

    return 0;
}
```

$Test Math$ ~~好吧，数学好像还不支持~~ 添加了支持  
$$ Test $$

~~这个代码块目测有点丑，还得改改[^1]~~ 改了一下，感觉好点了

[^1]: 注释测试
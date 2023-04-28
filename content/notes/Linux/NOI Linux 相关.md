---
title: "NOI Linux 指北"
date: 2023-03-28T22:12:58+08:00
tags: [Linux]
categories: Linux
description: 一些 Linux 的基本使用方法，基于 NOI Linux 2，也适用于其他使用 gnome 作 DE 的 Linux 环境 
image: https://pic.imgdb.cn/item/6416687ca682492fccfb973a.jpg
---

## 前言

现今 Windows 作为国外闭源系统随时可能被美方禁止使用却大行其道占据大陆巨额市场，可能操作系统真的引不起重视，就像光刻机一样，事情到了无可挽回的地步才开始用力建设吗？

写本文的起因是我一同学（实际上不止一个）并不会 NOI Linux 的基本使用，而这玩意是中国大陆 OI 中比较重要的环境之一，因此写一份基础教程

## 从终端开始

### 什么是终端？

对于一个计算机来说，输入输出设备显然不是必需品（脱离使用的范畴，没有输入输出设备它也能通电成为一个吉祥物），而最初人们为了与计算机互动，发明了终端，即人类用户与计算机交互的设备。

对于一个计算机操作系统，其运行过程中本身并没有显示任何东西，所谓桌面环境之类在显示器上显示的东西都是后话，最简单的输入输出方式即“我输入文字，计算机输出文字”，于是引入今天我们所要遇到的第一个概念，我称其为“文字终端”，即你输入文字，计算机输出文字的地方。

对于现在来说，以往奢侈的图形显示现在已经满地都是，但是为了还能用上简单快捷的文字终端的功能，出现了“终端模拟器”，实际上就是开一个小黑窗口，里面能够输入输出（类比 CMD）。

### 打开终端

![](https://pic.imgdb.cn/item/6422f9eca682492fcc1a0d39.jpg)

在桌面或者文件管理器中右键，点击“在终端中打开”

![](https://pic.imgdb.cn/item/6422fa27a682492fcc1a9748.jpg)

于是出现了一个如上的窗口，这就是一个终端模拟器，里面运行一个被称为“Shell”的进程，这里不多作介绍，理解它能够输入输出文字就行。

在这里你可以输入命令，做到创建文件，修改文件，删除文件等基本上你在图形界面下能做的所有事情。

### 先...管理文件？

在 Shell 中，`.` 表示当前目录，`..` 表示当前目录的父目录，`/` 表示根目录

Linux 下只有一个根目录，不会像 Windows 一样不同的盘有不同的根目录

也就是如下

```
Windows:
C:-
  |- Windows
  |- Program Files
  |_ Users

D:-
  |- Software
  |- Documents
  |_ Steam

Linux:
/ -
  |- usr
  |- bin
  |- home
  |- dev
  |- etc
  |.....
```

绝对路径：能唯一确定的路径，也就是说从根目录表示的确定的位置，比如 `/usr/local/bin`

相对路径：相对于某一目录的位置，比如当前目录是 `/usr/local/` 则上面的路径可以表示为 `bin` 或 `./bin`

先提供相应的命令：

```shell
# 这是一条注释，SHELL 中使用井号做单行注释

# 创建文件夹
mkdir folder_name
# 创建一个 modmodwhr 文件夹
mkdir modmodwhr


# 创建空文件
touch filename
# 创建一个名为 whrakioi 文件夹
touch whrakioi


# 进入一个目录
cd folder_name
# 当前目录下有一个 hhyakioi 文件夹，进入该文件夹
cd hhyakioi
# 或者 . 代表当前目录，./hhyakioi 相当于进入当前目录下的 hhyakioi 文件夹
cd ./hhyakioi
# 进入上一级文件夹
cd ..
# 进入根目录
cd /
# 进入某绝对路径
cd /usr/share
# 最后一个杠加不加无所谓
cd /usr/share/

# 复制一个文件到另一个位置
cp /path/to/file /path/to/direction
# 把 1.cpp 复制到 gjxakioi 文件夹
cp 1.cpp gjxakioi/1.cpp
# 复制文件夹需要添加 -r 参数
cp folder_name gjxakioi/folder_name -r

# 移动一个文件到另一个位置，同时可以当重命名用
mv path1 path2
# 重命名 1.cpp 为 2.cpp
mv 1.cpp 2.cpp
# 移动 2.cpp 到 gjxakioi 文件夹
mv 1.cpp gjxakioi/1.cpp
```

NOI Linux 2 的比赛环境下默认给你挂载 noip 文件夹并在 Linux 桌面上创建快捷方式，因此如果你在桌面上进入终端，则可以通过下面的命令进入 noip 文件夹

```shell
cd noip
```

或者

```shell
cd ./noip
```

## 开始写代码吧！

### 想编辑下文件？

如果你在 Windows 下写好了代码想要粘贴到 Linux 的文件里，则可以

```shell
# 新建 a.cpp 文件
touch a.cpp
# 使用 nano 编辑器编辑 a.cpp 文件
nano a.cpp
```

nano 是 GNU 的一款简易终端文本编辑器，相对于 Vi/Vim 来说更适合新手使用

复制了 Windows 的代码之后，确保 VMware 的共享剪贴板已经打开，然后按下 `Ctrl` + `Shift` + `V` 粘贴，然后按下 `Ctrl` + `S` 保存，`Ctrl` + `X` 退出。



当然，有图形化编辑器供你使用

```shell
# 使用闭源 Visual Studio Code 编辑 a.cpp
code a.cpp
```



在此之后你可以通过 `cat` 命令[^1]验证文件内容

```shell
cat a.cpp
```

### 编译 C++ 源码

#### GNU 编译套件

GNU Compile Collection 是 GNU 开发的一套编译器，在这里我们只使用 GNU C 编译器部分。

另：GNU 计划的目标是开发出一款完全自由的操作系统和软件生态，自由软件运动致力于推进全部软件的隐私保护以及自由化，自由软件属于全人类，自由软件开发者们留下的作品与学习资料是不朽的！如果你对自由软件运动有疑问或有兴趣，可前往 www.gnu.org 了解更多相关内容。

#### 开始编译

```shell
g++ grainrainisgod.cpp
```

这个命令会在当前目录下编译 grainrainisgod.cpp 文件，然后默认输出一个 a.out 可执行文件[^2]。

可是我懒得重命名文件，所以如何执行输出可执行文件的位置和名称呢？只需要加 `-o /path/to/direction` 即可

```shell
g++ grainrainisgod.cpp -o grainrainisgod
```

当然，你也可以加入其他比赛要求的编译选项，比如说吸个氧（加 `-O2` 参数）

```shell
g++ grainrainisgod.cpp -o grainrainisgod -O2
```

#### 开启编译警告

编译警告在编译的编译阶段和警告阶段发出，能帮你发现一些低级问题和简单 UB，比如著名 UB（但是萌妹 OPTIM 还是犯了这个问题）

```cpp
a = a++ + ++a;
```

编译器会发出警告

```log
warning: multiple unsequenced modifications to 'a' [-Wunsequenced]
    cout << a++ + ++a << endl;
```

这样的警告对于比赛选手来说肯定是越多越好越全越好，所以需要开启全部警告，即添加 `-Wall` 参数

```shell
g++ grainrainisgod.cpp -o grainrainisgod -O2 -Wall
```



#### 检查内存错误以及未定义行为

Sanitizer 是谷歌开发的一个检测程序问题的开源套件，一般集成在 gcc 以及 clang 编译器中。相应的，MinGW-W64 并没有为 Sanitizer 做 Windows 兼容，因而无法在比赛环境 Windows 使用（LLVM/Clang 编译器包含的 Sanitizer 支持 Windows，MSVC 的也支持但是只有 AddressSanitizer）（但是显然比赛环境没有这两个编译器），所以显得 Linux 环境非常重要。

Address Sanitizer 用于检测程序的内存错误，比如爆栈等问题。使用方法是添加 `-fsanitize=address` 选项

Undefined Behaviour Sanitizer 用于检测程序的未定义行为，比如访问空指针，数组越界访问等问题，使用方法是添加 `-fsanitize=undefined` 选项。

值得注意的是，这两个 Sanitizer 会显著拖慢程序运行速度，所以添加这两个编译选项编译出来的程序在运行时间和内存占用上参考意义不大。同时如果动态分配内存过多可能 Address Sanitizer 会抽风，UBSanitizer 倒是问题不大。所以酌情使用。

#### 总结：

```shell
g++ grainrainisgod.cpp -o grainrainisgod -O2 -Wall -fsanitize=address -fsanitize=undefined
```

如上是我赛时经常使用的编译命令。

### 执行可执行文件

```shell
./grainrainisgod
```

`.` （当前目录）+ `/` （目录分隔符）+ `grainrainisgod` 可执行文件名称

然后就可以快乐地输入了。

注意：在这个终端中复制使用 `Ctrl`+`Shift`+ `C`，粘贴使用 `Ctrl`+`Shift`+` V`

`Ctrl` + `C` 用于终止当前正在运行的程序，也可以在输入命令打错字时 `Ctrl` + `C` 重开一行。



[^1]: `cat filename` 即输出 `filename` 文件的内容

[^2]: Linux 下文件类型的判断与  Windows/DOS 略有不同。众所周知，Windows 下操作系统通过扩展名判断文件类型，也就是说无论你把什么文件改成 `.docx` 扩展名，Windows 都会用 Word 打开它。但是 Linux 一般通过文件头判断文件类型，也就是说无论你把可执行文件命名成 `a.out` 还是 `a.docx` 还是 `a.jpg` 甚至没有扩展名，输入 `.\a.jpg` 或相应命令之后终端都会把它当作可执行文件执行。

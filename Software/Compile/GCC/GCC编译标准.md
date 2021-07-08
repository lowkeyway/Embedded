# GCC -std编译标准

要知道，任何一门编程语言都有相关的组织和团体在不停的维护和更新。原因很简单，时代在发展，编程语言如果停滞不前，最终就会被淘汰。

以 C 语言为例，发展至今该编程语言已经迭代了诸多个版本，例如 C89（偶尔又称为 C90）、C94（C89 的修订版）、C99、C11、C17，以及当下正在开发的 C2X 新标准。甚至于在这些标准的基础上，GCC 编译器本身还对 C 语言的语法进行了扩展，先后产生了 GNU90、GNU99、GNU11 以及 GNU17 这 4 个版本。

```
有趣的是，GCC 编译器对 C 语言的很多扩展，往往会被 C 语言标准委员会所采纳，并添加到新的 C 语言标准中。
例如，GNU90 中对 C 语言的一些扩展，就融入到了新的 C99 标准中；GNU90、GNU99 中对 C 语言的一些扩展，被融入到了新的 C11 标准中。
```

C++ 语言的发展也历经了很多个版本，包括 C++98、C++03（C++98 的修订版）、C++11（有时又称为 C++0x）、C++14、C++17，以及即将要发布的 C++20 新标准。和 C 语言类似，GCC 编译器本身也对不同的 C++ 标准做了相应的扩展，比如 GNU++98、GNU++11、GNU++14、GNU++17。

读者可能会问，这么多标准，GCC 编译器使用的到底是哪一套呢？不同版本的 GCC 编译器，默认使用的标准版本也不尽相同。以当前最新的  GCC 10.1.0 版本为例，默认情况下 GCC 编译器会以 GNU11 标准（C11 标准的扩展版）编译 C 语言程序，以 GNU++14 标准（C++14 标准的扩展版）编译 C++ 程序。

那么，我们可以手动控制 GCC 编译器使用哪个编译标准吗？答案是肯定的，对于编译 C、C++ 程序来说，借助 -std 选项即可手动控制 GCC 编译程序时所使用的编译标准。也就是说，当使用 gcc 指令编译 C 语言程序时，我们可以借助 -std 选项指定要使用的编译标准；同样，当使用 g++ 指令编译 C++ 程序时，也可以借助 -std 选项指定要使用的编译标准。

-std 选项的使用方式很简单，其基本格式如下：
```
gcc/g++ -std=编译标准
```

注意，不同版本的 GCC 编译器，所支持使用的 C/C++ 编译标准也是不同的。表 1 罗列了常用的 GCC 版本对 C 语言编译标准的支持程度。
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Compile/pic/%E4%B8%8D%E5%90%8CGCC%E7%89%88%E6%9C%AC%E6%94%AF%E6%8C%81%E7%9A%84C%E8%AF%AD%E8%A8%80%E7%BC%96%E8%AF%91%E6%A0%87%E5%87%86.png">

```
注意，表头表示的是各个编译标准的名称，而表格内部的则为 -std 可用的值，例如 -std=c89、-std=c11、-std=gnu90 等（表 2 也是如此
```

表 2 罗列了常用的 GCC 版本对 C++ 程序编译标准的支持程度。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Compile/pic/%E4%B8%8D%E5%90%8CGCC%E7%89%88%E6%9C%AC%E6%94%AF%E6%8C%81%E7%9A%84C%2B%2B%E8%AF%AD%E8%A8%80%E7%BC%96%E8%AF%91%E6%A0%87%E5%87%86.png">


```
表 1、2 中，有些版本对应的同一编译标准有 2 种表示方式，例如对于 8.4~10.1 版本的 GCC 编译器来说，-std=c89 和 -std=c90 是一样的，
使用的都是 C89/C90 标准。另外，GCC 编译器还有其他版本，读者可查阅 GCC文档获得相关信息。
```

举个例子，如下是一个 C 语言源程序：
```
[root@bogon demo]# ls
main.c
[root@bogon demo]# cat main.c
#include <stdio.h>
int main(){
    for(int i=0;i<10;i++){
        printf("i=%d ",i);
    }
}
```
如果我们想以 c99 的标准编译它，在确认当前所有 GCC 编译器版本支持 C99 标准的前提下，通过执行如下指令，即可完成编译：
```
[root@bogon demo]# gcc -std=c99 main.c -o main.exe
[root@bogon demo]# ./main.exe
i=0 i=1 i=2 i=3 i=4 i=5 i=6 i=7 i=8 i=9
```
但是，对于在 for 循环中声明变量 i 的做法，是违反 C89 标准的。也就是说，如果我们以 C89 的编译标准编译 main.c，GCC 编译器会报错

```
[root@bogon demo]# gcc -std=c89 main.c -o main.exe
main.c: In function ‘main’:
main.c:3: error: ‘for’ loop initial declarations are only allowed in C99 mode
main.c:3: note: use option -std=c99 or -std=gnu99 to compile your code
```

这也就意味着，在编写程序前必须明确要使用的编译标准，并清楚得知道该标准下什么可用，什么不可用。

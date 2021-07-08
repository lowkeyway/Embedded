# __VA_ARGS__

__VA_ARGS__ 是在 C99 中增加的新特性。虽然 C89 引入了一种标准机制，允许定义具有可变数量参数的函数，但是 C89 中不允许这种定义可变数量参数的方式出现在宏定义中。C99 中加入了 __VA_ARGS__ 关键字，用于支持在宏定义中定义可变数量参数，用于接收 ... 传递的多个参数。

__VA_ARGS__ 只能出现在使用了省略号的像函数一样的宏定义里。例如 #define myprintf(...) fprintf(stderr, __VA_ARGS__)。

## 解析不定参

通过宏定义，将多个参数传递给函数，那么函数是如何解析不定参的呢？

这就需要使用标准库头文件 <stdarg.h> 中的三个宏，分别是 “va_start()”、“va_arg()”、“va_end()”，以及一个可变参类型 “va_list”，示例使用方式借用 RT-Thread 中的 rt_sprintf 的实现，代码如下所示：
```
rt_int32_t rt_sprintf(char *buf, const char *format, ...)
{
    rt_int32_t n;
    va_list arg_ptr;

    va_start(arg_ptr, format);
    n = rt_vsprintf(buf, format, arg_ptr);
    va_end(arg_ptr);

    return n;
}
```

+ 首先使用 “va_list” 类型定义一个变量 “arg_ptr”
+ 然后调用 “va_start(arg_ptr, format);” 函数，第一个入参是 “va_list” 类型，第二个参数是 “rt_sprintf” 函数参数列表中的最后一个定参 “format”
+ 然后，通过调用 “rt_vsprintf” 函数，根据 “format” 来解析不定参，并将结果存放到 “buf” 中
+ 最后，使用 “va_end(arg_ptr);” 来释放不定参列表占用的资源


## 带 ‘#’ 的标识符 #__VA_ARGS__

预处理标记 ‘#’ 用于将宏定义参数转化为字符串，因此 #__VA_ARGS__ 会被展开为参数列表对应的字符串。

示例：
```
#define showlist(...) put(#__VA_ARGS__)

测试如下：
showlist(The first, second, and third items.);
showlist(arg1, arg2, arg3);

输出结果分别为：
The first, second, and third items.
arg1, arg2, arg3
```


## 带 ‘##’ 的标识符 ##__VA_ARGS__

##__VA_ARGS__ 是 GNU 特性，不是 C99 标准的一部分，C 标准不建议这样使用，但目前已经被大部分编译器支持。

标识符 ##__VA_ARGS__ 的意义来自 ‘##’，主要为了解决一下应用场景：

```
#define myprintf_a(fmt, ...) printf(fmt, __VA_ARGS__)
#define myprintf_b(fmt, ...) printf(fmt, ##__VA_ARGS__)

应用：
myprintf_a("hello");
myprintf_b("hello");

myprintf_a("hello: %s", "world");
myprintf_b("hello: %s", "world");
```
这个时候，编译器会报错，如下所示：

```
applications\main.c: In function 'main':
applications\main.c:26:57: error: expected expression before ')' token
 #define myprintf_a(fmt, ...) printf(fmt, __VA_ARGS__)
                                                         ^
applications\main.c:36:5: note: in expansion of macro 'myprintf_a'
     myprintf_a("hello");
```


为什么呢？

我们展开 myprintf_a("hello"); 之后为 printf("hello",)。因为没有不定参，所以，__VA_ARGS__ 展开为空白字符，这个时候，printf 函数中就多了一个 ‘,’（逗号），导致编译报错。而 ##__VA_ARGS__ 在展开的时候，因为 ‘##’ 找不到连接对象，会将 ‘##’ 之前的空白字符和 ‘,’（逗号）删除，这个时候 printf 函数就没有了多余的 ‘,’（逗号）。

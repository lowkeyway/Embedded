在底层开发中，一般是使用C或者C++，但是有时候为了开发效率或者在写测试脚本的时候，会经常使用到python，所以这就涉及到一个问题，用C/C++写的底层库，怎么样直接被python来调用？

python作为一门胶水语言，当然有办法来处理这个问题，python提供的方案就是ctypes库。

# ctypes

ctypes是python的外部函数库，它提供了C语言的兼容类型，而且可以直接调用用C语言封装的动态库。
如果各位有较好的英语水平，可以参考[ctypes](https://docs.python.org/3/library/ctypes.html?highlight=ctypes#module-ctypes)官方文档，但是我会给出更详细的示例，以便各位更好地理解。

## 库的封装

C代码如果要能够被python调用，首先我们先得把被调用C接口封装成库，一般是封装成动态库。编译动态库的指令是这样的：
```
gcc --shared -fPIC -o target.c libtarget.so  
```

在这里，

+ --shared -fPIC 是编译动态库的选项。

+ -o 是指定生成动态库的名称

在linux下，一般的命名规则是：静态库为lib.a，动态库为lib.so

target.c为目标文件，在编译时常有更复杂的调用关系和依赖，这里就不详说，有兴趣的朋友可以去了解了解gcc编译规则。

## 在python中导入库

既然库已经封装好了，那肯定是就想把它用起来。我们可以在python中导入这个库，以导入libtarget.so为例：
```
import ctypes
target = cdll.LoadLibrary("./libtarget.so")
```

顺带提一下，如果在windows环境下，动态库文件是.dll文件，例如导入libtarget.dll:
```
import ctypes
target = windll.LoadLibrary("./libtarget.dll")
```
在这里，可以将target看成是动态库的示例，直接可以以变量target来访问动态库中的内容。

LoadLibrary("./libtarget.so")表示导入同目录下的libtarget.so文件。

细心的朋友已经发现了，在导入时，linux环境下使用的是cdll，而windows环境下使用的是windll。

这里涉及到C语言的调用约定，gcc使用的调用约定是cdecl，windows动态库一般使用stdcall调用约定，既然是调用约定，就肯定是关于调用时的规则，他们之间的主要区别就是cdecl调用时由调用者清除被调用函数栈，而stdcall规定由被调用者清除被调用函数栈。


# 例子

# 初级版本

## 1. 我们可以先写一个test.c，这个文件只做一件事情，printf("Hello World\n").
```
#include <stdio.h>                                                                                                                                                                                          

void hello_world(void)
{
  printf("Hello World\n");
}

int main()
{
 hello_world();
 return 0;
}
```

## 2. 搭配一个简单的Makefile，实现编译so库的目的：
```
GCC ?= gcc                                                                                                                                                                                                  
CFLAGS = -Wall -Wconversion -O3 -fPIC
SOFLAGS = -fPIC --shared 
SRC = test.c

all:
  $(GCC) $(CFLAGS) $(SRC)
lib:
  $(GCC) $(SOFLAGS) $(SRC) -o lowkeyway.so
clean:
  -rm -rf *.so *.o *.txt *.out
```

## 3. 执行make lib命令，编译出lowkeyway.so
```
make lib
```
这样我们就可以在当前目录下看到lowkeyway.so了。
```
lowkeyway@lowkeyway:/media/sf_ubuntu/code/my_code/test$ ls
lowkeyway.so  Makefile  test.c  test.py
```

## 4. 编写test.py，加载lowkeyway.so并且调用lowkeyway.so中的hello_world函数
```
from ctypes import *                                                                                                                                                                                        
test = cdll.LoadLibrary("./lowkeyway.so")
test.hello_world()
```

来执行一下看看？
```
lowkeyway@lowkeyway:/media/sf_ubuntu/code/my_code/test$ python3 test.py 
Hello World
```

正常输出！




参考：https://www.cnblogs.com/downey-blog/p/10483342.html

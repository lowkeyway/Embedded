# 概述

Android.mk是Android提供的一种makefile文件，用来指定诸如编译生成so库名、引用的头文件目录、需要编译的.c/.cpp文件和.a静态库文件等。要掌握jni，就必须熟练掌握Android.mk的语法规范。 由于该文件会被NDK的编译工具解析多次，因此应该尽量减少源码中声明变量，因为这些变量可能会被多次定义从而影响到后面的解析。

这个文件的语法允许把源代码组织成模块，每个模块属于下列类型之一：  
+ APK程序：一般的Android程序，编译打包生成apk文件。  
+ JAVA库：java类库，编译打包生成jar包文件。  
+ C\C++应用程序：可执行的C/C++应用程序。  
+ **C\C++静态库：编译生产C/C++静态库，并打包成.a文件。**
+ **C\C++共享库：编译生成共享库，并打包成.so文件，有且只有共享库才能被安装/复制到APK包中。**

## 举例

这里参考了网上一个通用的例子，编译简单的“Hello World”，来说明一下Android.mk编写。例如下面的文件：
1. sources/test/hello.c 
2. sources/test/Android.mk  

其中“hello.c”是一个JNI共享库，实现返回“hello world”字符串的原生方法。因此，Android.mk文件内容如下：

```
LOCAL_PATH := $(call my-dir)  
include $(CLEAR_VARS)  
LOCAL_MODULE := hello  
LOCAL_SRC_FILES := hello.c  
include $(BUILD_SHARED_LIBRARY) 
```

解释一下这几行代码：
1. LOCAL_PATH := $(call my-dir) ：    
一个Android.mk文件首先必须定义好LOCAL_PATH变量，用于在开发树中查找源文件。在这个例子中，宏函数my-dir由编译系统提供，用于返回当前路径（即包含Android.mk文件的目录）。
  
2. include $(CLEAR_VARS)：  
CLEAR_VARS由编译系统提供，指定让GNU MAKEFILE清除了LOCAL_PATH变量外的许多LOCAL_\***变量（例如：LOCAL_MODULE、LOCAL_SRC_FILES等）。这是非常有必要的，因为所有的编译文件都在同一个GUN MKAE执行环境中，所有的变量都是全局变量，不清除容易引起解析错误。
  
3. LOCAL_MODULE := hello：  
LOCAL_MODULE变量必须定义，用来标识在Android.mk文件描述的每一个模块。而且名称必须是唯一的，并且不能包含空格。编译系统会自动产生合适的前缀和后缀，比如一个被命名为hello的共享库模块，将会生成libhello.so文件。如果把库命名为libhello，编译系统将不会添加任何lib前缀，也会生成libhello.so文件。

4. LOCAL_SRC_FILES := hello.c：  
LOCAL_SRC_FILES变量必须包含将要编译打包进模块中的源代码文件。

5. include $(BUILD_SHARED_LIBRARY)：  
BUILD_SHARED_LIBRARY是编译系统提供的变量，指向一个GNU Makefile脚本（应该就是build/core目录下的shared_library.mk）,负责收集自从上次调用include $(CLEAR_VARS)以来，定义在LOCAL_\***变量中的所有信息，并且决定编译什么，如何正确地去做，并根据其规则生成动态库。

6. 解释一下Android.mk里变量定义字符”:=”。“:=”类似于c中的宏，即在定义处明确展开，完全进行文本替换。’+=’是追加的意思；‘$’表示引用某变量的值

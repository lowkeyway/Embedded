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
+ 1. LOCAL_PATH := $(call my-dir) ： 
  一个Android.mk文件首先必须定义好LOCAL_PATH变量，用于在开发树中查找源文件。在这个例子中，宏函数my-dir由编译系统提供，用于返回当前路径（即包含Android.mk文件的目录）。
  
+ 2. include $(CLEAR_VARS)：
  CLEAR_VARS由编译系统提供，指定让GNU MAKEFILE清除了LOCAL_PATH变量外的许多LOCAL_***变量（例如：LOCAL_MODULE、LOCAL_SRC_FILES等）。这是非常有必要的，因为所有的编译文件都在同一个GUN MKAE执行环境中，所有的变量都是全局变量，不清除容易引起解析错误。
  

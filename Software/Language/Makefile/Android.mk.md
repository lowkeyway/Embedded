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

## 实际应用

```
TOP_PATH := $(call my-dir)
LIB_PATH := $(TOP_PATH)/ifplibs

LOCAL_PATH := $(TOP_PATH)

# ifp
include $(CLEAR_VARS)
LOCAL_MODULE := libifp
LOCAL_SRC_FILES := $(LIB_PATH)/libifp.a
include $(PREBUILT_STATIC_LIBRARY)

# ndkifp
include $(CLEAR_VARS)
LOCAL_MODULE := libndkifp
LOCAL_STATIC_LIBRARIES := libifp
LOCAL_SRC_FILES := $(LIB_PATH)/libndkifp.a
include $(PREBUILT_STATIC_LIBRARY)

# ovtafehal.so
include $(CLEAR_VARS)
LOCAL_PATH := $(TOP_PATH)
LOCAL_CFLAGS := $(INCLUDE_PATH)
#LOCAL_CFLAGS += -Wno-unused-variable
#LOCAL_CFLAGS += -Wno-array-bounds
#LOCAL_CFLAGS += -fvisibility=hidden
LOCAL_CFLAGS += -O3 -c -std=c99 -w -pedantic -fstack-protector --param ssp-buffer-size=4 -fno-short-enums
LOCAL_CFLAGS += -fPIE -fPIC 
LOCAL_LDLIBS := -llog -lz
LOCAL_SRC_FILES := ovt_thp.c ovt_zeroflash.c fws.c ovt_reflash.c
LOCAL_STATIC_LIBRARIES := ndkifp ifp
LOCAL_MODULE := ovtafehal 
include $(BUILD_SHARED_LIBRARY)

# daemon
include $(CLEAR_VARS)
LOCAL_PATH := $(TOP_PATH)
LOCAL_CFLAGS += -Wno-unused-variable
LOCAL_CFLAGS += -Wno-array-bounds
LOCAL_CFLAGS += -fvisibility=hidden
LOCAL_CFLAGS += -O3 -w
LOCAL_CFLAGS += -fPIE -fPIC -fstack-protector
LOCAL_LDLIBS := -llog -lz
# LOCAL_SRC_FILES := ovt_thp.c ovt_zeroflash.c fws.c ovt_reflash.c
LOCAL_SRC_FILES += daemon.c
# LOCAL_STATIC_LIBRARIES := ndkifp ifp
LOCAL_MODULE := daemon
include $(BUILD_EXECUTABLE)
```

这里需要强调一个例子，就是.a 文件 和 .so 文件，（静态库&共享库）的区别:

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Language/Makefile/PIC/Android.mk_Static%26Shared_Diff.png">

include $(BUILD_STATIC_LIBRARY) & include $(BUILD_SHARED_LIBRARY)

# 一、Android.mk文件的用途

一个android子项目中会存在一个或多个Android.mk文件
## 1、单一的Android.mk文件

直接参考NDK的sample目录下的hello-jni项目，在这个项目中只有一个Android.mk文件

## 2、多个Android.mk文件

如果需要编译的模块比较多，我们可能会将对应的模块放置在相应的目录中，这样，我们可以在每个目录中定义对应的Android.mk文件（类似于上面的写法），最后，在根目录放置一个Android.mk文件，内容如下：
```
include $(call all-subdir-makefiles)
```

只需要这一行就可以了，它的作用就是包含所有子目录中的Android.mk文件。

## 3、多个模块共用一个Android.mk

这个文件允许你将源文件组织成模块，这个模块中含有：
+ 静态库(.a文件)
+ 动态库(共享库)(.so文件) ，只有共享库才能被安装/复制到您的应用软件（APK）包中

include $(BUILD_STATIC_LIBRARY)，编译出的是静态库

include $(BUILD_SHARED_LIBRARY)，编译出的是动态库

# 二、GNU Make系统变量

这些 GNU Make变量在你的 Android.mk 文件解析之前，就由编译系统定义好了。注意在某些情况下，NDK可能分析 Android.mk 几次，每一次某些变量的定义会有不同。  

+ **CLEAR_VARS:**  
指向一个编译脚本，几乎所有未定义的 LOCAL_XXX 变量都在”Module-description”节中列出。必须在开始一个新模块之前包含这个脚本：include$(CLEAR_VARS)，用于重置除LOCAL_PATH变量外的，所有LOCAL_XXX系列变量。  
+ **BUILD_SHARED_LIBRARY:**  
指向编译脚本，根据所有的在 LOCAL_XXX 变量把列出的源代码文件编译成一个共享库。  
注意，需要在这行代码之前定义 LOCAL_MODULE 和 LOCAL_SRC_FILES。
+ **BUILD_STATIC_LIBRARY:** 
用于编译一个静态库。静态库不会复制到的APK包中，但是能够用于编译共享库。  
示例：include $(BUILD_STATIC_LIBRARY)  
注意，这将会生成一个名为 lib$(LOCAL_MODULE).a 的文件  
+ **TARGET_ARCH:**  
目标 CPU平台的名字
+ **TARGET_PLATFORM:**  
Android.mk 解析的时候，目标 Android 平台的名字.详情可考/development/ndk/docs/stable- apis.txt.  
android-3 -> Official Android 1.5 system images  
android-4 -> Official Android 1.6 system images  
android-5 -> Official Android 2.0 system images  
+ **TARGET_ARCH_ABI:**  
暂时只支持两个 value，armeabi 和 armeabi-v7a。
+ **TARGET_ABI:**  目
标平台和 ABI 的组合.  

# 三 、自定义变量

以下是在 Android.mk中依赖或定义的变量列表，可以定义其他变量为自己使用，但是NDK编译系统保留下列变量名：  

+ 以 LOCAL_开头的名字（例如 LOCAL_MODULE）  
+ 以 PRIVATE_, NDK_ 或 APP_开头的名字（内部使用)  
+ 小写名字（内部使用，例如‘my-dir’）  

如果为了方便在 Android.mk 中定义自己的变量，建议使用 MY_前缀，一个小例子：

```
MY_SOURCES := foo.c 
ifneq ($(MY_CONFIG_BAR),) 
MY_SOURCES += bar.c 
endif 
LOCAL_SRC_FILES += $(MY_SOURCES) 
```

**注意**：‘:=’是赋值的意思；’+=’是追加的意思；‘$’表示引用某变量的值。

Reference Link:
https://blog.csdn.net/xx326664162/article/details/52875825


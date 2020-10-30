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


# 四 、模块描述变量

下面的变量用于向系统描述我们自己的模块，应该定义在include $(CLEAR_VARS)和include $(BUILD_\***)之间。  
+ **LOCAL_PATH：**  
这个变量用于给出当前文件的路径，必须在Android.mk的开头定义，可以这样使用：LOCAL_PATH := $(call my-dir)，这样这个变量不会被\$(CLEAR_VARS)清除，因为每个Android.mk只需要定义一次（即使一个文件中定义了多个模块的情况下）。

+ **LOCAL_MODULE：**  
当前模块的名称，这个名称应当是唯一的，并且不能包含空格。模块间的依赖关系就是通过这个名称来引用的。

+ **LOCAL_MODULE_CLASS：**  
标识所编译模块最后放置的位置。

  + ETC表示放置在/system/etc.目录下，
  + APPS表示放置在/system/app目录下，
  + SHARED_LIBRARIES表示放置在/system/lib目录下
  + 如果具体指定，则编译的模块不会放到编译系统中，最后会在out对应product的obj目录下的对应目录中。

+ **LOCAL_SRC_FILES：**  
这是要编译的源代码文件列表。只要列出要传递给编译器的文件即可，编译系统会自动计算依赖关系。源代码文件路径都是相相对于LOCAL_PATH的，因此可以使用相对路径进行描述。

  + 只要列出要传递给编译器的文件，编译系统自动计算依赖。注意源代码文件名称都是相对于 LOCAL_PATH的，你可以使用路径部分，例如：  
    LOCAL_SRC_FILES := foo.c toto/bar.c\  
    Hello.c  
    文件之间可以用空格或Tab键进行分割,换行请用”\”  
  + 如果是追加源代码文件的话，请用LOCAL_SRC_FILES +=
  + 可以LOCAL_SRC_FILES := $(call all-subdir-java-files)这种形式来包含local_path目录下的所有java文件。

+ **LOCAL_JAVA_LIBRARIES：**  
当前模块依赖的Java共享库，也叫Java动态库。例如framework.jar包。

+ **LOCAL_STATIC_JAVA_LIBRARIES：**  
当前模块依赖的Java静态库，在Android里，导入的jar包和引用的第三方工程都属于Java静态库。

+ **LOCAL_STATIC_LIBRARIES：**  
指定该模块需要使用哪些静态库，以便在编译时进行链接。

+ **LOCAL_SHARED_LIBRARIES：**  
指定模块在运行时要依赖的共享库（动态库），在链接时就需要，以便在生成文件时嵌入其相应的信息。

+ **LOCAL_C_INCLUDES：**   
可选变量，表示头文件的搜索路径。 默认的头文件的搜索路径是LOCAL_PATH目录。

+ **LOCAL_CFLAGS：**  
提供给C/C++编译器的额外编译参数。

+ **LOCAL_PACKAGE_NAME：**  
当前APK应用的名称。

+ **LOCAL_CERTIFICATE：**  
签署当前应用的证书名称。

+ **LOCAL_MODULE_TAGS：**  
当前模块所包含的标签，一个模块可以包含多个标签。标签的值可能是eng、user、debug、development、optional。其中，optional是默认标签。

+ **LOCAL_DEX_PREOPT：**  
apk的odex优化开关，默认是false。

+ **LOCAL_LDLIBS:**  
编译模块时要使用的附加的链接器选项。这对于使用‘-l’前缀传递指定库的名字是有用的。  
例如，LOCAL_LDLIBS := -lz表示告诉链接器生成的模块要在加载时刻链接到/system/lib/libz.so  
可查看 docs/STABLE-APIS.TXT 获取使用 NDK发行版能链接到的开放的系统库列表。  

+ **LOCAL_MODULE_PATH 和 LOCAL_UNSTRIPPED_PATH**  
在 Android.mk 文件中， 还可以用LOCAL_MODULE_PATH 和LOCAL_UNSTRIPPED_PATH指定最后的目标安装路径.  
不同的文件系统路径用以下的宏进行选择：  

  + TARGET_ROOT_OUT：表示根文件系统。
  + TARGET_OUT：表示 system文件系统。
  + TARGET_OUT_DATA：表示 data文件系统。
      如：LOCAL_MODULE_PATH :=$(TARGET_ROOT_OUT)
      至于LOCAL_MODULE_PATH 和LOCAL_UNSTRIPPED_PATH的区别，暂时还不清楚。

+ **LOCAL_JNI_SHARED_LIBRARIES：**  
定义了要包含的so库文件的名字，如果程序没有采用jni，不需要LOCAL_JNI_SHARED_LIBRARIES := libxxx 这样在编译的时候，NDK自动会把这个libxxx打包进apk； 放在youapk/lib/目录下  
除此之外，Build系统中还定义了一些函数方便在Android.mk中使用，包括：  

  + $(call my-dir)：获取当前文件夹的路径。
  + $(call all-java-files-under, )：获取指定目录下的所有java文件。
  + $(call all-c-files-under, )：获取指定目录下的所有c文件。
  + $(call all-Iaidl-files-under, )：获取指定目录下的所有AIDL文件。
  + $(call all-makefiles-under, )：获取指定目录下的所有Make文件。
  + $(call intermediates-dir-for, , , ,

# 五、NDK提供的函数宏

GNU Make函数宏，必须通过使用’$(call )’来调用，返回值是文本化的信息。  
+ **my-dir:**  
返回当前 Android.mk 所在的目录的路径，相对于 NDK 编译系统的顶层。在 Android.mk 文件的开头如此定义： LOCAL_PATH := $(call my-dir)  
+ **all-subdir-makefiles:**   
返回一个位于当前’my-dir’路径的子目录中的所有Android.mk的列表。  
例如，某一子项目的目录层次如下：

```
src/foo/Android.mk
src/foo/lib1/Android.mk
src/foo/lib2/Android.mk
```
如果 src/foo/Android.mk 文件中使用了： include $(call all-subdir-makefiles)那么它就会自动包含 src/foo/lib1/Android.mk 和 src/foo/lib2/Android.mk。  
这项功能用于向编译系统提供深层次嵌套的代码目录层次。  
注意，在默认情况下，NDK 将会只搜索在 src/*/Android.mk 中的文件。  

+ **this-makefile:**  
返回当前Makefile 的路径(即这个函数调用的地方)  
+ **parent-makefile:**  
返回调用树中父 Makefile 路径。即包含当前Makefile的Makefile 路径。
+ **grand-parent-makefile：**  
返回调用树中父Makefile的父Makefile的路径

# 六、 Android.mk示例

## 编译静态库

```
LOCAL_PATH := $(call my-dir) 
include $(CLEAR_VARS) 
LOCAL_MODULE = libhellos 
LOCAL_CFLAGS =$(L_CFLAGS) 
LOCAL_SRC_FILES = hellos.c 
LOCAL_C_INCLUDES = $(INCLUDES) 
LOCAL_SHARED_LIBRARIES := libcutils 
LOCAL_COPY_HEADERS_TO := libhellos 
LOCAL_COPY_HEADERS := hellos.h 
include $(BUILD_STATIC_LIBRARY) 
```

## 编译动态库

```
LOCAL_PATH := $(call my-dir) 
include $(CLEAR_VARS) 
LOCAL_MODULE = libhellod 
LOCAL_CFLAGS = $(L_CFLAGS) 
LOCAL_SRC_FILES = hellod.c 
LOCAL_C_INCLUDES = $(INCLUDES) 
LOCAL_SHARED_LIBRARIES := libcutils 
LOCAL_COPY_HEADERS_TO := libhellod 
LOCAL_COPY_HEADERS := hellod.h 
include $(BUILD_SHARED_LIBRARY
```

## 使用静态库

```
LOCAL_PATH := $(call my-dir) 
include $(CLEAR_VARS) 
LOCAL_MODULE := hellos 
LOCAL_STATIC_LIBRARIES := libhellos 
LOCAL_SHARED_LIBRARIES := 
LOCAL_LDLIBS += -ldl 
LOCAL_CFLAGS := $(L_CFLAGS) 
LOCAL_SRC_FILES := mains.c 
LOCAL_C_INCLUDES := $(INCLUDES) 
include $(BUILD_EXECUTABLE) 
```

## 使用动态库

```
LOCAL_PATH := $(call my-dir) 
include $(CLEAR_VARS) 
LOCAL_MODULE := hellod 
LOCAL_MODULE_TAGS := debug 
LOCAL_SHARED_LIBRARIES := libc libcutils libhellod 
LOCAL_LDLIBS += -ldl 
LOCAL_CFLAGS := $(L_CFLAGS) 
LOCAL_SRC_FILES := maind.c 
LOCAL_C_INCLUDES := $(INCLUDES) 
include $(BUILD_EXECUTABLE) 
```


Reference Link:
https://blog.csdn.net/xx326664162/article/details/52875825


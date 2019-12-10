# 硬件接口-Camera Driver IC

在了解Camera在Android中的软件结构之前，我们还是要先看一看在实际工程中，Camera的硬件接口。这样有助于我们回顾前面讲的模组相关内容，也便于理解后面的Camera Driver以及HAL的知识。

## 原理图

一个合格的驱动工程师怎么能不会看原理图呢？我们可以以MT6797中Defaule Design中窥视一下Camera的接口：

### **前置13M S5K3**

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E7%A1%AC%E4%BB%B6%E6%8E%A5%E5%8F%A3.png">

### **后置23M OV23850**

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E7%A1%AC%E4%BB%B6%E6%8E%A5%E5%8F%A3%20%E5%90%8E%E7%BD%AE%E6%91%84%E5%83%8F%E5%A4%B4.png">

从上面的原理图，我们就可以总结出AP和Sensor之间的接口PIN：

+ MIPI 数据输出接口：
  + CLK Mipi Clk P/N
  + D0  Mipi Data0 P/N
  + D1  Mipi Data1 P/N(Option)
  + D2  Mipi Data2 P/N(Option)
  + D3  Mipi Data3 P/N(Option)
  
+ POWER 电源控制：
  + AVDD     2.8V, 模拟电，用于感光Sensor区和ADC等；
  + DOVDD    1.8V，IO电，I2C/mipi接口等；
  + DVDD     1.2V, 核心电，用于ISP等；
  + AF_VDD   2.8V, Lens的供电
  
+ CTRL 控制PIN：
  + PWDN     上电/休眠控制
  + RESET    复位
  + MCLK     AP给模组的主时钟（6~27M）
  
+ SCCB(Serial Camera Control Bus)
  + SCL  I2C时钟线
  + SDA  I2C数据线

## 芯片手册

不会看芯片手册的程序员也不是好程序员。我挑选了两款主流厂家的Sensor（OV48B和IMX327），对比一下：

### OV48B

#### 支持特性

我们以OV最新的48M的Sensor IC为例，看看在官方描述中，都会提及它的那些特性：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E7%89%B9%E5%BE%81%E6%94%AF%E6%8C%81%EF%BC%88%E7%AE%80%E6%B4%81%EF%BC%89.png">
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E7%89%B9%E5%BE%81%E6%94%AF%E6%8C%81.png">

注：怎么感觉不是很清晰？可以在[这里](http://www.ovt.com.cn/wp-content/uploads/2019/10/OmniVision_OV48B.pdf)查看官方版本的PDF。

#### 内部框图

最后，放一张芯片内部框图，看看他们是如何配合工作的：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E5%86%85%E9%83%A8%E6%A1%86%E5%9B%BE.png">

### IMX327

#### 支持特性

看看索尼的这一款Sensor描述，比较一下主流的Sensor厂的侧重点是否一样：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E7%89%B9%E5%BE%81%E6%94%AF%E6%8C%81%EF%BC%88IMX327%EF%BC%89.png">

#### 内部框图

以及放一下IMX327的内部框图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E5%86%85%E9%83%A8%E6%A1%86%E5%9B%BE(Sony%20Imx327).png">

## 初始化

千里之行，始于足下。Camera IC再厉害，也离不开上电、CLK、I2C控制这一套。由上面的图示可以看出来，一般的Camera Sensor有三路电，一路CLK，一套SCCB。如果系统无法识别IC，那么肯定是初始化过程中的这三者遇到了问题，可以逐一排查。

先看上电时序：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E4%B8%8A%E7%94%B5%E6%97%B6%E5%BA%8F.png">

再看整个初始化过程：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E5%88%9D%E5%A7%8B%E5%8C%96%E6%B5%81%E7%A8%8B.png">


# Android Camera

## Android Architecture

在了解Camera在Android中的地位之前，总是要先搞清楚Android的架构。（什么是Android应该可以略过吧。）
从Android官网摘来分层图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E6%A1%86%E6%9E%B6%E5%88%86%E5%B1%82%E5%9B%BE.png">

从这张图上可以看出，Android分为5层：

+ **Applications**:

最上层的应用，编译后生成某应用APK。很多此类 API 都可以直接映射到底层 HAL 接口，并可提供与实现驱动程序相关的实用信息。
+ **Application Framework/JNI**:
  + 主要为Applications提供API。
  + JNI 使Application Framework和Libraries可交互;
+ **Native Libraries/Android Runtime**:
  + 包括Framework和Service。系统服务是专注于特定功能的模块化组件，例如窗口管理器、搜索服务或通知管理器。 应用框架 API 所提供的功能可与系统服务通信，以访问底层硬件。Android 包含两组服务：“系统”（诸如窗口管理器和通知管理器之类的服务）和“媒体”（与播放和录制媒体相关的服务）；
  + Android最新的ART虚拟机，替代之前的Java虚拟机，可以直接把Application字节转换成机器码；
+ **HAL/HIDL**:

  + 硬件抽象层（Hardware Abstract Layer） 用来链接driver和 Service。HAL 可定义一个标准接口以供硬件供应商实现，这可让 Android 忽略较低级别的驱动程序实现。借助 HAL，您可以顺利实现相关功能，而不会影响或更改更高级别的系统。HAL 实现会被封装成模块，并会由 Android 系统适时地加载。
  + HAL接口描述语言（HAL Interface Description Language），Android 8.0 重新设计了 Android 操作系统框架（在一个名为“Treble”的项目中），以便让制造商能够以更低的成本更轻松、更快速地将设备更新到新版 Android 系统。在这种新架构中，HAL 接口定义语言（HIDL，发音为“hide-l”）指定了 HAL 和其用户之间的接口，让用户能够替换 Android 框架，而无需重新编译 HAL。
+ **Kernel**:

硬件driver的驱动。发设备驱动程序与开发典型的 Linux 设备驱动程序类似。Android 使用的 Linux 内核版本包含几个特殊的补充功能，例如：Low Memory Killer（一种内存管理系统，可更主动地保留内存）、唤醒锁定（一种 PowerManager 系统服务）、Binder IPC 驱动程序以及对移动嵌入式平台来说非常重要的其他功能。这些补充功能主要用于增强系统功能，不会影响驱动程序开发。您可以使用任意版本的内核，只要它支持所需功能（如 Binder 驱动程序）即可。

这里，我们重点关注Kernel->HAL->Native Libratries这三层。

## Android Camera Architecture

我们拿Camera相关内容去映射到上面的分层中：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E5%9B%BE.png">

由上图可以看出，我们可以着重关注三个部分：
+ **Camera Service**: 

即在Native Libraries层，是Camera Framework的范畴，包括了Camera Service和Camera Client；
+ **Camera HA**L:

即在HAL/HIDL层，用来连接Camera Driver和Camera Service的（这部分Android提供了原型API，各个平台厂也会修改）
+ **Camera Driver**: 

即在Kernel这一层，用的是Linux的V4L2架构，然后贴上Platform Vendor和Sensor Driver IC Vendor的Patch，用来驱动各自的Camera IC的；


## Android Camera Implement

在了解了框架之后，我们尝试把框架内的一些关键接口和方法描述一下，还是先采用Android官方的图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20%E5%AE%9E%E7%8E%B0%E7%BB%93%E6%9E%84%E5%9B%BE.png">

在展开这个图之前，有必要先回顾一下Android各个版本的代号以及时间：
+ Android4 K 2013年 
  + 相机 API1.Android 4.4 及更低版本设备上的应用级相机框架，通过 android.hardware.Camera 类提供。
  + 相机 HAL3.1.随 Android 4.4 发布的相机设备 HAL 版本。
+ Android5 L 2014年 
  + Camera v2 API. Android 5.0对拍照API进行了全新的设计，新增了全新设计的Camera v2 API，这些API不仅大幅提高了Android系统拍照的功能，还能支持RAW照片输出，甚至允许程序调整相机的对焦模式、曝光模式、快门等。
  + 相机 HAL3.2
+ Android6 M 2015年
+ Android7 N 2016年
+ Android8 O 2017年 
  + 引入了 Treble，用于将 CameraHal API 切换到由 HAL 接口描述语言 (HIDL) 定义的稳定接口
+ Android9 P 2018年
+ Android10 Q 2019年
  + Camera HAL 3.5

为什么要回顾这些呢？因为Android更新太快了，应用层的API Version有API1和API2，HAL层的Version有HAL1和HAL3，而且到了Android O后，又因为引入了Treble机制，导致Android HAL又大改了一次，所以我们后面将要提到的流程中，都是基于Android 8之后的API2 + HAL3。

当然，碎片化虽然严重，但是Android工程师们可不是吃素的，各个版本都可以兼容，参考下图：

+ [官方版本](https://source.android.com/devices/camera)

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20%E6%97%A7%E7%89%88%E5%85%BC%E5%AE%B9%E5%AE%9E%E7%8E%B0%E7%BB%93%E6%9E%84%E5%9B%BE.png">

+ [民间版本](https://www.jianshu.com/p/4e838a5cdb05)

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20API%E5%92%8CHAL%E7%89%88%E6%9C%AC%E7%9A%84%E5%AF%B9%E5%BA%94%E5%85%B3%E7%B3%BB.JPG">



# 什么是ISP？

ISP(Image Signal Processor)，即图像处理，主要作用是对前端图像传感器输出的信号做后期处理，主要功能有线性纠正、噪声去除、坏点去除、内插、白平衡、自动曝光控制等，依赖于ISP才能在不同的光学条件下都能较好的还原现场细节，ISP技术在很大程度上决定了摄像机的成像质量。</br>
光线始于Lens，终于Sensor，CIS之后再无模拟。光线被Sensor转换为电讯号之后，经过ADC的采样、量化，最终形成了我们熟悉的01世界的数字图像。数字图像的下一个加工厂就是ISP了！

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ARM_ISP.png">

看ISP的直译就知道，它也是一个处理器，所以根据它的位置，我们可以把它分为集成式和分离式：

## 分离式

分离式也称为外置ISP架构，这里的分离和外置的对象都是主处理器AP。所以，只要ISP没有被集成在AP内部的都可以成为分离式。
在Camera分辨率还没有1M的年代，这个量级的处理量Camera Driver IC厂商有能力研发自己的ISP，他们可以把这部分集成在自己的Driver IC中；但是现如今的Camera动不动就要上亿了，总有些专业的数字图像处理的公司接管了这部分业务，有了自己的ISP（比如：socionext，arm和ov），所以我们现如今说的分离式的ISP都是这种在AP外下挂一个专业ISP的形式：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E5%A4%96%E7%BD%AEISP.png">

## 集成式

集成的就很容易理解了，现如今高通、苹果、三星、海思、MTK主流的手机AP供应商都会把ISP集成到他们的芯片内部，这样可以给用户提供更便捷的调试体验（算法平台一手承包，真是工程师们的福音），还节约了成本，节约了宝贵的PCB空间资源。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E5%86%85%E7%BD%AEISP.png">

# ISP的内部架构

如下图所示，ISP内部包含 CPU、SUP IP、IF 等设备，事实上，可以认为 ISP 是一个 SOC（system of chip），可以运行各种算法程序，实时处理图像信号。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E7%BB%93%E6%9E%84.png">

+ **CPU**

CPU 即中央处理器，可以运行 AF、LSC 等各种图像处理算法，控制外围设备。现代的 ISP 内部的 CPU 一般都是 ARM Cortex-A 系列的，例如 Cortex-A5、Cortex-A7。

+ **SUB IP**

SUB IP 是各种功能模块的通称，对图像进行各自专业的处理。常见的 SUB IP 如 DIS(Digital Image Stabilization)、CSC(Color Space Conversion)、VRA(Vehicle Routing Algorithm) 等。

+ **图像传输接口**

图像传输接口主要分两种，并口 ITU(International Telecommunication Union) 和串口 CSI(Camera Serial Interface)。CSI 是 MIPI CSI 的简称，鉴于 MIPI CSI 的诸多优点，在手机相机领域，已经广泛使用 MIPI-CSI 接口传输图像数据和各种自定义数据。

+ **通用外围设备**

通用外围设备指 I2C、SPI、PWM、UART、WATCHDOG 等。ISP 中包含 I2C 控制器，用于读取 OTP 信息，控制 VCM 等。对于外置 ISP，ISP 本身还是 I2C 从设备。AP 可以通过 I2C 控制 ISP 的工作模式，获取其工作状态等。


# ISP的控制结构

控制结构可以从三方面看，包括： 
+ 1、ISP前端逻辑    
+ 2、ISP内部逻辑
+ 3、ISP后端逻辑

## 前端逻辑（与Sensor/Lens的关系）

如图所示，lens 将光信号投射到sensor 的感光区域后，sensor 经过光电转换，将Bayer 格式的原始图像送给ISP，ISP 经过算法处理，输出RGB空间域的图像给后端的视频采集单元。在这个过程中，ISP通过运行在其上的firmware（固件）对ISP逻辑，从而对lens 和sensor 进行相应控制，进而完成自动光圈、自动曝光、自动白平衡等功能。其中，firmware的运转靠视频采集单元的中断驱动。PQ Tools 工具通过网口或者串口完成对ISP 的在线图像质量调节。

ISP 由ISP逻辑及运行在其上的Firmware组成，逻辑单元除了完成一部分算法处理外，还可以统计出当前图像的实时信息。Firmware 通过获取ISP 逻辑的图像统计信息，重新计算，反馈控制lens、sensor 和ISP 逻辑，以达到自动调节图像质量的目的。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E6%8E%A7%E5%88%B6%E6%9E%B6%E6%9E%84.png">

## 内部逻辑（Firmware行为）

ISP 的Firmware包含三部分：
+ ISP 控制单元和基础算法库；
+ AE/AWB/AF 算法库；
+ sensor 库；

Firmware 设计的基本思想是单独提供3A算法库，由ISP控制单元调度基础算法库和3A 算法库，同时sensor 库分别向ISP 基础算法库和3A 算法库注册函数回调，以实现差异化的sensor 适配。ISP firmware 架构如图所示。

不同的sensor 都以回调函数的形式，向ISP 算法库注册控制函数。ISP 控制单元调度基础算法库和3A 算法库时，将通过这些回调函数获取初始化参数，并控制sensor，如调节曝光时间、模拟增益、数字增益，控制lens 步进聚焦或旋转光圈等。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20FW%20ALG.png">


## 后端逻辑（与AP的关系）

CPU处理器包括：AP、BP、CP。 BP：基带处理器、AP：应用处理器、CP：多媒体加速器

这里所说的控制方式是AP 对 ISP 的操控方式 。

+ **I2C/SPI**：这一般是外置 ISP 的做法。SPI 一般用于下载固件、I2C 一般用于寄存器控制。在内核的 ISP 驱动中，外置 ISP 一般是实现为 I2C 设备，然后封装成 V4L2-SUBDEV。
+ **MEM MAP**：这一般是内置 ISP 的做法。将 ISP 内部的寄存器地址空间映射到内核地址空间，
+ **MEM SHARE**：这也是内置 ISP 的做法。AP 这边分配内存，然后将内存地址传给 ISP，二者实际上共享同一块内存。因此 AP 对这段共享内存的操作会实时反馈到 ISP 端。



# SDIO

## 概述

SDIO，全称：Secure Digital Input and Output ，即安全数字输入输出接口。
SDIO协议是由SD卡的协议演化升级而来的，很多地方保留了SD卡的读写协议，同时SDIO协议又在SD卡协议之上添加了CMD52和CMD53命令。

## 特点

+ 它采用HOST-DEVICE模式，所有通信都由HOST端发命令，DEVICE设备只要解析HOST命令就可与HOST进行通信。
+ 支持三种不同的数据总线模式：1位(默认)、4位和8位，同时支持SPI模式。
+ 支持Low-Speed, High-Speed, 高速传输速率可达100Mb/sec
+ 支持热拔插
+ 支持Slave给Host发送中断（唤醒功能）
+ 2.7~3.6V的初始化电平，枚举后支持3.3V或者1.8V
+ 支持全Size的SDIO和miniSDIO


## 硬件接口

本篇内容主要是想了解SDIO设备，但是如果翻开SDIO的官方Spec，面对缭乱的功能，很容易混淆不知所以然。即便是更复杂的Mipi协议感觉也没有如此混乱。
其实要了解SDIO的架构，还是要从历史发展的角度理一理，说不准能有一个新方向。SDIO接口并不是从无到有的顶层设计，而是自然演进过来的，从MMC->SD->SDIO，我们先来看看他们的演进历程。

### 1 MMC

MMC（Multimedia Card），中文翻译为多媒体卡，设计之初就是用来做多媒体文件存储的，所以它只具备存储功能。

熟悉Linux kernel的人都知道，kernel使用MMC subsystem统一管理MMC、SD、SDIO等设备，为什么呢？
从1997年MMC规范发布至今，基于不同的考量（物理尺寸、电压范围、管脚数量、最大容量、数据位宽、clock频率、安全特性、是否支持SPI mode、是否支持DDR mode、等等），进化出了MMC、SD、microSD、SDIO、eMMC等不同的规范（如下面图片1所示）。虽然乱花迷人，其本质终究还是一样的，丝毫未变，这就是Linux kernel将它们统称为MMC的原因。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20MMC-SD-SDIO%E6%BC%94%E8%BF%9B.gif">

从上图可以看出MMC是SD/SDIO/eMMC等一系列的祖师爷，1997年由西门子和闪迪共同开发，技术基于东芝的Nand-Flash。

#### 1.1 实物展示

MMC作为一种硬件存储设备所以MMC的接口最为简单，只有7个Pin。大小跟一张油票差不多，约24mm x 32mm x 1.5mm。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20MMC%20%E5%AE%9E%E7%89%A9.jfif">

#### 1.2 接口展示

因为只有7个Pin，所以只能支持SPI和1 Bit模式（现在有发展出新的MMC接口和协议，已经可以支持4bit/8bit）

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20MMC%20%E7%AE%A1%E8%84%9A%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg">


MMC 的 SPI mode 时钟最高只能到25MHz，因此读取速度通常低于3MB/s。之所以在MMC支持SPI模式，是因为：
+ 1、SPI总线是一个通用总线，大部份芯片都用硬件模块；
+ 2、SPI模式支持不带CRC校验的传输方式，可以降低硬件要求；
+ 3、SD的CMD线与DATA线之间有可能同时产生数据，对没有SD硬件模块的主机支持起来难度较高。

了解了MMC的最初设计初衷以及硬件接口，为了保证兼容性，后面的演进SD/SDIO等都支持SPI模式就会更容易理解一些。


### 2 SD Card

1999年，由日本松下、东芝及美国SanDisk公司共同研制完成。2000年，这几家公司发起成立了SD协会（Secure Digital Association简称SDA）
SD卡（Security Digtial card）中文翻译成安全数码卡。它是从MMC的基础上发展来的，做了什么修改呢？从字面上看，我们就知道它一定增加了安全加密功能：
+ SD卡比MMC多了2个管脚，所以可以支持到4Bit总线模式。
+ SD卡使用卡内智能控制模块进行FLASH操作控制，包括协议、安全算法、数据存取、ECC算法、缺陷处理和分析、电源管理、时钟管理。
+ 大部分SD卡的侧面设有写保护控制，以避免一些数据意外地写入，而少部分的SD卡甚至支持数字版权管理的技术。
+ SD卡是基于flash的存储卡。
+ SD卡和MMC卡的区别在于初始化过程不同。
+ SD卡的通信协议包括SD总线和SPI两类。
+ 通信电压范围：2.0-3.6V；工作电压范围:2.0-3.6V
+ 最大10 个堆叠的卡（20MHz,Vcc=2.7-3.6V)

SD卡的设计也是为了存储功能，最直观的感受就是PIN脚会比MMC多了2个（可以支持到了SD 4Bit模式），同时SD也是从外观到协议不断演进的，比如为了适应移动数码产品的需要，演进除了Mini DS（迷你SD）和microSD（TF-TransFlash 卡）。


#### 2.1 实物展示

如下图，是SD卡、Mini SD、Micro SD的实物对比：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%E5%8D%A1%E5%AE%9E%E7%89%A9.png">

#### 2.2 接口展示

除了大小上的不同，在管脚上，SD/MINI/Micro也不相同：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%E5%8D%A1%E7%AE%A1%E8%84%9A%E6%8E%A5%E5%8F%A3.gif">

+ miniSD相对于标准SD，增加了2个NC引脚
+ microSD相对于标准SD，减少了1个VSS引脚


但是需要指出的是，在UHS-II以及UHS-III中，管脚接口又增加了8个，这样速度可以成倍增加。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%20UHS-II%20%E6%8E%A5%E5%8F%A3.png">

#### 2.3 管脚定义

因为Mini SD和Micro SD本质上都是SD，所以从数据传输协议上看，都是一样的，只是对地的修改。（这也可以理解为什么Mini SD或Micro SD加个套套就可以当做SD来用）

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%E5%8D%A1%20Pin%E8%84%9A%E5%AE%9A%E4%B9%89.jpg">



#### 2.4 容量规范

SDHC（SD high capacity）中文翻译为SD大容量卡, SDXC(SD eXtended Capacity，中文名：容量扩大化的安全存储卡)。

目前市场上的SD、Mini SD、Micro SD卡遵循的是SD Spec Ver1.0或1.1规范，最大可能容量仅为**2GB**。2006年，SDA协会发布了SD Spec Ver2.0规范，符合此新规范的SD卡容量可达4GB或更高。
符合2.0规范的SD卡，称为SDHC（SD high capacity）卡。SDHC卡外形维持与SD卡一致，但是文件系统从FAT12、FAT16改为FAT32型；SDHC卡的最大容量可达32GB。除了SDHC卡外,还有Mini SDHC，Micro SDHC类型的卡。

SDHC卡与标准SD卡不再兼容，必须符合SD Spec Ver2.0的设备才能支持SDHC卡，这样的设备都会带有SDHC logo。而支持SDHC卡的设备可以向下兼容标准SD卡。
为了充分发挥SDHC卡的性能，保证兼容性，SDA协会为SDHC卡定义了3个速度等级：2，4，6；其含义是各等级分别可以忍受的写速率至少是2MB/S，4MB/S，6MB/S.速度等级定义中使用的是数据写速率，数据读速率要比数据写速率快。

有三种容量等级：
+ SD最大容量为2GB， 支持FAT 12, 16。
+ HC内存卡最大容量为32GB，支持FAT 32。
+ XC内存卡最大容量为2TB，支持exFAT。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%E6%A0%87%E8%AF%86%E5%8F%8A%E5%85%B6%E9%80%9F%E5%BA%A6%E5%AF%B9%E6%AF%94.png">


#### 2.5 速度规范

速度上，可以通过Class和UHS(Ultra High-Speed)两个量级来衡量。

SD2.0的规范：
普通卡和高速卡的速率定义为Class2、Class4、Class6 和Class10 四个等级。在Class10卡问世之前，存在过一阵Class11和Class13的卡,但这种标准最终没有被SDA共识。

SD3.01规范：
又被称为超高速卡，速率定义为UHS-I（理论上可以支持最大 104MB/s 的总线速度）和UHS-II（理论总线速度骤然提高到了 312MB/s）。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%20Class%20%E6%A0%87%E8%AF%86%E5%8F%8A%E5%85%B6%E9%80%9F%E5%BA%A6%E5%AF%B9%E6%AF%94.png">

可以参考如下演进图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SD%20UHS.jpg">


### 3 SDIO

SDIO是在SD内存卡接口的基础上发展起来的外设接口，SDIO接口兼容以前的SD内存卡，并且可以连接SDIO接口的设备，目前根据SDIO协议的SPEC，SDIO接口支持的设备总类有：
+ Wi-Fi card（无线网络卡） 
+ CMOS sensor card（照相模块） 
+ GPS card 
+ GSM/GPRS modem card 
+ Bluetooth card 
+ Radio/TV card

我们可以简单的理解成公式：SDIO = SD + IO，即这种总线既要支持SD存储，满足之前SD的所有协议，也要支持IO功能，作为数据吞吐的总线接口。

### 3.1 硬件展示

SDIO从SD中进化而来，却又抽象了SD，因为它终于脱离了存储这个约束，开始走向总线协议上来了，这也是我们本篇文章来研究它的初衷。
因此，从硬件上就没有我们看到的MMC/SD这么直观的实物了，但是作为一个Host+Slave模式的总线接口，我们可以通过承载这种硬件接口的Host和Slave侧的硬件原理图窥探一下。

+ Host:

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20MSDC%20AP%20Side.png">

这个图采自MTK6595的MSCS(Memory Stick And SD card Controller)，即AP侧的SDIO的接口，可以看到SDIO的CMD/CLK/DATA0~Data3的四总线模式以及CMD/CLK/DATA0~DATA7的八总线模式（这种模式一般都连在eMMC上，eMMC是embedded MMC）。我们重点关注4总线模式即可。

+ Slave：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%2BWIFI%20%E6%8E%A5%E5%8F%A3.png">

这个图展现的是SDIO接口的WIFI模块。

可以看出在Host和Slave之间是采用直连模式（如果不放心，可以下挂TVS器件以改善ESD性能）

#### 3.1 接口介绍

有了如上的硬件展示，接口部分就没什么可说的了。截取SDIO官方的一个示意图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20%E8%BF%9E%E6%8E%A5%E4%B8%A4%E4%B8%AA4-bit%20Cards.png">


+ CLK信号:HOST给DEVICE的时钟信号。
+ CMD信号：双向的信号，用于传送命令和反应。
+ DAT0-DAT3 信号:四条用于传送的数据线。

对于SDIO总线下挂的设备可以通称为SD卡设备。

==========================================================================

总结一下他们之间的关系：

+ MMC、SD、SDIO的技术本质是一样的（使用相同的总线规范，等等），都是从MMC规范演化而来；
  + MMC强调的是多媒体存储（MM，MultiMedia）；
  + SD强调的是安全和数据保护（S，Secure）；
  + SDIO是从SD演化出来的，强调的是接口（IO，Input/Output），不再关注另一端的具体形态（可以是WIFI设备、Bluetooth设备、GPS等等）。

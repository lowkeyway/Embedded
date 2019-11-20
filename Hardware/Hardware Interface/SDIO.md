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

那么对应的UHS-II之后增加的八个管脚可以对应下图：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20UHS-II%20Interface%20Pin%20define.png">

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

#### 3.1 硬件展示

SDIO从SD中进化而来，却又抽象了SD，因为它终于脱离了存储这个约束，开始走向总线协议上来了，这也是我们本篇文章来研究它的初衷。
因此，从硬件上就没有我们看到的MMC/SD这么直观的实物了，但是作为一个Host+Slave模式的总线接口，我们可以通过承载这种硬件接口的Host和Slave侧的硬件原理图窥探一下。

+ Host:

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20MSDC%20AP%20Side.png">

这个图采自MTK6595的MSCS(Memory Stick And SD card Controller)，即AP侧的SDIO的接口，可以看到SDIO的CMD/CLK/DATA0-Data3的四总线模式以及CMD/CLK/DATA0-DATA7的八总线模式（这种模式一般都连在eMMC上，eMMC是embedded MMC）。我们重点关注4总线模式即可。

+ Slave：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%2BWIFI%20%E6%8E%A5%E5%8F%A3.png">

这个图展现的是SDIO接口的WIFI模块。

可以看出在Host和Slave之间是采用直连模式（如果不放心，可以下挂TVS器件以改善ESD性能）

##### 3.2 接口介绍

有了如上的硬件展示，接口部分就没什么可说的了。截取SDIO官方的一个示意图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20%E8%BF%9E%E6%8E%A5%E4%B8%A4%E4%B8%AA4-bit%20Cards.png">

以及参考SD协议中对PIN定义的Pin Number（重点关注Pin1，在SD模式下即可以做Data3，也可以做Card Detect）：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20Pin%20%E5%AE%9A%E4%B9%89.png">

+ CLK信号:HOST给DEVICE的时钟信号。
+ CMD信号：双向的信号，用于传送命令和反应。
+ DAT0-DAT3 信号:四条用于传送的数据线。

对于SDIO总线下挂的设备可以通称为SD卡设备。


### 4.总结

总结一下他们之间的关系：

+ MMC、SD、SDIO的技术本质是一样的（使用相同的总线规范，等等），都是从MMC规范演化而来；
  + MMC强调的是多媒体存储（MM，MultiMedia）；
  + SD强调的是安全和数据保护（S，Secure）；
  + SDIO是从SD演化出来的，强调的是接口（IO，Input/Output），不再关注另一端的具体形态（可以是WIFI设备、Bluetooth设备、GPS等等）。
  
  
  
## 协议

因为SDIO是从SD协议中进化出来的，所以SDIO(IO Only）的协议必须要兼容SD卡的协议。通过上面的一些历史铺垫，相信对SDIO有了初步的认识，背着历史包袱的SDIO也注定会比较繁杂，而这繁杂从哪里开始讲起呢？
当然是枚举！因为Host在上电的瞬间就要知道它面对的Slave是什么设备，才能化繁为简，

### 1. 初始化（枚举）

借鉴USB的枚举概念，我们更容易理解。初始化的过程就是为了辨别Slave是MMC还是SD，是SDSC还是SDHC/SDXC，或者是我们今天的主角SDIO IO Only Card？

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20IO%20Only%20%E6%9E%9A%E4%B8%BE.png">

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20IO%20Only%20%E6%9E%9A%E4%B8%BE%20%E5%90%8D%E8%AF%8D%E8%A7%A3%E6%9E%90.png">

如上图是摘自最新的SDIO简明手册的初始化部分。依然复杂，但对于我们来说，为了便于理解，我们可以直接默认为Slave是SDIO IO Only设备，看它在这个流程中都经过哪些Case，然后再通过这些Case抽丝剥茧去介绍每个命令是什么含义！

+ (1) Power On               （我们以上电初始化为例）
+ (2) IO=0, MEM=0, F2=0      （参考上图的IO/MEM/F2的含义，这里是初始化的时候把一些Flag清零，以便后面流程中判断）
+ (3) CMD0 Pin1=High         （Pin1 对应的是Data3，即在Data3 Pin拉高的情况下由Host发送CMD0命令，强制Slave进入Idle状态）
+ (4) CMD8                   （发送CMD8命令，做电压检测，初始化电压3.3V）
+ (5) Check Responses        （其实就是检查CMD8的返回R7命令是否正常）
+ (6) F8=1                   （假想实验中，R7当然正常，表明Host提供的电源下Slave可以正常工作）
+ (7) Initialize SDIO        （测试IO Flag是否需要初始化，很明显，IO Flag在我们上电的时候被清零了）
+ (8) CMD5 Arg=0             （发送CMD5命令，OCR参数为0，询问Slave的电压支持范围）
+ (9) Check Response         （Slave对CMD5的命令返回正常，R4命令格式正确，OCR中包含了支持的电压返回）
+ (10) CMD5 Arg=S18R, WV      （收到Slave返回的支持电压范围假设为1.8V，再次发送CMD5命令请求切换1.8V的）
+ (11) Check Response         （再次检查R4，看是否正常返回，如果正常返回看R4的C位）
+ (12) IO=1                   （R4的C位为1，表示初始化Ok，可以进行下一步操作了。设置IO Flag为1）
+ (13) Test MP Flag           （因为我们是IO Only，所以R4的Memory Present位为0）
+ (14) Check IO               （跳过了Memory Initialization， 检查IO Flag，我们在12步的时候已经设置为1了）
+ (15) Check S18A(OR)         （R4的S18A位，假设我们的Slave可以支持1.8V，所以这一位应该为1）
+ (16) CMD11                  （Host发送CMD11，切换电平为1.8V）
+ (17) Check Response         （检查CMD11的返回R1是否正确，当然正确）
+ (18) Voltage Switch Sequence  （电平切换中）
+ (19) Check Error            （检查有没有Error，当然没有）
+ (20) Check MEM, F2          （MEM和F2 我们在初始化为0后就再也没动过）
+ (21) CMD3                   （Host发送CMD3， 请求Slave提供relative address）
+ (22) Check IO, MEM          （IO我们设置过为1， MEM没有变为0）
+ (23) IO Only Card           （SDIO IO Only设备）

至此，枚举成功，并且结束。


### 2. 命令和响应

要想搞清楚枚举过程，就要对SDIO的命令有所认识。
我们知道，标准的SDIO有6跟线，其中一条CLK, 一条CMD， 四条数据总线DAT0~DTA7。在执行命令的时候，只有CLK和CMD参与工作，不管是Host主动发的CMD还是Slave返回的Response。

#### 2.1 命令和响应的形式

前面也提到了，SDIO总线属于主从结构，所有的命令都要由Host发起。对于所有的SDIO命令来说，可以分为有返回和无返回两种情况。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20No%20response%20and%20No%20data.png">

那么从协议角度，对所有的命令有通用的格式规定：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20Command%20Token%20Format.png">

同时，对有返回的返回格式也有规定：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20Response%20Token%20Format.png">

#### 2.2 枚举常用命令简析

好了，了解了这一点，我们可以通过几个例子来窥探一下SDIO CMD的轮廓。SDIO的命令浩如烟海，有上百之多，我们先关心一下刚才提到的枚举流程中遇到的。相信枚举得过程弄懂了，其他的流程都不在话下。
枚举中遇到的命令依次是CMD0->CMD8->CMD5->CMD11->CMD3，可以通过下表看看都是什么意思。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20command%200-8-11-3.png">

可以看出只有CMD0没有Response，其他CMD都有对应的而且是一对一的回应。可以通过command description了解该命令的基本作用。
当然，看这个表格只能雾里看花，如果一个命令只有一个功能还好，如果可以带参数，就不能理解枚举过程中的Check Response等一系列骚操作了。比如CMD5。

#### 2.3 CMD5详解
所以，我把CMD5单独拎出来，一方面是因为CMD5确实在枚举中起的作用最大，另一方面是因为CMD5是对传统SD协议的补充，用来替代ACMD41的。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20CMD5.png">
在这个框图中，就可以对何为S18R、何为OCR。

#### 2.4 R4详解
当然，我们也举CMD5的返回R4，来详细看一看内部结构：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20R4.png">
在这个图中，我们就可以对照枚举流程关注C位、Memory Present位、S18A位、OCR位。


### 3. 数据

枚举完成之后，我们就对SDIO的相关CMD和Response格式有了初步的了解。当然，我们使用SDIO不是为了纯粹的发送指令，否则DAT0~DAT3太浪费了。
对于MMC或者SD这种存储设备来说，不是我们讨论的重点，我们既然枚举成了SDIO IO Only设备，那么就来看一看SDIO 设备是怎么传输数据的。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20Data%20Packet%20format.png">

SDIO设备为了和SD内存卡兼容，SD卡所有Command和Response完全兼容，同时加入了一些新的Command和Response。例如，初始化SD内存卡使用ACMD41，而SDIO卡设备则用CMD5通知DEVICE进行初始化。但二者最重要的区别是，SDIO卡比SD内存卡多了CMD52和CMD53命令，这两个命令可以方便的访问某个功能的某个地址寄存器。

#### 3.1 单Byte数据(CMD52 / R5)

CMD52是由HOST发往DEVICE的，它必须有DEVICE返回来的Response。 **CMD52不需要占用DAT线，读写的数据是通过CMD52或者Response来传送。每次CMD52只能读或者写一个byte．**

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20CMD52.png">

这张图中有几个需要注意的地方：
+ Function Number： 每个SDIO设备都可以有自己的7个Function，分别是F1-F7，其中F0对应的是**CIA(Common I/O area)**, 3Bit刚好可以对应8个Function。
+ Register Address: 选定好一个Function后，就可以通过Register Address定位操作的地址。每个Function有128K个字节，所以Register Address需要17个Bit。
+ Write Data or Stuff Bits: 写命令的时候，就把数据放在这里，1个Byte；读的时候，清零即可。

以及CMD52的Response R5

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20R5.png">

这张图中需要注意的两个地方：
+ Response Flags Bits: Slave以及BUS 状态，详细可以参考上图中的5-1表。
+ Read or Write Data: 如果CMD52是读命令，那么该为装载的是在特定寄存器地址中读出来的值；如果是写命令，则要参考CMD52中的RAW Flag，是否要回读。

#### 3.2 多Byte数据(CMD53)

同CMD52命令不同的是，CMD53没有返回的命令的，这里判断是否DEVICE设备读写完毕是需要驱动里面自己判断的，一般有2个方法，1.设置相应的读写完毕中断。如果DEVICE设备读写完毕，则对HOST设备发送中断。2.HOST设备主动查询DEVICE设备是否读写完毕，可以通过CMD命令是否有返回来判断是否DEVICE是否读写完毕。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20CMD53.png">

这张图中需要注意的地方就多了些：
+ Block Mode: 用来设置传输是否用Block Mode的
  + 当设置为0时： 传输的的单位为Byte，大小为Byte/Block Count的寄存器中获取, 支持的最大数可以从CIS中的TPLFE_MAX_BLK_SIZE中获取；
  + 当设置为1时： 传输的单位为Block，Block的个数在Byte/Block Count的寄存器中获取,每个Block的大小还要根据不同Function来区分：
    + 如果是Function 0，则在CCCR中的FN0 Block Size中获取
    + 如果是Function 1-7，则则从FBR的I/O block size中获取。
  + 因为Block Mode并不是必须要支持的Option，所以Host可以通过CCCR中的Card Supports MBIO bit(SMB)
  
+ OP Code： 用来设置是否从固定地址做读取操作
  + 当设置为0： 普通模式，根据Function Number + Register Address进行读写。对地址0固定读写16个字节，相当于16次读写的地址0.
  + 当设置为1： 递增模式。意思是做完操作后地址直接加1，连续进行读写操作。对地址0增量读写16个字节，相当于读写0~15地址的数据。
  
  
至此，数据传输的部分也了解完了！其实就是CMD52和CMD53两个命令。


  
## 常见寄存器

为什么总感觉少点什么？因为我们在了解CMD53的时候，引入了好几个名词，CIS/CCCC/FBR这些都是什么东西？

如果有过USB驱动的经验可能会容易理解一些，但是没有也没关系。简单的说，SDIO的所有操作其实都是针对的Slave的寄存器的操作，什么寄存器呢？

这一切，我们可以结合CMD53，先从CIA说起:

### 1. CIA(Common I/O Area)

CIA 可以被翻译成通用I/O 区域，它是所有SDIO设备都需要包含的，还记得我们在CMD52/53命令的时候提过的Register Address吗？CIA就是Function 0，它的大小就是128K。
同样的，如果一个SDIO Slave卡设备支持好多Function（最多是Function1-7），那么它也应该有对应的大小区域给每个Function，如下图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20CIA.png">


在0x0000 0000 - 0x0001 FFFF这128K的CIA中，包括了三个单独的寄存器，他们是：
+ CCCR(Card Common Control Register)
+ FBR(Function Basic Register)
+ CIS(Card Information Structure)


#### 1.1 CIS(Card Information Structure)

先说CIS， 原因是CIS里面存储了卡设备的一些通用信息，在初始化完成之后，Host会从这里面获取一些信息，以便知道对面的卡设备的支持功能。

+ 地址寻址范围0x001000~0x017FFF（其中0x0001 8000 - 0x0001 7fff是RFU保留区域）

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20CIS.png">

当然在32767这么大的区域存储的也是由对应的寄存器寻址的。比如我们在CMD53中提到的“支持的最大数可以从CIS中的TPLFE_MAX_BLK_SIZE中获取；”

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20TPLFE_MAX_BLK_SIZE.png">

#### 1.2 CCCR(Card Common Control Register)

话不多述，可以看一下官方的CCCR介绍。我们在介绍CMD53的时候提过“Host可以通过CCCR中的Card Supports MBIO bit(SMB)”判断SDIO Slave是否支持Block Mode

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20CCCR%20SMB.png">



#### 1.3 FBR(Function Basic Register)

我们在CMD53中提过，“如果是Function 1-7，则则从FBR的I/O block size中获取。”

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20SDIO%20FBR%20IO%20block%20size.png">



## 总结

到这里，SDIO的基本概念就入门了，至少可以通过操作CMD和Data了。至于在SDIO总线的Suspend/Resume，Idle/Busy这种状态。还是要在实际工作用遇到问题，去查阅SPEC。这样才能理解得更透彻。

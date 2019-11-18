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

### MMC

熟悉Linux kernel的人都知道，kernel使用MMC subsystem统一管理MMC、SD、SDIO等设备，为什么呢？
从1997年MMC规范发布至今，基于不同的考量（物理尺寸、电压范围、管脚数量、最大容量、数据位宽、clock频率、安全特性、是否支持SPI mode、是否支持DDR mode、等等），进化出了MMC、SD、microSD、SDIO、eMMC等不同的规范（如下面图片1所示）。虽然乱花迷人，其本质终究还是一样的，丝毫未变，这就是Linux kernel将它们统称为MMC的原因。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20MMC-SD-SDIO%E6%BC%94%E8%BF%9B.gif">

从上图可以看出MMC是SD/SDIO/eMMC等一系列的祖师爷，1997年由西门子和闪迪共同开发，技术基于东芝的Nand-Flash。

+ 实物展示

MMC作为一种硬件存储设备所以MMC的接口最为简单，只有7个Pin。大小跟一张油票差不多，约24mm x 32mm x 1.5mm。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20MMC%20%E5%AE%9E%E7%89%A9.jfif">

+ 接口展示

因为只有7个Pin，所以只能支持SPI和1 Bit模式（现在有发展出新的MMC接口和协议，已经可以支持4bit/8bit）

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/SDIO/SDIO%20MMC%20%E7%AE%A1%E8%84%9A%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg">


MMC 的 SPI mode 时钟最高只能到25MHz，因此读取速度通常低于3MB/s。之所以在MMC支持SPI模式，是因为：
+ 1、SPI总线是一个通用总线，大部份芯片都用硬件模块；
+ 2、SPI模式支持不带CRC校验的传输方式，可以降低硬件要求；
+ 3、SD的CMD线与DATA线之间有可能同时产生数据，对没有SD硬件模块的主机支持起来难度较高。

了解了MMC的最初设计初衷以及硬件接口，为了保证兼容性，后面的演进SD/SDIO等都支持SPI模式就会更容易理解一些。

总结一下他们之间的关系：
+ MMC、SD、SDIO的技术本质是一样的（使用相同的总线规范，等等），都是从MMC规范演化而来；
  + MMC强调的是多媒体存储（MM，MultiMedia）；
  + SD强调的是安全和数据保护（S，Secure）；
  + SDIO是从SD演化出来的，强调的是接口（IO，Input/Output），不再关注另一端的具体形态（可以是WIFI设备、Bluetooth设备、GPS等等）。

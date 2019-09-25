# SPI

## 简介

SPI是串行外设接口(Serial Peripheral Interface)的缩写。是 Motorola 公司推出的一种同步串行接口技术，是一种**高速的，全双工，同步**的串行通信总线。

+ 全双工
+ 串行
+ 非差分
+ 主从模式

速率上没有明确的标准，一般从1M ~ 50M bps不等。

## 硬件接口

PIN Name  | Function |  Description  
-|-|-
CS | Chip Select | 片选，对于Slave来说，一般为低有效。 |
MOSI | Master Out Slave In | 主设备数据输出，从设备数据输入 |
MISO | Master In Slave Out | 主设备数据输入，从设备数据输出 |
SCLK | Serial Clock | 时钟信号，主设备输出 |

## 通讯模式

SPI通信有4种不同的模式，不同的从设备可能在出厂是就是配置为某种模式，这是不能改变的；但我们的通信双方必须是工作在同一模式下，所以我们可以对我们的主设备的SPI模式进行配置，通过CPOL（时钟极性）和CPHA（时钟相位）来控制我们主设备的通信模式，具体如下
+ Mode0：CPOL=0，CPHA=0
+ Mode1：CPOL=0，CPHA=1
+ Mode2：CPOL=1，CPHA=0
+ Mode3：CPOL=1，CPHA=1
时钟极性CPOL是用来配置SCLK的电平出于哪种状态时是空闲态或者有效态，时钟相位CPHA是用来配置数据采样是在第几个边沿：
+ CPOL=0，表示当SCLK=0时处于空闲态，所以有效状态就是SCLK处于高电平时
+ CPOL=1，表示当SCLK=1时处于空闲态，所以有效状态就是SCLK处于低电平时
+ CPHA=0，表示数据采样是在第1个边沿，数据发送在第2个边沿
+ CPHA=1，表示数据采样是在第2个边沿，数据发送在第1个边沿

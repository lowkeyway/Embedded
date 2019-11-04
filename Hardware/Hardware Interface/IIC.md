# IIC

## **概述**
I2C(Inter-Integrated Circuit BUS) 集成电路总线，该总线由NXP（原PHILIPS）公司设计，多用于主控制器和从器件间的主从通信，在小数据量场合使用，传输距离短，任意时刻只能有一个主机等特性。
分为物理层（两线结构）和协议层（主机，从机，时钟极性，时钟相位）。

## **物理层**

+ 只要求两条总线线路，一条是串行数据线ＳＤＡ，一条是串行时钟线ＳＣＬ。（IIC是半双工，而不是全双工）。
+ 每个连接到总线的器件都可以通过唯一的地址和其它器件通信，主机/从机角色和地址可配置，主机可以作为主机发送器和主机接收器。
+ IIC是真正的多主机总线，（IIC可以在通讯过程中，改变主机），如果两个或更多的主机同时请求总线，可以通过冲突检测和仲裁防止总线数据被破坏。
+ 传输速率在标准模式下可以达到100kb/s,快速模式下可以达到400kb/s，在IIC 2.0协议中，增加了HS模式，可以达到3.4MBit/s。
+ 连接到总线的IC数量只是受到总线的最大负载电容(400pf)限制。

如下是是IIC设备的物理层的典型连接方式：

<img src='https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%20%E5%85%B8%E5%9E%8B%E6%8E%A5%E6%B3%95.png'>

## **协议层**

IIC的协议层才是掌握IIC的关键。现在简单概括如下：
+ 数据有效性
SDA 线上的数据必须在**时钟的高电平**周期保持稳定数据线的高或低电平状态只有在SCL 线的时钟信号是低电平时才能改变

<img src='https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%20%E6%95%B0%E6%8D%AE%E6%9C%89%E6%95%88%E6%80%A7.png'>


+ 起始和结束条件

在I2C 总线中唯一出现的是被定义为起始S 和停止P 条件。
其中一种情况是在SCL 线是高电平时SDA 线从高电平向低电平切换这个情况表示起始条件；当SCL 是高电平时SDA 线由低电平向高电平切换表示停止条件。
起始和停止条件一般由主机产生总线在起始条件后被认为处于忙的状态在停止条件的某段时间后，总线被认为再次处于空闲状态。

如果产生重复起始Sr 条件而不产生停止条件总线会一直处于忙的状态此时的起始条件S和重复起始Sr 条件在功能上是一样的。

<img src='https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%20%E8%B5%B7%E5%A7%8B%E5%92%8C%E7%BB%88%E6%AD%A2.png'>


+ 应答

每当主机向从机发送完一个字节的数据，主机总是需要等待从机给出一个应答信号，以确认从机是否成功接收到了数据，从机应答主机所需要的时钟仍是主机提供的，应答出现在每一次主机完成8个数据位传输后紧跟着的时钟周期，低电平0表示应答，1表示非应答，如下图所示。：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%20%E5%93%8D%E5%BA%94.png">


+ 数据帧格式

I2C总线上传送的数据信号是广义的，既包括地址信号，又包括真正的数据信号。
在起始信号后必须传送一个从机的地址（7位），第8位是数据的传送方向位（R/T），用“0”表示主机发送数据（T），“1”表示主机接收数据（R）。每次数据传送总是由主机产生的终止信号结束。但是，若主机希望继续占用总线进行新的数据传送，则可以不产生终止信号，马上再次发出起始信号对另一从机进行寻址。

**举个例子**

驱动Slave模块的时候，写从机地址是0x68,因为发送从机地址的时候，要加一位读写方向位。如果是向从机里写入某个寄存器的地址，所以应该是7位地址   0x68(1101000)+二进制位0=11010000。

在总线的一次数据传输过程中，可以有以下几种组合方式：

1. 主机向从机发送数据，数据传送方向在整个传送过程中不变：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%207%E4%BD%8D%E5%9C%B0%E5%9D%80%20%E4%BC%A0%E8%BE%93%E6%96%B9%E5%90%91%E4%B8%8D%E5%8F%98.png">

2. 主机在第一个字节后，立即从从机读数据:
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%207%E4%BD%8D%E5%9C%B0%E5%9D%80%20%E8%AF%BB%E6%95%B0%E6%8D%AE.png">

3. 在传送过程中，当需要改变传送方向时，起始信号和从机地址都被重复产生一次，但两次读/写方向位正好反相：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/IIC%207%E4%BD%8D%E5%9C%B0%E5%9D%80%20%E6%9C%89%E8%AF%BB%E6%9C%89%E5%86%99%E5%A4%8D%E5%90%88%E6%A0%BC%E5%BC%8F.png">

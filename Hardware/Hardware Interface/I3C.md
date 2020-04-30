# 第一章

目前随着手机等移动设备包含的sensor越来越多，传统应用在sensor上的I2C/SPI接口的局限性也越来越明显，典型的缺陷如下：  
1、sensor等设备的增加，对控制总线的速度和功耗提出了更加严苛的要求；  
2、虽然I2C是一种2线接口，但是往往此类device需要额外增加一条中断INT信号线；  

处于解决上述问题的原因，推出了I3C的接口总线和协议，下面一起来看下I3C总线的特性。

## 1. I3C的应用场景

+ I3C总线可以应用在各种sensor中；  
+ 可以使用在任何传统的I2C/SPI/UART等接口的设备中;  

## 2. 什么是I3C

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/I3C/I3C_01%E4%BB%80%E4%B9%88%E6%98%AFI3C.png">

I3C吸纳了I2C和SPI的关键特性，并将其统一起来，同时在I2C的基础上，保留了2线的串行接口结构，这样工程师就可以在单个设备中连接大量的传感器。  


从上图中我们可以将特性具体一下：  
1. I3C总线可以支持multi-master即多主设备;  
2. I3C总线与传统的I2C设备仍然是兼容的;  
3. 可以支持软中断;  
4. 相比较于I2C总线的功耗更低;
5. 速度更快，可以支持到12.5MHZ;  

从下图中可以看到在传统的I2C接口设备中包含了太多的I/0口了(碎片式的接口），将之（I2C/SPI)替换成I3C之后可以节省很大部分的信号线（省去了中断信号的一根线EINT，若取代SPI，可以省的更多）的开销，在布局布线时也更方便.





https://www.cnblogs.com/gcws/p/8995542.html

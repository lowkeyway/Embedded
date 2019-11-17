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

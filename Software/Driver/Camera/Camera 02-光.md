# 光

## 回顾

我们先来理一理Camera成像的基本步骤：

+ 1、景物通过镜头（LENS）生成的光学图像投射到图像传感器(Sensor)表面上；
+ 2、Sensor 将模拟的电信号，经过 A/D（模数转换）转换后变为数字图像信号；
+ 3、图像图像将被送往数字信号处理芯片（DSP/ISP）中加工处理；
+ 4、处理后的数据再通过 IO 接口传输到 CPU 中处理；
+ 5、CPU会将处理后的图像以LCD接受的数据格式最终显示在屏幕上；

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%A8%A1%E7%BB%84%E6%88%90%E5%83%8F%E5%8E%9F%E7%90%86.png">

Camera的世界里，知识浩如烟海，很容易迷失在一个个散落的点里面，没办法梳理成系统的体系，从而影响大对整体的理解。
我有一个想法，把自己想象成一束光，那么我记录下自己穿梭过程中的所见所闻，是不是就能呈现出一个比较完整的视角？

## 我是谁

我是谁？我是一束可见光！也许是直接从主动发光的光源处来，也许是经过了反射而来。

### 光的本质

从爱因斯坦之后，光的波粒二象性就深入人心。在这里，我们只认为光是一种电磁波。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%94%B5%E7%A3%81%E6%B3%A2.png">

### 可见光

从下图可以看出，可见光其实是整个光谱中非常小的一部分。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E5%85%89%E6%98%AF%E4%BB%80%E4%B9%88.jpg">

### 光的一些性质

我们既然把光看做一种特殊的电磁波，那么就可以用波的一些性质来描述它：



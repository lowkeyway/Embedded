# Camera 成像原理

## 1 模组

### 1.1 人眼结构及其成像
都说眼睛是心灵的窗口，可在生物出现后的上亿年间，都生活在没有视觉的世界里。时至今日，眼睛已经进化成几乎完美的存在，而照相机的出现也应该算是鸟飞派的胜利。
我们在了解人类创造的Camera之前，不妨先来看看大自然创造的Camera。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%9C%BC%E7%9D%9B%E7%9A%84%E5%88%A8%E9%9D%A2%E5%9B%BE.png">

比较重要的：
+ 晶状体：可以依靠肌肉的收缩、扩张来控制焦距。
+ 视网膜：成像的温床。传说中的视锥细胞和色杆细胞就是在组成视网膜的一部分。
+ 中央凹：黄斑中央的凹陷称为中央凹，是视力最敏锐的地方，因为他在1.5mm的圆形区域中集中了视网膜中的大部分视锥细胞。
+ 盲点：视神经集中的地方（传输信号给大脑），视锥/视杆细胞无法分布，所以光照在这个地方会是一个盲区。

以及，人眼的成像原理。其实就是是小孔成像
+ 1、光照会通过晶状体的调节，最终把角点落在视网膜上（尤其是中央凹）；
+ 2、视网膜上的视锥细胞和视杆细胞会感受不同的强度和色彩，转换成对应的电讯号；
+ 3、电讯号会被视神经传导到大脑中处理

这样，我们就以为自己看到了图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E4%BA%BA%E7%9C%BC%E6%88%90%E5%83%8F.jpg">

### 1.2 Camera 模组及其成像

先上一张图，看一看手机或嵌入式设备中的Camera模组长什么样子：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%A8%A1%E7%BB%84%E5%AE%9E%E7%89%A9.jpg">

虽然从实物图上看Camera跟人眼大相径庭，但是实际的原理都是一样的。来看看他内部的组成到底是什么样的？

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%A8%A1%E7%BB%84%E7%88%86%E7%82%B8%E5%9B%BE.png">

+ FPC: Flexible Printed Circuit 可挠性印刷电路板，现在很多会把Sensor Driver IC贴在FPC上，它的一端是连接器要扣在主板上，另一端是整颗模组的基座。
+ Sensor:图象传感器，光电传感器，可以把模拟的光信号转换成数字信号。
+ IR:红外滤波片，可以滤掉人眼不可见的红外光等。
+ Holder:镜头的基座，如果支持AF功能，一般会嵌入VCM马达，可以对镜头的位置进行调节。
+ Lens:镜头，玻璃或其他透明材质的凸面镜。

我们也可以看一下实物的散列图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%A8%A1%E7%BB%84%E5%AE%9E%E7%89%A9%E8%A7%A3%E5%88%A8%E5%9B%BE.png">

以及，Camra的成像原理，及其流程：
+ 1、景物通过镜头（LENS）生成的光学图像投射到图像传感器(Sensor)表面上；
+ 2、Sensor 将模拟的电信号，经过 A/D（模数转换）转换后变为数字图像信号；
+ 3、图像图像将被送往数字信号处理芯片（DSP/ISP）中加工处理；
+ 4、处理后的数据再通过 IO 接口传输到 CPU 中处理；
+ 5、CPU会将处理后的图像以LCD接受的数据格式最终显示在屏幕上；

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%A8%A1%E7%BB%84%E6%88%90%E5%83%8F%E5%8E%9F%E7%90%86.png">





# 结构

## 在模组中Camera Sensor

Camera 传感器作为Camera模组中最重要的一部分，它肩负着把光信号转换成电信号的重任，从此之后才正式开启了数字图像世界。</br>
我们先来回顾一下Camera Image Sensor在整个模组中的位置：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20Image%20Sensor%20%E5%9C%A8%E6%A8%A1%E7%BB%84%E4%B8%AD%E7%9A%84%E4%BD%8D%E7%BD%AE.jpg">

虽然我们的AF/AE/AWB系列都讲在了Camera Sensor的前面，但是毫无疑问，这些图像Tunning技术无一不是跟Camera Sensor强相关的，甚至像PDAF、Rolling Shutter这种还必须要对应特殊的Sensor设计。

## Camera Sensor的内部结构

我们知道了Image Sensor在整个模组中的位置，又了解了它应该在整个光路中扮演的角色，那么想要知道它是如何完成这一任务的，就必须要从它的硬件剖析图开始说起：

### CCD的总图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20%E5%9B%BE%E5%83%8F%E4%BC%A0%E6%84%9F%E5%99%A8%E7%BB%93%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg">


### CMOS的总图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20Image%20Sensor%20%E7%BB%93%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg">

从结构上看CCD跟CMOS的内部结构图其实是差不多的，我们这里以比较详细的CMOS为例，List一下Sensor中的各个部件：

+ **A – Colour filter array（彩色滤光阵列）**</br>
它的作用是将光纤过滤成R/G/B三种颜色其中之一的单色光。排列方式如流行的“拜耳阵列(Bayer Array)”。</br>
拜耳阵列模拟人眼对色彩的敏感程度，采用1红2绿1蓝的排列方式（4×4阵列，有8个绿色、4个蓝色和4个红色像素）将灰度信息转换成彩色信息。采用这种技术的传感器实际每个像素仅有一种颜色信息，需要利用反马赛克算法进行插值计算，最终获得一张图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20%E6%8B%9C%E8%80%B3%E9%98%B5%E5%88%97.png">

+ B – Low-pass filter / Anti-aliasing filter（低通/抗混叠滤波片）</br>
跟IR一样，在Sensor内部有时候也可以设计一些低通滤波片，这样可以阻隔更高频的紫外波等，同时也可以保证有些光纤在Sensor内部反射导致的混叠现象；

+ C – Infrared filter (hot mirror)（红外滤波片）</br>
因为光电传感器对红外线是极度敏感的，所以有时候会把红外线滤波片设计在Sensor内部。

+ **D – Circuitry（电路走线）**</br>
既然是数字传感器，就必须要靠集成电路获取最终的数字图像数据。

+ **E – Pixel**</br>
宏观意义上的像素，也可以表示一个光电传感器。它是Camera Image Sensor的核心部件，完成了光->电的转换。

+ **F – Microlenses**</br>
微镜头。同样是凸透镜，从本质上讲跟模组中的Lens可调节焦距的作用是一致的，它的主要作用就是把各个方向的光汇聚到光电传感器方向。因为每个像素都跟它的四邻域颜色不一样，所以这样设计可以有效防止不同颜色的叠加干扰。

+ **G – Black pixels**</br>
NG的像素。一个2M的Camera并不是只有2M个Pixels的，而是必须要多于这个数，那么没有被用到的Pixels就是“黑色像素”。这些像素一般都被跟光隔绝开来，这样可以给Camera Sensor一个没有光照的基准电流，以衡量整颗Sensor的Noise.

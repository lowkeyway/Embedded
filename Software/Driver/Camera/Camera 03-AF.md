# AF

Camera Tunning有三宝：AF/AE/AWB。

我是一束光，那么从进入Camera模组的那一刻，最开始接触的就是Lens，那么我们今天就来看一看AF的相关内容。

## 写在前面

AF（Auto Focusing）是自动对焦的简称。作为9012年的现代人，我相信没有人会对对焦的概念不了解。不过，还是Mark一下。

### 焦距

我们知道，Camera虽然复杂，但是其成像本质跟小孔成像也没有什么区别。当把小孔换成了凸透镜，那么凸透镜的焦距、物距、相聚这些概念总要了解一下。
千言万语不如一张图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%84%A6%E8%B7%9D.jpg">

### 对焦

如上图，如果手机和光源的位置不变，我们可以通过调节凸透镜的位置来达到改变物距和相距的目的。
那么，什么标准下算是对焦成功了呢？

+ 从视觉效果上看，就是Camera对真是世界的输出是清晰的
+ 从原理上说，就是光源中的任意一点，经过凸透镜后都应该又汇聚成一点（上图中的A1点发出的红线和绿线经过凸透镜要在感光Sensor上刚好汇聚成一点A2）

### 景深

当某一物体聚焦清晰时，从该物体前面的某一段距离到其后面的某一段距离内的所有景物也都相当于是清晰的（肉眼不可辨即清晰）。

焦点相当清晰的这段从前到后的距离就叫做景深。可以参考下图理解：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%99%AF%E6%B7%B1.jpg">


## 谁来支持对焦?

从上面的逻辑可以看出来，在光源和手机都不移动的情况下，我们仅仅可以通过移动凸透镜就可以来实现对焦的。那么在设计中的硬件至少要包括两个部分：
+ 凸透镜
+ 马达

### 凸透镜

在单反的世界里，镜头是相机的灵魂，一个镜头上万是很随意的事。它相当于人眼中的晶状体，利用透镜的折射原理，景物光线透过镜头在聚焦平面上形成清晰的像。

#### 为什么选用凸透镜？

凸透镜可以改变光线的传播方向，把相距缩短到很小，这样我们才可以把很远的物体通过一个很小的距离，聚焦到很小的感光材料商。

#### 只要是透光的都可以吗？

理论上是的，从材质可分为我们一般选用下面两种材质：
+ 塑胶透镜(plastic)
+ 玻璃透镜(glass)

玻璃镜片比树脂镜片贵。塑胶透镜其实是树脂镜片，透光率和感光性等光学指标比不上玻璃镜片。

#### 可以用多片凸透镜吗？

Of Course! 而且现在的相机都是采用多片镜片组合的方式。</br>
1P、2P、1G1P、1G2P、2G2P、2G3P、4G、5G等（P:plastic, G:glass ）。透镜越多，成本越高，但是光线会更均匀一些，改善畸变等缺点，相对成像效果会更出色。
比如小米的这颗号称一亿像素的摄像头：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E9%95%9C%E5%A4%B4.png">

### 马达

凸透镜有了，下一步就是考虑如何移动凸透镜的问题了。
在单反的世界里，镜头如此之大，我们可以用手去旋转镜头，以达到对焦之目的，这就是手动对焦。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E5%8D%95%E5%8F%8D%E9%95%9C%E5%A4%B4%20%E6%89%8B%E5%8A%A8%E5%AF%B9%E7%84%A6.jfif">

但是在手机中，Camera模组就这么大，手动拧镜头显然是不可行的，那怎么做到呢？马达就应运而生了。

#### VCM

VCM概述：全称Voice Coil Motor，电子学里面的音圈电机，是马达的一种。因为原理和扬声器类似，所以叫音圈电机，具有高频响、高精度的特点。</br>
其主要原理是在一个永久磁场内，**通过改变马达内线圈的直流电流大小，来控制弹簧片的拉伸位置，从而带动上下运动**。手机摄像头广泛的使用VCM实现自动对焦功能，通过VCM可以调节镜头的位置，呈现清晰的图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20VCM%20%E7%BB%93%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE.png">

#### 自动对焦过程

既然是可以通过改变电流的大小来达到移动镜片位置的目的，那么如何做到自动对焦就容易理解多了，比如介绍一种最原始的自动对焦模式：
+ 进入自动调焦模式后，电流驱动能力从0到最大值，使得镜头从原地移动到最大位移处；
+ sensor成像面自动拍摄图片并保存到DSP内，DSP通过这些图片，计算每一副图片的MTF（Modulation transfer function）值；
+ 在这条MTF曲线中找到最大值，通过算法，得到这个点对应的电流大小；
+ 再一次指示Driver提供给音圈这个电流，而使镜头稳定在这个成像面，使得达到自动变焦；

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20VCM%20%E5%B7%A5%E4%BD%9C%E6%96%B9%E5%BC%8F.png">


## 自动对焦种类

在工业界，照相机的自动对焦方式有很多：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20AF%E6%96%B9%E5%BC%8F%E5%AF%B9%E6%AF%94.png">

</br></br>
在手机领域中，最常见的还是有三种： 反差式、相位检测式、反射时间测距法。

### 反差对焦

反差对焦-CDAF（Contrast Detection Auto Focus）
这是手机中最常见的方法，如在VCM中描述的AF流程就是反差对焦的过程。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E5%8F%8D%E5%B7%AE%E5%BC%8F%E5%AF%B9%E7%84%A6%20%E5%8A%A8%E5%9B%BE.gif">

+ 1 当我们对准被摄物体时，镜头模组内的马达便会驱动镜片从底部向顶部移动;
+ 2 在这个过程中，像素传感器将会对整个场景范围进行纵深方向上的全面检测，并持续记录对比度等反差数值;
+ 3 找出反差最大位置后，运动到顶部的镜片则会重新回到该位置，完成最终的对焦。

</br>
**可以想象，这种方式对对比度很小的大片物体（比如取景中全部都是绿色）的对焦效果很差。**

### 相位检测式

相位对焦——PDAF：它的全称是Phase Detection Auto Focus，字面意思就是“相位检测自动对焦”。
相位检测对焦系统构造相对复杂，并且对光线的要求比较高。需要额外的相位检测装置或者对sensor的pixel进行重新设计。通过分离镜头和线性传感器将图像分离出2个图像，然后通过线性传感器检测出两个图像之间的距离。
</br>
如果要基本原理，却也简单，三步走：

+ **1 分离镜头和线性传感器将图像，分离出2个图像**
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%9B%B8%E4%BD%8D%E5%AF%B9%E7%84%A6%20Sensor%20%E8%AE%BE%E8%AE%A1.png">

</br>
对于Sensor的设计，不同的厂商有不同的方案，比如Sony、Samsung、OV三家的方案：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%9B%B8%E4%BD%8D%E5%AF%B9%E7%84%A6%20%E4%B8%89%E7%A7%8DSensor%E8%AE%BE%E8%AE%A1%E6%96%B9%E5%BC%8F.jpg">
  
```
shield pixel（OPPO R7）
屏蔽掉像素一般的感光区域（黑色部分），值获得一半信号。需要另外的像素屏蔽掉另一半 信号，得到完整的相位差信息。
SP越多，对焦越快，但信号损失越严重，目前SP密度控制在1%~3%。

super PD（OPPO R9S）
将相邻的像素共用一个on chip microlens得到相位差信息，一般在Green上处理。
同样的，二合一的PD越多，对焦越快，但信号损失越严重，目前密度也控制在1%~3%

dual PD（Samsung S7）
将同一个像素底部的感光区域（即光电二极管）一分为二，在同一个像素内即可完成相位信 息捕获。
dual PD 也有叫 2PD、全像素双核对焦，这种像素覆盖率100%，所以对焦体验最佳。
但由于将光电二极管一分为二，井口变小，FWC急剧衰减，dynamic range衰减严重，拍照 非常容易过曝。
```

+ **2 根据两个镜头对同一物体在分离镜头和传感器上的图像，计算相位差**
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%9B%B8%E4%BD%8D%E5%AF%B9%E7%84%A6%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86.png">

</br>
两者相位差最小的时候就是对焦清晰的点

+ **3 根据相位差，查表算出需要如何推动玻璃。（方向和距离）**
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E7%9B%B8%E4%BD%8D%E5%AF%B9%E7%84%A6%20Table%E8%A1%A8.jpg">

</br>
软件计算出两者的相位差，通过查询table表，计算出马达所要到达的理想位置，直接驱动马达，实现快速对焦。

### 反射时间测距法

反射时间测距一般采用激光测距。LDAF（Laser Detection Auto Focus）。
这种对焦方式就很简单了，一般采用计算极光发射和返回的时间差来推算待测物体到手机的距离，从而可以直接算出马达需要推动玻璃的位置。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20TOF%20%E6%B5%8B%E8%B7%9D.jpg">


### 三种自动对焦方式的对比

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E4%B8%89%E7%A7%8D%E5%AF%B9%E7%84%A6%E6%96%B9%E5%BC%8F%E7%9A%84%E5%AF%B9%E6%AF%94.png">


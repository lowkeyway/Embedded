# 什么是AE

AE（Auto Exposure）是自动曝光（调节）的意思。是相机根据外界光线的强弱自动调整曝光量和增益，防止曝光过度或者不足的一种机制。</br>
如果我是一束光，在穿过了Lens后，就要达到感光Sensor上了。那么AE就是就是用来控制有多少光可以抵达Sendor的一种机制。

## 曝光

曝光（英语：Exposure）是指摄影的过程中允许进入镜头照在感光媒体（胶片相机的底片或是数字照相机的图像传感器）上的光量。</br>
要说起精确的曝光定义，1960年的时候美国标准机构ASA就提出了APEX(Additive System of Photographic Exposure)，为了方便计算胶片机的曝光参数，提出的一个经验公式，也称为曝光方程。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%9B%9D%E5%85%89%E6%96%B9%E7%A8%8B.png">

+ **Av**表示光圈大小，其值每增1加个单位，表示进光量减少一半。
+ **Tv**表示快门快慢，其值每增1加个单位，表示进光量减少一半。
+ **Sv**表示相机感度，其值每增1加个单位，表示相机对同样进光量的敏感度增加一倍，体现在照片上就是亮度增加一倍。
+ **Bv**表示环境光的平均亮度，其每增加1个单位，表示环境光平均亮度增加一倍。

从直观上理解曝光公式：当等式右边增加时，表示相机敏感度或环境光增加，为了保持照片同样亮度，需要减少进光量，即减少等式左边的和。同理，当相机敏感度或环境光减少时，需要增加进光量，即增加左边的和。因此，在测出环境光亮度Bv时，根据公式可以计算出合适的曝光参数**光圈、快门、ISO组合**（当然这里有很多组合可以满足等式，至于如何选择有专门的算法和图表可查）

### 曝光的标准

既然曝光意指通光量的大小，那么多少的同光量算是一个正常标准呢？</br>
我们知道物体的亮度与色彩是由物体对光线的反射率来决定的。例如纯黑色的放射率是0，纯白色的反射率是100%，处于中间的灰度的反射率是18%，这就是18%中间灰度。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E8%87%AA%E5%8A%A8%E6%9B%9D%E5%85%89%E7%9A%84%E6%A0%87%E5%87%86.jpg">

具有一定反射率的物体在最终的图像中被还原到了其相应的灰度级，这就意味着达到了正确的曝光。例如摄影师们通常在拍摄之前使用中性灰卡测试曝光是否正常。

有标准，那么就肯定有偏大和偏小之说。简单的说
+ 过曝就是通光量太大了，导致画面中亮度太高照片泛白；
+ 相反，欠曝就是通光量太小了，导致画面亮度太低照片暗淡；

找个对比图就看得明白：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%9B%9D%E5%85%89%E6%AD%A3%E5%B8%B8%E3%80%81%E8%BF%87%E6%9B%9D%E3%80%81%E4%B8%8D%E8%B6%B3%E7%9A%84%E5%AF%B9%E6%AF%94.png">




## 曝光和直方图

从另外一个角度做曝光的定量分析，我想起来从直方图片。直方图分析图片中每个像素的亮度值，并且做统计，可以参考雷神的图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E8%BF%87%E6%9B%9D%E5%92%8C%E6%9B%9D%E5%85%89%E4%B8%8D%E8%B6%B3%E4%B8%8B%E7%9A%84%E7%9B%B4%E6%96%B9%E5%9B%BE.png">

+ 过曝的图，直方图亮度趋于集中数值大的一端；
+ 曝光不足的图，直方图的亮度统计趋于集中数值较小的一端；

PS： 雷神说可以用直方图均衡的方法做后期处理，可AE现在要做的是在前端就把把这种现象扼杀在摇篮中。

## 自动曝光

那么问题来了，一般来说是什么因素导致了这种过曝和欠曝的现象呢？我们又改如何解决呢？
1. “曝光”可以经由光圈，快门和感光媒体的感光度的组合来控制。所以肯定是这三者某一个或者某些因素配合出了问题。
2. 如何解决？AE自动曝光就通过调节这三者之间的关系来解决这个问题的。

自动曝光其实就是一个根据实际图片做反馈来调节到目标亮度的过程。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E8%87%AA%E5%8A%A8%E6%9B%9D%E5%85%89%E6%B5%81%E7%A8%8B.png">

# 支持AE的几个硬件要素



# 支持AE的软件算法


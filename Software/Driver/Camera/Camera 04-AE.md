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

## 光圈（Aperture）

光圈（英语：Aperture）：
+ 硬件：是照相机上用来控制镜头孔径大小的部件，以控制景深、镜头成像素质、以及和快门协同控制进光量。对于已经制造好的镜头，不能随意改变镜头的直径，但是可以通过在镜头内部加入多边形或者圆型，并且面积可变的孔状光栅来达到控制镜头通光量，这个装置就叫做光圈，

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%85%89%E5%9C%88.jpg">

+ 抽象：也可以用来表示光圈值。光学系统中的焦比（英语：f-number，或称F值、F比例、相对孔径、光圈值等，习惯上也简称“光圈”）表达镜头的焦距和光圈直径大小的关系。即公式：

ƒ/"#=N=f/D</br>
**光圈值=镜头的焦距/光圈口径**。
上式中，ƒ 是焦距，而 D 是光圈直径。

特性：**光圈值越小光圈就越大，进光量也越大，景深越浅**。意即D值(光圈有效孔径)越大，N值(光圈值)越小。

### 光圈大小的表示

#### f表示法
我们最常见的光圈大小表示应该是这个样子的， f/# 或直接写成f #:

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%85%89%E5%9C%88f%E5%80%BC%E8%A1%A8%E7%A4%BA%E6%B3%95.jpg">

书写时数值替代符号中的#，即#就代表了焦距和光圈直径大小的比值（而不是光圈直径D了，变成了表示比值N），这里的f可以不理解成公式中的焦距，/可以不用理解成公式中的除号。
举个例子，若焦距值是光圈直径的16倍，那样 ƒ 值是 ƒ/16，或 N = 16。

#### f表示法的等级划分

看多了f表示法，就会看到各种各样的值：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%85%89%E5%9C%88f%E5%80%BC%E8%A1%A8%E7%A4%BA%E6%B3%95%E5%88%86%E7%BA%A7.jpg">


### 光圈和进光量

光圈系数= 镜头焦距/光圈孔径；常用的镜头的光圈数序列为

1， 1.4， 2， 2.8， 4， 5.6， 8， 11， 16， 22， 32， 45， 64，90，128

通过镜头到达感光底片或芯片的光的照度和光圈孔径的平方成正比，和光圈数的平方成反比。光圈数序列中相邻两个光圈数的平方比为 1：2，因此，光圈环转动一格，照度相差一倍：

2x 1.42 =22
2x5.62 = 82
2x82=112

这个很好理解，在曝光时间固定的情况下，光圈越大（f值越小）进光量越大嘛：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%85%89%E5%9C%88%E5%92%8C%E8%BF%9B%E5%85%89%E9%87%8F.jpg">

### 光圈和景深

还记得景深的定义吗？
在焦点前后各有一个容许弥散圆，这两个弥散圆之间的距离就叫焦深，其对应在被摄主体（对焦点）前后，其影像仍然有一段清晰范围的，就是景深。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%20%E6%99%AF%E6%B7%B1.jpg">

从这个图中可以看出来：
+ 当光圈缩小（对应凸透镜的直径变小），弥散圆直径不变时，焦深和景深都变大；
+ 当光圈变大（对应凸透镜的直径变大），弥散圆直径不变时，焦深和景深都变小；

用下图表示：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%85%89%E5%9C%88%E5%92%8C%E6%99%AF%E6%B7%B1.jpg">

其中：
+ 浅景深：指的是照片看上去主体很清晰，背景很模糊，一眼就能看到照片上的主体。
+ 深景深：指的是照片看上去整个画面都是清晰的，没有背景模糊。

所以，人物照适合用浅景深，大光圈；风景照适合用深景深，小光圈；


## 快门（Shutter）

快门也叫光闸，（粤语区有按英文"Shutter"音译成“失打”一词）是照相机中控制曝光时间的重要部件，快门时间越短，曝光时间越少。
在摄影术最初发明的那些年，拍张照片曝光时间一般都需要好几分钟，大部分照相机是不需要快门的，开始曝光的时候把镜头盖取下，然后看表，五分钟后盖上，照片完成。后来，胶片的感光速度越来越快（ISO越来越高），曝光时间变为一分钟，几秒钟，1/10秒甚至几百分之一秒，这时候用手取镜头盖就不够快了。我们需要一个能准确控制曝光时间的东西，这个东西就是快门。

### 衡量标准

如果把相机的Lens当做进光的管子，那么光圈就可以理解成管子的粗细，而快门就相当于阀门。光圈固定的情况下，快门打开的时间越长，进光量就越大！
所以，快门的恒量标准应该是以时间为单位
+ 单反：我们以尼康D5为例看它的规格(1/8000 ~ 30秒)：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%BF%AB%E9%97%A8%20%E6%97%B6%E9%97%B4%E6%A0%87%E5%87%86.png">

+ 手机上：我们找一个对比，比如华为的P30 Pro的专业模式（1/4000 ~ 3S）:

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E6%89%8B%E6%9C%BA%E4%B8%AD%E5%BF%AB%E9%97%A8%E9%80%9F%E5%BA%A6.jpg">

### 工作原理及适用场景

至于如何做到快慢的，可以参考如下动图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%BF%AB%E9%97%A8%20%E5%BF%AB%E3%80%81%E4%B8%AD%E3%80%81%E6%85%A2.gif">

不同的快门时间有一些典型的应用场景，如下图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%BF%AB%E9%97%A8%E7%9A%84%E9%80%82%E7%94%A8%E5%9C%BA%E6%99%AF.jpg">

### 快门的分类

如果我们从大历史观的角度看待这个问题，那么当年蒙在笨重照相机上的黑布都可以被称为“人肉快门”。现如今，我们可以把快门分成机械快门和电子快门两大典型类：

#### **机械快门**

机械快门现在还广泛应用在单反上，如工作原理中展示的图一样，它就是横在Lens和Sensor之间的金属挡板，挡板打开，即打开快门，挡板放下，即关闭快门。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%BF%AB%E9%97%A8%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B.gif">

#### **电子快门**

如果说单反还有很大的体积可以放机械快门，那么手机上这么小的模组是肯定放不下金属挡板的。那难道就没有快门的概念了吗？当然有，怎么实现呢？我们可以在感光Sensor上下功夫。如果感光Sensor是一个像素一个像素排列的点阵结构，那么，我们规定每个像素是否通电，通多长时间的电来接受曝光，这个过程就是电子快门的实现基础。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E6%89%8B%E6%9C%BA%E4%B8%AD%E7%9A%84%E7%94%B5%E5%AD%90%E5%BF%AB%E9%97%A8%20%E5%8D%B7%E5%B8%98%E5%BC%8F.jpg">

如上示意图，每个时刻手机只能得到感光Sensor一列的所有像素点的数据。所以通过逐行扫描的方式得到每一列的数据最后形成一张完整的照片。而这个扫描的时间我们可以认为它就是手机的快门速度了（这就是为什么我们查看手机照片的exif信息时，虽然没有机械快门但是也同样有快门速度的原因）。

需要注意的是，现在的电子快门一般采用“卷帘快门”的形式。什么是“卷帘快门”？

#### **卷帘快门和全局快门**

我们以电子快门为例。从字面上就能理解，简单的说就是：
+ 如果快门不是一次性全部打开（不是所有的感光Sensor一次性全部接受曝光），而是像卷帘一样一条Line一条Line的这样依次进行，就叫做卷帘快门；
+ 如果快门一次性全部打开（所有感光Sensor一次性同时接受曝光），这样就叫做全局快门。

文字总是无力，直接看图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%8D%B7%E5%B8%98%E5%BF%AB%E9%97%A8%E5%92%8C%E5%85%A8%E5%BF%AB%E9%97%A8.gif">

卷帘快门在拍摄高速运动的物体时是由缺陷的，会造成影像的畸变（果冻效应），如何畸变呢？可以参考下图，一个“俄罗斯方块的石头”被切成了条条

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%8D%B7%E5%B8%98%E5%BF%AB%E9%97%A8%E7%9A%84%E7%95%B8%E5%8F%98%20%E6%9E%9C%E5%86%BB%E7%8E%B0%E8%B1%A1%E6%BC%94%E7%A4%BA%E5%9B%BE.gif">

难道科学家们不知道这个问题吗？为什么还要设计成卷帘模式呢？
因为模数转换器是逐行共用，因此只能一行一行地进行转换，也就是说想要实现全局快门，就需要更多地铺设模数转换器来实现更快的读取速度，

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%BF%AB%E9%97%A8%20%E5%8D%B7%E5%B8%98%E7%9A%84%E5%8E%9F%E5%9B%A0.jpg">

全局快门(global shutter)最主要的区别是在每个像素处增加了采样保持单元，在指定时间达到后对数据进行采样然后顺序读出，这样虽然后读出的像素仍然在进行曝光，但存储在采样保持单元中的数据却并未改变。这种结构的主要缺点在于增加了每个像素的元件数目，使得填充系数降低，所以高分辨率的sensor设计难度和生产成本都很高，另外采样保持单元还引入了新的噪音源。

难道就没有办法在电子快门上实现全局曝光了吗？Sony说，他有办法：
如下图是块146万像素堆栈式背照CMOS采用双层结构，上层为1639 X 896像素阵列，下层则是816 X 1792的模数转换器和408个信号中继器阵列，像素与模数转换器数量几乎是1:1！

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E6%89%8B%E6%9C%BA%E5%85%A8%E5%BF%AB%E9%97%A8%E7%9A%84%E5%A0%86%E6%A0%88%E8%AE%BE%E8%AE%A1.jpg">

这种方案的先进之处在于使用了堆栈，以往的全局快门CMOS设计是在成像像素A旁边设计一个存储像素B，A得到的模拟数据就传输给B来短暂保存，等待模数转换器进行逐行转换，期间A可以继续进行拍摄。但问题在于这种比较取巧的方案会浪费大量CMOS面积来摆放存储像素，也没有办法完全利用所有的入射光线，换句话说就是模拟信号先天不足，需要进行数字放大，因此这种设计的信噪比和动态范围都明显更低。

### 果冻效应

如果体积了卷帘快门，就不能不说“果冻效应”

果冻效应在照片和视频拍摄里都会出现，而且机械快门和电子快门都有，胶片相机也躲不掉……照片里的果冻效应最明显的是物体扭曲，示例图如下：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%8D%B7%E5%B8%98%E5%BF%AB%E9%97%A8%E7%9A%84%E6%9E%9C%E5%86%BB%E6%95%88%E5%BA%94.gif">

可以看到扇叶高速旋转时，很容易就拍出右图的扭曲变形效果，形成果冻效应的根本原因在于感光Sensor读取速度太慢.CMOS最上面一排像素的曝光早于最下面一排接近1/250秒，对于高速运转的物体来说，1/250秒已经可以运动很长一段距离了，所以这就是果冻效应的一个根本原因。
感光Sensor需要按行读取的原因又是因为采用了“卷帘快门”的模式。

我们以行进中的小人来模拟这个过程：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E5%8D%B7%E5%B8%98%E5%BF%AB%E9%97%A8%E7%9A%84%E6%9E%9C%E5%86%BB%E6%95%88%E5%BA%94%E7%9A%84%E8%A1%8C%E4%BA%BA%E7%A4%BA%E6%84%8F%E5%9B%BE.gif">


### 手机中怎么实现Shutter改变？

因为Sensor本身并没有时间的概念，它是通过pixel clock数和pixel clock的频率来表示sensor曝光时间的。为了得到简便的表达方式，就用曝光行数表示曝光时间了。曝光行数=pixel clock数/每条line的pixel数。 只需要知道sensor的pixel clock频率和每行的pixel数（有效pixel+dummy pixel），便可以计算出任何曝光时间，sensor需要曝光多少行。

## 感光度（ISO）

感光度，又称为ISO值，是衡量**底片对于光的灵敏程度**，由敏感度测量学及测量数个数值来决定，**ISO的值越大代表对光线越敏感**。
+ 在胶卷时代，ISO代表了胶片对光线的敏感程度；
+ 在数码时代，ISO则代表了感光Sensor对光线的敏感程度；

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20ISO%20%E6%8F%90%E9%AB%98ISO%E7%9A%84%E6%95%88%E6%9E%9C.jpg">

相信你在电视上经常听到什么什么产品经过了ISO9001质量体系认证，但是现在听到相机的感光度叫ISO，不知道是否会认为他们是同一个东西。

### ISO标准

ISO，英文全称：「 International Organization for Standardization 」，是国际标准化组织的英文简称。在胶片时代，胶片感光度被称为 ISO 感光度，人们习惯用 ISO + 数字表示不同底片的感光性能，原则上来说 ISO 后面的数字越大，感光效果越明显。虽然现在来到了电子时代，但这个习惯一直沿用至今。</br>
比如如果用ISO100的胶卷，相机2秒可以正确曝光的话，同样光线条件下用ISO200的胶卷只需要1秒即可，用ISO400则只要0.5秒。 
在数码时代，数码相机的主菜单里都有ISO选择，100，200，400或者800，这和胶卷上的一样。看机型不同，低的到ISO50，最高有到25600的，数字越大越敏感（感光度越高）。

我们知道了ISO后面数字的定性分析是数值越大，对光越敏感，那么它具体是怎么来的呢？Show一张来自国标：GBT 2924-2008的表格：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20ISO%20%E6%84%9F%E5%85%89%E5%BA%A6%E6%A0%87%E5%BA%A6.png">

表中的算数值就是我们常见的ISO 值。

+ 算术感光度值：　　    S= √2 / Hm
+ 算术值对应公式为：　　Hm=（HG x HL）

Hm 表示曝光量; HL 为最慢层曝光量，例如感红层最慢，则 HL = HR(R=Red;B=Blue;G=Green)

从公式中可知，只要把曝光量测试出来，求出了感光度(Hm)，将其代入上述公式就可以得到对应的 ISO 感光度值了。

### ISO和光圈、快门的配合

ISO主要应用是增加对光线敏感度，提高相机的高度曝光性能，可以使画面变得更亮，但必须对信号进行电子放大增幅，在这个过程中也会**产生噪声**，对图片质量有很大影响。所以即便在比较暗的环境中，我们也不能一味地提高ISO来达到图片变亮的目的，可以通过低ISO + 慢快门来解决（其实就是光亮不够，时间来凑），毕竟可以把感光Sensor的成像想象成一个动量的概念。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20ISO%20ISO%E5%92%8C%E5%BF%AB%E9%97%A8%E9%85%8D%E5%90%88.jpg">

### ISO和增益

有一个概念必须澄清一下，因为ISO是由历史包袱的，从胶片的感光材料不同而来，所以有时我们理所当然的套用到现在的感光芯片上，认为相机提高ISO就是提高感光芯片的感光能力（敏感程度），其实不然！与胶片不同，数码相机的「感光能力」或者「对光的敏感度」由影像传感器（芯片中的某一层）决定，**它在出厂封装后便已是固定不变，与ISO值无关**。那数码相机的感光度又是如何计算的呢？这个时候我们需要了解一下增益的概念：

光信号进入镜头后，在数码相机的传感器上发生光电效应后转化为电信号，此时还是一种模拟信号，在进入模数信号转换器（ADC）之前，还要经过电路中的「放大器」，后者用于处理传感器输出的电信号，提高相机的感光度本质上就是放大影像传感器的输出信号。我们看到提高感光度后的图像更明亮了，其实是因为放大了原有信号，而非增加了图像曝光量。数码相机利用不同的放大倍率，提供一种类似于不同胶片感光度的功能。如今的相机基本都能达到几万ISO，甚至可以扩展至几十万ISO。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20ISO%20%E5%A2%9E%E7%9B%8A.jpg">

然而，在放大有用信号的同时，也无可避免地放大了有害的噪声（noise），后者会对图像数据的处理构成干扰。因此，ISO调得越高，照片的噪点也会越多，画质也会变差。

调节亮度增益说白了就是改变ISO，改变CMOS传感器的感光性能，但是会影响到画质。调节曝光补偿则是为了改变快门速度，不改变ISO不会影响画质。



# 支持AE的软件算法

## 流程

先参考AE算法思路和流程：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E6%9B%9D%E5%85%89%E7%AE%97%E6%B3%95%E7%A4%BA%E6%84%8F%E5%9B%BE.png">

可见，AE的输入为当前影像的亮度值Y，输出为sensor的曝光时间和增益，isp增益和镜头光圈（如果镜头光圈可调）。当AE algorithm得到当前帧的亮度后，便会与target Y做比较，然后计算出下一次需要调整的参数，以便让影像的亮度越来越接近target Y，如下所示。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E7%AE%97%E6%B3%95%20Target%20Y.jpg">

## 算法分类及过程

目前比较常见的算法有平均亮度法、权重均值法、亮度直方图等。其中最普遍的就是平均亮度法。
+ 平均亮度法：平均亮度法就是对图像所以像素亮度求平均值，通过不断调整曝光参数最终达到目标亮度；
+ 权重均值法：权重均值法是对图像不同区域设置不同权重来计算图像亮度，例如相机中的各种测光模式的选择就是改变不同区域的权重。
+ 亮度直方图法：亮度直方图法是通过为直方图中峰值分配不同权重来计算图像亮度。


自动曝光实现的过程：

+ 第一步：对当前图像进行亮度统计；
+ 第二步：根据当前图像亮度确定曝光值；
+ 第三步：计算新的曝光参数，曝光时间、光圈、增益；
+ 第四步：将新的曝光参数应用到相机；
+ 第五步：重复步骤一到四，直到亮度满足要求

## AE中遇到的一些问题

### flicker

现象：图像因为sensor曝光时间不是光源频率的整数倍导致图像上出现Banding即明暗相间的条纹。

产生原因：

曝光时间小于1/100秒，且曝光时间处于波峰时，图片亮度比较亮
曝光时间小于1/100秒，且曝光时间处于波谷时，图片亮度比较暗
从能量的角度看，就是当sensor逐像素吸收外界能量成像时，外界能量有时大有时小，像素就有的亮有的暗。因此产生flicker现象。

解决办法：

所以只有曝光时间=光源周期的整数倍的时候，保证每个像素吸收的光能是稳定的。才可以避免flicker。(国内市电，50HZ)

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2004-AE%20%E9%97%AE%E9%A2%98%20Flicker.png">

### 过曝、曝光不足

照片如上面所示。
一般都是AE算法中没有匹配好Shutter/Gain之间的关系。

### AE peak

AE的峰值，图像亮度平稳时，突然出现亮度高的画面。

### AE震荡

图像亮度忽明忽暗。

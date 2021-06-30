# 硬件接口-Camera Driver IC

在了解Camera在Android中的软件结构之前，我们还是要先看一看在实际工程中，Camera的硬件接口。这样有助于我们回顾前面讲的模组相关内容，也便于理解后面的Camera Driver以及HAL的知识。

## 原理图

一个合格的驱动工程师怎么能不会看原理图呢？我们可以以MT6797中Defaule Design中窥视一下Camera的接口：

### **前置13M S5K3**

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E7%A1%AC%E4%BB%B6%E6%8E%A5%E5%8F%A3.png">

### **后置23M OV23850**

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E7%A1%AC%E4%BB%B6%E6%8E%A5%E5%8F%A3%20%E5%90%8E%E7%BD%AE%E6%91%84%E5%83%8F%E5%A4%B4.png">

从上面的原理图，我们就可以总结出AP和Sensor之间的接口PIN：

+ MIPI 数据输出接口：
  + CLK Mipi Clk P/N
  + D0  Mipi Data0 P/N
  + D1  Mipi Data1 P/N(Option)
  + D2  Mipi Data2 P/N(Option)
  + D3  Mipi Data3 P/N(Option)
  
+ POWER 电源控制：
  + AVDD     2.8V, 模拟电，用于感光Sensor区和ADC等；
  + DOVDD    1.8V，IO电，I2C/mipi接口等；
  + DVDD     1.2V, 核心电，用于ISP等；
  + AF_VDD   2.8V, Lens的供电
  
+ CTRL 控制PIN：
  + PWDN     上电/休眠控制
  + RESET    复位
  + MCLK     AP给模组的主时钟（6~27M）
  
+ SCCB(Serial Camera Control Bus)
  + SCL  I2C时钟线
  + SDA  I2C数据线

## 芯片手册

不会看芯片手册的程序员也不是好程序员。我挑选了两款主流厂家的Sensor（OV48B和IMX327），对比一下：

### OV48B

#### 支持特性

我们以OV最新的48M的Sensor IC为例，看看在官方描述中，都会提及它的那些特性：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E7%89%B9%E5%BE%81%E6%94%AF%E6%8C%81%EF%BC%88%E7%AE%80%E6%B4%81%EF%BC%89.png">
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E7%89%B9%E5%BE%81%E6%94%AF%E6%8C%81.png">

注：怎么感觉不是很清晰？可以在[这里](http://www.ovt.com.cn/wp-content/uploads/2019/10/OmniVision_OV48B.pdf)查看官方版本的PDF。

#### 内部框图

最后，放一张芯片内部框图，看看他们是如何配合工作的：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E5%86%85%E9%83%A8%E6%A1%86%E5%9B%BE.png">

### IMX327

#### 支持特性

看看索尼的这一款Sensor描述，比较一下主流的Sensor厂的侧重点是否一样：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E7%89%B9%E5%BE%81%E6%94%AF%E6%8C%81%EF%BC%88IMX327%EF%BC%89.png">

#### 内部框图

以及放一下IMX327的内部框图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20Camera%20Driver%20IC%20%E5%86%85%E9%83%A8%E6%A1%86%E5%9B%BE(Sony%20Imx327).png">

## 初始化

千里之行，始于足下。Camera IC再厉害，也离不开上电、CLK、I2C控制这一套。由上面的图示可以看出来，一般的Camera Sensor有三路电，一路CLK，一套SCCB。如果系统无法识别IC，那么肯定是初始化过程中的这三者遇到了问题，可以逐一排查。

先看上电时序：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E4%B8%8A%E7%94%B5%E6%97%B6%E5%BA%8F.png">

再看整个初始化过程：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Android/Camera%2005-Android%20%E5%88%9D%E5%A7%8B%E5%8C%96%E6%B5%81%E7%A8%8B.png">

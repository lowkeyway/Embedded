# 硬件接口-Camera Driver IC

在了解Camera在Android中的软件结构之前，我们还是要先看一看在实际工程中，Camera的硬件接口。

## 接口分类

常见类型有MIPI、DVP和usb接口：

+ DVP总线：
  + PCLK极限大约在96M左右，而且走线长度不能过长，所有DVP最大速率最好控制在72M以下，故PCB layout会较好画。一般而言，96M pclk是DVP的极限;
  + DVP是并口传输，速度较慢，传输的带宽低，使用需要PCLK\sensor输出时钟、MCLK（XCLK）\外部时钟输入、VSYNC\场同步、HSYNC\行同步、D[0：11]\并口数据——可以是8/10/12bit数据位数大小;
    + PCLK：像素点同步时钟信号，每个PCLK对应一个像素点，可以为48MHz；对于时钟信号，一般做包地处理，减少对其他信号的干扰，还需要在源端加电阻和电容，减少过冲和振铃，从而减少对其他信号的干扰。
    + MCLK（XCLK）：外部时钟输入，可由主控或晶振提供，由sensor规格书确定，可以为24MHZ；
    + VSYNC：帧同步信号，一帧一个信号，频率为几十Hz（30Hz）
    + HSYNC：行同步信号（频率为几十KHz）
    + 例如：分别率 320×240的屏，每一行需要输入320个脉冲来依次移位、锁存这一行的数据，然后来个HSYNC 脉冲换一行；这样依次输入240行之后换行同时来个VSYNC脉冲把行计数器清零，又重新从第一行开始刷新显示。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2011-DVP.jpg">

+ MIPI总线：
  + 速率随便就几百M，而且是lvds接口耦合，走线必须差分等长，并且注意保护，故对PCB走线以及阻抗控制要求高一点;
  + MIPI是LVDS（Low Voltage Differential Signaling，低电压差分信号），低压差分串口。只需要要CLKP/N、DATAP/N——最大支持4-lane;

MIPI接口比DVP的接口信号线少，由于是低压差分信号，产生的干扰小，抗干扰能力也强。最重要的是DVP接口在信号完整性方面受限制，速率也受限制。500W还可以勉强用DVP，800W及以上都采用MIPI接口。

** 注：下面我们只讲MIPI接口总线的。 **

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

## 帧同步

在DVP接口中，有HSYNC和VSYNC以及PCLK，AP和Camera之间保证了同步。那么，对于MIPI接口呢?
在camera领域人们所说的MIPI接口一般是指MIPI CSI-2规范，该规范使用长、短两种封包格式，其中长包用来传输图像数据，短包用来传输HSYNC,VSYNC等控制信号。

### 帧结构

Sensor的实际像点数量通常比标准的图像分辨率（如2048x1536，1920x1080等）要大一些，多出来的像点主要有两种作用，

+ 黑像素，即dark pixel，像点上方覆盖的是不透光的金属，相当于零输入的情况，用于检测像素的暗电流水平
+ 滤波像素，即filter pixel，很多ISP算法会使用3x3或5x5大小的滤波窗口，因此需要在输出分辨率的基础上增加若干行、列使滤波窗口内全是有效像素。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2011-%E5%B8%A7%E7%BB%93%E6%9E%84.sensor.png">

Sensor 的输出信号中除了像素数据之外，还需要一些控制信号用于时间同步。每一帧开始时会有一个FrameStart 同步信号，在一帧内会用HSYNC信号指示哪些时钟周期携带了有效像素，以及会用VSYNC信号指示哪些行中携带了有效像素。未携带有效像素的时钟周期称为消隐周期，又分为水平消隐（horizontal blanking）和垂直消隐（vertical blanking），如下图所示。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2011-%E5%B8%A7%E7%BB%93%E6%9E%84.Signal.png">  

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2011-%E5%B8%A7%E7%BB%93%E6%9E%84.Data.png">

以上图为例，sensor输出的每一行由以下单元组成

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2011-%E5%B8%A7%E7%BB%93%E6%9E%84.Line.png">

而sensor输出的每一帧图像由以下行组成

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Camera%2011-%E5%B8%A7%E7%BB%93%E6%9E%84.Frame.png">


Sensor 处理每一行都需要若干个水平消隐周期(Porch)用于处理一些内部逻辑，在此基础上用户也可以增加消隐周期用于调节每行的曝光时间，起到间接调整帧率的作用。同理，sensor 每帧都需要若干个垂直消隐周期用于为下一帧做准备，在此基础上用户也可以增加一些消隐周期用于调整帧率。

因此，当sensor 主频确定后，垂直消隐的行数是用户控制sensor 输出帧率的主要手段，而水平消隐的周期数是调整帧率的辅助手段。尽管有两个参数都可以起到调整帧率的作用，但为了简单起见，人们往往以SONY 厂家推荐的水平消隐参数作为参考标准，使sensor 每一行的行长（时钟周期数）与SONY 同类型号保持一致，在此基础上继续调节垂直消隐，达到用户需要的帧率。

需要注意的是，当用户配置的积分时间（行数）参数大于一帧的有效行数时，sensor 会把超出的行数自动视作额外的垂直消隐，因此会在两帧之间插入额外的延时，导致sensor 输出帧率下降。

再举一个OV7725的例子，下图是OV7725的VGA时序，其特点是:

+ 每一帧开始时同步信号VSYNC有效并持续4行时间（4 tline），
+ 每一行开始时同步信号HSYNC有效并持续64个tp （1tp=2PCLK），
+ 在HREF为高电平时每一行有效时间为640tp，无效时间为144tp，每一行时间为tline=784tp；
+ 每一帧总行数是510，其中有效行数是480
+ 采集一帧数据的时间是784*510tp。


### 数据帧格式

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/camera%2011-%E6%95%B0%E6%8D%AE%E6%B5%81.shortpackage.png">

字节(byte)为基本传输单元，每个byte中有8位(bit)

+ SYNC BYTE：用来同步数据开始，告知接下来为有效数据

+ DATA TYPE：该包传输的是什么格式的数据

  + YUV422, (1E)
  + RAW8, (2A)
  + RAW10, (2B)


+ WORD COUNT： 16bits, PAYLOAD中的byte数量（即输出窗口的1行中有多少个字节，也即列数。注意raw10为列数的1.25倍，raw12为列数的1.5倍）

+ ECC：校验datatype和wc是否出错

+ PAYLOAD : image data

+ CSC：PAYLOAD数据传输校验

*由于插入了许多数据标识，所以会影响hb或者vb的最小值

通过D-Phy数据通道发送的载荷数据采用的是数据包的格式。它可以是长的数据包，也可以是短的数据包。长数据包包含32位的包头、有效载荷数据和16位的数据包脚注。短数据包只包含32位的包头。

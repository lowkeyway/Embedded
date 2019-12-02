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
跟IR一样，在Sensor内部有时候也可以设计一些低通滤波片，这样可以阻隔更高频的紫外波等，同时也可以保证有些光线在Sensor内部反射导致的混叠现象；

+ C – Infrared filter (hot mirror)（红外滤波片）</br>
因为光电传感器对红外线是极度敏感的，所以有时候会把红外线滤波片设计在Sensor内部。

+ **D – Circuitry（电路走线）**</br>
既然是数字传感器，就必须要靠集成电路获取最终的数字图像数据。

+ **E – Pixel（像素）**</br>
宏观意义上的像素，也可以表示一个光电传感器。它是Camera Image Sensor的核心部件，完成了光->电的转换。

+ **F – Microlenses（微镜头）**</br>
微镜头。同样是凸透镜，从本质上讲跟模组中的Lens可调节焦距的作用是一致的，它的主要作用就是把各个方向的光汇聚到光电传感器方向。因为每个像素都跟它的四邻域颜色不一样，所以这样设计可以有效防止不同颜色的叠加干扰。

+ **G – Black pixels**</br>
NG的像素。一个2M的Camera并不是只有2M个Pixels的，而是必须要多于这个数，那么没有被用到的Pixels就是“黑色像素”。这些像素一般都被跟光隔绝开来，这样可以给Camera Sensor一个没有光照的基准电流，以衡量整颗Sensor的Noise.


# Sensor分类

我们在深入到光电传感器以及电路设计之前，有个概念是总也绕不过去，那就是CCD和CMOS传感器之分。但是不管是CCD还是CMOS，他们只是技术实现上的不同。我们提纲挈领抽象一下，抛开具体的内部构造，看看Sensor是做的什么工作？

## Sensor的本质

我们知道了Camera Image Sensor其实就是光电转换的过程，多强的光我就要对应给多少电荷，再由专门的放大器和模数转换电路采样、量化成每个Pixel对应的DN值。这个值就可以通过CPU加工、运算、输出了。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20%E5%85%89%E5%BC%BA%E5%88%B0%E7%94%B5%E5%BC%BA%E7%9A%84%E8%BD%AC%E6%8D%A2.jpg">

那么针对如何转换光电和如何导出电荷这门技术就可以继续细分了。

## CCD 和 CMOS

在分开介绍CCD和CMOS之前，我觉得有必要知道CCD和CMOS到底区分的是什么？
CCD与CMOS传感器是当前最被普遍采用的两种影像感测组件，基本上两者都是利用感光二极管（photodiode）进行光与电转换，将影像转换为数字数据，而其主要差异则**在数字数据传送方式的不同**。所以从宏观上看，我们可以直接把整个Camera Sensor分成CCD和CMOS两大类了。

### CCD

CCD (Charge Couple Device)：电荷耦合器件，矽晶半导体，类似太阳能电池，透过光电效应，转化为存储电荷；是一种集成电路，上有许多排列整齐的电容，能感应光线，并将影像转变成数字信号。经由外部电路的控制，每个小电容能将其所带的电荷转给它相邻的电容。

#### 实物图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CCD.jpg">

#### 工作框图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CCD%20%E4%BC%A0%E6%84%9F%E5%99%A8%E9%98%B5%E5%8E%9F%E7%90%86%E7%A4%BA%E6%84%8F%E5%9B%BE.png">

电荷耦合组件（CCD：Charge Coupled Device）是一种推电荷（电子）前进的组件，利用 3 个金属电极不同电压依序推电荷前进，如图 （b）所示，
+ 像素 D 的光传感器内的电子经由旁边的 CCD 组件由 4 向上推到 3;
+ 像素 C 的电子由 3 推到 2;
+ 像素 B 的电子由 2 推到 1;
+ 像素 A 的电子由 1 推到水平线;

依此类推，第一行的电子推完，再推第二行，再推第三行，依此类推，必须把影像传感器内每一个像素的电子依序推到水平线，经由“模拟前端”（AFE：Analog Front End）将模拟讯号转换成数字讯号，也就是影像的“模拟数字转换器”（ADC：Analog to Digital Converter），再输入处理器（Processor）进行数字讯号处理。

#### 工作原理

在一个用于感光的CCD中，有一个光敏区域（硅的外延层），和一个由移位寄存器制成的传感区域（狭义上的CCD）。

+ 图像通过透镜投影在一列电容上（光敏区域），导致每一个电容都积累一定的电荷，而电荷的数量则正比于该处的入射光强；
+ 一旦电容阵列曝光，一个控制回路将会使每个电容把自己的电荷传给相邻的下一个电容（传感区域）；
+ 阵列中最后一个电容里的电荷，则将传给一个电荷放大器，并被转化为电压信号；
+ 通过重复这个过程，控制回路可以把整个阵列中的电荷转化为一系列的电压信号。在数字电路中，会将这些信号采样、数字化，通常会存储起来；

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CCD%20%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.gif">

在栅电极（G）中，施加正电压会产生势阱（黄），并把电荷包（电子，蓝）收集于其中。只需按正确的顺序施加正电压，就可以传导电荷包。

#### 总结

CCD元件电荷转移也就是数据读取的方法：由下图示可以看出，CCD是全部传输出来再统一处理，CCD传感器中每一行中每一个象素的电荷数据都会依次传送到下一个象素中，由最底端部分输出，再经由传感器边缘的放大器进行放大输出。这是由于在构造上，CCD可以充分保证电荷信号在传送时不会失真，每个像素可以集合至单一放大器统一处理。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CCD%20%E9%98%B5%E5%88%97%E8%AF%BB%E5%8F%96%E7%94%B5%E5%AE%B9%E5%80%BC.gif">

### CMOS

CMOS (Complementary Metal Oxide Semiconductor)：互补金属氧化物半导体，由硅和锗这两种元素做成的半导体，使其在CMOS上共存着带N(-)和P(+)级的半导体 。对于CMOS这个名词来说，学习过数电的同学们肯定都不陌生，我们今天主要了解一下，CMOS在Camera Image Sensor中的应用：

#### 实物图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS.jpg">

#### 工作框图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20%E5%B7%A5%E4%BD%9C%E6%A1%86%E5%9B%BE.png">

跟CCD一样，每个像素包括感光区和读出电路，不同的是每个像素的信号经由模拟信号处理后，交由ADC进行模数转换后即可输出到数字处理模块。像素阵列的信号读出如下：

+ 每个像素在进行reset，进行曝光。
+ 行扫描寄存器，一行一行的激活像素阵列中的行选址晶体管。
+ 列扫描寄存器，对于每一行像素，一个个的激活像素的列选址晶体管。
+ 读出信号，并进行放大。

注：PD：photodiode

#### 工作原理

我们希望列出来单个Pixel的工作原理，先看看它的等效电路图：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20%E5%8D%95%E5%83%8F%E7%B4%A0%E7%AD%89%E6%95%88%E7%94%B5%E8%B7%AF.jpg">

从CMOS的等效电路图可以看出来，CMOS的工作原理比CCD要简单很多，即：单个像素的光电传感器的电荷值经过CMOS开关直接传出来即可。单即便是这个简单的过程中个，工程师们也设计出来很多措施不断改善、演进，简单的分类是无源型和有源型：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20Active%26Passive.png">

##### Passive Pixel Sensor（无源像素型（PPS））
简单的说，就是一个PN节作为photodiode感光，以及一个与它相连的reset晶体管作为一个开关。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20PPS%20%E6%97%A0%E6%BA%90%E5%83%8F%E7%B4%A0.png">

它的工作原理：
+ 1、在开始曝光之前，该像素的行选择地址会上电（图中未画出），从而RS会激活，连通PN结与column bus。同时列选择器会上电，此时PN结会被加载高反向电压（例如3.3 V）。在Reset（即PN结内电子空穴对达到平衡）完成后，RS将会被停止激活，停止PN结与column bus的连通。
+ 2、在曝光时间内，PN结内的硅在吸收光线后，会产生电子-空穴对。由于PN结内电场的影响，电子-空穴对会分成两个电荷载体，电子会流向PN结的n+n+端，空穴会流向PN结的p-substrate。因此，经过曝光后的的PN结，其反向电压会降低。
+ 3、在曝光结束后，RS会被再次激活，读出电路会测量PN结内的电压，该电压与原反向电压之间的差，就是PN结接受到的光信号。（在主流sensor设计中，电压差与光强成正比关系）
+ 4、在读出感光信号后，会对PN结进行再次reset，准备下次曝光。

这种像素结构，其读出电路完全位于像素外面，称为Passive Pixel。Passive Pixel的读出电路简单，整个Pixel的面积可以大部分用于构造PN结，所以其满阱电容一般会高于其他结构。但是，由于其信号的读出电路位于Pixel外面，它受到电路噪声的影响比Active Pixel大。Passivel Pixel噪声较大有2个主要原因：

+ 1、相对读出电路上的寄生电容，PN结的电容相对较小。代表其信号的电压差相对较小，这导致其对电路噪声很敏感。
+ 2、PN结的信号，先经过读出电路，才进行放大。这种情况，注入到读出信号的噪声会随着信号一起放大。


##### Active Pixel Sensor（无源像素型（APS））

Active Pixel指的是在像素内部有信号读出电路和放大电路的像素结构。如对比图(a)，信号传出Pixel之前，就已经读出并放大，这减少了读出信号对噪声的敏感性。随着工艺的发展，基于Active Pixel的CMOS传感器在暗电流和噪声表现上有很大提升，Active Pixel结构随之成为了CMOS传感器的主流设计。
如今的APS设计中，最常用的就是基于PPD结构（Pinned Photodiode Pixel）的像素结构。PPD pixel包括一个PPD的感光区，以及4个晶体管，所以也称为**4T像素结构**。PPD的出现，是CMOS性能的巨大突破，它允许相关双采样（CDS）电路的引入，消除了复位引入的kTC噪声，运放器引入的1/f噪声和offset噪声。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20PPPD%20%E6%9C%89%E6%BA%90%E5%83%8F%E7%B4%A0.png">


对于PPD，右边部分电路只是信号读出电路。读出电路与光电转换结构通过TX完全隔开，这样可以将光感区的设计和读出电路完全隔离开，有利于各种信号处理电路的引入（如CDS，DDS等）。另外，PPD感光区的设计采用的是p-n-p结构，减小了暗电流。PPD像素的工作方式如下：
+ 1、 曝光。光照射产生的电子-空穴对会因PPD电场的存在而分开，电子移向n区，空穴移向p区。
+ 2、 复位。在曝光结束时，激活RST，将读出区（n+区）复位到高电平。
+ 3、 复位电平读出。复位完成后，读出复位电平，其中包含运放的offset噪声，1/f噪声以及复位引入的kTC噪声，将读出的信号存储在第一个电容中。
+ 4、 电荷转移。激活TX，将电荷从感光区完全转移到n+区用于读出，这里的机制类似于CCD中的电荷转移。
+ 5、 信号电平读出。接下来，将n+n+区的电压信号读出到第二个电容。这里的信号包括：光电转换产生的信号，运放产生的offset，1/f噪声以及复位引入的kTC噪声
+ 6、 信号输出。将存储在两个电容中的信号相减（如采用CDS，即可消除Pixel中的主要噪声），得到的信号在经过模拟放大，然后经过ADC采样，即可进行数字化信号输出。

但是需要指出的是：由于PPD像素结构在暗电流和噪声方面的优异表现，近年来市面上的CMOS传感器都是以PPD结构为主。但是，PPD结构有4个晶体管，有的设计甚至有5个，这大大降低了像素的填充因子（即感光区占整个像素面积的比值），这会影响传感器的光电转换效率，进而影响传感器的噪声表现。

#### 总结

跟CCD的先做电子转移，后处理不同的是，CMOS的电子转电压及缓冲放大等作用都是在成像单元内完成的（PPD）。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CMOS%20%E9%98%B5%E5%88%97%E8%AF%BB%E5%8F%96%E7%94%B5%E5%AE%B9%E5%80%BC.gif">


### CCD和CMOS的一些差别

先回顾一下两者在结构上的对比：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CCD%26CMOS%20%E7%BB%93%E6%9E%84%E5%8C%BA%E5%88%AB.gif">

由于数据传送方式不同，因此CCD与CMOS传感器在效能与应用上亦有诸多差异，这些差异包括灵敏度、成本、分辨率、噪声与耗电量等。

+ **灵敏度差异**

由于CMOS传感器每个画素由四个晶体管与一个感光二极管所构成（含放大器与A/D转换电路），使得每个画素的感光区域远小于画素本身的表面积，因此在画素尺寸（pixel size）相同的情况下，CMOS传感器的灵敏度会低于CCD传感器。


+ **成本差异**

由于CMOS传感器采用一般半导体电路最常用的CMOS制程，可以轻易地将周边电路（如AGC、CDS、Timing generator或DSP等）整合到传感器芯片中，因此可以节省外围芯片所需负担的成本；而CCD由于采用电荷传递的方式传送数据，只要其中有一个画素不能运作，就会导致一整排的数据不能传送，因此控制CCD传感器的良率比CMOS传感器更加困难，即使有经验的厂商也很难在产品问世的半年内就突破50％的水平，因此通常CCD传感器的成本会高于CMOS传感器。


+ **分辨率差异**

如上所述，CMOS传感器的每个画素都比CCD传感器更加复杂，其pixel size很难达到CCD传感器的水平，因此，当我们比较相同尺寸的CCD与CMOS传感器时，CCD传感器的分辨率通常会优于CMOS传感器的水平。举例来说，目前市面上210万画素的水平（OmniVision的OV2610，2002/6推出），其尺寸为1/2吋，pixel size为4.25微米，但Sony在2002/12推出的ICX452，其尺寸与OV2610相差不多（1/1.8吋），但分辨率却能高达513万画素，pixel size亦只有2.78微米的水平。

+ **噪声差异**

由于CMOS传感器每个感光二极管都需搭配一个放大器，而放大器属于模拟电路，很难让每个放大器所得到的结果维持一致性，因此与只有一个放大器放在芯片边缘的CCD传感器比较之下，CMOS传感器的噪声就会增加很多，影响影像质量。


+ **耗电量差异**

CMOS传感器的影像撷取方式为主动式，感光二极管所产生的电荷会直接由旁边的晶体管放大输出，但CCD传感器则为被动式撷取，需外加电压让每个画素中的电荷移动，而这外加电压通常需要**12～18V**的水平；因此，CCD传感器除了在电源管理线路设计上的难度更高之外（需外加power IC），高驱动电压更使其耗电量远高于CMOS传感器。


综合以上分析，CCD传感器在灵敏度、分辨率、以及噪声控制等方面均优于CMOS传感器，而CMOS传感器则具有低成本、低耗电以及高整合度的特性。不过，随着CCD与CMOS传感器技术的进步，两者的差异似乎有逐渐缩小的态势，例如CCD传感器持续在耗电量上作改进，以期应用于行动通讯市场（这方面的代表业者为Sanyo）；CMOS传感器则持续改善分辨率与灵敏度的不足，以期应用于更高阶的影像产品市场；

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/Sensor/Camera%2006-Sensor%20CCD%26CMOS%20%E6%80%A7%E8%83%BD%E5%8C%BA%E5%88%AB.png">






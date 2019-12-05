# 什么是ISP？

ISP(Image Signal Processor)，即图像处理，主要作用是对前端图像传感器输出的信号做后期处理，主要功能有线性纠正、噪声去除、坏点去除、内插、白平衡、自动曝光控制等，依赖于ISP才能在不同的光学条件下都能较好的还原现场细节，ISP技术在很大程度上决定了摄像机的成像质量。</br>
光线始于Lens，终于Sensor，CIS之后再无模拟。光线被Sensor转换为电讯号之后，经过ADC的采样、量化，最终形成了我们熟悉的01世界的数字图像。数字图像的下一个加工厂就是ISP了！

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ARM_ISP.png">

看ISP的直译就知道，它也是一个处理器，所以根据它的位置，我们可以把它分为集成式和分离式：

## 分离式

分离式也称为外置ISP架构，这里的分离和外置的对象都是主处理器AP。所以，只要ISP没有被集成在AP内部的都可以成为分离式。
在Camera分辨率还没有1M的年代，这个量级的处理量Camera Driver IC厂商有能力研发自己的ISP，他们可以把这部分集成在自己的Driver IC中；但是现如今的Camera动不动就要上亿了，总有些专业的数字图像处理的公司接管了这部分业务，有了自己的ISP（比如：socionext，arm和ov），所以我们现如今说的分离式的ISP都是这种在AP外下挂一个专业ISP的形式：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E5%A4%96%E7%BD%AEISP.png">

## 集成式

集成的就很容易理解了，现如今高通、苹果、三星、海思、MTK主流的手机AP供应商都会把ISP集成到他们的芯片内部，这样可以给用户提供更便捷的调试体验（算法平台一手承包，真是工程师们的福音），还节约了成本，节约了宝贵的PCB空间资源。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E5%86%85%E7%BD%AEISP.png">

# ISP的内部架构

如下图所示，ISP内部包含 CPU、SUP IP、IF 等设备，事实上，可以认为 ISP 是一个 SOC（system of chip），可以运行各种算法程序，实时处理图像信号。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E7%BB%93%E6%9E%84.png">

+ **CPU**

CPU 即中央处理器，可以运行 AF、LSC 等各种图像处理算法，控制外围设备。现代的 ISP 内部的 CPU 一般都是 ARM Cortex-A 系列的，例如 Cortex-A5、Cortex-A7。

+ **SUB IP**

SUB IP 是各种功能模块的通称，对图像进行各自专业的处理。常见的 SUB IP 如 DIS(Digital Image Stabilization)、CSC(Color Space Conversion)、VRA(Vehicle Routing Algorithm) 等。

+ **图像传输接口**

图像传输接口主要分两种，并口 ITU(International Telecommunication Union) 和串口 CSI(Camera Serial Interface)。CSI 是 MIPI CSI 的简称，鉴于 MIPI CSI 的诸多优点，在手机相机领域，已经广泛使用 MIPI-CSI 接口传输图像数据和各种自定义数据。

+ **通用外围设备**

通用外围设备指 I2C、SPI、PWM、UART、WATCHDOG 等。ISP 中包含 I2C 控制器，用于读取 OTP 信息，控制 VCM 等。对于外置 ISP，ISP 本身还是 I2C 从设备。AP 可以通过 I2C 控制 ISP 的工作模式，获取其工作状态等。


# ISP的控制结构

控制结构可以从三方面看，包括： 
+ 1、ISP前端逻辑    
+ 2、ISP内部逻辑
+ 3、ISP后端逻辑

## 前端逻辑（与Sensor/Lens的关系）

如图所示，lens 将光信号投射到sensor 的感光区域后，sensor 经过光电转换，将Bayer 格式的原始图像送给ISP，ISP 经过算法处理，输出RGB空间域的图像给后端的视频采集单元。在这个过程中，ISP通过运行在其上的firmware（固件）对ISP逻辑，从而对lens 和sensor 进行相应控制，进而完成自动光圈、自动曝光、自动白平衡等功能。其中，firmware的运转靠视频采集单元的中断驱动。PQ Tools 工具通过网口或者串口完成对ISP 的在线图像质量调节。

ISP 由ISP逻辑及运行在其上的Firmware组成，逻辑单元除了完成一部分算法处理外，还可以统计出当前图像的实时信息。Firmware 通过获取ISP 逻辑的图像统计信息，重新计算，反馈控制lens、sensor 和ISP 逻辑，以达到自动调节图像质量的目的。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E6%8E%A7%E5%88%B6%E6%9E%B6%E6%9E%84.png">

## 内部逻辑（Firmware行为）

ISP 的Firmware包含三部分：
+ ISP 控制单元和基础算法库；
+ AE/AWB/AF 算法库；
+ sensor 库；

Firmware 设计的基本思想是单独提供3A算法库，由ISP控制单元调度基础算法库和3A 算法库，同时sensor 库分别向ISP 基础算法库和3A 算法库注册函数回调，以实现差异化的sensor 适配。ISP firmware 架构如图所示。

不同的sensor 都以回调函数的形式，向ISP 算法库注册控制函数。ISP 控制单元调度基础算法库和3A 算法库时，将通过这些回调函数获取初始化参数，并控制sensor，如调节曝光时间、模拟增益、数字增益，控制lens 步进聚焦或旋转光圈等。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20FW%20ALG.png">


## 后端逻辑（与AP的关系）

CPU处理器包括：AP、BP、CP。 BP：基带处理器、AP：应用处理器、CP：多媒体加速器

这里所说的控制方式是AP 对 ISP 的操控方式 。

+ **I2C/SPI**：这一般是外置 ISP 的做法。SPI 一般用于下载固件、I2C 一般用于寄存器控制。在内核的 ISP 驱动中，外置 ISP 一般是实现为 I2C 设备，然后封装成 V4L2-SUBDEV。
+ **MEM MAP**：这一般是内置 ISP 的做法。将 ISP 内部的寄存器地址空间映射到内核地址空间，
+ **MEM SHARE**：这也是内置 ISP 的做法。AP 这边分配内存，然后将内存地址传给 ISP，二者实际上共享同一块内存。因此 AP 对这段共享内存的操作会实时反馈到 ISP 端。


# ISP能做什么

不同厂商的ISP肯定会支持各种各样的功能，这也可以在网络上搜索到的各式各样的框图中略见一斑。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B1.gif">
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B2.jpg">
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B4.png">
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B5.jpeg">
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B6.png">

ISP的功能比较杂，基本上跟图像效果有关的它都有份。它内部包含多个图像算法处理模块，其中比较有代表性的是：扣暗电流（去掉底电流噪声），线性化（解决数据非线性问题），shading（解决镜头带来的亮度衰减与颜色变化），去坏点（去掉sensor中坏点数据），去噪（去除噪声），demosaic（raw数据转为RGB数据），3A（自动白平衡，自动对焦，自动曝光），gamma（亮度映射曲线，优化局部与整体对比度），旋转（角度变化），锐化（调整锐度），缩放（放大缩小），色彩空间转换（转换到不同色彩空间进处理），颜色增强（可选，调整颜色），肤色增强（可选，优化肤色表现）等。
实际情况下，不同芯片的ISP，其处理流程和模块可能会稍有不同，但是其原理、实现功能都是一样的。我还是列出来在网上搜的一些名词解析，供参考：


+ **DEMOSAIC**

DEMOSAIC 是 ISP 的主要功能之一。SENSOR 的像素点上覆盖着 CFA，光线通过 CFA 后照射到像素上。CFA 由 R、G、B 三种颜色的遮光罩组成，每种遮光罩只允许一种颜色通过，因此每个像素输出的信号只包含 R、G、B 三者中的一种颜色信息。SENSOR 输出的这种数据就是 BAYER 数据，即通常所说的 RAW 数据。显而易见，RAW 数据所反映的颜色信息不是真实的颜色信息。DEMOSAIC 就是通过插值算法将将每个像素所代表的真实颜色计算出来。

+ **FOCUS**

根据光学知识，景物在传感器上成像最清晰时处于合焦平面上。通过更改 LENS 的位置，使得景物在传感器上清晰的成像，是 ISP FOCUS 功能所需要完成的任务。FOCUS 分为手动和自动两种模式。ISP 可以运行 CONTRAST AF、PDAF、LASER AF 等算法实现自动对焦。

+ **EXPOSURE**

曝光。EXPOSURE 主要影响图像的明暗程度。ISP 需要实现 AE 功能，通过控制曝光程度，使得图像亮度适宜。
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%2015.AE%EF%BC%88Automatic%20Exposure%EF%BC%89----%E8%87%AA%E5%8A%A8%E6%9B%9D%E5%85%89.png">

+ **WB**

白平衡。白平衡与色温相关，用于衡量图像的色彩真实性和准确性。ISP需要实现 AWB 功能，力求在各种复杂场景下都能精确的还原物体本来的颜色。
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%208-WB.png">

+ **LSC**

用于消除图像周边和图片中心的不一致性，包含亮度和色度两方面。ISP 需要借助 OTP 中的校准数据完成 LSC 功能。

+ **GAMMA CORRECTION**

伽玛校正。传感器对光线的响应和人眼对光线的响应是不同的。伽玛校正就是使得图像看起来符合人眼的特性。

+ **CROP/RESIZE**

图像剪裁，即改变图像的尺寸。可用于输出不同分辨率的图像。

+ **VRA**

视觉识别。用于识别特定的景物，例如人脸识别，车牌识别。ISP 通过各种 VRA 算法，准确的识别特定的景物。

+ **DRC**

动态范围校正。动态范围即图像的明暗区间。DRC 可以使得暗处的景物不至于欠曝，而亮处的景物不至于过曝。ISP 需要支持 DRC 功能。

+ **CSC**

颜色空间转换。例如，ISP 会将 RGB 信号转化为 YUV 信号输出。

+ **IS**

图像稳定。IS 的主要作用是使得图像不要因为手持时轻微的抖动而模糊不清。IS 有很多种，例如 OIS、DIS、EIS。ISP 可以实现 DIS 和 EIS。


+ **TestPattern------测试图像**

Test Pattern主要用来做测试用。不需要先在片上ROM存储图片数据，直接使用生成的测试图像，用生成的测试图像进行后续模块的测试验证。以下是常用的两种测试图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%201-Test%20Pattern.png">

+ **BLC(BlackLevel Correction)------黑电平校正**

Black Level 是用来定义图像数据为 0 时对应的信号电平。由于暗电流的影响， 传感器出来的实际原始数据并不是我们需要的黑平衡（ 数据不为0） 。 所以，为减少暗电流对图像信号的影响，可以采用的有效的方法是从已获得的图像信号中减去参考暗电流信号。一般情况下， 在传感器中，实际像素要比有效像素多， 像素区头几行作为不感光区（ 实际上， 这部分区域也做了 RGB 的 color filter） ， 用于自动黑电平校正， 其平均值作为校正值， 然后在下面区域的像素都减去此矫正值， 那么就可以将黑电平矫正过来了。如下图所示，左边是做黑电平校正之前的图像，右边是做了黑电平校正之后的图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%202-BLC(BlackLevel%20Correction).png">

+ **LSC(Lens Shade Correction)------镜头阴影校正**

由于相机在成像距离较远时，随着视场角慢慢增大，能够通过照相机镜头的斜光束将慢慢减少，从而使得获得的图像中间比较亮，边缘比较暗，这个现象就是光学系统中的渐晕。由于渐晕现象带来的图像亮度不均会影响后续处理的准确性。因此从图像传感器输出的数字信号必须先经过镜头矫正功能块来消除渐晕给图像带来的影响。同时由于对于不同波长的光线透镜的折射率并不相同，因此在图像边缘的地方，其R、G、B的值也会出现偏差，导致CA(chroma aberration)的出现，因此在矫正渐晕的同时也要考虑各个颜色通道的差异性。

常用的镜头矫正的具体实现方法是，首先确定图像中间亮度比较均匀的区域，该区域的像素不需要做矫正；以这个区域为中心，计算出各点由于衰减带来的图像变暗的速度，这样就可以计算出相应R、G、B通道的补偿因子(即增益)。下图左边图像是未做镜头阴影校正的，右边图像是做了镜头阴影校正的。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%203-BLC(BlackLevel%20Correction).png">

+ **DPC(Bad Point Correction)------坏点校正**

所谓坏点，是指像素阵列中与周围像素点的变化表现出明显不同的像素，因为图像传感器是成千上万的元件工作在一起，因此出现坏点的概率很大。一般来讲，坏点分为三类：第一类是死点，即一直表现为最暗值的点；第二类是亮点，即一直表现为最亮值的点：第三类是漂移点，就是变化规律与周围像素明显不同的像素点。由于图像传感器中CFA的应用，每个像素只能得到一种颜色信息，缺失的两种颜色信息需要从周围像素中得到。如果图像中存在坏点的话，那么坏点会随着颜色插补的过程往外扩散，直到影响整幅图像。因此必须在颜色插补之前进行坏点的消除。

+ **GB（Green Balance）------绿平衡**
 
由于感光器件制造工艺和电路问题，Gr,Gb数值存在差异,将出现格子迷宫现象可使用均值算法处理Gr,Gb通道存在的差异,同时保留高频信息。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%205-GB%EF%BC%88Green%20Balance%EF%BC%89.png">

+ **Denoise-----去除噪声**

使用 cmos sensor 获取图像，光照程度和传感器问题是生成图像中大量噪声的主要因素。同时， 当信号经过 ADC 时， 又会引入其他一些噪声。 这些噪声会使图像整体变得模糊， 而且丢失很多细节， 所以需要对图像进行去噪处理空间去噪传统的方法有均值滤波、 高斯滤波等。

但是， 一般的高斯滤波在进行采样时主要考虑了像素间的空间距离关系， 并没有考虑像素值之间的相似程度， 因此这样得到的模糊结果通常是整张图片一团模糊。 所以， 一般采用非线性去噪算法， 例如双边滤波器， 在采样时不仅考虑像素在空间距离上的关系， 同时加入了像素间的相似程度考虑， 因而可以保持原始图像的大体分块， 进而保持边缘。

+ **Demosaic------颜色插值**

光线中主要包含三种颜色信息，即R、G、B。但是由于像素只能感应光的亮度，不能感应光的颜色，同时为了减小硬件和资源的消耗，必须要使用一个滤光层，使得每个像素点只能感应到一种颜色的光。目前主要应用的滤光层是bayer GRBG格式。如下图所示：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%207.1-Demosaic.png">

这样，经过滤色板的作用之后，每个像素点只能感应到一种颜色。必须要找到一种方法来复原该像素点其它两个通道的信息，寻找该点另外两个通道的值的过程就是颜色插补的过程。由于图像是连续变化的，因此一个像素点的R、G、B的值应该是与周围的像素点相联系的，因此可以利用其周围像素点的值来获得该点其它两个通道的值。目前最常用的插补算法是利用该像素点周围像素的平均值来计算该点的插补值。如下图所示，左侧是RAW域原始图像，右侧是经过插值之后的图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%207.2-Demosaic.png">

+ **CCM（Color Correction Matrix）------颜色校正**

颜色校正主要为了校正在滤光板处各颜色块之间的颜色渗透带来的颜色误差。一般颜色校正的过程是首先利用该图像传感器拍摄到的图像与标准图像相比较，以此来计算得到一个校正矩阵。该矩阵就是该图像传感器的颜色校正矩阵。在该图像传感器应用的过程中，及可以利用该矩阵对该图像传感器所拍摄的所有图像来进行校正，以获得最接近于物体真实颜色的图像。

一般情况下，对颜色进行校正的过程，都会伴随有对颜色饱和度的调整。颜色的饱和度是指色彩的纯度，某色彩的纯度越高，则其表现的就越鲜明；纯度越低，表现的则比较黯淡。RGB三原色的饱和度越高，则可显示的色彩范围就越广泛。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%209-CCM%EF%BC%88Color%20Correction%20Matrix%EF%BC%89.png">

+ **RGB Gamma------Gamma校正**

伽马校正的最初起源是CRT屏幕的非线性，研究CRT电子枪的物理表明，电子枪的输入电压和输出光之间满足5．2幂函数关系，即荧光屏上显示的亮度正比于输入电压的5／2次方，这个指数被称为伽马。这种关系源于阴极、光栅和电子束之间的静电相互作用。由于对于输入信号的发光灰度，不是线性函数，而是指数函数，因此必需校正。校正原理如下图1-9所示：

但是实际情况是，即便CRT显示是线性的，伽马校正依然是必须的，是因为人类视觉系统对于亮度的响应大致是成对数关系的，而不是线性的。人类视觉对低亮度变化的感觉比高亮度变化的感觉来的敏锐，当光强度小于1lux时，常人的视觉敏锐度会提高100倍t2118]。伽马校正就是为了校正这种亮度的非线性关系引入的一种传输函数。校正过程就是对图像的伽玛曲线进行编辑，检出图像信号中的深色部分和浅色部分，并使两者比例增大，从而提高图像对比度效果，以对图像进行非线性色调编辑。由于视觉环境和显示设备特性的差异，伽马一般取2．2～2．5之间的值。当用于校正的伽马值大于1时，图像较亮的部分被压缩，较暗的部分被扩展；而伽马值小于1时，情况则刚好相反。

现在常用的伽马校正是利用查表法来实现的，即首先根据一个伽马值，将不同亮度范围的理想输出值在查找表中设定好，在处理图像的时候，只需要根据输入的亮度，既可以得到其理想的输出值。在进行伽马校正的同时，可以一定范围的抑制图像较暗部分的噪声值，并提高图像的对比度。还可以实现图像现显示精度的调整，比如从l0bit精度至8bit精度的调整。上图分别是未做Gamma校正的，下图是做了Gamma校正的。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%2010-RGB%20Gamma------Gamma%E6%A0%A1%E6%AD%A3.png">

+ **RGBToYUV**

YUV 是一种基本色彩空间， 人眼对亮度改变的敏感性远比对色彩变化大很多， 因此， 对于人眼而言， 亮度分量 Y 要比色度分量 U、 V 重要得多。 另外，YUV色彩空间分为YUV444,YUV422，YUV420等格式，这些格式有些比原始RGB图像格式所需内存要小很多，这样亮度分量和色度分量分别存储之后，给视频编码压缩图像带来一定好处。

+ **WDR（Wide Dynamic Range）------宽动态**

动态范围(Dynamic Range)是指摄像机支持的最大输出信号和最小输出信号的比值，或者说图像最亮部分与最暗部分的灰度比值。普通摄像机的动态范围一般在1:1000(60db)左右，而宽动态(Wide Dynamic Range,WDR)摄像机的动态范围能达到1:1800-1:5600(65-75db)。

宽动态技术主要用来解决摄像机在宽动态场景中采集的图像出现亮区域过曝而暗区域曝光不够的现象。简而言之，宽动态技术可以使场景中特别亮的区域和特别暗的区域在最终成像中同时看清楚。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%2012-WDR%EF%BC%88Wide%20Dynamic%20Range%EF%BC%89------%E5%AE%BD%E5%8A%A8%E6%80%81.png">

+ **Sharp------锐化**

CMOS输入的图像将引入各种噪声，有随机噪声、量化噪声、固定模式噪声等。ISP降噪处理过程中，势必将在降噪的同时，把一些图像细节给消除了，导致图像不够清晰。为了消除降噪过程中对图像细节的损失，需要对图像进行锐化处理，还原图像的相关细节。如下图所示，左图是未锐化的原始图像，右图是经过锐化之后的图像。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%2014.Sharp------%E9%94%90%E5%8C%96.png">


</br></br></br></br></br>
<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ISP%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B3.png">

+ **RAW域**

是指从DPC一直到demosaic阶段（此流程图）

+ **DPC**

坏点矫正(bed pixel corr),坏点由于芯片制造工艺等问题产生的，坏点是指亮度或者色彩与周围其他像素的点有非常大的区别，常用检测方法是在全黑环境下看亮点和彩点和在盖白板的情况下看黑点和彩点，ISP端一般通过在亮度域上取其他周围像素点均值来消除坏点

+ **BLC**

黑电平矫正(Black level corr)，黑电平是指图像数据为0时对应的信号电平，进行黑电平矫正的目的；一是由于sensor本身会存在暗电流，导致在没有光照进来的条件下pixel也有电压输出，不过这部分一般在sensor端就已经处理掉了，还有一个原因是因为sensor进行模数转换时精度不够，以8bit为例，每个pixel有效范围是0-255，sensor可能无法将接近于0的信息转化出来，由于人眼特性（对暗处细节比较敏感，）所以sensor厂商一般在转换时会加一个固定的偏移量使像素输出在5（非固定值）—255之间，然后传输在ISP端再做一个减法，将5（非固定值）变为0

+ **Denosice**

降噪. 噪声在图像上常表现为一引起较强视觉效果的孤立像素点或像素块。一般在暗态下噪声表现尤为明显。影响人的主观视觉感受及对目标的观测，所以进行降噪，但是降噪一般伴随着细节的损失

+ **LSC**

镜头亮度矫正（lens shading corr）由于镜头光学系统原因（CRA），sensor中心光轴附件的pixle感光量比四周多，所以导致呈现出来的画面会中心亮四周暗（同时由于边缘入射角大，会造成相邻像素间串扰，严重时会导致角落偏色）。 所以进行lsc的主要目的是为了让画面四周亮度与中心亮度一直，简单理解就是用过增加四周像素的gain值，来达到亮度一致

+ **AWB**

自动白平衡（auto white balance），白平衡顾名思义就是让白色在任何色温下camera都能把它还原成白，由于色温的影响，一张白纸在低色温下会偏黄，高色温下会偏蓝，白平衡的目的就是白色物体在任何色问下都是R=G=B呈现出白色，比较常用的AWB算法有灰度世界，完美反射法等

+ **Demosica**

颜色插值。SENSOR每个pixel只感知一种颜色分量（如流程图一开始所示），由于人眼对绿色比较敏感所以G的分量是R与B的两倍，所形成的图像称之为Bayer图，所以要通过颜色插值使每个pixel上同时包含RGB三个分量

+ **CCM** 

色彩校正（color corr matrix），AWB已经将白色校准了，CCM就是用来校准白色除白色以外其他颜色的准确度的，用一个3X3的CCM矩阵来校准, 其中每一列系数r1+g1+b1等于一个恒定值1。Ccm矫正最终结果可以通过拍摄24色卡图片然后用imatest分析来做分析参考

+ **Ygamma**

由于最早期的显示器端，亮度与电流之间响应不线性的，而是以曲线形式（曲线称之为gamma曲线），camera为了配合显示器显示出正确的亮度所以有了摄像头的gamma曲线与显示器gamma曲线成反比（不是绝对的），后来随着显示器的工艺发展，显示器亮度与电流之间已经可以做成显性关系了，但是人们发现由于gamma曲线的存在，摄像头暗部才能信息更好保留显示，更符合人眼视觉感受，我们可以通过调整gamma曲线来调整摄像头的亮度，对比度，动态范围等等的效果


+ **EE**

锐化，当物体锐化值过低时会出现边缘模糊，图像给人感觉不清晰，锐化过高就会导致图像出现锯齿白边等现象

+ **CSM**

色彩空间转化（color space matrix），RGB图像通过一个转转举止向SRGB等色彩空间转化的过程



参考：

https://blog.csdn.net/lz0499/article/details/71156291
http://camera.geek-docs.com/camera-isp/camera-isp-flow-intro.html
https://blog.csdn.net/tyfwin/article/details/90487890

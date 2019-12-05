# 什么是ISP？

ISP(Image Signal Processor)，即图像处理，主要作用是对前端图像传感器输出的信号做后期处理，主要功能有线性纠正、噪声去除、坏点去除、内插、白平衡、自动曝光控制等，依赖于ISP才能在不同的光学条件下都能较好的还原现场细节，ISP技术在很大程度上决定了摄像机的成像质量。
光线始于Lens，终于Sensor，CIS之后再无模拟。光线被Sensor转换为电讯号之后，经过ADC的采样、量化，最终形成了我们熟悉的01世界的数字图像。数字图像的下一个加工厂就是ISP了！

<img src"https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20ARM_ISP.png">

看ISP的直译就知道，它也是一个处理器，所以根据它的位置，我们可以把它分为集成式和分离式：

## 分离式

分离式也称为外置ISP架构，这里的分离和外置的对象都是主处理器AP。所以，只要ISP没有被集成在AP内部的都可以成为分离式。
在Camera分辨率还没有1M的年代，这个量级的处理量Camera Driver IC厂商有能力研发自己的ISP，他们可以把这部分集成在自己的Driver IC中；但是现如今的Camera动不动就要上亿了，总有些专业的数字图像处理的公司接管了这部分业务，有了自己的ISP（比如：socionext，arm和ov），所以我们现如今说的分离式的ISP都是这种在AP外下挂一个专业ISP的形式：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E5%A4%96%E7%BD%AEISP.png">

## 集成式

集成的就很容易理解了，现如今高通、苹果、三星、海思、MTK主流的手机AP供应商都会把ISP集成到他们的芯片内部，这样可以给用户提供更便捷的调试体验（算法平台一手承包，真是工程师们的福音），还节约了成本，节约了宝贵的PCB空间资源。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/ISP/Camera%2007-ISP%20%E5%86%85%E7%BD%AEISP.png">

# ISP的内部架构



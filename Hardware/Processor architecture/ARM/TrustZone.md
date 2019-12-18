# 1. TrustZone介绍
## 1.1 安全背景

在介绍TrustZone前有必要简单回顾下目前的一些安全手段。

CPU通过内存映射手段给每个进程营造一个单独的地址空间来隔离多个进程的代码和数据，通过内核空间和用户空间不同的特权级来隔离操作系统和用户进程的代码和数据。但由于内存中的代码和数据都是明文，容易被同处于内存中的其它应用偷窥，因此出现了扩展的安全模块，应用将加密数据送往安全模块，由安全模块处理完后再返回结果给相应的应用。

很多消费电子设备都使用扩展的安全模块来确保数据安全，目前常见的方式有：

+ 外部挂接硬件安全模块

  数据的处理交由外部的安全模块实现，这些模块能够保护自己的资源和密钥等数据的安全，如SIM卡、各种智能卡或连接到外部的硬件加解密模块等，但其同主芯片的通信线路暴露在外部，容易被监听破解。另外，通信的速率比较低。

+ 内部集成硬件安全模块
  
  将外部安全模块的功能集成到芯片内，因此一个芯片上至少有两个核：一个普通核和一个安全核。优点是核与核之间的通信在芯片内部实现，不再暴露在外面。缺点是核之间的通信速度仍然较低，而且单独的安全核性能有限，还会会占用SoC面积，成本较高。

## 1.2 TrustZone是个什么鬼？

TrustZone是ARM针对消费电子设备设计的一种硬件架构，其目的是为消费电子产品构建一个安全框架来抵御各种可能的攻击。

TrustZone在概念上将SoC的硬件和软件资源划分为安全(Secure World)和非安全(Normal World)两个世界，所有需要保密的操作在安全世界执行（如指纹识别、密码处理、数据加解密、安全认证等），其余操作在非安全世界执行（如用户操作系统、各种应用程序等），安全世界和非安全世界通过一个名为Monitor Mode的模式进行转换，如图1：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Processor%20architecture/ARM/Picture/ARM-01%20TrustZone/01%20ARM%20Trustzone%20Nw-Sw.png">

图1. ARM的安全世界和非安全世界

处理器架构上，TrustZone将每个物理核虚拟为两个核，一个非安全核（Non-secure Core, NS Core），运行非安全世界的代码；和另一个安全核（Secure Core），运行安全世界的代码。

两个虚拟的核以基于时间片的方式运行，根据需要实时占用物理核，并通过Monitor Mode在安全世界和非安全世界之间切换，类似同一CPU下的多应用程序环境，不同的是多应用程序环境下操作系统实现的是进程间切换，而Trustzone下的Monitor Mode实现了同一CPU上两个操作系统间的切换。

AMBA3 AXI(AMBA3
Advanced eXtensble Interface)系统总线作为TrustZone的基础架构设施，提供了安全世界和非安全世界的隔离机制，确保非安全核只能访问非安全世界的系统资源，而安全核能访问所有资源，因此安全世界的资源不会被非安全世界（或普通世界）所访问。

设计上，TrustZone并不是采用一刀切的方式让每个芯片厂家都使用同样的实现。总体上以AMBA3 AXI总线为基础，针对不同的应用场景设计了各种安全组件，芯片厂商根据具体的安全需求，选择不同的安全组件来构建他们的TrustZone实现。

其中主要的组件有：

+ 必选组件
  + AMBA3 AXI总线，安全机制的基础设施
  + 虚拟化的ARM Core，虚拟安全和非安全核
  + TZPC (TrustZone Protection Controller)，根据需要控制外设的安全特性
  + TZASC (TrustZone Address Space Controller)，对内存进行安全和非安全区域划分和保护

+ 可选组件
  + TZMA (TrustZone Memory Adapter)，片上ROM或RAM安全区域和非安全区域的划分和保护
  + AXI-to-APB bridge，桥接APB总线，配合TZPC使APB总线外设支持TrustZone安全特性

除了以上列出的组件外，还有诸如 Level 2 Cache Controller, DMA Controller, Generic Interrupt Controller等。

逻辑上，安全世界中，安全系统的OS提供统一的服务，针对不同的安全需求加载不同的安全应用TA(Trusted Application)。 例如：针对某具体DRM的TA，针对DTCP-IP的TA，针对HDCP 2.0验证的TA等。

图2是一个ARM官网对TrustZone介绍的应用示意图：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Processor%20architecture/ARM/Picture/ARM-01%20TrustZone/01%20ARM%20Trustzone%20TEE.png">

图中左边蓝色部分Rich OS Application Environment(REE)表示用户操作环境，可以运行各种应用，例如电视或手机的用户操作系统，图中右边绿色部分Trusted Execution Envrionment(TEE)表示系统的安全环境，运行Trusted OS，在此基础上执行可信任应用，包括身份验证、授权管理、DRM认证等，这部分隐藏在用户界面背后，独立于用户操作环境，为用户操作环境提供安全服务。

```
可信执行环境（TEE, Trusted Execution Environment）是Global Platform（GP）提出的概念。对应于TEE还有一个REE(Rich Execution Environment)概念，分别对应于安全世界(Secure World)和非安全世界(Non-secure World, Normal World)。

GlobalPlatform（GP）是跨行业的国际标准组织，致力于开发、制定并发布安全芯片的技术标准，以促进多应用产业环境的管理 及其安全、可互操作的业务部署。目标是创建一个标准化的基础架构, 加快安全应用程序及其关联资源的部署，如数据和密钥，同时保护安全应用程序及其关联资源免受软件方面的攻击。
```

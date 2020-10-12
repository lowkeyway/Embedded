# MIPI

## D-PHY

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20D%20Option.png">

### 1、传输模式

+ LP（Low-Power）模式：用于传输控制信号，最高速率 10 MHz。
  + LP传输不需要CLK做基准，Host和Slave内部会同步采样，而且只通过Data0传输。
+ HS（High-Speed）模式：用于高速传输数据，速率范围 [80 Mbps， 1Gbps] per Lane。
  + HS传输需要用CLK作双边缘采样，而且在多Lane模式下数据会拆开给每一条Lane。
    + 数据拆分
    <img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20%E5%A4%9A%E6%80%BB%E7%BA%BF%E6%95%B0%E6%8D%AE%E5%88%86%E9%85%8D.png">

    + 数据集合
    <img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20%E5%A4%9A%E6%80%BB%E7%BA%BF%E6%95%B0%E6%8D%AE%E9%9B%86%E5%90%88.png">
    
传输的最小单元为 1 个字节，采用小端的方式及 LSB first，MSB last。


### 2、Lane States

+ LP mode 有 4 种状态： LP00、LP01（0）、LP10（1）、LP11 （Dp、Dn）
+ HS mode 有 2 种状态： HS-0、HS-1

HS 发送器发送的数据 LP 接收器看到的都是 LP00，


### 3、Lane Levels

+ LP： 0 ~ 1.2V
+ HS： 100 ~ 300mV，HS common level = 200mV，swing = 200 mv


### 4、操作模式

在数据线上有 3 种可能的操作模式：Escape mode, High-Speed (Burst) mode and Control mode，下面是从停止状态进入相应模式需要的时序：

+ **Escape mode** 
  + 进入时序：LP11→LP10→LP00→LP01→LP00，
  + 退出时序：LP10→LP11

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Escape%20Enter.png">

+ **High-Speed mode**
  + 进入时序：LP11→LP01→LP00→SoT(0001_1101)
  + 退出时序：EoT→LP11
  
  <img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20HS-LP.png">
  
+ **Turnaround**
  + 进入时序：LP11→LP10→LP00→LP10→LP00
  + 退出时序：LP00→LP10→LP11

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Turn%20Around.png">

+ **ULPS和低速传输**
当进入 Escape mode 需要发送 8-bit entry command 表明请求的动作，比如要进行低速数据传输则需要发送 **cmd： 0x87**，进入超低功耗模式则发送 **cmd： 0x78**。在 DSI 中 LP 通讯只用 Data Lane 0。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/ULPS%20Enter.png">


 ### 5、时序要求
 
 在调试 DSI 或者 CSI 的时候， HS mode 下的几个时序非常重要：T_LPX，T_HS-SETTLE ≈ T_HS-PREPARE + T_HS-ZERO，T_HS-TRAIL，一般遵循的原则为：Host 端的 T_HS-SETTLE > Slave 端的 T_HS-SETTLE。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Transfer%20Timming.png">

可以参考如下典型值：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Transfer%20Timming%20Data.png">

### 6、电平要求

有的时候，为了在更高速度下增强抗干扰能力、降低功耗，会人为得降低P和N之间的电平差，但是Mipi同时也规范了这种操作：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Voltage%20Levels.png">

可以参考如下典型值：
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Voltage%20Levels%20Data.png">

## C-PHY

### Feature


+ Uses a group of three conductors, rather than conventional pairs. The group of three wires is called a Lane, and the individual Lines of the Lane are called A, B and C. C-PHY does not have a separate clock Lane. 

+ Within a three-wire Lane, two of the three wires are driven to opposite levels; the third wire is terminated to a mid-level (at either one end or both ends), and the voltages at which the wires are driven changes at every symbol. 

+ Multiple bits are encoded into each symbol epoch, the data rate is ~2.28x the symbol rate. There is no additional overhead for line coding, such as 8b10b, which is not needed 

+ Clock timing is encoded into each symbol. This is accomplished by requiring that the combination of voltages driven onto the wires must change at every symbol boundary. This simplifies clock recovery. 

+ The signal is received using a group of three differential receivers 

+ The C-PHY interface can co-exist on the same pins/pads as the D-PHY interface signals 


### 协议框图

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20C%20Option.png">

### 电平标准

虽然C-PHY也分为LP和HS两种模式，但是C-PHY的电平跟D-PHY有少许不一样。

LP： 0 ~ 1.1V
HS：250mV Normal(V=0.5V). Swing 250mV


<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/%E7%94%B5%E5%B9%B3%E7%A4%BA%E6%84%8F%E5%9B%BE.png">

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/%E7%94%B5%E5%B9%B3%E6%8C%87%E6%A0%87.png">


### 状态迁移

我们知道了C-PHY每条Lane上是有3跟物理总线的，那么这三条物理总线是如何表达成数字信号的呢？简单的说，C-PHY的3条物理总线可以构成6种状态（没有000/111这两种），这6中状态之间每种状态都有5种迁移的可能，迁移的可能性就可以被编码成数字信号。

+ 6种状态
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/6%E7%A7%8D%E7%8A%B6%E6%80%81.png">

+ 5种迁移
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/5%E7%A7%8D%E7%8A%B6%E6%80%81%E8%BF%81%E7%A7%BB.png">

### 连续状态编码

我们知道了C-PHY使用了5进制的编码，那么5进制是如何跟AP侧的2进制进行转换的呢？简单的说，就是靠连续状态迁移来实现的，即一个16bit的数据可以翻译成连续的7个5进制（状态迁移）来表示。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/16bit-7symbol%E7%9A%84%E5%8E%8B%E7%BC%A9.png">


## DSI Protocol

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20DSI%20layer.png">

### 1、线路构成

在 DSI 中需要 1 根时钟线以及 1 ~ 4 根数据线。

### 2、两种接口的 LCD

+ Command mode（对应 MPU 接口）
+ Video mode（对应 RGB 接口）
  + 该模式下视频数据只能通过 HS mode 传输
  
### 3、数据包类型

#### OverView

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20HS%20Transmist%20with%20EoTP.png">

+ 短包：4 bytes，由 3 部分组成：
  + Data Identifier (DI) * 1byte： Contains the Virtual Channel[7:6] and Data Type[5:0].
  + Packet Data * 2byte：Length is fixed at two bytes
  + Error Correction Code (ECC) * 1byte：allows single-bit errors to be corrected and 2-bit errors to be detected.

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Short%20Package%20Struct.png">

+ 长包：6 ~ 65541 bytes，同样由 3 部分组成：
  + Packet Header(4 bytes) - 包头
```
Data Identifier (DI) * 1byte：Contains the Virtual Channel[7:6] and Data Type[5:0].
Word Count (WC) * 2byte：defines the number of bytes in the Data Payload.
Error Correction Code (ECC) * 1byte：allows single-bit errors to be corrected and 2-bit errors to be detected.
```

+ Data Payload(0~65535 bytes) - 有效数据

```
Length = WC × bytes
```

  + Packet Footer(2 bytes)：Checksum - 包尾
  
```
If the payload has length 0, then the Checksum calculation results in FFFFh
If the Checksum isn’t calculated, the Checksum value is 0000h
```
<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Long%20Package%20Struct.png">

### 4、从控制器到外设发送的包类型

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI%20DATA%20TYPE.png">

如果希望从外设读取数据或者状态，则在处理器发送完读取命令后还需要发送 BTA 命令，非读取命令在外设接收成功后会返回 trigger message 0x84。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20CMD%20Identifier%20Byte.png">

### 5、从外设到处理器数据包类型

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Package%20Peripheral-to-Processor.bmp">

返回的数据一般分为 4 个类型：

+ Tearing Effect (TE)：trigger message (BAh)
+ Acknowledge：trigger message (84h)
+ Acknowledge and Error Report：short packet (Data Type is 02h)
+ Response to Read Request：short packet or long packet
Generic Read Response、DCS Read Response（1byte, 2byte, multi byte）

### 6、Vedio Mode

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Video%20Mode.png">

在Vedio Mode传输里，也分三种模式：
+ **Non-Burst Mode with Sync Pulses** – enables the peripheral to accurately reconstruct original video 2025 timing, including sync pulse widths. 

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Video%20Mode-Non-Burst%20with%20Sync%20Start%20and%20End.png">

+ **Non-Burst Mode with Sync Events** – similar to above, but accurate reconstruction of sync pulse 2027 widths is not required, so a single Sync Event is substituted. 

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Video%20Mode-Non-Burst%20with%20Sync%20Events.png">

+ **Burst mode** – RGB pixel packets are time-compressed, leaving more time during a scan line for 2029 LP mode (saving power) or for multiplexing other Transmissions onto the DSI Link.

<img src="https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/MIPI/MIPI%20Video%20Mode-Burst.png">

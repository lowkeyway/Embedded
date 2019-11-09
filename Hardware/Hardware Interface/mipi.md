# MIPI

## D-PHY

### 1、传输模式

+ **LP（Low-Power）模式**：用于传输控制信号，最高速率 10 MHz
+ **HS（High-Speed）模式**：用于高速传输数据，速率范围 [80 Mbps， 1Gbps] per Lane

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


+ **High-Speed mode**
  + 进入时序：LP11→LP01→LP00→SoT(0001_1101)
  + 退出时序：EoT→LP11
  
  
+ **Turnaround**
  + 进入时序：LP11→LP10→LP00→LP10→LP00
  + 退出时序：LP00→LP10→LP11

当进入 Escape mode 需要发送 8-bit entry command 表明请求的动作，比如要进行低速数据传输则需要发送 **cmd： 0x87**，进入超低功耗模式则发送 **cmd： 0x78**。在 DSI 中 LP 通讯只用 Data Lane 0。


 ### 5、时序要求
 
 在调试 DSI 或者 CSI 的时候， HS mode 下的几个时序非常重要：T_LPX，T_HS-SETTLE ≈ T_HS-PREPARE + T_HS-ZERO，T_HS-TRAIL，一般遵循的原则为：Host 端的 T_HS-SETTLE > Slave 端的 T_HS-SETTLE。



## DSI

# UART

 通用异步收发传输器（Universal Asynchronous Receiver/Transmitter），通常称作UATR，是一种异步收发传输器。将数据由串行通信与并行通信间做传输转换，作为并行输入称为串行输出的芯片。UART是一种通用串行数据总线，用于异步通信。该总线双向通信，可以实现全双工传输和接收。
 
 所以，UART的几个重要特征：
 + 异步
 + 串行
 + 全双工
 + 协商波特率
 
 **1. UART通信协议**

UART作为异步串口通信协议的一种，工作原理是将传输数据的每一个字符一位一位地传输。其中每一位（bit）的意义如下：

+ 起始位：先发出一个逻辑“0”的信号，表示传输字符开始。
+	数据位：紧接着起始位之后。数据位的个数可以是4、5、6、7、8等，构成一个字符。通常采用ASCII码。从最低位开始传送，靠时钟定位。
+	奇偶校验位：数据位加上这一位后，使得“1”的位数应为偶数（偶校验）或奇数（奇校验），以次来校验数据传送的正确性。
+	停止位：它是一个字符数据的结束标志。可以是1位、1.5位、2位的高电平。由于数据是在传输线上定时的，并且每一个设备有其自己的时钟，很可能在通信中两台设备间出现了小小的不同步。因此停止位不仅仅是表示传输的结束，并且提供计算机校正时钟同步的机会。适用于停止位的位数越多，不同时钟同步的容忍程度越大，但是数据传输率也就越慢。
+	空闲位：处于逻辑“1”状态，表示当前线路上没有数据传输。
 
 <img src='https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/UART-%E6%95%B0%E6%8D%AE%E6%A0%BC%E5%BC%8F.jpeg'>
 
 **2. UART工作原理**
 
+ 发送数据过程：空闲状态，线路处于高电平；当收到发送指令后，拉低线路的一个数据位的时间T，接着数据按低位到高位依次发送，数据发送完毕后，接着发送奇偶校验位和停止位，一帧数据发送完成。

+ 数据接收过程：空闲状态，线路处于高电平；当检测到线路的下降沿（高电平变为低电平）时说明线路有数据传输，按照约定的波特率从低位到高位接收数据，数据接收完毕后，接着接收并比较奇偶校验位是否正确，如果正确则通知后续设备接收数据或存入缓冲。

由于UART是异步传输，没有传输同步时钟，为了保证数据的正确性，UART采用16倍数据波特率的时钟进行采样。每个数据有16个时钟采样，取中间的采样值，以保证采样不会滑码或误码。一般UART一帧的数据位数为8，这样即使每个数据有一个时钟的误差，接收端也能正确地采样到数据。

 <img src='https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/UART%20%E6%95%B0%E6%8D%AE%E9%87%87%E6%A0%B7.jpeg'>
 
 **3. 波特率与比特率**
 
+ 比特率 

在数字信道中，比特率是数字信号的传输速率，它用单位时间内传输的二进制代码的有效位(bit)数来表示，其单位为每秒比特数bit/s(bps)、每秒千比特数(Kbps)或每秒兆比特数(Mbps)来表示(此处K和M分别为1000和1000000，而不是涉及计算机存储器容量时的1024和1048576)。

+ 波特率

波特率指数据信号对载波的调制速率，它用单位时间内载波调制状态改变次数来表示，其单位为波特(Baud)。 波特率与比特率的关系为：比特率=波特率X单个调制状态对应的二进制位数。
如何区分两者？ 显然，两相调制(单个调制状态对应1个二进制位)的比特率等于波特率；四相调制(单个调制状态对应2个二进制位)的比特率为波特率的两倍；八相调制(单个调制状态对应3个二进制位)的比特率为波特率的三倍；依次类推。

 <img src='https://github.com/lowkeyway/Embedded/blob/master/Hardware/Hardware%20Interface/PictureSrc/UART%20%E6%B3%A2%E7%89%B9%E7%8E%87%20%E6%AF%94%E7%89%B9%E7%8E%87.jpeg'>

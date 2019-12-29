V4L2是V4L的升级版本，为Video For Linux之意。是Linux中专门给Video设计的一套框架。
V4L2支持的设备十分广泛，但是其中只有很少一部分在本质上是真正的视频设备：

+ **Video capture device** ： 从摄像头等设备上获取视频数据。对很多人来讲，video capture是V4L2的基本应用。设备名称为/dev/video,主设备号81，子设备号0-63
+ **Video output device** ： 将视频数据编码为模拟信号输出。与video capture设备名相同。
+ **Video overlay device** ： 将同步锁相视频数据（如TV）转换为VGA信号，或者将抓取的视频数据直接存放到视频卡的显存中。
+ **Video output overlay device** ：也被称为OSD(On-Screen Display)
+ **VBI device** ： 提供对VBI（Vertical Blanking Interval）数据的控制，发送VBI数据或抓取VBI数据。设备名/dev/vbi0-vbi31,主设备号81,子设备号224-255
+ **Radio device** ： FM/AM发送和接收设备。设备名/dev/radio0-radio63,主设备号81，子设备号64~127

在Camera这里，我们只关心Video Capture Device。
我们要了解V4L2可以从用户态和内核态两个部分来看看如何使用和实现V4L2的一些API和关键结构体。

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/V4L2/Camera%2010-V4L2%20%E7%BB%93%E6%9E%84%E5%9B%BE.png">

# 用户态

可以看出,V4L2是一个字符设备，而V4L2的大部分功能都是通过设备文件的ioctl导出的。
可以将这些ioctl分类如下：

1. Query Capability:查询设备支持的功能，只有VIDIOC_QUERY_CAP一个。
2. 优先级相关：包括VIDIOC_G_PRIORITY,VIDIOC_S_PRIORITY,设置优先级。
3. capture相关：视频捕获相关Ioctl。

capture ioctl list

| ID |	描述 |
|-|-|
|VIDIOC_ENUM_FMT |	枚举设备所支持的所有数据格式|
|VIDIOC_S_FMT |	设置数据格式|
|VIDIOC_G_FMT |	获取数据格式|
|VIDIOC_TRY_FMT |	与VIDIOC_S_FMT一样，但不会改变设备的状态|
|VIDIOC_REQBUFS |	向设备请求视频缓冲区，即初始化视频缓冲区|
|VIDIOC_QUERYBUF |	查询缓冲区的状态|
|VIDIOC_QBUF |	从设备获取一帧视频数据|
|VIDIOC_DQBUF |	将视频缓冲区归回给设备|
|VIDIOC_OVERLAY |	开始或者停止overlay|
|VIDIOC_G_FBUF |	获取video overlay设备或OSD设备的framebuffer参数|
|VIDIOC_S_FBUF |	设置framebuffer参数|
|VIDIOC_STREAMON |	开始流I/O操作，capture or output device|
|VIDIOC_STREAMOFF |	关闭流I/O操作|

4. TV视频标准：
TV Standard
|ID |	描述|
-|-
VIDIOC_ENUMSTD |	枚举设备支持的所有标准
VIDIOC_G_STD |	获取当前正在使用的标准
VIDIOC_S_STD |	设置视频标准
VIDIOC_QUERYSTD |	有的设备支持自动侦测输入源的视频标准，此时使用此ioctl查询侦测到的视频标准

5. input/output：

Input / Output

| ID |	描述 |
|- | - |
| VIDIOC_ENUMINPUT |	枚举所有input端口 |
| VIDIOC_G_INPUT |	获取当前正在使用的input端口 |
| VIDIOC_S_INPUT |	设置将要使用的input端口 |
| VIDIOC_ENUMOUTPUT |	枚举所有output端口 |
| VIDIOC_S_OUTPUT |	设置将要使用的output端口 |
| VIDIOC_ENUMAUDIO |	枚举所有audio input端口 |
| VIDIOC_G_AUDIO |	获取当前正在使用的audio input端口 |
| VIDIOC_S_AUDIO |	设置将要使用的audio input端口 |
| VIDIOC_ENUMAUDOUT |	枚举所有audio output端口 |
| VIDIOC_G_AUDOUT |	获取当前正在使用的audio output端口 |
| VIDIOC_S_AUDOUT |	设置将要使用的audio output端口 |

6. controls：设备特定的控制，例如设置对比度，亮度

| ID |	描述 |
| - | - | 
| VIDIOC_QUERYCTRL |	查询指定control的详细信息 |
| VIDIOC_G_CTRL |	获取指定control的值 |
| VIDIOC_S_CTRL |	设置指定control的值 |
| VIDIOC_G_EXT_CTRLS |	获取多个control的值 |
| VIDIOC_S_EXT_CTRLS |	设置多个control的值 |
| VIDIOC_TRY_EXT_CTRLS |	与VIDIOC_S_EXT_CTRLS相同，但是不改变设备状态 |
| VIDIOC_QUERYMENU |	查询menu |

7. 其他杂项：

| ID |	描述 |
| - | - |
| VIDIOC_G_MODULATOR |	  |
| VIDIOC_S_MODULATOR |	  |
| VIDIOC_G_CROP	 |  |
| VIDIOC_S_CROP	 |  |
| VIDIOC_G_SELECTION |	  |
| VIDIOC_S_SELECTION |	  |
| VIDIOC_CROPCAP |	  |
| VIDIOC_G_ENC_INDEX |	  |
| VIDIOC_ENCODER_CMD |	  |
| VIDIOC_TRY_ENCODER_CMD |	  |
| VIDIOC_DECODER_CMD |	  |
| VIDIOC_TRY_DECODER_CMD |	  |
| VIDIOC_G_PARM |	  |
| VIDIOC_S_PARM |	  |
| VIDIOC_G_TUNER |	  |
| VIDIOC_S_TUNER |	  |
| VIDIOC_G_FREQUENCY |	  |
| VIDIOC_S_FREQUENCY |	  |
| VIDIOC_G_SLICED_VBI_CAP |	  |
| VIDIOC_LOG_STATUS |	  |
| VIDIOC_DBG_G_CHIP_IDENT |	  |
| VIDIOC_S_HW_FREQ_SEEK |	  |
| VIDIOC_ENUM_FRAMESIZES |	  |
| VIDIOC_ENUM_FRAMEINTERVALS |	  |
| VIDIOC_ENUM_DV_PRESETS |	  |
| VIDIOC_S_DV_PRESET |	  |
| VIDIOC_G_DV_PRESET |	  |
| VIDIOC_QUERY_DV_PRESET |	  |
| VIDIOC_S_DV_TIMINGS |	  |
| VIDIOC_G_DV_TIMINGS |	  |
| VIDIOC_DQEVENT |	  |
| VIDIOC_SUBSCRIBE_EVENT |	  |
| VIDIOC_UNSUBSCRIBE_EVENT |	  |
| VIDIOC_CREATE_BUFS |	  |
| VIDIOC_PREPARE_BUF | |

# 内核态


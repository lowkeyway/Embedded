IS 图像稳定 (Image Stabilization)
 
RDI 原始数据转储接口 (Raw Dump Interface)
 
RoI AF 感兴趣区域 (AF Region of Interest)
 
SNoC 系统 NoC (System NoC)
 
SOF：start of frame
 
ANR：application not response
 
MCC: mutil camera control
 
LPM: low power manager(低功耗下运行)
 
CTS/ITS ：Android Camera Image Test Suite (ITS) is part of Android Compatibility Test Suite (CTS)
 
Android相机图像测试套件（ITS）是Android兼容性测试套件（CTS）的一部分
 
验证程序，包括验证图像内容的测试。
 
从CTS 7.0_r8开始，CTS Verifier通过一体式摄像机ITS支持ITS测试自动化。
 
继续支持手动测试，以确保涵盖所有Android设备尺寸。
 
HAF：混合自动变焦
 
CRM: camera request manager
 
Sensor CRA(主光线角）
 
从镜头的传感器一侧，可以聚焦到像素上的光线的最大角度被定义为一个参数，称为主光角(CRA)。对于主光角的一般性定义是：此角度处的像素响应降低为零度角像素响应(此时，此像素是垂直于光线的)的80%
 
 
lens CRA与sensor 不配会使sensor 的pixel 出现在光检测区域周围，使pixel 曝光不足，亮度不够，会使整个画面造成亮度不均匀的情况。还有可能造成chroma shading 或局部色偏。局部色偏比较严重，无法用算法补偿
 
Sensor HDR：
 
sensor在一幅图像里能够同时体现高光和阴影部分内容的能力
 
lens fov：视场角
 
视场角与焦距的关系：一般情况下，视场角越大，焦距就越短
 
IFE ：Image Front End， Bayer processing for video/preview only， HDR/De-mosic， color correction ，scaler，也可以直接输出Raw到RDI
 
RDI : Raw Dump Interface，直接从IFE吐出来用于capture的
 
STATS:for 3A,ISP硬件给出的3A数据，用于后面的3A算法
 
PDPC：PhaseDetection Pixel Correction，相位检测像素校正
 
ASD: Auto scene detection，自动场景检测
 
CSIC:Camera serial interface decoder,摄像机串行接口解码器
 
CAMIF: Ideal Raw的第一个dump点
 
FD: Face-based,基于人脸，也就是人脸识别
 
ICA：图像校正和调整      是一个硬件单元，主要用于由镜头和运动引起的几何扭曲
 
LENR：低/中频增强和降噪
 
TMC：色调映射控制
 
CSID：摄像机串行接口解码器模块
 
IFE：图像前端
 
IFE_Lite：？
 
BPS：Bayer处理段
 
IPE：图像处理引擎
 
VPU：视频处理单元
 
DPU：显示处理单元
 
BPC：坏像素校正
 
BCC：坏群集校正
 
ABF：自适应拜耳滤波器
 
GIC：绿色不平衡校正
 
GTM：全局色调映射
 
HNR：混合降噪
 
ANR：先进的降噪功能
 
TF：时间过滤器
 
MFNR：多帧降噪
 
LTM：局部色调映射
 
CS：色度抑制
 
ASF：自适应空间滤波器
 
Upscaler：升频器
 
GRA：Grain Adder（纹理增加器？）？？
 
CPAS：相机外围设备和支持
 
CAMIF：摄像头接口？？？它是VFE（video front-end）硬件的第一部分，主要任务是同步sensor发送数据过程中涉及到的行、场同步信号。另外它还具有图像提取和图像镜像能力，CAMIF hardware使外部camera sensor能够通过一些简单的外部协议链接到用户单元。为camera提供了数据和时钟接口，但并没有提供控制接口，最具代表行的是用I2C做配置和状态接口。当然，也可以是其他的一次控制信号做一些静态控制。例如：睡眠唤醒模式控制。
 
NPS：噪声处理部分
 
PPS：后处理部分
 
MCTF：运动补偿时间滤波
 
CAC、CCM、GLUT、2D LUT（二维查找表？）、CV（颜色转换）、CC（颜色校正）、SCE（肤色增强）、MCE（记忆色彩增强）：？？？
 
SIMO：单输入多输出
 
Pedestal Correction：基座校正
 
Down Scaler：降低规模（尺寸）
 
Chroma Enhancement：色度增强
 
Chroma Suppression：色度抑制
 
PDAF：相位检测自动对焦
 
LSC：镜头阴影校正
 
PNR：峰值降噪
 
ADRC：自动动态范围压缩
 
Backlit scene：背光场景
 
Garage scene：车库场景
 
HNR：降低亮度噪声，但保持纹理细节
 
LDC：镜头畸变校正
 
EIS：电子稳像
 
Multi pass spatial noise filtering：多通空间噪声滤波
 
LNR：镜头降噪
 
Invert gamma：反转伽玛
 
hue, saturation, lightness：色调，饱和度，亮度
 
Upscaler：升频器
 
ACE：高级色度增强
 
CPP：相机后处理器（相当于新版的BPS、IPE）
 
BSP
 
board support package，板级支持安装包？
 
也就是“做出支持安装包，来实现手机上各个硬件的基本功能”。
 
CCT：correlated color temperature，相关色温，具体不详；
 
chi-cdk：
 
Camera Hardward Interface，相机硬件接口；
 
Camera Development Kit，相机开发包；
 
HFR：High Frame Rate, min HFR=90, means>=90时，需要enable HFR高帧率，目前最高960，但是是利用插值算法计算得出的，非实际960

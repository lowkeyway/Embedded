# 一、LCD vs OLED

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/LCD/00_LCD%26OLED%E7%BB%93%E6%9E%84%E5%9B%BE.png">

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/LCD/00_LCD%26OLED%E5%AF%B9%E6%AF%94.png">

# 二、Resolution & PPI

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/LCD/01_Resolution.png">

+ Portrait M<N
+ Landscape M>N

## Pixel and Sub-pixel
+ RGB strip pixel structure
+ 3 sub-pixel -> 1 pixel

## PPI 
+ Pixel density Per Inch
+ PPI=√(𝑀^2+𝑁^2 )/𝑠𝑖𝑧𝑒 

## Main resolution for MP
+ QQVGA(128 x 160)
+ QVGA(240 x 320)
+ WQVGA(240 x 400 or 432)
+ HVGA(320 x 480)
+ nHD(360 x 640)
+ WVGA(480 x 800)
+ FWVGA(480 x 854 or 864)
+ QHD/+(540/640 x 960) 
+ HD720(720 x 1280)
+ WXGA(768/800 x 1280)
+ FHD(1080 x 1920)

# 三、Gray scale & Color depth

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/LCD/02_Gray%20scale%20%26%20Color%20depth.png">

In 16 bits Driving Case (5R6G5B) = 2<sup>5</sup>R × 2<sup>6</sup>G × 2<sup>5</sup>B  = 32R × 64G × 32B = 65,536 color depth display
Color depth decided by driver IC function and module design



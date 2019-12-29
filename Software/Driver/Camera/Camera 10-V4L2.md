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


至于如何使用这些命令，可以参考下面的代码：

```

/* 
 *  V4L2 video capture example 
 * 
 *  This program can be used and distributed without restrictions. 
 * 
 *      This program is provided with the V4L2 API 
 * see http://linuxtv.org/docs.php for more information 
 */  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <string.h>  
#include <assert.h>  
  
#include <getopt.h>             /* getopt_long() */  
  
#include <fcntl.h>              /* low-level i/o */  
#include <unistd.h>  
#include <errno.h>  
#include <sys/stat.h>  
#include <sys/types.h>  
#include <sys/time.h>  
#include <sys/mman.h>  
#include <sys/ioctl.h>  
#include <linux/videodev2.h>  

#define CLEAR(x) memset(&(x), 0, sizeof(x))  
  
enum io_method {  
        IO_METHOD_READ,  
        IO_METHOD_MMAP,  
        IO_METHOD_USERPTR,  
};  
  
struct buffer {  
        void   *start;  
        size_t  length;  
};  
  
static char            *dev_name;  
static enum io_method   io = IO_METHOD_MMAP;  
static int              fd = -1;  
struct buffer          *buffers;  
static unsigned int     n_buffers;  
static int              out_buf;  
static int              force_format;  
static int              frame_count = 4;  
 
static void errno_exit(const char *s)  
{  
        fprintf(stderr, "%s error %d, %s\n", s, errno, strerror(errno));  
        exit(EXIT_FAILURE);  
}  
  
static int xioctl(int fh, int request, void *arg)  
{  
        int r;  
  
        do {  
                r = ioctl(fh, request, arg);  
        } while (-1 == r && EINTR == errno);  
  
        return r;  
}  
  
static void process_image(const void *p, int size)  
{
        if (out_buf)  
                fwrite(p, size, 1, stdout);  
  
        fflush(stderr);  
        fprintf(stderr, ".");  
        fflush(stdout);  
}  

static void store_image(const char *buf_start, int size, int index)
{
    char path[20];
    
    snprintf(path, sizeof(path), "./yuyv%d.yuv", index); 
    int fd = open(path, O_WRONLY|O_CREAT, 00700);
        if (-1 == fd) {  
                fprintf(stderr, "Cannot open '%s': %d, %s\n",  
                         path, errno, strerror(errno));  
                exit(EXIT_FAILURE);  
        }  

    write(fd, buf_start, size);  
    close(fd);  
}

static int read_frame(void)  
{  
        struct v4l2_buffer buf;  
        unsigned int i;  
  
        switch (io) {  
        case IO_METHOD_READ:  
                if (-1 == read(fd, buffers[0].start, buffers[0].length)) {  
                        switch (errno) {  
                        case EAGAIN:  
                                return 0;  
  
                        case EIO:  
                                /* Could ignore EIO, see spec. */  
  
                                /* fall through */  
  
                        default:  
                                errno_exit("read");  
                        }  
                }  
  
                process_image(buffers[0].start, buffers[0].length);  
                break;  
  
        case IO_METHOD_MMAP:  
            CLEAR(buf);  

            buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
            buf.memory = V4L2_MEMORY_MMAP;  
            if (-1 == xioctl(fd, VIDIOC_DQBUF, &buf)) {  
                    switch (errno) {  
                    case EAGAIN:  
                            return 0;  

                    case EIO:  
                            /* Could ignore EIO, see spec. */  

                            /* fall through */  

                    default:  
                            errno_exit("VIDIOC_DQBUF");  
                    }  
            }  
            assert(buf.index < n_buffers);  

          //printf("buf.bytesused = %d\n", buf.bytesused);
            process_image(buffers[buf.index].start, buf.bytesused);  
            store_image(buffers[buf.index].start, buf.bytesused, buf.index);
  
            if (-1 == xioctl(fd, VIDIOC_QBUF, &buf))  
                    errno_exit("VIDIOC_QBUF");  
            break;  
  
        case IO_METHOD_USERPTR:  
                CLEAR(buf);  
  
                buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                buf.memory = V4L2_MEMORY_USERPTR;  
  
                if (-1 == xioctl(fd, VIDIOC_DQBUF, &buf)) {  
                        switch (errno) {  
                        case EAGAIN:  
                                return 0;  
  
                        case EIO:  
                                /* Could ignore EIO, see spec. */  
  
                                /* fall through */  
  
                        default:  
                                errno_exit("VIDIOC_DQBUF");  
                        }  
                }  
  
                for (i = 0; i < n_buffers; ++i)  
                        if (buf.m.userptr == (unsigned long)buffers[i].start  
                            && buf.length == buffers[i].length)  
                                break;  
  
                assert(i < n_buffers);  
  
                process_image((void *)buf.m.userptr, buf.bytesused);  
  
                if (-1 == xioctl(fd, VIDIOC_QBUF, &buf))  
                        errno_exit("VIDIOC_QBUF");  
                break;  
        }  
  
        return 1;  
}  
  
/* two operations 
 * step1 : delay 
 * step2 : read frame 
 */  
static void mainloop(void)  
{  
        unsigned int count;  
  
        count = frame_count;  
  
        while (count-- > 0) {  
                for (;;) {  
                        fd_set fds;  
                        struct timeval tv;  
                        int r;  
  
                        FD_ZERO(&fds);  
                        FD_SET(fd, &fds);  
  
                        /* Timeout. */  
                        tv.tv_sec = 2;  
                        tv.tv_usec = 0;  
  
                        r = select(fd + 1, &fds, NULL, NULL, &tv);  
  
                        if (-1 == r) {  
                                if (EINTR == errno)  
                                        continue;  
                                errno_exit("select");  
                        }  
  
                        if (0 == r) {  
                                fprintf(stderr, "select timeout\n"); 
                                exit(EXIT_FAILURE);  
                        }  
  
                        if (read_frame())  
                                break;  
                        /* EAGAIN - continue select loop. */  
                }  
        }  
}  
/* 
 * one operation 
 * step1 : VIDIOC_STREAMOFF 
 */  
static void stop_capturing(void)  
{  
        enum v4l2_buf_type type;  
        switch (io) {  
        case IO_METHOD_READ:  
                /* Nothing to do. */  
                break;  
  
        case IO_METHOD_MMAP:  
        case IO_METHOD_USERPTR:  
                type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                if (-1 == xioctl(fd, VIDIOC_STREAMOFF, &type))  
                        errno_exit("VIDIOC_STREAMOFF");  
                break;  
        }  
}  
  
/* tow operations 
 * step1 : VIDIOC_QBUF(insert buffer to queue) 
 * step2 : VIDIOC_STREAMOFF 
 */  
static void start_capturing(void)  
{  
        unsigned int i;  
        enum v4l2_buf_type type;  
  
        switch (io) {  
        case IO_METHOD_READ:  
                /* Nothing to do. */  
                break;  
  
        case IO_METHOD_MMAP:  
                for (i = 0; i < n_buffers; ++i) {  
                        struct v4l2_buffer buf;  
  
                        CLEAR(buf);  
                        buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                        buf.memory = V4L2_MEMORY_MMAP;  
                        buf.index = i;  
  
                        if (-1 == xioctl(fd, VIDIOC_QBUF, &buf))  
                                errno_exit("VIDIOC_QBUF");  
                } 
                type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                if (-1 == xioctl(fd, VIDIOC_STREAMON, &type))  
                        errno_exit("VIDIOC_STREAMON");  
        break;  
  
        case IO_METHOD_USERPTR:  
                for (i = 0; i < n_buffers; ++i) {  
                        struct v4l2_buffer buf;  
  
                        CLEAR(buf);  
                        buf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                        buf.memory = V4L2_MEMORY_USERPTR;  
                        buf.index = i;  
                        buf.m.userptr = (unsigned long)buffers[i].start;  
                        buf.length = buffers[i].length;  
  
                        if (-1 == xioctl(fd, VIDIOC_QBUF, &buf))  
                                errno_exit("VIDIOC_QBUF");  
                }  
                type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                if (-1 == xioctl(fd, VIDIOC_STREAMON, &type))  
                        errno_exit("VIDIOC_STREAMON");  
                break;  
        }  
}  
  
/* two operations 
 * step1 : munmap buffers 
 * steo2 : free buffers 
 */  
static void uninit_device(void)  
{  
        unsigned int i;  
  
        switch (io) {  
        case IO_METHOD_READ:  
                free(buffers[0].start);  
                break;  
  
        case IO_METHOD_MMAP:  
                for (i = 0; i < n_buffers; ++i)  
                        if (-1 == munmap(buffers[i].start, buffers[i].length))  
                                errno_exit("munmap");  
                break;  
  
        case IO_METHOD_USERPTR:  
                for (i = 0; i < n_buffers; ++i)  
                        free(buffers[i].start);  
                break;  
        }  
  
        free(buffers);  
}  
  
static void init_read(unsigned int buffer_size)  
{  
        buffers = calloc(1, sizeof(*buffers));  
  
        if (!buffers) {  
                fprintf(stderr, "Out of memory\n");  
                exit(EXIT_FAILURE);  
        }  
  
        buffers[0].length = buffer_size;  
        buffers[0].start = malloc(buffer_size);  
  
        if (!buffers[0].start) {  
                fprintf(stderr, "Out of memory\n");  
                exit(EXIT_FAILURE);  
        }  
}  
  
static void init_mmap(void)  
{  
        struct v4l2_requestbuffers req;  
  
        CLEAR(req);  
  
        req.count = 4;  
        req.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
        req.memory = V4L2_MEMORY_MMAP;  

        if (-1 == xioctl(fd, VIDIOC_REQBUFS, &req)) {  
                if (EINVAL == errno) {  
                        fprintf(stderr, "%s does not support "  
                                 "memory mapping\n", dev_name);  
                        exit(EXIT_FAILURE);  
                } else {  
                        errno_exit("VIDIOC_REQBUFS");  
                }  
        }  
  
        if (req.count < 2) {  
                fprintf(stderr, "Insufficient buffer memory on %s\n",  
                         dev_name);  
                exit(EXIT_FAILURE);  
        }  
        buffers = calloc(req.count, sizeof(*buffers));  
  
        if (!buffers) {  
                fprintf(stderr, "Out of memory\n");  
                exit(EXIT_FAILURE);  
        }  
  
        for (n_buffers = 0; n_buffers < req.count; ++n_buffers) {  
                struct v4l2_buffer buf;  
  
                CLEAR(buf);  
  
                buf.type        = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                buf.memory      = V4L2_MEMORY_MMAP;  
                buf.index       = n_buffers;  
  
                if (-1 == xioctl(fd, VIDIOC_QUERYBUF, &buf))  
                        errno_exit("VIDIOC_QUERYBUF");  
  
                buffers[n_buffers].length = buf.length;
                printf("buffers[%d].length=%d\n", n_buffers, buffers[n_buffers].length);
                buffers[n_buffers].start =  
                        mmap(NULL /* start anywhere */,  
                              buf.length,  
                              PROT_READ | PROT_WRITE /* required */,  
                              MAP_SHARED /* recommended */,  
                              fd, buf.m.offset);  
  
                if (MAP_FAILED == buffers[n_buffers].start)  
                        errno_exit("mmap");  
        }  
}  
  
static void init_userp(unsigned int buffer_size)  
{  
        struct v4l2_requestbuffers req;  
  
        CLEAR(req);  
  
        req.count  = 4;  
        req.type   = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
        req.memory = V4L2_MEMORY_USERPTR;  
  
        if (-1 == xioctl(fd, VIDIOC_REQBUFS, &req)) {  
                if (EINVAL == errno) {  
                        fprintf(stderr, "%s does not support "  
                                 "user pointer i/o\n", dev_name);  
                        exit(EXIT_FAILURE);  
                } else {  
                        errno_exit("VIDIOC_REQBUFS");  
                }  
        }  
  
        buffers = calloc(4, sizeof(*buffers));  
  
        if (!buffers) {  
                fprintf(stderr, "Out of memory\n");  
                exit(EXIT_FAILURE);  
        }  
  
        for (n_buffers = 0; n_buffers < 4; ++n_buffers) {  
                buffers[n_buffers].length = buffer_size;  
                buffers[n_buffers].start = malloc(buffer_size);  
  
                if (!buffers[n_buffers].start) {  
                        fprintf(stderr, "Out of memory\n");  
                        exit(EXIT_FAILURE);  
                }  
        }  
}  
  
/* five operations 
 * step1 : cap :query camera's capability and check it(is a video device? is it support read? is it support streaming?) 
 * step2 : cropcap:set cropcap's type and get cropcap by VIDIOC_CROPCAP 
 * step3 : set crop parameter by VIDIOC_S_CROP (such as frame type and angle) 
 * step4 : set fmt 
 * step5 : mmap 
 */  
static void init_device(void)  
{  
        struct v4l2_capability cap;  
        struct v4l2_cropcap cropcap;  
        struct v4l2_crop crop;  
        struct v4l2_format fmt;  
        unsigned int min; 

    
        if (-1 == xioctl(fd, VIDIOC_QUERYCAP, &cap)) {  
                if (EINVAL == errno) {  
                        fprintf(stderr, "%s is no V4L2 device\n",  
                                 dev_name);  
                        exit(EXIT_FAILURE);  
                } else {  
                        errno_exit("VIDIOC_QUERYCAP");  
                }  
        }  
        if (!(cap.capabilities & V4L2_CAP_VIDEO_CAPTURE)) {  
                fprintf(stderr, "%s is no video capture device\n",  
                         dev_name);  
                exit(EXIT_FAILURE);  
        }  

        switch (io) {  
        case IO_METHOD_READ:  
                if (!(cap.capabilities & V4L2_CAP_READWRITE)) {  
                        fprintf(stderr, "%s does not support read i/o\n",  
                                 dev_name);  
                        exit(EXIT_FAILURE);  
                }  
                break;  
  
        case IO_METHOD_MMAP:  
        case IO_METHOD_USERPTR:  
                if (!(cap.capabilities & V4L2_CAP_STREAMING)) {  
                        fprintf(stderr, "%s does not support streaming i/o\n",  
                                 dev_name);  
                        exit(EXIT_FAILURE);  
                }  
                break;  
        }  
  
  
        /* Select video input, video standard and tune here. */  
        CLEAR(cropcap);  
  
        cropcap.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
        /* if device support cropcap's type then set crop */  
        if (0 == xioctl(fd, VIDIOC_CROPCAP, &cropcap)) {  
                crop.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
                crop.c = cropcap.defrect; /* reset to default */  
  
                if (-1 == xioctl(fd, VIDIOC_S_CROP, &crop)) {  
                        switch (errno) {  
                        case EINVAL:  
                                /* Cropping not supported. */  
                                break;  
                        default:  
                                /* Errors ignored. */  
                                break;  
                        }  
                }  
        } else {  
                /* Errors ignored. */  
        }  
  
  
        CLEAR(fmt);  
  
        fmt.type = V4L2_BUF_TYPE_VIDEO_CAPTURE;  
        if (force_format) {  
                fmt.fmt.pix.width       = 640;  
                fmt.fmt.pix.height      = 480;  
                fmt.fmt.pix.pixelformat = V4L2_PIX_FMT_YUYV;  
                fmt.fmt.pix.field       = V4L2_FIELD_INTERLACED; 
  
          printf("set %d*%d YUYV format\n", fmt.fmt.pix.width, fmt.fmt.pix.height);
                if (-1 == xioctl(fd, VIDIOC_S_FMT, &fmt))  
                        errno_exit("VIDIOC_S_FMT");  
  
                /* Note VIDIOC_S_FMT may change width and height. */  
        } else {  
                /* Preserve original settings as set by v4l2-ctl for example */  
                if (-1 == xioctl(fd, VIDIOC_G_FMT, &fmt))  
                        errno_exit("VIDIOC_G_FMT");  
        }  
  
        /* Buggy driver paranoia. */  
        min = fmt.fmt.pix.width * 2;  
        if (fmt.fmt.pix.bytesperline < min)  
                fmt.fmt.pix.bytesperline = min;  
        min = fmt.fmt.pix.bytesperline * fmt.fmt.pix.height;  
        if (fmt.fmt.pix.sizeimage < min)  
                fmt.fmt.pix.sizeimage = min;  
  
        switch (io) {  
        case IO_METHOD_READ:  
                init_read(fmt.fmt.pix.sizeimage);  
                break;  
  
        case IO_METHOD_MMAP:  
                init_mmap();  
                break;  
  
        case IO_METHOD_USERPTR:  
                init_userp(fmt.fmt.pix.sizeimage);  
                break;  
        }  
}  
  
/* 
 * close (fd) 
 */  
static void close_device(void)  
{  
        if (-1 == close(fd))  
                errno_exit("close");  
  
        fd = -1;  
}  
  
/* three operations 
 * step 1 : check dev_name and st_mode 
 * step 2 : open(device) 
 */  
static void open_device(void)  
{  
        struct stat st;  
  
        if (-1 == stat(dev_name, &st)) {  
                fprintf(stderr, "Cannot identify '%s': %d, %s\n",  
                         dev_name, errno, strerror(errno));  
                exit(EXIT_FAILURE);  
        }  
  
        if (!S_ISCHR(st.st_mode)) {  
                fprintf(stderr, "%s is no device\n", dev_name);  
                exit(EXIT_FAILURE);  
        }  
  
        fd = open(dev_name, O_RDWR /* required */ | O_NONBLOCK, 0);  
  
        if (-1 == fd) {  
                fprintf(stderr, "Cannot open '%s': %d, %s\n",  
                         dev_name, errno, strerror(errno));  
                exit(EXIT_FAILURE);  
        }  
}  
  
static void usage(FILE *fp, int argc, char **argv)  
{  
        fprintf(fp,  
                 "Usage: %s [options]\n\n"  
                 "Version 1.3\n"  
                 "Options:\n"  
                 "-d | --device name   Video device name [%s]\n"  
                 "-h | --help          Print this message\n"  
                 "-m | --mmap          Use memory mapped buffers [default]\n"  
                 "-r | --read          Use read() calls\n"  
                 "-u | --userp         Use application allocated buffers\n"  
                 "-o | --output        Outputs stream to stdout\n"  
                 "-f | --format        Force format to 640x480 YUYV\n"  
                 "-c | --count         Number of frames to grab [%i]\n"  
                 "",  
                 argv[0], dev_name, frame_count);  
}  
  
static const char short_options[] = "d:hmruofc:";  
  
static const struct option  
long_options[] = {  
        { "device", required_argument, NULL, 'd' },  
        { "help",   no_argument,       NULL, 'h' },  
        { "mmap",   no_argument,       NULL, 'm' },  
        { "read",   no_argument,       NULL, 'r' },  
        { "userp",  no_argument,       NULL, 'u' },  
        { "output", no_argument,       NULL, 'o' },  
        { "format", no_argument,       NULL, 'f' },  
        { "count",  required_argument, NULL, 'c' },  
        { 0, 0, 0, 0 }  
};  
  
int main(int argc, char **argv)  
{  
        dev_name = "/dev/video0";  
  
        for (;;) {  
                int idx;  
                int c;  
  
                c = getopt_long(argc, argv,  
                                short_options, long_options, &idx);  
  
                if (-1 == c)  
                        break;  
  
                switch (c) {  
                case 0: /* getopt_long() flag */  
                        break;  
  
                case 'd':  
                        dev_name = optarg;  
                        break;  
  
                case 'h':  
                        usage(stdout, argc, argv);  
                        exit(EXIT_SUCCESS);  
  
                case 'm':  
                        io = IO_METHOD_MMAP;  
                        break;  
  
                case 'r':  
                        io = IO_METHOD_READ;  
                        break;  
  
                case 'u':  
                        io = IO_METHOD_USERPTR;  
                        break;  
  
                case 'o':  
                        out_buf++;  
                        break;  
  
                case 'f':  
                        force_format++;  
                        break;  
  
                case 'c':  
                        errno = 0;  
                        frame_count = strtol(optarg, NULL, 0);  
                        if (errno)  
                                errno_exit(optarg);  
                        break;  
  
                default:  
                        usage(stderr, argc, argv);  
                        exit(EXIT_FAILURE);  
                }  
        }  
  
        open_device();  
        init_device(); 

        start_capturing();  
        mainloop();  
        stop_capturing();  
        uninit_device(); 
        close_device();  
        fprintf(stderr, "\n");  
        return 0;  
}


```

调用逻辑：

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/Driver/Pic/Camera/V4L2/Camera%2010-V4L2%20%E5%BA%94%E7%94%A8%E5%B1%82%E8%B0%83%E7%94%A8%E6%AC%A1%E5%BA%8F.png">

# 内核态




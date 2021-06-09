Show code directly.

```
enum log_level{
    LOG_OFF     = 0,
    LOG_INFO    = 1,
    LOG_DEBUG   = 2,
    LOG_ON      = 3,
};

int g_current_dbg_level = LOG_OFF;

#define DEBUG(Level, _fmt, ...) \
{\
    if(Level < g_current_dbg_level) \
    {\
        printf(_fmt, ##__VA_ARGS__); \
    } \
}
#define PRINT_D(Fmt, ...)   \
{\
    DEBUG(LOG_DEBUG, Fmt, ##__VA_ARGS__);\
}
```

# Window

```
#define LOU_P(_fmt, ...) printf("[LOUIS]%s, %d - "##_fmt, __FUNCTION__, __LINE__, ##__VA_ARGS__)
```

# Linux 

```
#define __output(...) \
    printf(__VA_ARGS__);
 
#define __format(__fmt__) "%s(%d)-<%s>: " __fmt__ "\n"
 
#define TRACE_CMH(__fmt__, ...) \
    __output(__format(__fmt__), __FILE__, __LINE__, __FUNCTION__, ##__VA_ARGS__);
```

## 使用的宏

　　1) __VA_ARGS__   是一个可变参数的宏，这个可宏是新的C99规范中新增的，目前似乎gcc和VC6.0之后的都支持（VC6.0的编译器不支持）。宏前面加上##的作用在于，当可变参数的个数为0时，这里的##起到把前面多余的","去掉的作用。
　　2) __FILE__    宏在预编译时会替换成当前的源文件名
　　3) __LINE__   宏在预编译时会替换成当前的行号
　　4) __FUNCTION__   宏在预编译时会替换成当前的函数名称

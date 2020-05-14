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

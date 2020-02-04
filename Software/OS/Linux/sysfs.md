# 1. sysfs接口函数DEVICE_ATTR和sysfs_create_group

在调试驱动，可能需要对驱动里的某些变量进行读写，或函数调用。可通过sysfs接口创建驱动对应的属性，使得可以在用户空间通过sysfs接口的show和store函数与硬件交互；

 

Sysfs接口可通过sysfs_create_group()来创建，如果设备驱动要创建，需要用到函数宏DEVICE_ATTR；

另外总线对应**BUS_ATTR**、设备驱动对应**DRIVER_ATTR**、类(class)对应**CLASS_ATTR**，均在kernel/include/linux/device.h下定义：

```
//下面的show和store只是简单举例
static ssize_t gpio_show(struct device *d, struct device_attribute*attr, char *buf)
{
       printk("gpio_show()\n");
       returnpr_info("store\n");
}
 
static ssize_t gpio_store(struct device *d, struct device_attribute *attr,const char *buf,size_t count)
{
       printk("gpio_store()\n");
       returnpr_info("store\n");
}
 
//用DEVICE_ATTR宏创建属性gpio文件，如果show()或是store()没有功能，就以NULL代替
static DEVICE_ATTR(gpio, S_IWUSR |S_IRUGO, gpio_show, gpio_store);
 
//属性结构体数组最后一项必须以NULL结尾。
static struct attribute *gpio_attrs[] = {
       &dev_attr_gpio.attr,
       NULL
};
```

## DEVICE_ATTR：

DEVICE_ATTR 的定义DEVICE_ATTR(_name,_mode, _show, _store);可知这里gpio是name，mode是S_IWUSR |S_IRUGO，读操作_show是gpio_show函数，写操作_store 是gpio_store函数；

因为：

```
#define DEVICE_ATTR(_name, _mode, _show, _store) \
    struct device_attribute dev_attr_##_name = __ATTR(_name, _mode, _show, _store)
```

device_attribute：

```
/* interface for exporting device attributes */
struct device_attribute {
    struct attribute    attr;
    ssize_t (*show)(struct device *dev, struct device_attribute *attr,
            char *buf);
    ssize_t (*store)(struct device *dev, struct device_attribute *attr,
             const char *buf, size_t count);
};
```

Mode是权限位，在kernel/include/uapi/linux/stat.h；
```
#define S_IRWXU 00700 //用户可读写和执行
#define S_IRUSR 00400//用户可读
#define S_IWUSR 00200//用户可写
#define S_IXUSR 00100//用户可执行
 
#define S_IRWXG 00070//用户组可读写和执行
#define S_IRGRP 00040//用户组可读
#define S_IWGRP 00020//用户组可写
#define S_IXGRP 00010//用户组可执行
 
#define S_IRWXO 00007//其他可读写和执行
#define S_IROTH 00004//其他可读
#define S_IWOTH 00002//其他可写
#define S_IXOTH 00001//其他可执行
```

##device_attribute结构体

为了使对属性的读写变得有意义，一般将attribute结构嵌入到其他数据结构中。子系统通常都会定义自己的属性结构，并且提供添加和删除属性文件的包装函数，比如设备属性结构体定义：

```
/* interface for exporting device attributes */  
struct device_attribute {  
       struct attribute    attr;  
       ssize_t (*show)(structdevice *dev, struct device_attribute *attr,  
                     char*buf);  
       ssize_t (*store)(structdevice *dev, struct device_attribute *attr,  
                      const char *buf, size_t count);  
};
```



# 2.     定义attribute属性结构体数组到属性组中：


```
static const struct attribute_group gpio_attr_grp = {
       .attrs = gpio_attrs,
}
//我们这里只有一个属性结构体数组只有一个成员，可以有多个，比如：
static struct attribute *gpio_keys_attrs[] = {
       &dev_attr_keys.attr,
       &dev_attr_switches.attr,
       &dev_attr_disabled_keys.attr,
       &dev_attr_disabled_switches.attr,
       &dev_attr_test.attr,
       NULL,
};
```
属性attribute结构体定义：
```
struct attribute {  
       const char           *name;  
       umode_t                     mode;  
#ifdef CONFIG_DEBUG_LOCK_ALLOC  
       bool                     ignore_lockdep:1;  
       struct lock_class_key *key;  
       struct lock_class_key skey;  
#endif  
};
```

创建sysfs接口后，就可以在adb shell 终端查看到和操作接口了。当我们将数据 echo 到接口中时，在用户空间完成了一次 write 操作，对应到 kernel ，调用了驱动中的”store”。当我们cat一个接口时则会调用”show” 。这样就建立了 android 层到 kernel 的桥梁，操作的细节在”show”和”store” 中完成的。

# 3.     创建属性文件的sysfs接口：

```
ret = sysfs_create_group(&pdev->dev.kobj,&gpio_attr_grp);
sysfs_create_group()在kobj目录下创建一个属性集合，并显示集合中的属性文件。如果文件已存在，会报错。
 
//删除接口
sysfs_remove_group(&pdev->dev.kobj,&gpio_keys_attr_group);
sysfs_remove_group()在kobj目录下删除一个属性集合，并删除集合中的属性文件
```



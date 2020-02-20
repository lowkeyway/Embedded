# container_of(ptr,type,member)
+ **题目**
描述下面XXX 这个宏的作用：
#define offsetof(TYPE,MEMBER)((size_t)&((TYPE*)0)->MEMBER)
#define XXX(ptr,type,member({\
const typeof(((type*)0)->member)*__mptr=(ptr);\
(type*)(char*)__mptr – offsetof(type,member));})

+ **分析**

很明显，这里的XXX是从linux里面抄出来的container_of
先看第一个宏
```C
#define offsetof(TYPE,MEMBER)((size_t)&((TYPE*)0)->MEMBER)
```
 (TYPE*)0 表示把0地址转换成指向TYPE类型的指针，&((TYPE*)0)->MEMBER就表示TYPE类型下MEMBER成员的地址，因为TYPE的地址是0，所以这个地址就是MEMBER相对于TYPE类型的偏移量。

再看第二个宏
```C
#define XXX(ptr,type,member({\
const typeof(((type*)0)->member)*__mptr=(ptr);\
(type*)(char*)__mptr – offsetof(type,member));})
```
第一句话看typeof关键字里面的信息((type*)0)->member，根据上面的讲解，它代表了从0指针指向的一个type类型下的member成员变量, 所以第一句话就代表把ptr这个指针地址赋值给了指向type.member的指针__mptr，并且这个type的首地址应该为0；
第二句话是要把__mptr先转换成(char*)这样的目的是在加减运算中数值的偏移为1，通过把__mptr数值跟member相对于type的偏移量相减，就应该得到这个member成员对应的type类型的首地址，当然最后再把结果强转成(type*).

+ **答案**

这两个宏分别代表：
1. 计算成员参数在本结构体中的偏移量
2. 在知道成员参数指针地址的情况下，得到该成员参数对应结构体的首地址

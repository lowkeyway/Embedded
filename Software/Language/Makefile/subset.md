# 一、替换函数

subst是一个替换函数，这个函数有三个参数，第一个参数是被替换字串，第二个参数是替换字串，第三个参数是替换操作作用的字串。

例：    
```
    comma:= ,
    empty:=    
    space:= $(empty) $(empty)
    foo:=a b c

    bar:= $(subst $(space),$(comma),$(foo)) 
```

这个函数也就是把$(foo)中的空格替换成逗号，所以$(bar)的值是 "a,b,c"


# 二、字符串替换

$(subst <aa>,<bb>,<text>)

把text中的aa替换成bb

例：

```
$(subst ee,EE,feet on the street)
```

把“ feet on the street”中的“ ee”替换成“ EE”，返回结果是“ fEEt on the strEEt”。

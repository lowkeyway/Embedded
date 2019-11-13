# lambda

## 1. 什么是lambda

简单来说，编程中提到的 lambda 表达式，通常是在需要一个函数，但是又不想费神去命名一个函数的场合下使用，也就是指匿名函数。

举例子说明：
```Python
g = lambda x : x+1
g(1)
```

输出：
```Python
2
```

可以这样认为,lambda作为一个表达式，定义了一个匿名函数，上例的代码x为入口参数，x+1为函数体，用函数来表示为：

```Python
def g(x):
   return x+1
g(1)
```


## 2. lambda的其他应用：filter, map, reduce
```Python
foo = [2, 18, 9, 22, 17, 24, 8, 12, 27]
print(list(filter(lambda x: x % 3 == 0, foo)))
print(list(map(lambda x: x * 2 + 10, foo)))
print([x * 2 + 10 for x in foo])
```

输出：
```Python
[18, 9, 24, 12, 27]
[14, 46, 28, 54, 44, 58, 26, 34, 64]
[14, 46, 28, 54, 44, 58, 26, 34, 64]
```


需要注意的是lamba函数只会提升代码的简洁度，而不是运算效率。但是带来的是代码的可读性下降。


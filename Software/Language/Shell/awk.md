# 1. 简介

awk 是一种处理文本文件的语言，是一个强大的文本分析工具。

awk 其实不仅仅是工具软件，还是一种编程语言。

awk 是以文件的一行内容为处理单位的。awk读取一行内容，然后根据指定条件判断是否处理此行内容，若此行文本符合条件，则按照动作处理文本，否则跳过此行文本，读取下一行进行判断。


# 2. 基本用法

condition：条件。若此行文本符合该条件，则按照 action 处理此行文本。不添加条件时则处理每一行文本；

action：动作。按照动作处理符合要求的内容。一般用于打印指定的内容信息；

+ 注意下面的引号为英文的单引号

## 2.1 处理指定文件的内容

awk   'condition { action }'   filename

## 2.2 处理某个命令的执行结果

command | awk ' condition { action }'

## 2.3  常用参数

### 2.3.1  F（指定字段分隔符）

默认使用空格作为分隔符。

```
lowkeyway@lowkeyway:~$ echo "aa bb cc dd ee" | awk '{printf $2}'
bb
lowkeyway@lowkeyway:~$ echo "aa bb cc dd ee" | awk -F 'bb' '{printf $2}'
 cc dd ee

```

# 3. 变量

## 3.1  FS（字段分隔符）

默认是空格和制表符。

**$0 表示当前整行内容，$1，$2 表示第一个字段，第二个字段**

```
lowkeyway@lowkeyway:~/code/test$ echo "aa bb cc dd ee" | awk '{print $0}'
aa bb cc dd ee
lowkeyway@lowkeyway:~/code/test$ echo "aa bb cc dd ee" | awk '{print $1}'
aa
lowkeyway@lowkeyway:~/code/test$ echo "aa bb cc dd ee" | awk '{print $3}'
cc
```

## 3.2 NF（当前行的字段个数）

$NF就代表最后一个字段，$(NF-1)代表倒数第二个字段
```
lowkeyway@lowkeyway:~/code$ ll
总用量 20
drwxr-x---  5 lowkeyway lowkeyway 4096 2月  19 10:54 ./
drwxr-xr-x 24 lowkeyway lowkeyway 4096 2月  26 15:31 ../
drwxr-x---  8 lowkeyway lowkeyway 4096 2月  26 11:36 ldd3/
drwxr-x---  3 lowkeyway lowkeyway 4096 2月  19 10:54 sysfs/
drwxr-x---  2 lowkeyway lowkeyway 4096 2月  25 21:24 test/
lowkeyway@lowkeyway:~/code$ ll | awk '{print $NF}'
20
./
../
ldd3/
sysfs/
test/
lowkeyway@lowkeyway:~/code$ ll | awk '{print $(NF-1)}'
总用量
10:54
15:31
11:36
10:54
21:24
lowkeyway@lowkeyway:~/code$ ll | awk '{print NR") ", $0}'
1)  总用量 20
2)  drwxr-x---  5 lowkeyway lowkeyway 4096 2月  19 10:54 ./
3)  drwxr-xr-x 24 lowkeyway lowkeyway 4096 2月  26 15:31 ../
4)  drwxr-x---  8 lowkeyway lowkeyway 4096 2月  26 11:36 ldd3/
5)  drwxr-x---  3 lowkeyway lowkeyway 4096 2月  19 10:54 sysfs/
6)  drwxr-x---  2 lowkeyway lowkeyway 4096 2月  25 21:24 test
```
逗号表示输出的变量之间用空格分隔；

右括号必需使用 双引号 才可以原样输出

## 3.4 FILENAME(当前文件名)

```
lowkeyway@lowkeyway:~/code/test$ awk '{print FILENAME, NR ")", $0}' awk.txt 
awk.txt 1) aaa
awk.txt 2) bbb
awk.txt 3) ccc
awk.txt 4) ddd
lowkeyway@lowkeyway:~/code/test$ cat awk.txt | awk '{print FILENAME, NR")", $0}'
- 1) aaa
- 2) bbb
- 3) ccc
- 4) ddd
```

awk   '{ condition  action }'   filename 这种形式时可以打印文件名；

通过 |（管道符）读取内容时打印的是 - 


## 3.5 其他变量

+ RS：行分隔符，用于分割每一行，默认是换行符。
+ OFS：输出字段的分隔符，用于打印时分隔字段，默认为空格。
+ ORS：输出记录的分隔符，用于打印时分隔记录，默认为换行符。
+ OFMT：数字输出的格式，默认为％.6g。


https://www.cnblogs.com/kanwan/p/11962607.html

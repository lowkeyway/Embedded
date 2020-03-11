# Grep

Linux 中的搜索三剑客之一grep，用来搜索一个文件中的关键字。

+ -a 或 --text : 不要忽略二进制的数据。
+ -A<显示行数> 或 --after-context=<显示行数> : 除了显示符合范本样式的那一列之外，并显示该行之后的内容。
+ -b 或 --byte-offset : 在显示符合样式的那一行之前，标示出该行第一个字符的编号。
+ -B<显示行数> 或 --before-context=<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前的内容。
+ -c 或 --count : 计算符合样式的列数。
+ -C<显示行数> 或 --context=<显示行数>或-<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前后的内容。
+ -d <动作> 或 --directories=<动作> : 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。
+ -e<范本样式> 或 --regexp=<范本样式> : 指定字符串做为查找文件内容的样式。
+ -E 或 --extended-regexp : 将样式为延伸的正则表达式来使用。
+ -f<规则文件> 或 --file=<规则文件> : 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。
+ -F 或 --fixed-regexp : 将样式视为固定字符串的列表。
+ -G 或 --basic-regexp : 将样式视为普通的表示法来使用。
+ -h 或 --no-filename : 在显示符合样式的那一行之前，不标示该行所属的文件名称。
+ -H 或 --with-filename : 在显示符合样式的那一行之前，表示该行所属的文件名称。
+ -i 或 --ignore-case : 忽略字符大小写的差别。
+ -l 或 --file-with-matches : 列出文件内容符合指定的样式的文件名称。
+ -L 或 --files-without-match : 列出文件内容不符合指定的样式的文件名称。
+ -n 或 --line-number : 在显示符合样式的那一行之前，标示出该行的列数编号。
+ -o 或 --only-matching : 只显示匹配PATTERN 部分。
+ -q 或 --quiet或--silent : 不显示任何信息。
+ -r 或 --recursive : 此参数的效果和指定"-d recurse"参数相同。
+ -s 或 --no-messages : 不显示错误信息。
+ -v 或 --revert-match : 显示不包含匹配文本的所有行。
+ -V 或 --version : 显示版本信息。
+ -w 或 --word-regexp : 只显示全字符合的列。
+ -x --line-regexp : 只显示全列符合的列。
+ -y : 此参数的效果和指定"-i"参数相同。 


## 其中比较常用的是： 

**-i 或 --ignore-case : 忽略字符大小写**  
**-n 或 --line-number : 显示行号**  
**-v 或 --revert-match : 显示不包含匹配文本的所有行**  
**-E 或 --extended-regexp : 将样式为延伸的正则表达式来使用，同egrep**  
**-r 或 --recursive : 递归查找**  
**-l 或 --file-with-matches : 只列出文件名**  
**-w 或 --word-regexp : 全词匹配**  
**--color=auto: 匹配到的输出结果变色**  

## 示例

```
lowkeyway@lowkeyway:~/code/test$ grep -irnw foo_test  
匹配到二进制文件 .test.c.swp  
test.c:35:static void foo_test(char *msg, ...)  
test.c:110:	//foo_test("Hello", "World","!", "");  
匹配到二进制文件 a.out  

lowkeyway@lowkeyway:~/code/test$ find . -name test.c | xargs grep -irnw foo_test  
35:static void foo_test(char *msg, ...)  
110:	//foo_test("Hello", "World","!", "");  

```

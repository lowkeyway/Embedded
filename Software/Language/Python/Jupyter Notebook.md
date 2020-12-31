# 一、什么是Jupyter Notebook？

## 1. 简介

```
Jupyter Notebook是基于网页的用于交互计算的应用程序。其可被应用于全过程计算：开发、文档编写、运行代码和展示结果。

——Jupyter Notebook官方介绍
```

简而言之，Jupyter Notebook是以网页的形式打开，可以在网页页面中直接编写代码和运行代码，代码的运行结果也会直接在代码块下显示的程序。如在编程过程中需要编写说明文档，可在同一个页面中直接编写，便于作及时的说明和解释。

## 2. 组成部分

+ **网页应用**  
网页应用即基于网页形式的、结合了编写说明文档、数学公式、交互计算和其他富媒体形式的工具。简言之，网页应用是可以实现各种功能的工具。

+ **文档**  
即Jupyter Notebook中所有交互计算、编写说明文档、数学公式、图片以及其他富媒体形式的输入和输出，都是以文档的形式体现的。  
这些文档是保存为后缀名为.ipynb的JSON格式文件，不仅便于版本控制，也方便与他人共享。  
此外，文档还可以导出为：HTML、LaTeX、PDF等格式。  

## 3. Jupyter Notebook的主要特点

+ 编程时具有语法高亮、缩进、tab补全的功能。
+ 可直接通过浏览器运行代码，同时在代码块下方展示运行结果。
+  以富媒体格式展示计算结果。富媒体格式包括：HTML，LaTeX，PNG，SVG等。
+ 对代码编写说明文档或语句时，支持Markdown语法。
+ 支持使用LaTeX编写数学性说明。


# 二、安装Jupyter Notebook

## 1 安装前提
安装Jupyter Notebook的前提是需要安装了Python（3.3版本及以上，或2.7版本）。

## 2 使用Anaconda安装

如果你是小白，那么建议你通过安装Anaconda来解决Jupyter Notebook的安装问题，因为Anaconda已经自动为你安装了Jupter Notebook及其他工具，还有python中超过180个科学包及其依赖项。  
常规来说，安装了Anaconda发行版时已经自动为你安装了Jupyter Notebook的，但如果没有自动安装，那么就在终端（Linux或macOS的“终端”，Windows的“Anaconda Prompt”，以下均简称“终端”）中输入以下命令安装：

```
conda install jupyter notebook
```

## 3 使用pip命令安装
如果你是有经验的Python玩家，想要尝试用pip命令来安装Jupyter Notebook，那么请看以下步骤吧！接下来的命令都输入在终端当中的噢！  

### 3.1 把pip升级到最新版本
+ Python 3.x
```
pip3 install --upgrade pip
```

+ Python 2.x
```
pip install --upgrade pip
```
**注意：**老版本的pip在安装Jupyter Notebook过程中或面临依赖项无法同步安装的问题。因此强烈建议先把pip升级到最新版本。

### 3.2 安装Jupyter Notebook

+ Python 3.x
```
pip3 install jupyter
```

+ Python 2.x
```
pip install jupyter
```

# 三、使用

## 1 快捷键
快捷方式是 Jupyter Notebooks 最大的优势之一。当你想运行任意代码块时，只需要按 Ctrl+Enter 就行了。Jupyter Notebooks 提供了很多键盘快捷键，可以帮助我们节省很多时间。

下面是我们手动选择的一些对你的上手会有莫大帮助的快捷方式。我强烈建议你在阅读本文时逐一尝试一下。未来你会离不开它们的！

Jupyter Notebooks 提供了两种不同的键盘输入模式——命令和编辑。命令模式是将键盘和笔记本层面的命令绑定起来，并且由带有蓝色左边距的灰色单元边框表示。编辑模式让你可以在活动单元中输入文本（或代码），用绿色单元边框表示。

**你可以分别使用 Esc 和 Enter 在命令模式和编辑模式之间跳跃。现在就试试看吧！**

进入命令模式之后（此时你没有活跃单元），你可以尝试以下快捷键：
+ A 会在活跃单元之上插入一个新的单元，B 会在活跃单元之下插入一个新单元。
+ 连续按两次 D，可以删除一个单元。
+ 撤销被删除的单元，按 Z。
+ Y 会将当前活跃的单元变成一个代码单元。
+ 按住 Shift +上或下箭头可选择多个单元。在多选模式时，按住 Shift + M 可合并你的选择。
+ 按 F 会弹出「查找和替换」菜单。
+ 处于编辑模式时（在命令模式时按 Enter 会进入编辑模式），你会发现下列快捷键很有用：
+ Ctrl + Home 到达单元起始位置。
+ Ctrl + S 保存进度。
+ 如之前提到的，Ctrl + Enter 会运行你的整个单元块。
+ Alt + Enter 不止会运行你的单元块，还会在下面添加一个新单元。
+ Ctrl + Shift + F 打开命令面板。
+ 要查看键盘快捷键完整列表，可在命令模式按「H」或进入「Help > Keyboard Shortcuts」。你一定要经常看这些快捷键，因为常会添加新的。



## 2. 有用的 Jupyter Notebooks 扩展

扩展/附加组件是一种非常有生产力的方式，能帮你提升在 Jupyter Notebooks 上的生产力。我认为安装和使用扩展的最好工具之一是 Nbextensions。在你的机器上安装它只需简单两步（也有其它安装方法，但我认为这个最方便）：

+ 第一步：从 pip 安装它：

pip install jupyter_contrib_nbextensions

+ 第二步：安装相关的 JavaScript 和 CSS 文件：

jupyter contrib nbextension install –user

完成这个工作之后，你会在你的 Jupyter Notebook 主页顶部看见一个 Nbextensions 选项卡。点击一下，你就能看到很多可在你的项目中使用的扩展。

要启用某个扩展，只需勾选它即可。下面我给出了 4 个我觉得最有用的扩展：

1. Code prettify：它能重新调整代码块内容的格式并进行美化。
2. Printview：这个扩展会添加一个工具栏按钮，可为当前笔记本调用 jupyter nbconvert，并可以选择是否在新的浏览器标签页显示转换后的文件。
3. Scratchpad：这会添加一个暂存单元，让你可以无需修改笔记本就能运行你的代码。当你想实验你的代码但不想改动你的实时笔记本时，这会是一个非常方便的扩展。
4. Table of Contents (2)：这个很棒的扩展可以收集你的笔记本中的所有标题，并将它们显示在一个浮动窗口中。
这只是少量几个扩展。我强烈建议你查看完整扩展列表并实验它们的功能。


参考： https://zhuanlan.zhihu.com/p/33105153

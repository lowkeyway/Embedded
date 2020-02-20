# 一、Git工作流程

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Others/Pic/Git/git%E5%B7%A5%E4%BD%9C%E5%8C%BA%E5%9F%9F.png">

以上包括一些简单而常用的命令，但是先不关心这些，先来了解下面这4个专有名词。

Workspace：工作区

Index / Stage：暂存区

Repository：仓库区（或本地仓库）

Remote：远程仓库

## 工作区

程序员进行开发改动的地方，是你当前看到的，也是最新的。

平常我们开发就是拷贝远程仓库中的一个分支，基于该分支进行开发。在开发过程中就是对工作区的操作。

## 暂存区

.git目录下的index文件, 暂存区会记录git add添加文件的相关信息(文件名、大小、timestamp...)，不保存文件实体, 通过id指向每个文件实体。可以使用git status查看暂存区的状态。暂存区标记了你当前工作区中，哪些内容是被git管理的。

当你完成某个需求或功能后需要提交到远程仓库，那么第一步就是通过git add先提交到暂存区，被git管理。

## 本地仓库

保存了对象被提交 过的各个版本，比起工作区和暂存区的内容，它要更旧一些。

git commit后同步index的目录树到本地仓库，方便从下一步通过git push同步本地仓库与远程仓库的同步。

## 远程仓库

远程仓库的内容可能被分布在多个地点的处于协作关系的本地仓库修改，因此它可能与本地仓库同步，也可能不同步，但是它的内容是最旧的。

## 小结

任何对象都是在工作区中诞生和被修改；

任何修改都是从进入index区才开始被版本控制；

只有把修改提交到本地仓库，该修改才能在仓库中留下痕迹；

与协作者分享本地的修改，可以把它们push到远程仓库来共享。

下面这幅图更加直接阐述了四个区域之间的关系，可能有些命令不太清楚，没关系，下部分会详细介绍。

<img src=https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Others/Pic/Git/git%E5%B7%A5%E4%BD%9C%E5%8C%BA%E5%9F%9F%E5%92%8C%E5%91%BD%E4%BB%A4%E5%AF%B9%E5%BA%94%E5%9B%BE.png>

二、常用Git命令

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Others/Pic/Git/git%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%E9%80%9F%E6%9F%A5%E8%A1%A8.png">

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

# 二、常用Git命令

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Others/Pic/Git/git%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%E9%80%9F%E6%9F%A5%E8%A1%A8.png">


## 1. 查看log
git log --name-status 每次修改的文件列表, 显示状态

git log --name-only 每次修改的文件列表

git log --stat 每次修改的文件列表, 及文件修改的统计

git whatchanged 每次修改的文件列表

git whatchanged --stat 每次修改的文件列表, 及文件修改的统计

git show 显示最后一次的文件改变的具体内容

## 2. 打patch
git diff > ../sync.patch # 生成补丁
git apply ../sync.patch # 打补丁
git apply --check ../sync.patch #测试补丁能否成功

## 3. Git暂存管理
git stash # 暂存

git stash list # 列所有stash

git stash apply # 恢复暂存的内容

git stash drop # 删除暂存区

## 4. 远程仓库

git remote -v # 查看远程服务器地址和仓库名称

git remote show origin # 查看远程服务器仓库状态

git remote add origin git@ github:robbin/robbin_site.git # 添加远程仓库地址

git remote set-url origin git@ github.com:robbin/robbin_site.git # 设置远程仓库地址(用于修改远程仓库地址) 

git remote rm <repository> # 删除远程仓库

创建远程仓库

git clone --bare robbin_site robbin_site.git # 用带版本的项目创建纯版本仓库

scp -r my_project.git git@ git.csdn.net:~ # 将纯仓库上传到服务器上

mkdir robbin_site.git && cd robbin_site.git && git --bare init # 在服务器创建纯仓库

git remote add origin git@ github.com:robbin/robbin_site.git # 设置远程仓库地址

git push -u origin master # 客户端首次提交

git push -u origin develop # 首次将本地develop分支提交到远程develop分支，并且track

git remote set-head origin master # 设置远程仓库的HEAD指向master分支也可以命令设置跟踪远程库和本地库

git branch --set-upstream master origin/master

git branch --set-upstream develop origin/develop


## 5. 查看、添加、提交、删除、找回，重置修改文件

git help <command> # 显示command的help

git show # 显示某次提交的内容 git show $id

git co -- <file> # 抛弃工作区修改

git co . # 抛弃工作区修改

git add <file> # 将工作文件修改提交到本地暂存区

git add . # 将所有修改过的工作文件提交暂存区

git rm <file> # 从版本库中删除文件

git rm <file> --cached # 从版本库中删除文件，但不删除文件

git reset <file> # 从暂存区恢复到工作文件

git reset -- . # 从暂存区恢复到工作文件

git reset --hard # 恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改

git ci <file> git ci . git ci -a # 将git add, git rm和git ci等操作都合并在一起
  做　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　

git ci -am "some comments"

git ci --amend # 修改最后一次提交记录

git revert <$id> # 恢复某次提交的状态，恢复动作本身也创建次提交对象

git revert HEAD # 恢复最后一次提交的状态


git branch -a#查看所有分支。

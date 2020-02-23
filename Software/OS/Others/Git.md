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

### 1.1 查看某一个文件的修改历史

+ 1. git log filename

可以看到fileName相关的commit记录

+ 2. git log -p filenam

可以显示每次提交的diff

+ 3. git show c5e69804bbd9725b5dece57f8cbece4a96b9f80b filename

只看某次提交中的某个文件变化，可以直接加上fileName

+ 4. git log --pretty=oneline 文件路径\文件名

查看某个文件的修改记录



## 2. 打patch
git diff > ../sync.patch # 生成补丁
git apply ../sync.patch # 打补丁
git apply --check ../sync.patch #测试补丁能否成功

## 3. Git暂存管理

git stash save "save message"  : 执行存储时，添加备注，方便查找，只有git stash 也要可以的，但查找时不方便识别。

git stash list  ：查看stash了哪些存储

git stash show ：显示做了哪些改动，默认show第一个存储,如果要显示其他存贮，后面加stash@{$num}，比如第二个 git stash show stash@{1}

git stash show -p : 显示第一个存储的改动，如果想显示其他存存储，命令：git stash show  stash@{$num}  -p ，比如第二个：git stash show  stash@{1}  -p

git stash apply :应用某个存储,但不会把存储从存储列表中删除，默认使用第一个存储,即stash@{0}，如果要使用其他个，git stash apply stash@{$num} ， 比如第二个：git stash apply stash@{1} 

git stash pop ：命令恢复之前缓存的工作目录，将缓存堆栈中的对应stash删除，并将对应修改应用到当前的工作目录下,默认为第一个stash,即stash@{0}，如果要应用并删除其他stash，命令：git stash pop stash@{$num} ，比如应用并删除第二个：git stash pop stash@{1}

git stash drop stash@{$num} ：丢弃stash@{$num}存储，从列表中删除这个存储

git stash clear ：删除所有缓存的stash

## 4. 远程仓库

git remote -v # 查看远程服务器地址和仓库名称

git remote show origin # 查看远程服务器仓库状态

git push origin "本地分支":“远程分支”		//创建远程分支

git push origin :"远程分支"				//删除远程分支

git checkout -b "本地分支" “origin/远程分支”  #从远程分支上闯进本地分支

git push -u origin master # 客户端首次提交

git push -u origin develop # 首次将本地develop分支提交到远程develop分支，并且track

git branch -vv #查看当前分支和远程分支的对应关系。


## 5. 查看、添加、提交、删除、找回，重置修改文件

git help <command> # 显示command的help

git show # 显示某次提交的内容 git show $id

git checkout -- <file> # 抛弃工作区修改

git checkout . # 抛弃工作区修改

git add <file> # 将工作文件修改提交到本地暂存区

git add . # 将所有修改过的工作文件提交暂存区

git rm <file> # 从版本库中删除文件

git rm <file> --cached # 从版本库中删除文件，但不删除文件

git reset <file> # 从暂存区恢复到工作文件

git reset -- . # 从暂存区恢复到工作文件

git reset --hard # 恢复最近一次提交过的状态，即放弃上次提交后的所有本次修改　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　

git commit -am "some comments"

git commit --amend # 修改最后一次提交记录

git revert <$id> # 恢复某次提交的状态，恢复动作本身也创建次提交对象

git revert HEAD # 恢复最后一次提交的状态

git branch -a#查看所有分支。

### 5.1 Git Reset

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Others/Pic/Git/git%20reset.png">

+ **区别：**

--hard：重置位置的同时，直接将 working Tree工作目录、 index 暂存区及 repository 都重置成目标Reset节点的內容,所以效果看起来等同于清空暂存区和工作区。

--soft：重置位置的同时，保留working Tree工作目录和index暂存区的内容，只让repository中的内容和 reset 目标节点保持一致，因此原节点和reset节点之间的【差异变更集】会放入index暂存区中(Staged files)。所以效果看起来就是工作目录的内容不变，暂存区原有的内容也不变，只是原节点和Reset节点之间的所有差异都会放到暂存区中。

--mixed（默认）：重置位置的同时，只保留Working Tree工作目录的內容，但会将 Index暂存区 和 Repository 中的內容更改和reset目标节点一致，因此原节点和Reset节点之间的【差异变更集】会放入Working Tree工作目录中。所以效果看起来就是原节点和Reset节点之间的所有差异都会放到工作目录中。

+ **使用场景:**

--hard：(1) 要放弃目前本地的所有改变時，即去掉所有add到暂存区的文件和工作区的文件，可以执行 git reset -hard HEAD 来强制恢复git管理的文件夹的內容及状态；(2) 真的想抛弃目标节点后的所有commit（可能觉得目标节点到原节点之间的commit提交都是错了，之前所有的commit有问题）。

--soft：原节点和reset节点之间的【差异变更集】会放入index暂存区中(Staged files)，所以假如我们之前工作目录没有改过任何文件，也没add到暂存区，那么使用reset --soft后，我们可以直接执行 git commit 將 index暂存区中的內容提交至 repository 中。为什么要这样呢？这样做的使用场景是：假如我们想合并「当前节点」与「reset目标节点」之间不具太大意义的 commit 记录(可能是阶段性地频繁提交,就是开发一个功能的时候，改或者增加一个文件的时候就commit，这样做导致一个完整的功能可能会好多个commit点，这时假如你需要把这些commit整合成一个commit的时候)時，可以考虑使用reset --soft来让 commit 演进线图较为清晰。总而言之，可以使用--soft合并commit节点。

--mixed（默认）：(1)使用完reset --mixed后，我們可以直接执行 git add 将這些改变果的文件內容加入 index 暂存区中，再执行 git commit 将 Index暂存区 中的內容提交至Repository中，这样一样可以达到合并commit节点的效果（与上面--soft合并commit节点差不多，只是多了git add添加到暂存区的操作）；(2)移除所有Index暂存区中准备要提交的文件(Staged files)，我们可以执行 git reset HEAD 来 Unstage 所有已列入 Index暂存区 的待提交的文件。(有时候发现add错文件到暂存区，就可以使用命令)。(3)commit提交某些错误代码，或者没有必要的文件也被commit上去，不想再修改错误再commit（因为会留下一个错误commit点），可以回退到正确的commit点上，然后所有原节点和reset节点之间差异会返回工作目录，假如有个没必要的文件的话就可以直接删除了，再commit上去就OK了。

## 6. TAG

git tag -a "tag name" -m "注释"			//创建一个本地tag，写注释

git push origin "tag name"				//上传本地tag

git tag -d "tag name"

git push origin :refs/tags/"tag name "		//删除远程tag

<img src="https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Others/Pic/Git/git%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE.png">

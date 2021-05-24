# 一、查看log
  
git log --name-status 每次修改的文件列表, 显示状态  
  
git log --name-only 每次修改的文件列表  
  
git log --stat 每次修改的文件列表, 及文件修改的统计  
  
git whatchanged 每次修改的文件列表  
  
git whatchanged --stat 每次修改的文件列表, 及文件修改的统计  
  
git show 显示最后一次的文件改变的具体内容  
  

  
# 二、打patch
  
git diff > ../sync.patch # 生成补丁  
  
git apply ../sync.patch # 打补丁  
  
git apply --check ../sync.patch #测试补丁能否成功  
  

  
# 三、Git暂存管理  
  
git stash # 暂存  
  
git stash list # 列所有stash  
  
git stash apply # 恢复暂存的内容  
  
git stash drop # 删除暂存区    
  
  
  
# 四、远程仓库
  
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
  
git remote set-head origin master # 设置远程仓库的HEAD指向master分支
  
也可以命令设置跟踪远程库和本地库
  
git branch --set-upstream master origin/master
  
git branch --set-upstream develop origin/develop
  

  
# 五、查看、添加、提交、删除、找回，重置修改文件
  
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
  
git ci <file> git ci . git ci -a # 将git add, git rm和git ci等操作都合并在一起做　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
  
git ci -am "some comments"
  
git ci --amend # 修改最后一次提交记录
  
git revert <$id> # 恢复某次提交的状态，恢复动作本身也创建次提交对象
  
git revert HEAD # 恢复最后一次提交的状态
  

  
git branch -a#查看所有分支。
  

  

  

  
git config --global user.name "你的名字" 让你全部的Git仓库绑定你的名字
  
git config --global user.email "你的邮箱" 让你全部的Git仓库绑定你的邮箱
  
git init 初始化你的仓库
  
git add . 把工作区的文件全部提交到暂存区
  
git add ./<file>/ 把工作区的<file>文件提交到暂存区
  
git commit -m "xxx" 把暂存区的所有文件提交到仓库区，暂存区空空荡荡
  
git remote add origin https://github.com/name/name_cangku.git 把本地仓库与远程仓库连接起来
  
git push -u origin master 把仓库区的主分支master提交到远程仓库里
  
git push -u origin <其他分支> 把其他分支提交到远程仓库
  
git status查看当前仓库的状态
  
git diff 查看文件修改的具体内容
  
git log 显示从最近到最远的提交历史
  
git clone + 仓库地址下载克隆文件
  
git reset --hard + 版本号 回溯版本，版本号在commit的时候与master跟随在一起
  
git reflog 显示命令历史
  
git checkout -- <file> 撤销命令，用版本库里的文件替换掉工作区的文件。我觉得就像是Git世界的ctrl + z
  
git rm 删除版本库的文件
  
git branch 查看当前所有分支
  
git branch <分支名字> 创建分支
  
git checkout <分支名字> 切换到分支
  
git merge <分支名字> 合并分支
  
git branch -d <分支名字> 删除分支,有可能会删除失败，因为Git会保护没有被合并的分支
  
git branch -D + <分支名字> 强行删除，丢弃没被合并的分支
  
git log --graph 查看分支合并图
  
git merge --no-ff <分支名字> 合并分支的时候禁用Fast forward模式,因为这个模式会丢失分支历史信息
  
git stash 当有其他任务插进来时，把当前工作现场“存储”起来,以后恢复后继续工作
  
git stash list 查看你刚刚“存放”起来的工作去哪里了
  
git stash apply 恢复却不删除stash内容
  
git stash drop 删除stash内容
  
git stash pop 恢复的同时把stash内容也删了
  
git remote 查看远程库的信息，会显示origin，远程仓库默认名称为origin
  
git remote -v 显示更详细的信息
  
git pull 把最新的提交从远程仓库中抓取下来，在本地合并,和git push相反
  
git rebase 把分叉的提交历史“整理”成一条直线，看上去更直观
  
git tag 查看所有标签，可以知道历史版本的tag
  
git tag <name> 打标签，默认为HEAD。比如git tag v1.0
  
git tag <tagName> <版本号> 把版本号打上标签，版本号就是commit时，跟在旁边的一串字母数字
  
git show <tagName> 查看标签信息
  
git tag -a <tagName> -m "<说明>" 创建带说明的标签。-a指定标签名，-m指定说明文字
  
git tag -d <tagName> 删除标签
  
git push origin <tagname> 推送某个标签到远程
  
git push origin --tags 一次性推送全部尚未推送到远程的本地标签
  
git push origin :refs/tags/<tagname> 删除远程标签<tagname>
  
git config --global color.ui true 让Git显示颜色，会让命令输出看起来更醒目
  
git add -f <file> 强制提交已忽略的的文件
  
git check-ignore -v <file> 检查为什么Git会忽略该文件
  

  

  
git commit 没有push，想要修改commit内容：
  
git commit --amend
  

  

  

  

  

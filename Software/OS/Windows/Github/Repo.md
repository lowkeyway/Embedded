# 1. 概要

repo是Android为了方便管理多个git库而开发的Python脚本。repo的出现，并非为了取代git，而是为了让Android开发者更为有效的利用git。

Android源码包含数百个git库，仅仅是下载这么多git库就是一项繁重的任务，所以在下载源码时，Android就引入了repo。 Android官方推荐下载repo的方法是通过Linux curl命令，下载完后，为repo脚本添加可执行权限：

```
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
由于国内Google访问受限，所以上述命令不一定能下载成功。其实，我们现在可以从很多第三方渠道找到repo脚本，只需要取下来，确保repo可以正确执行即可

例如Harmony的gitee中的下载：
```
curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > /usr/local/bin/repo  #如果没有权限，可下载至其他目录，并将其配置到环境变量中
chmod a+x /usr/local/bin/repo
pip3 install -i https://repo.huaweicloud.com/repository/pypi/simple requests
```

# 2. 工作原理


repo需要关注当前git库的数量、名称、路径等，有了这些基本信息，才能对这些git库进行操作。通过集中维护所有git库的清单，repo可以方便的从清单中获取git库的信息。 这份清单会随着版本演进升级而产生变化，同时也有一些本地的修改定制需求，所以，repo是通过一个git库来管理项目的清单文件的，这个git库名字叫manifests。

当打开repo这个可执行的python脚本后，发现代码量并不大(不超过1000行)，难道仅这一个脚本就完成了AOSP数百个git库的管理吗？并非如此。 repo是一系列脚本的集合，这些脚本也是通过git库来维护的，这个git库名字叫repo。

在客户端使用repo初始化一个项目时，就会从远程把manifests和repo这两个git库拷贝到本地，但这对于Android开发人员来说，又是近乎无形的(一般通过文件管理器，是无法看到这两个git库的)。 repo将自动化的管理信息都隐藏根目录的.repo子目录中。


# 3. 使用介绍

repo命令的使用格式如下所示：
```
$ repo <COMMAND> <OPTIONS>
```
可选的的有：help、init、sync、upload、diff、download、forall、prune、start、status，每一个命令都有实际的使用场景， 下面我们先对这些命令做一个简要的介绍：

## 3.1 init
```
$ repo init -u <URL> [<OPTIONS>]
```

+ -u：指定manifests这个远程git库的URL，manifests库是整个项目的清单。默认情况，这个git库只包含了default.xml一个文件，其内容可以参见Android的样本

+ -m， –manifest-name：指定所需要的manifests库中的清单文件。默认情况下，会使用maniftests/default.xml

+ -b， –manifest-branch：指定manifest.xml文件中的一个版本，，也就是俗称的“分支”



运行该命令后，会在当前目录下新建一个.repo子目录：
```
.repo
├── manifests       # 一个git库，包含default.xml文件，用于描述repo所管理的git库的信息
├── manifests.git   # manifest这个git库的实体，manifest/.git目录下的所有文件都会链接到该目录
├── manifest.xml    # manifests/default.xml的一个软链接
└── repo            # 一个git库，包含repo运行的所有脚本
```
这些本地的目录是如何生成的呢？执行repo命令时，可以通过--trace参数，来看实际发生了什么。
```
$ repo --trace init -u $URL -b $BRANCH -m $MANIFEST
  --------------------
  mkdir .repo; cd .repo
  git clone --bare $URL manifests.git
  git clone https://android.googlesource.com/tools/repo
  mkdir -p manifests/.git; cd manifests/.git
  for i in ../../manifests.git/*; do ln -s $ı .; done
  cd ..
  git checkout $BRANCH -- .
  cd ..
  ln -s manifests/$MANIFEST manifest.xml
```

首先，在当前目录下创建.repo子目录，后续所有的操作都在.repo子目录下完成;

然后，clone了两个git库，其中一个是-u参数指定的manifests，本地git库的名称是manifest.git;另一个是默认的repo，后面我们会看到这个URL也可以通过参数来指定;

接着，创建了manifest/.git目录，里面的所有文件都是到manifests.git这个目录的链接，这个是为了方便对manifests目录执行git命令，紧接着，就会将manifest切换到-b参数指定的分支;

最后，在.repo目录下，创建了一个软链接，链接到-m参数制定的清单文件，默认情况是manifests/default.xml。

这样，就完成了一个多git库的初始化，之后，就可以执行其他的repo命令了。

我们还介绍几个不常用的参数，在国内下载Android源码时，会用到：

+ –repo-url：指定远程repo库的URL，默认情况是https://android.googlesource.com/tools/repo，但国内访问Google受限，会导致这个库无法下载，从而导致repo init失败，所以可以通过该参数指定一个访问不受限的repo地址

+ –repo-branch：同manifest这个git库一样，repo这个git库也是有版本差异的，可以通过该参数来指定下载repo这个远程git库的特定分支

+ –no-repo-verify：在下载repo库时，会对repo的源码进行检查。通过–repo-url指定第三方repo库时，可能会导致检查不通过，所以可以配套使用该参数，强制不进行检查

## 3.2 sync


```
$ repo sync [PROJECT_LIST]
```
下载远程代码，并将本地代码更新到最新，这个过程称为“同步”。如果不使用任何参数，那么会对所有repo管理的进行同步操作;也可以PROJECT_LIST参数，指定若干要同步的PROJECT。 根据本地git库代码不同，同步操作会有不同的行为：

+ 当本地的git库是第一次触发同步操作时，那么，该命令等价于git clone，会将远程git库直接拷贝到本地

+ 当本地已经触发过同步操作时，那么，该命令等价于git remote update && git rebase origin/<BRANCH>，<BRANCH>就是当前与本地分支所关联的远程分支 代码合并可能会产生冲突，当冲突出现时，只需要解决完冲突，然后执行git rebase --continue即可。

+ 当sync命令正确执行完毕后，本地代码就同远程代码保持一致了。在一些场景下，我们会用到sync命令的一些参数：

  + -j：开启多线程同步操作，这会加快sync命令的执行速度。默认情况下，使用4个线程并发进行sync

  + -c, –current-branch：只同步指定的远程分支。默认情况下，sync会同步所有的远程分支，当远程分支比较多的时候，下载的代码量就大。使用该参数，可以缩减下载时间，节省本地磁盘空间

  + -d, –detach：脱离当前的本地分支，切换到manifest.xml中设定的分支。在实际操作中，这个参数很有用，当我们第一次sync完代码后，往往会切换到dev分支进行开发。如果不带该参数使用sync， 则会触发本地的dev分支与manifest设定的远程分支进行合并，这会很可能会导致sync失败

  + -f, –force-broken：当有git库sync失败了，不中断整个同步操作，继续同步其他的git库

  + –no-clone-bundle：在向服务器发起请求时，为了做到尽快的响应速度，会用到内容分发网络(CDN, Content Delivery Network)。同步操作也会通过CDN与就近的服务器建立连接， 使用HTTP/HTTPS的$URL/clone.bundle来初始化本地的git库，clone.bundle实际上是远程git库的镜像，通过HTTP直接下载，这会更好的利用网络带宽，加快下载速度。 当服务器不能正常响应下载$URL/clone.bundle，但git又能正常工作时，可以通过该参数，配置不下载$URL/clone.bundle，而是直接通过git下载远程git库

## 3.3 upload
```
$ repo upload [PROJECT_LIST]
```
从字面意思理解，upload就是要上传，将本地的代码上传到远程服务器。upload命令首先会找出本地分支从上一次同步操作以来发生的改动，然后会将这些改动生成Patch文件，上传至Gerrit服务器。 如果没有指定PROJECT_LIST，那么upload会找出所有git库的改动;如果某个git库有多个分支，upload会提供一个交互界面，提示选择其中若干个分支进行上传操作。

upload并不会直接将改动合并后远程的git库，而是需要先得到Reviewer批准。Reviewer查看改动内容、决定是否批准合入代码的操作，都是通过Gerrit完成。 Gerrit服务器的地址是在manifests中指定的：打开.repo/manifest.xml，<remote>这个XML TAG中的review属性值就是Review服务器的URL:
  
```
<remote name="aosp"
fetch=".."
review="https://android-review.googlesource.com/" />
```  

Gerrit的实现机制不是本文讨论的内容，但有几个与Gerrit相关的概念，是需要代码提交人员了解的：

+ Reviewer：代码审阅人员可以是多个，是需要人为指定的。Gerrit提供网页的操作，可以填选Reviewer。当有多个git库的改动提交时，为了避免在网页上频繁的填选Reviewer这种重复劳动， upload提供了–re, –reviewer参数，在命令行一次性指定Reviewer

+ Commit-ID：git为了标识每个提交，引入了Commit-ID，是一个SHA-1值，针对当次提交内容的一个Checksum，可以用于验证提交内容的完整性

+ Change-ID：Gerrit针对每一个Review任务，引入了一个Change-ID，每一个提交上传到Gerrit，都会对应到一个Change-ID， 为了区分于Commit-ID，Gerrit设定Change-ID都是以大写字母 “I” 打头的。 Change-ID与Commit-ID并非一一对应的，每一个Commit-ID都会关联到一个Change-ID，但Change-ID可以关联到多个Commit-ID

+ Patch-Set：当前需要Review的改动内容。一个Change-ID关联多个Commit-ID，就是通过Patch-Set来表现的，当通过git commit --amend命令修正上一次的提交并上传时， Commit-ID已经发生了变化，但仍可以保持Change-ID不变，这样,在Gerrit原来的Review任务下，就会出现新的Patch-Set。修正多少次，就会出现多少个Patch-Set， 可以理解，只有最后一次修正才是我们想要的结果，所以，在所有的Patch-Set中，只有最新的一个是真正有用的，能够合并的。
  
  
## 3.4 download
  
  
## 3.5 forall
  
## 3.6 prune
  
## 3.7 start
  
## 3.8 status
  
  
# 4. 使用实践

Android推荐的开发流程是：

+ repo init初始化工程，指定待下载的分支
+ repo sync下载代码
+ repo start将本地git库切换到开发分支(TOPIC BRANCH)
+ 在本地进行修改，验证后，提交到本地
+ repo upload上传到服务器，等待review
在实际使用过程中，我们会用到repo的一些什么子命令和参数呢？哪些参数有助于提高开发效率呢？下面我们以一些实际场景为例展开说明。

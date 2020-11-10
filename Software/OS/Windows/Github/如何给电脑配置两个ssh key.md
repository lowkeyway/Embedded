说明下我的环境是 Win10 + GitBash.

# 生成并添加第一个ssh key

第一次使用ssh生成key，默认会在用户~（根目录）下生成 id_rsa, id_rsa.pub 2个文件；所以需要添加多个ssh key时也会生成对应的私钥和公钥。
```
$ ssh-keygen -t rsa -C "youremail@yourcompany.com"
```
在Git Bash中执行这条命令一路回车，会在 ~/.ssh/ 目录下生成 id_rsa 和 id_rsa.pub 两个文件，用文本编辑器将 id_rsa_pub 中的内容复制一下粘贴到github(gitlab)上。


# 生成并添加第二个ssh key
```
$ ssh-keygen -t rsa -C "youremail@gmail.com"
```
注意不要一路回车，要给这个文件起一个名字， 比如叫 id_rsa_github, 所以相应的也会生成一个 id_rsa_github.pub 文件

目录结构:
{img src="https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Pic/Windows/SSH%20Key.png"}

# 添加私钥

```
$ ssh-add ~/.ssh/id_rsa
$ ssh-add ~/.ssh/id_rsa_github
```
如果执行ssh-add时提示"Could not open a connection to your authentication agent"，可以现执行命令：
```
$ ssh-agent bash
```
然后再运行ssh-add命令。

```
# 可以通过 ssh-add -l 来确私钥列表
$ ssh-add -l

# 可以通过 ssh-add -D 来清空私钥列表
$ ssh-add -D
```

# 修改配置文件

在 ~/.ssh 目录下新建一个config文件,添加内容：
```
Host gitlab
  HostName Omnivision
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa
Host github
  HostName github.com
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/github_id_rsa
```


# 测试

```
$ ssh -T git@github.com
```

输出
Hi user! You've successfully authenticated, but GitHub does not provide shell access. 就表示成功的连上github了.





参考这篇文章：

https://www.cnblogs.com/fanyong/p/3962455.html

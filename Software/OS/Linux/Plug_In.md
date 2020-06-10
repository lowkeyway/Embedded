# 1. 安装vim_plug

可以参考这个链接：https://github.com/junegunn/vim-plug

当然，也可以在[这里](https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Linux/Pic/plug.vim)download，另存为 ~/.vim/autoload/plug.vim

# 2. 修改vim.rc

可以参考这个[链接](https://github.com/lowkeyway/Embedded/blob/master/Software/OS/Linux/vimrc.md)。

## 2.1 Ycm

需要指出的是，[Ycm](https://github.com/ycm-core/YouCompleteMe)的安装比较复杂一点，需要手动装一下。

在目录： ~/.vim/plugged/YouCompleteMe 下：

```
git submodule update --init --recursive
./install.py --clang-completer
```

# 3. 更新

在VIM界面中用如下两个命令就可以更新了。
```
:PlugStatus
:PlugUpdate
```

# YouCompleteMe 安装

对vim安装了插件管理器 [vim-plug](https://github.com/junegunn/vim-plug)之后，所有的插件都简单成怂了，但是偏偏一个最重要的代码补全工具YouCompleteMe插件在每次安装的时候都有奇怪的问题，在这里记录一下。  

## 环境准备

```
sudo apt install build-essential cmake python3-dev  python-dev
sudo apt install clang libclang-dev
```

## 插件安装

### 插件安装：

```
Plug 'Valloric/YouCompleteMe'
```

这里是设定在.vimrc里面的利用vim-plug管理器的PlugInstall命令安装的，原理是会根据写的内容（Valloric/YouCompleteMe）去对应的github地址去download文件，真的要抱怨一下github，太慢了。
如果发现默认的地址很难download，可以尝试在[HERE](https://github.com/ycm-core/YouCompleteMe)这里下载，注意奥，跟插件中的不是一个仓！但其实是一个人做的core，可以把比较大的文件down下来copy过去。

### 编译插件：

```
cd ~/.vim/plugged/YouCompleteMe
./install.py --clang-completer
```

# 配置插件：

```
let g:ycm_add_preview_to_completeopt = 0
let g:ycm_show_diagnostics_ui = 0
let g:ycm_server_log_level = 'info'
let g:ycm_min_num_identifier_candidate_chars = 2
let g:ycm_collect_identifiers_from_comments_and_strings = 1
let g:ycm_complete_in_strings=1
let g:ycm_key_invoke_completion = '<c-z>'
highlight PMenu ctermfg=0 ctermbg=242 guifg=black guibg=darkgrey
highlight PMenuSel ctermfg=242 ctermbg=8 guifg=darkgrey guibg=black
set completeopt=menu,menuone

noremap <c-z> <NOP>

let g:ycm_semantic_triggers =  {
            \ 'c,cpp,python,java,go,erlang,perl': ['re!\w{2}'],
            \ 'cs,lua,javascript': ['re!\w{2}'],
            \ }

let g:ycm_filetype_whitelist = { 
            \ "c":1,
            \ "cpp":1, 
            \ "objc":1,
            \ "sh":1,
            \ "zsh":1,
            \ "zimbu":1,
            \ }
            
let g:ycm_global_ycm_extra_conf='~/.ycm_extra_conf.py'

let g:ycm_confirm_extra_conf = 0
```

## 指定.ycm_extra_conf.py路径:

将 .ycm_extra_conf.py 拷贝到home 目录
```
cp ~/.vim/plugged/YouCompleteMe/third_party/ycmd/.ycm_extra_conf.py ~/
```

最后一步的配置必不可少，否则可能出现结构体变量的不识别问题。



set nocompatible " 关闭 vi 兼容模式  
syntax on " 自动语法高亮  
set number " 显示行号  
set cursorline " 突出显示当前行  
set ruler " 打开状态栏标尺  
set shiftwidth=4 " 设定 << 和 >> 命令移动时的宽度为 4  
set softtabstop=4 " 使得按退格键时可以一次删掉 4 个空格  
set tabstop=4 " 设定 tab 长度为 4  
set nobackup " 覆盖文件时不备份  
set autochdir " 自动切换当前目录为当前文件所在的目录  
filetype plugin indent on " 开启插件  
set backupcopy=yes " 设置备份时的行为为覆盖  
set ignorecase smartcase " 搜索时忽略大小写，但在有一个或以上大写字母时仍保持对大小写敏感  
set nowrapscan " 禁止在搜索到文件两端时重新搜索  
set incsearch " 输入搜索内容时就显示搜索结果  
set hlsearch " 搜索时高亮显示被找到的文本  
set noerrorbells " 关闭错误信息响铃  
set novisualbell " 关闭使用可视响铃代替呼叫  
set t_vb= " 置空错误铃声的终端代码  
"set showmatch " 插入括号时，短暂地跳转到匹配的对应括号  
"set matchtime=2 " 短暂跳转到匹配括号的时间  
set magic " 设置魔术  
set hidden " 允许在有未保存的修改时切换缓冲区，此时的修改由 vim 负责保存  
set guioptions-=T " 隐藏工具栏  
set guioptions-=m " 隐藏菜单栏  
set smartindent " 开启新行时使用智能自动缩进  
set backspace=indent,eol,start  
" 不设定在插入状态无法用退格键和 Delete 键删除回车符  
set cmdheight=1 " 设定命令行的行数为 1  
set laststatus=2 " 显示状态栏 (默认值为 1, 无法显示状态栏)  
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\   
" 设置在状态行显示的信息  
set foldenable " 开始折叠  
set foldmethod=syntax " 设置语法折叠  
set foldcolumn=0 " 设置折叠区域的宽度  
setlocal foldlevel=1 " 设置折叠层数为  
" set foldclose=all " 设置为自动关闭折叠   
" nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>  
" 用空格键来开关折叠  
  
set fenc=  
  
" return OS type, eg: windows, or linux, mac, et.st..  
function! MySys()  
if has("win16") || has("win32") || has("win64") || has("win95")  
return "windows"  
elseif has("unix")  
return "linux"  
endif  
endfunction  
  
" 用户目录变量$VIMFILES  
if MySys() == "windows"  
let $VIMFILES = $VIM.'/vimfiles'  
elseif MySys() == "linux"  
let $VIMFILES = $HOME.'/.vim'  
endif  
  
" 设定doc文档目录  
let helptags=$VIMFILES.'/doc'  
  
" 设置字体 以及中文支持  
if has("win32")  
set guifont=Inconsolata:h12:cANSI  
endif  
  
" 配置多语言环境  
if has("multi_byte")  
" UTF-8 编码  
set encoding=utf-8  
set termencoding=utf-8  
set formatoptions+=mM  
set fencs=utf-8,gbk  
  
if v:lang =~? '^\(zh\)\|\(ja\)\|\(ko\)'  
set ambiwidth=double  
endif  
  
if has("win32")  
source $VIMRUNTIME/delmenu.vim  
source $VIMRUNTIME/menu.vim  
language messages zh_CN.utf-8  
endif  
else  
echoerr "Sorry, this version of (g)vim was not compiled with +multi_byte"  
endif  
  
" Buffers操作快捷方式!  
nnoremap <C-RETURN> :bnext<CR>  
nnoremap <C-S-RETURN> :bprevious<CR>  
  
" Tab操作快捷方式!  
nnoremap <C-TAB> :tabnext<CR>  
nnoremap <C-S-TAB> :tabprev<CR>  
  
"关于tab的快捷键  
" map tn :tabnext<cr>  
" map tp :tabprevious<cr>  
" map td :tabnew .<cr>  
" map te :tabedit  
" map tc :tabclose<cr>  
  
"窗口分割时,进行切换的按键热键需要连接两次,比如从下方窗口移动  
"光标到上方窗口,需要<c-w><c-w>k,非常麻烦,现在重映射为<c-k>,切换的  
"时候会变得非常方便.  
nnoremap <C-h> <C-w>h  
nnoremap <C-j> <C-w>j  
nnoremap <C-k> <C-w>k  
nnoremap <C-l> <C-w>l  
  
"一些不错的映射转换语法（如果在一个文件中混合了不同语言时有用）  
nnoremap <leader>1 :set filetype=xhtml<CR>  
nnoremap <leader>2 :set filetype=css<CR>  
nnoremap <leader>3 :set filetype=javascript<CR>  
nnoremap <leader>4 :set filetype=php<CR>  
  
" set fileformats=unix,dos,mac  
" nmap <leader>fd :se fileformat=dos<CR>  
" nmap <leader>fu :se fileformat=unix<CR>  
  
" use Ctrl+[l|n|p|cc] to list|next|previous|jump to count the result  
" map <C-x>l <ESC>:cl<CR>  
" map <C-x>n <ESC>:cn<CR>  
" map <C-x>p <ESC>:cp<CR>  
" map <C-x>c <ESC>:cc<CR>  
  
  
" 让 Tohtml 产生有 CSS 语法的 html  
" syntax/2html.vim，可以用:runtime! syntax/2html.vim  
let html_use_css=1  
  
" Python 文件的一般设置，比如不要 tab 等  
autocmd FileType python set tabstop=4 shiftwidth=4 expandtab  
autocmd FileType python map <F12> :!python %<CR>  
  
" 选中状态下 Ctrl+c 复制  
vmap <C-c> "+y  
  
" 打开javascript折叠  
let b:javascript_fold=1  
" 打开javascript对dom、html和css的支持  
let javascript_enable_domhtmlcss=1  
" 设置字典 ~/.vim/dict/文件的路径  
autocmd filetype javascript set dictionary=$VIMFILES/dict/javascript.dict  
autocmd filetype css set dictionary=$VIMFILES/dict/css.dict  
autocmd filetype php set dictionary=$VIMFILES/dict/php.dict  
  
"-----------------------------------------------------------------  
" plugin - bufexplorer.vim Buffers切换  
" \be 全屏方式查看全部打开的文件列表  
" \bv 左右方式查看 \bs 上下方式查看  
"-----------------------------------------------------------------  
  
  
"-----------------------------------------------------------------  
" plugin - taglist.vim 查看函数列表，需要ctags程序  
" F4 打开隐藏taglist窗口  
"-----------------------------------------------------------------  
if MySys() == "windows" " 设定windows系统中ctags程序的位置  
let Tlist_Ctags_Cmd = '"'.$VIMRUNTIME.'/ctags.exe"'  
elseif MySys() == "linux" " 设定windows系统中ctags程序的位置  
let Tlist_Ctags_Cmd = '/usr/bin/ctags'  
endif  
nnoremap <silent><F4> :TlistToggle<CR>  
let Tlist_Show_One_File = 1 " 不同时显示多个文件的tag，只显示当前文件的  
let Tlist_Exit_OnlyWindow = 1 " 如果taglist窗口是最后一个窗口，则退出vim  
let Tlist_Use_Right_Window = 1 " 在右侧窗口中显示taglist窗口  
let Tlist_File_Fold_Auto_Close=1 " 自动折叠当前非编辑文件的方法列表  
let Tlist_Auto_Open = 0  
let Tlist_Auto_Update = 1  
let Tlist_Hightlight_Tag_On_BufEnter = 1  
let Tlist_Enable_Fold_Column = 0  
let Tlist_Process_File_Always = 1  
let Tlist_Display_Prototype = 0  
let Tlist_Compact_Format = 1  
  
  
"-----------------------------------------------------------------  
" plugin - mark.vim 给各种tags标记不同的颜色，便于观看调式的插件。  
" \m mark or unmark the word under (or before) the cursor  
" \r manually input a regular expression. 用于搜索.  
" \n clear this mark (i.e. the mark under the cursor), or clear all highlighted marks .  
" \* 当前MarkWord的下一个 \# 当前MarkWord的上一个  
" \/ 所有MarkWords的下一个 \? 所有MarkWords的上一个  
"-----------------------------------------------------------------  
  
  
"-----------------------------------------------------------------  
" plugin - NERD_tree.vim 以树状方式浏览系统中的文件和目录  
" :ERDtree 打开NERD_tree :NERDtreeClose 关闭NERD_tree  
" o 打开关闭文件或者目录 t 在标签页中打开  
" T 在后台标签页中打开 ! 执行此文件  
" p 到上层目录 P 到根目录  
" K 到第一个节点 J 到最后一个节点  
" u 打开上层目录 m 显示文件系统菜单（添加、删除、移动操作）  
" r 递归刷新当前目录 R 递归刷新当前根目录  
"-----------------------------------------------------------------  
" F3 NERDTree 切换  
map <F3> :NERDTreeToggle<CR>  
imap <F3> <ESC>:NERDTreeToggle<CR>  
  
  
"-----------------------------------------------------------------  
" plugin - NERD_commenter.vim 注释代码用的，  
" [count],cc 光标以下count行逐行添加注释(7,cc)  
" [count],cu 光标以下count行逐行取消注释(7,cu)  
" [count],cm 光标以下count行尝试添加块注释(7,cm)  
" ,cA 在行尾插入 /* */,并且进入插入模式。 这个命令方便写注释。  
" 注：count参数可选，无则默认为选中行或当前行  
"-----------------------------------------------------------------  
let NERDSpaceDelims=1 " 让注释符与语句之间留一个空格  
let NERDCompactSexyComs=1 " 多行注释时样子更好看  
  
  
"-----------------------------------------------------------------  
" plugin - DoxygenToolkit.vim 由注释生成文档，并且能够快速生成函数标准注释  
"-----------------------------------------------------------------  
let g:DoxygenToolkit_authorName="Asins - asinsimple AT gmail DOT com"  
let g:DoxygenToolkit_briefTag_funcName="yes"  
map <leader>da :DoxAuthor<CR>  
map <leader>df :Dox<CR>  
map <leader>db :DoxBlock<CR>  
map <leader>dc a /* */<LEFT><LEFT><LEFT>  
  
  
"-----------------------------------------------------------------  
" plugin – ZenCoding.vim 很酷的插件，HTML代码生成  
" 插件最新版：http://github.com/mattn/zencoding-vim  
"-----------------------------------------------------------------  
  
  
"-----------------------------------------------------------------  
" plugin – checksyntax.vim JavaScript常见语法错误检查  
" 默认快捷方式为 F5  
"-----------------------------------------------------------------  
let g:checksyntax_auto = 0 " 不自动检查  
  
  
"-----------------------------------------------------------------  
" plugin - NeoComplCache.vim 自动补全插件  
"-----------------------------------------------------------------  
let g:AutoComplPop_NotEnableAtStartup = 1  
let g:NeoComplCache_EnableAtStartup = 1  
let g:NeoComplCache_SmartCase = 1  
let g:NeoComplCache_TagsAutoUpdate = 1  
let g:NeoComplCache_EnableInfo = 1  
let g:NeoComplCache_EnableCamelCaseCompletion = 1  
let g:NeoComplCache_MinSyntaxLength = 3  
let g:NeoComplCache_EnableSkipCompletion = 1  
let g:NeoComplCache_SkipInputTime = '0.5'  
let g:NeoComplCache_SnippetsDir = $VIMFILES.'/snippets'  
" <TAB> completion.  
inoremap <expr><TAB> pumvisible() ? "\<C-n>" : "\<TAB>"  
inoremap <C-j> <Esc>o  
inoremap ' ''<ESC>i  
inoremap " ""<ESC>i  
inoremap ( ()<ESC>i  
inoremap [ []<ESC>i  
inoremap { {<CR>}<ESC>O  
  
" snippets expand key  
imap <silent> <C-e> <Plug>(neocomplcache_snippets_expand)  
smap <silent> <C-e> <Plug>(neocomplcache_snippets_expand)  
  
  
"-----------------------------------------------------------------  
" plugin - matchit.vim 对%命令进行扩展使得能在嵌套标签和语句之间跳转  
" % 正向匹配 g% 反向匹配  
" [% 定位块首 ]% 定位块尾  
"-----------------------------------------------------------------  
  
  
"-----------------------------------------------------------------  
" plugin - vcscommand.vim 对%命令进行扩展使得能在嵌套标签和语句之间跳转  
" SVN/git管理工具  
"-----------------------------------------------------------------  
"-----------------------------------------------------------------  
" plugin – a.vim  
"-----------------------------------------------------------------  
  
" Specify a directory for plugins  
" - For Neovim: stdpath('data') . '/plugged'  
" - Avoid using standard Vim directory names like 'plugin'  
call plug#begin('~/.vim/plugged')  
  
" Make sure you use single quotes  
  
" Shorthand notation; fetches https://github.com/junegunn/vim-easy-align  
Plug 'junegunn/vim-easy-align'  
  
" Any valid git URL is allowed  
Plug 'https://github.com/junegunn/vim-github-dashboard.git'  
  
" Multiple Plug commands can be written in a single line using | separators  
Plug 'SirVer/ultisnips' | Plug 'honza/vim-snippets'  
  
" On-demand loading  
Plug 'scrooloose/nerdtree', { 'on':  'NERDTreeToggle' }  
Plug 'tpope/vim-fireplace', { 'for': 'clojure' }  
  
" Using a non-master branch  
Plug 'rdnetto/YCM-Generator', { 'branch': 'stable' }  
  
" Using a tagged release; wildcard allowed (requires git 1.9.2 or above)  
Plug 'fatih/vim-go', { 'tag': '*' }  
  
" Plugin options  
Plug 'nsf/gocode', { 'tag': 'v.20150303', 'rtp': 'vim' }  
  
" Plugin outside ~/.vim/plugged with post-update hook  
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }  
  
" Unmanaged plugin (manually installed and updated)  
" Plug '~/my-prototype-plugin'  
  
" Install vim-gutentags  
Plug 'ludovicchabant/vim-gutentags'  
" gutentags 搜索工程目录的标志，碰到这些文件/目录名就停止向上一级目录递归  
let g:gutentags_project_root = ['.root', '.svn', '.git', '.hg', '.project']  
  
" 所生成的数据文件的名称  
let g:gutentags_ctags_tagfile = '.tags'  
  
" 将自动生成的 tags 文件全部放入 ~/.cache/tags 目录中，避免污染工程目录  
let s:vim_tags = expand('~/.cache/tags')  
let g:gutentags_cache_dir = s:vim_tags  
  
" 配置 ctags 的参数  
let g:gutentags_ctags_extra_args = ['--fields=+niazS', '--extra=+q']  
let g:gutentags_ctags_extra_args += ['--c++-kinds=+px']  
let g:gutentags_ctags_extra_args += ['--c-kinds=+px']  
  
" 检测 ~/.cache/tags 不存在就新建  
if !isdirectory(s:vim_tags)  
   silent! call mkdir(s:vim_tags, 'p')  
endif  
  
" Louis Install ALE  
Plug 'dense-analysis/ale'  
" "Plug 'w0p/ale'  
  
" Louis install YCM  
Plug 'Valloric/YouCompleteMe'  
let g:ycm_add_preview_to_completeopt = 0  
let g:ycm_show_diagnostics_ui = 0  
let g:ycm_server_log_level = 'info'  
let g:ycm_min_num_identifier_candidate_chars = 2  
let g:ycm_collect_identifiers_from_comments_and_strings = 1  
let g:ycm_complete_in_strings=1  
let g:ycm_key_invoke_completion = '<c-z>'  
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
  
" Louis Install taglist  
Plug 'vim-scripts/taglist.vim'  
map <silent> <F12> :TlistToggle<cr>  
  
call plug#end()  
" Table to space" 
set tabstop=2
set softtabstop=2
set shiftwidth=2                                                                                                                                                                                        
set ts=2  
set expandtab  
set autoindent  

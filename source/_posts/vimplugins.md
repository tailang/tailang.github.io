---
title: vim小记录：常用插件介绍
date: 2012-05-19 12:54:03
categories:
    - tech
tags:
    - vim
---

### [snipmate](https://github.com/msanders/snipmate.vim)
为了加快效率，当然需要vim能实现tab键补全代码的功能了。我使用的是snipmate.vim这个插件，当你安装了snipmate.vim，已经有一些snippets了，如ruby，php，java......
比如在写ruby的时候，输入```def```在按tab键，就能显示为
```
def method_name

end
```
当然你可以修改相应的snippets，大概文件位置在这
```
～/.vim/bundle/snipmate.vim/snippets
```
特别注意的是其中的_.snippets文件，他可以设置全局snippets，就比如我在其中设置了
```
snippet =
	<%= ${1} %>
snippet <
	<% ${1} %>
```
用于快速输出
```
<%  %>
<%=  %>
```


### [nerdtree](https://github.com/scrooloose/nerdtree)
NERDTree的作用就是列出当前路径的目录树，一般IDE都是有的。可以方便的浏览项目的总体的目录结构和创建删除重命名文件或文件名。
至于它的配置我做了如下修改
```
 " NERDTree config
 map <F2> :NERDTreeToggle<CR>
 autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") &&b:NERDTreeType == "primary") | q | endif
```
第一条是说使用F2键快速调出和隐藏它；
第二条是关闭vim时，如果打开的文件除了NERDTree没有其他文件时，它自动关闭，减少多次按:q!。
如果想打开vim时自动打开NERDTree，可以如下设定
```
autocmd vimenter * NERDTree
```

下面总结一些命令,非常的多，只要记住一些你常用的就行了，后名添加！！！的是我常用的(原文地址：[http://yang3wei.github.io/blog/2013/01/29/nerdtree-kuai-jie-jian-ji-lu/](http://yang3wei.github.io/blog/2013/01/29/nerdtree-kuai-jie-jian-ji-lu/)）

```
ctrl + w + h    光标 focus 左侧树形目录
ctrl + w + l    光标 focus 右侧文件显示窗口
ctrl + w + w    光标自动在左右侧窗口切换 #！！！
ctrl + w + r    移动当前窗口的布局位置
```

```
o       在已有窗口中打开文件、目录或书签，并跳到该窗口
go      在已有窗口 中打开文件、目录或书签，但不跳到该窗口
t       在新 Tab 中打开选中文件/书签，并跳到新 Tab
T       在新 Tab 中打开选中文件/书签，但不跳到新 Tab
i       split 一个新窗口打开选中文件，并跳到该窗口
gi      split 一个新窗口打开选中文件，但不跳到该窗口
s       vsplit 一个新窗口打开选中文件，并跳到该窗口
gs      vsplit 一个新 窗口打开选中文件，但不跳到该窗口
!       执行当前文件
O       递归打开选中 结点下的所有目录
x       合拢选中结点的父目录
X       递归 合拢选中结点下的所有目录
e       Edit the current dif

双击    相当于 NERDTree-o
中键    对文件相当于 NERDTree-i，对目录相当于 NERDTree-e

D       删除当前书签

P       跳到根结点
p       跳到父结点
K       跳到当前目录下同级的第一个结点
J       跳到当前目录下同级的最后一个结点
k       跳到当前目录下同级的前一个结点
j       跳到当前目录下同级的后一个结点

C       将选中目录或选中文件的父目录设为根结点
u       将当前根结点的父目录设为根目录，并变成合拢原根结点
U       将当前根结点的父目录设为根目录，但保持展开原根结点
r       递归刷新选中目录
R       递归刷新根结点
m       显示文件系统菜单 #！！！然后根据提示进行文件的操作如新建，重命名等
cd      将 CWD 设为选中目录

I       切换是否显示隐藏文件
f       切换是否使用文件过滤器
F       切换是否显示文件
B       切换是否显示书签

q       关闭 NerdTree 窗口
?       切换是否显示 Quick Help
```
```
:tabnew [++opt选项] ［＋cmd］ 文件      建立对指定文件新的tab
:tabc   关闭当前的 tab
:tabo   关闭所有其他的 tab
:tabs   查看所有打开的 tab
:tabp   前一个 tab
:tabn   后一个 tab

标准模式下：
gT      前一个 tab
gt      后一个 tab

MacVim 还可以借助快捷键来完成 tab 的关闭、切换
cmd+w   关闭当前的 tab
cmd+{   前一个 tab
cmd+}   后一个 tab
```


### [ctrlp](https://github.com/kien/ctrlp.vim)
这是一个超赞的插件，如果使用过sublime-text2，那么肯定很熟悉ctrlp。它可以快速的帮助我们找到项目中的文件。在vim normal模式下，按下ctrl+p，然后输入你要寻找的文件就行了。当然还有其他一些快捷查找键，如正则查找等，
```
Press <F5> to purge the cache for the current directory to get new files, remove deleted files and apply new ignore options.
Press <c-f> and <c-b> to cycle between modes.
Press <c-d> to switch to filename only search instead of full path.
Press <c-r> to switch to regexp mode.
Use <c-j>, <c-k> or the arrow keys to navigate the result list.
Use <c-t> or <c-v>, <c-x> to open the selected entry in a new tab or in a new split.
Use <c-n>, <c-p> to select the next/previous string in the prompt's history.
Use <c-y> to create a new file and its parent directories.
Use <c-z> to mark/unmark multiple files and <c-o> to open them.
```

### [neocomplcache.vim](https://github.com/Shougo/neocomplcache.vim)
它有什么作用呢？直接上图你就知道了
![](http://ww2.sinaimg.cn/large/bf0b41c3gw1e8vd0mmk99j206j057mxc.jpg)
明白了吧，就是代码的提示。
当然这个太强大了，有时会和其他插件冲突，有时会出现下面错误而无法打开
```
Another plugin set completefunc! Disabled neocomplcache
```
所以我在.vimrc做了如下的配置
```
 let g:neocomplcache_enable_at_startup = 1 "打开vim时自动打开
 let g:neocomplcache_force_overwrite_completefunc = 1 "解决上面那个错误
```
### [vim-autoclose](https://github.com/Townk/vim-autoclose)
它的作用其实就是'(' '[' '{'补全，如当你输入'('后自动显示出'()'

######[vim-powerline](https://github.com/Lokaltog/vim-powerline)
更美观的状态栏
![http://ww3.sinaimg.cn/large/bf0b41c3gw1e8vd7d1ngvj20jy0623z5.jpg](http://ww3.sinaimg.cn/large/bf0b41c3gw1e8vd7d1ngvj20jy0623z5.jpg)
我的设置
```
"powerline config
 set laststatus=2
 set t_Co=256   
 set encoding=utf-8  
 set fillchars+=stl:\ ,stlnc:\
```

 ### NERD_commenter.vim:   
添加注释的插件，使用如下： 
[count],cc:光标以下count行添加注释(2,cc) 
[count],cu:光标以下count行取消注释(2,cu) 
[count],cm:光标以下count行添加块注释(2,cm) 

### matchit
快速匹配的功能。如果我们在写html的时候，文件非常大，其中出现了大量<div></div>,那我们怎么快速匹配跳转呢？就用这个插件，我们可以将光标停留在一个<div>上，然后在标准模式下输入%，就能快速的跳转到相应的</div>.当然这不单 支持<div>其他也一样

### Ctags
什么是ctags呢？wiki上是这样解释的：
>Ctags是一个用于从程序源代码树产生索引文件（或tag文件)，从而便于文本编辑器来实现快速定位的实用工具。

也许初次看到这个的同学还是云里雾里的,就比如我第一次看到时就不知道这到底是做什么用的。下面我简单介绍一下它的作用：
假设我们在看一个别人写的项目，项目下有非常多的文件，而且在不同文件中又定义了一些函数或方法。当你阅读到一段代码时，你看到了一个调用的方法，但是你不知道这个方法到底是做什么用的，那我们会怎么办，第一个反应就是去看这个方法的定义的源代码，然而在这么多的文件中如何快速的找到这个方法的源代码呢？那我们就应该使用Ctags了。

如何安装使用ctags？
如果你使用的debian，ubuntu，mint可以使用apt-get安装，如果是其他发行版的话，就使用相应的方法安装ctangs。
如在mint下
```
sudo apt-get install ctags
```

安装好后，然后我们要进入你的项目主目录，执行下列命令
```
ctags -R 
```

然后你就可以使用了，比如你在一个文件下调用了my_method方法，然后你想看它的定义的地方，你只要将光标移到该方法名上，按```Ctrl+]```这两个键后，就会自动跳转到方法定义的地方。

### vim-rails
这是最吸引我的一个插件，当你在写rails应用的时候可以提高效率。
一些常用功能：

gf功能：将光标停在对应的地方，然后按gf两个键就可以智能的跳到相应的文件，例如在user.rb中有这么一句has_many :comments, 然后将光标移到comments上，按gf就智能的跳到comment.rb文件；

:find :可以快速切换到对应的文件，如:find user.rb 就跳到user.rb文件，支持tab补全

:Rmodel/:Rcontroller/:Rview  :可以快速切换到对应的model controller view，如你在user.rb这个文件下，输入:Rcontroller命令就会切换到user的controller文件下。

......

一些命令
```
:Rake      :Rake db:migrate,  :Rake db:create, ...... 
  :Rmodel     :Rmodel info (查找model名称为info的文件) 
  :Rview      :Rview  infos/new (查找infos控制器下的new视图文件) 
  :Rcontroller     :Rcontroller infos(查找控制器名称为infos的文件) 
  :find       :Rfind infos_controller(查找infos_controller.rb文件) 
  :Rails       :Rails console 或  :Rails generate model info age:integer或........ 
  :Rscript     :Rscript console 或 :Rscript generate model info age:integer或......(注意Rscript相当于script/rails命令) 
  :Redit       :Redit 相对路径 
  :Rlog        :Rlog development  打开development.log日志文件 
  :Rpreview     打开一个浏览器，http://localhost:3000 
  :Rrefresh     刷新 
  R             在目录下直接shift+r，可以刷新目录 
  gf            根据当前光标处内容跳转到文件 
  :Rmigration   查找migration文件 
  :Rlayout      查找layout文件 
  :Rhelper      查找helper文件 
  :Rstylesheet 
  :Rjavascript 
  :Rplugin 
  :Rlib 
  :Rtask 
  :Rserver 
```
这里有两个非常棒的介绍：
[http://ruby-china.org/topics/4478](http://ruby-china.org/topics/4478)
[http://railscasts-china.com/episodes/rails-with-vim?autoplay=true](http://railscasts-china.com/episodes/rails-with-vim?autoplay=true)

### 其它设置
```
 set nu "打开vim后显示行号
 set mouse=a "是能鼠标
 set tabstop=2 "设置tab键的空格
```
如果你使用ubuntu等发行版，在使用的时候出现不能将vim的文本复制粘帖到其他软件，使用+y +p都不行
用下面的方法可以解决（原文地址:[http://www.liurongxing.com/ubuntu-system-vim-to-use-the-system-clipboard.html](http://www.liurongxing.com/ubuntu-system-vim-to-use-the-system-clipboard.html)）
>1 问题来源
用 apt-get install安装的vim不能使用系统剪贴板，即复制："+y，和粘贴"+p不能用；用:reg 命令查看没有"+寄存器
2 软件版本
操作系统：ubuntu 12.04；vim版本 7.3.429
3 安装过程
3.1 安装相关软件包
```
$ sudo apt-get install build-essential
$ sudo apt-get install ncurses-dev
$ sudo apt-get install xorg-dev
$ sudo apt-get install libgtk2.0-dev
```
3.2 安装vim
```
sudo apt-get install vim vim-scripts vim-gnome vim-gtk
sudo apt-get install exuberant-ctags cscope
```
设置映射配置.vimrc：
```
let mapleader = ","
let g:mapleader = ","
map c "+y
map p "+p
```

### Gvim
今天使用Gvim编辑文档的时候，突然发现不能输入中文了，寻找了这种办法都没有解决，也不清楚是什么原因导致这个问题的。因为前几天还用的好好，而且vim也可以正常时候。最后的解决方法是这样的(前提是 我使用的是fcitx)：  
首先安装这个插件，[fcitx.vim](https://github.com/vim-scripts/fcitx.vim)  ,然后在~/.bashrc最后添加下面的代码  
```bash
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE=xim
export GTK_IM_MODULE=xim
```
保存退出后，注销重启就OK了。   
这里特别扯一下fcitx.vim，真的是无意中发现的宝贝啊。以前如果在插入模式下用的是中文输入，现在要切换到其他模式下的话，我们先要将输入发设置成英文输入，再输入命令。这样是不是很麻烦呢？fcitx.vim非常好的解决了这个问题，这是它的README写到的
>在离开或重新进入插入模式时自动记录和恢复每个缓冲区各自的输入法状态，以便在普通模式下始终是英文输入模式，切换回插入模式时恢复离开前的输入法输入模式。

非常cool
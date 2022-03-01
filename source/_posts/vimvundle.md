---
title: vim小记录：使用vundle进行插件管理
date: 2012-05-19 12:51:03
categories:
    - tech
tags:
    - vim
---
如果不使用vundle的话，进行插件的安装，配置和管理相对会麻烦，曾经没使用vundle的时候我经常遇到无法安装一些vim插件。但使用vundle后你只要在文件中添加一行你的插件名再安装就OK了。先简单说一下vundle的使用，相信你会爱上它。
这是它的github地址:[https://github.com/gmarik/vundle](https://github.com/gmarik/vundle)
#### 1.先安装vim
#### 2.创建文件夹~/.vim和文件~/.vimrc
进入你的home目录创建.vim文件夹和 .vimrc文件
#### 3.安装vundle
```
$ git clone https://github.com/gmarik/vundle.git ~/.vim/bundle/vundle
```
#### 4.编辑如下内容到.vimrc文件

`注：在最新的vundle中，下面的配置不在有效，最新配置请参考该项目`[vimrc](https://github.com/tailang/vimrc)
```
set nocompatible               " be iMproved
 filetype off                   " required!

 set rtp+=~/.vim/bundle/vundle/
 call vundle#rc()

 " let Vundle manage Vundle
 " required! 
 Bundle 'gmarik/vundle'

 " My Bundles here:
 "
 " original repos on github
 Bundle 'tpope/vim-fugitive'
 Bundle 'Lokaltog/vim-easymotion'
 Bundle 'rstacruz/sparkup', {'rtp': 'vim/'}
 Bundle 'tpope/vim-rails.git'
 " vim-scripts repos
 Bundle 'L9'
 Bundle 'FuzzyFinder'
 " non github repos
 Bundle 'git://git.wincent.com/command-t.git'
 " git repos on your local machine (ie. when working on your own plugin)
 Bundle 'file:///Users/gmarik/path/to/plugin'
 " ...

 filetype plugin indent on     " required!
 "
 " Brief help
 " :BundleList          - list configured bundles
 " :BundleInstall(!)    - install(update) bundles
 " :BundleSearch(!) foo - search(or refresh cache first) for foo
 " :BundleClean(!)      - confirm(or auto-approve) removal of unused bundles
 "
 " see :h vundle for more details or wiki for FAQ
 " NOTE: comments after Bundle command are not allowed..
```
下面针对上面的文件做一些解释
```
set nocompatible               " be iMproved
 filetype off                   " required!

 set rtp+=~/.vim/bundle/vundle/
 call vundle#rc()

 " let Vundle manage Vundle
 " required! 
 Bundle 'gmarik/vundle'

 " My Bundles here:
 #以后你想安装什么插件可以写在下面
 "
 " original repos on github 
#如果你的插件来自github，写在下方，只要作者名/项目名就行了
 Bundle 'tpope/vim-fugitive' #如这里就安装了vim-fugitive这个插件
 Bundle 'Lokaltog/vim-easymotion'
 Bundle 'rstacruz/sparkup', {'rtp': 'vim/'}
 Bundle 'tpope/vim-rails.git'
 " vim-scripts repos
#如果插件来自 vim-scripts，你直接写插件名就行了
 Bundle 'L9'
 Bundle 'FuzzyFinder'
 " non github repos
#如使用自己的git库的插件，像下面这样做
 Bundle 'git://git.wincent.com/command-t.git'
 " git repos on your local machine (ie. when working on your own plugin)
 Bundle 'file:///Users/gmarik/path/to/plugin'
 " ...

 filetype plugin indent on     " required!
#下面是 vundle的一些命令代会会用到
 "
 " Brief help
 " :BundleList          - list configured bundles
 " :BundleInstall(!)    - install(update) bundles
 " :BundleSearch(!) foo - search(or refresh cache first) for foo
 " :BundleClean(!)      - confirm(or auto-approve) removal of unused bundles
 "
 " see :h vundle for more details or wiki for FAQ
 " NOTE: comments after Bundle command are not allowed..
#这里可以写一些你自己的配置
```
下面显示一些我暂时的配置文件
```
  filetype off                   " required!

 set rtp+=~/.vim/bundle/vundle/
 call vundle#rc()

 " let Vundle manage Vundle
 " required! 
 Bundle 'gmarik/vundle'

 " My Bundles here:
 "
 " original repos on github
 Bundle 'tpope/vim-fugitive'
 Bundle 'Lokaltog/vim-easymotion'
 Bundle 'rstacruz/sparkup', {'rtp': 'vim/'}
 Bundle 'tpope/vim-rails.git'
 Bundle 'scrooloose/nerdtree'
 Bundle 'kien/ctrlp.vim'
 Bundle 'msanders/snipmate.vim'
 Bundle 'mileszs/ack.vim'
 Bundle 'Shougo/neocomplcache.vim'
 Bundle 'Townk/vim-autoclose'
 Bundle 'Lokaltog/vim-easymotion'
 Bundle 'Lokaltog/vim-powerline'
 " vim-scripts repos
 Bundle 'L9'
 Bundle 'FuzzyFinder'
 " non github repos
 Bundle 'git://git.wincent.com/command-t.git'
 " git repos on your local machine (ie. when working on your own plugin)
 " ...

 filetype plugin indent on     " required!
 "
 " Brief help
 " :BundleList          - list configured bundles
 " :BundleInstall(!)    - install(update) bundles
 " :BundleSearch(!) foo - search(or refresh cache first) for foo
 " :BundleClean(!)      - confirm(or auto-approve) removal of unused bundles
 "
 " see :h vundle for more details or wiki for FAQ
 " NOTE: comments after Bundle command are not allowed..

 " NERDTree config
 map <F2> :NERDTreeToggle<CR>
 autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") &&  b:NERDTreeType == "primary") | q | endif

 "neocomplache config
 let g:neocomplcache_enable_at_startup = 1
 let g:neocomplcache_force_overwrite_completefunc = 1

 "other config
 set nu
 set mouse=a
 set tabstop=2
 let mapleader = ","  
 let g:mapleader = ","  
 map Y "+y  
 map P "+p  ""

 "easymotion
 let g:EasyMotion_leader_key = '<Leader>'

 "powerline config
 set laststatus=2
 set t_Co=256   
 set encoding=utf-8  
 set fillchars+=stl:\ ,stlnc:\
```
#### 5.安装你的插件
（1）保存退出当前的vim
（2）~~重新打开vim，输入命令```:BundleInstall```,然后就开始安装你的插件了。~~
重新打开vim,在命令模式下输入命令`PluginInstall`。
#### 6.如何移除插件
（1）编辑.vimrc文件移除的你要移除的插件行
（2）保存退出当前的vim
（3）~~重新打开vim，输入命令```:BundleClean```。~~
重打开vim，在命令模式下输入命名`PluginClean`。

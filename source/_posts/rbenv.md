---
title: rbenv实践
date: 2014-01-02 12:19:48
categories:
    - tech
tags:
    - ruby 
    - rbenv
---
从开始学习Ruby到现在，机子上一直在使用Rvm进行Ruby的版本控制。最近租了个服务器，准备学习如何部署Rails应用，看了[happypeter](http://happycasts.net/)前辈的一些视频，他竭力推介使用rbenv，并“扁”了一下Rvm。给我最直观的感觉：Rbenv相比Rvm，功能虽然减少了，但是更加简洁。所以现在想尝试一下Rbenv，并进行简单的使用记录。
#### 1.如何安装：
面对如何安装的问题，首先想到的当然是查看Rbenv的github了，你可以参考[这里](https://github.com/sstephenson/rbenv#basic-github-checkout)。但是我们有更简单的一种方法（详细在[这](https://github.com/fesplugas/rbenv-installer)）,下面一条命令解决
```
curl https://raw.github.com/fesplugas/rbenv-installer/master/bin/rbenv-installer | bash
```
当然这也不是绝对的，大多情况下，执行上面的命令后仍然找不到rbenv这个命令。那是因为Rbenv还没加载到你的路径下，你可以添加一下内容到~/.bashrc等文件中
```
export RBENV_ROOT="${HOME}/.rbenv"

if [ -d "${RBENV_ROOT}" ]; then
  export PATH="${RBENV_ROOT}/bin:${PATH}"
  eval "$(rbenv init -)"
fi
```
然后从新登入系统，检查Rbenv是否可用
```
rbenv -v
```
#### 2.使用Rbenv安装Ruby
安装ruby的时候需要安装一些相应的文件类库等。如果你使用的是ubuntu12.04，那就很方便了（注意如果按照上面的方法安装的话，不单单安装了Rbenv还安装了一些rbenv的相关插件，下面这条命令就是依赖其中的一个插件）
```
rbenv bootstrap-ubuntu-12-04 #安装了所需要的一些类库
```
然后开始安装ruby，先查看一下有哪些ruby版本可以安装
```
rbenv install --list
```
然后就可以安装相应的ruby版本了
```
rbenv install 2.0.0-rc2 #这样就安装了2.0.0这个版本
```
设定默认的ruby版本，以后除特殊要求，默认使用这个本版的ruby
```
 rbenv global  2.0.0-rc2 #这样以后默认使用2.0.0
```
但在某些项目中要使用对应的ruby版本，你可以先进入项目的目录，执行
```
rbenv local 1.9.3 #当前目录使用 1.9.3, 会生成一个 `.rbenv-version` 文件
```
#### 3.相对与rvm，使用rbenv的话特别注意以下这个问题
```
rbenv rehash                 # 每当切换 ruby 版本和执行 bundle install 之后必须执行这个命令
```
#### 4.其他
查看所有安装的ruby版本
```
rbenv versions
```

查看当前使用的ruby版本
```
rbenv version
```
如何删除一个ruby本版，这非常的简单，只要rm -rf 相应的版本
>As time goes on, Ruby versions you install will accumulate in your ~/.rbenv/versions directory.

>To remove old Ruby versions, simply rm -rf the directory of the version you want to remove. You can find the directory of a particular Ruby version with the rbenv prefix command, e.g. rbenv prefix 1.8.7-p357.

>The ruby-build plugin provides an rbenv uninstall command to automate the removal process.

参考：
[http://happycasts.net/episodes/75?autoplay=true](http://happycasts.net/episodes/75?autoplay=true)
[http://ruby-china.org/wiki/rbenv-guide](http://ruby-china.org/wiki/rbenv-guide)


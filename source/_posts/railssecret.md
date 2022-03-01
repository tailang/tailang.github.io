---
title: Rails小记录：token secret
date: 2013-08-04 14:00:06
categories:
    - tech
tags:
    - rails
---
token secret就像一个密钥，可以对cookic进行加密，从而保护网站的安全性。有时我们从github上clone一个Rails项目进行本地测试，可能会发现下面的现象

错误现象：
```
ArgumentError (A secret is required to generate an integrity hash for cookie session data. Use config.secret_token = "some secret phrase of at least 30 characters"in config/initializers/secret_token.rb)
```
那是因为项目的开发者为了安全考虑，对token secret做了处理。

解决方法：
```rake secret```
然后会生成一串代码，将它赋值给config/initializers/secret_taken.rb下
```
Rabel::Application.config.secret_token =‘...’
```

这个错误是其次的，而最重要的问题是：我们在写项目和部署是对token secret进行处理。让我们看看下面这篇博文，
>每一个Rails app都会获取一个很长，而且是随机生成的secret token，当使用rails new的时候，它会被生成并保存在config/initializers/secret_token.rb。里面的内容类似这样：  

>WebStore::Application.config.secret_token = '4f06a7a…72489780f'
因为rails自动创建secret token，所以很多开发者会忽略掉它。但是这个secret token就像是你的应用的管理员钥匙。如果你拥有了secret token，那样伪造会话和提升权限就会变得很容易。这是其中一个十分重要而且敏感的数据需要去保护的。加密是保护你的钥匙的最佳办法。
但是很不幸，rails并不能很好的处理这些secret token。secret_token.rb文件会被加入到版本控制当中，复制到GitHub，CI服务器和每一个开发人员的电脑。
最佳实践：在不同的环境中使用不同secret token。在应用中插入ENV变量就可以实现这个目的。另外一个替代方法是，在部署过程中把secret token作为符号链接。
修复：至少，rails必须通过.gitignore来忽略config/initializers/secret_token.rb文件。开发人员在部署的时候，用一个符号链接来替代生产环境的token，或者把初始化器转变为使用ENV 变量来初始化（例如Heroku）
我将会进一步提出rails创建一个保存serect token的机制方案。我们有大量的库提供安装指引如何把secret token加入到初始化器中，但是这并不是一个好的实践。同时，至少还有俩个解决方案来处理这个问题：ENV变量和初始化器的符号链接。
rails提供一个简单的API给开发人员来管理secret token，而且后台还是可插拔的（就像缓存和会话存储）。

总结的两个方法就是1,将secret token设置成环境变量; 2，用软连接
注意的地方是：不要将secret token文件加入git版本库

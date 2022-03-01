---
title: Rails小记录：数据库及model层唯一性验证
date: 2013-03-6 14:00:06
categories:
    - tech
tags:
    - rails
---
上学期看《ruby on rails tutorial》的时候，特别留意了下面的两段话，虽然当时很困惑，但也只能强记在心里。
>确保 Email 地址的唯一性(这样才能作为用户名),要使用 validates 方法的 unique 参数。提前说明,实现的
过程中存在一个很大的陷阱,所以不要轻易的跳过本小节,要认真的阅读。

>唯一性验证的不足
现在还有一个小问题,在此衷心的提醒你:唯一性验证无法真正保证唯一性。
唯一性验证无法真正保证唯一性。
不会吧,哪里出了问题呢?下面我来解释一下。
1. Alice 用 alice@wonderland.com 注册;
2. Alice 不小心按了两次提交按钮,连续发送了两次请求;
3. 然后就会发生下面的事情:请求 1 在内存中新建了一个用户对象,通过验证;请求 2 也一样。请求 1 创建
的用户存入了数据库,请求 2 创建的用户也存入了数据库。
4. 结果是,尽管有唯一性验证,数据库中还是有两条用户记录的 Email 地址是一样的。
相信我,上面这种难以置信的过程是可能会发生的,只要有一定的访问量,在任何 Rails 网站中都可能发生。幸好
解决的办法很容易实现,只需在数据库层也加上唯一性限制。我们要做的是在数据库中为 email 列建立索引,然
后为索引加上唯一性限制。  

最近翻阅了一些网上的资料，终于解开了心中的疑问，虽然只停留在问题的表面。
######既然既能在model层进行验证，又能在database验证，那我们应该怎么选择呢？
如果遇到unique的验证，应该在database层进行验证，有需要再在model层验证一下。详细的可以参考下面两个讨论，[stackoverflow](http://stackoverflow.com/questions/13122791/rails-validation-in-model-vs-migration)  （请仔细看第一个回答的3个reasons）
[rubychina](http://ruby-china.org/topics/4717),相信你会受益匪浅。  
###### 那为什么在unique验证的时候，只在model验证不安全呢？
除了上文说的“不小心按了两次提交按钮,连续发送了两次请求” 还可能出现下面这种情况，如果 你开启了多个rails进程，这时有非常多的用户访问，当两个或多个用户同时提交了相同的信息，也会同时将信息写入数据库，它的大概过程是这样的：
>假设启动了两个进程，两个用户同时提交了相同的信息
1.Model.save in PROCESS 1 #进程1在model层准备保存提交的数据
2.Model.save in PROCESS 2#进程2在model层准备保存提交的数据
3.SELECT FROM PROCESS 1 (result – no record in DB)#进程1查找数据库发现没有记录
4.SELECT FROM PROCESS 2 (result – no record in DB)#进程2查找数据库发现也没有记录
5.INSERT FROM PROCESS 1#进程1将数据插入数据库
6.INSERT FROM PROCESS 1#进程2将数据插入数据库

详细的解释你可以查看一下下面的文章[netmaniac](http://nhw.pl/wp/2009/07/20/are-activerecord-validations-worth-anything)

所以为了确保唯一性，最好在数据库层进行验证。

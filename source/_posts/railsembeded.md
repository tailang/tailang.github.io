---
title: Rails小记录：嵌套表单
date: 2013-04-18 14:00:06
categories:
    - tech
tags:
    - rails
---
什么是嵌套表单呢？举个简单的例子吧，比如你有两个表，一个User表，另一个Account表，他们是一对一的关系（也可以一对多等）。现在需要提交一个表单的时候同时提交User，Account对应的字段数据。
在Rails中有一种简单的方法解决，分别使用了这些方法
```
accepts_nested_attributes_for 
attr_accessible
fields_for
```
下面我们用代码进行说明一下。
```
class User < ActiveRecord::Base
   has_one :account 

   accepts_nested_attributes_for :account  #注意添加这两行
   attr_accessible  :account_attributes

end 
```
```
class Account<ActiveRecord::Base
belongs_to :user
```

```
class UsersController < ApplicationController
  def new
    @user = User.new
    @user.account.build #不要遗漏
  end

 def create
     @user = User.new(params[:user])
     if @user.save
       ...         
     end
  end
end
```

users/new.html.erb:
```
 <% form_for @user do |f| %>
    <%= f.text_field :name %>
    <% f.fields_for :account do |pf| %> #注意这里
      <%= pf.text_field :age %>
    <% end %>
 <% end %>
```

这是一对一的情况，假设是一对多呢，如User has_many :accounts ; Account belongs_to :user
这种情况下只要将对应的account改写成accounts，就ok了

一些深入参考：
[http://robots.thoughtbot.com/post/52960938209/accepts-nested-attributes-for-with-has-many-through](http://robots.thoughtbot.com/post/52960938209/accepts-nested-attributes-for-with-has-many-through)

[http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html)


以上是Rails3中，如果在Rails4中使用了Using Strong Parameters ,参考这里
去掉model中的`attr_accessible  :account_attributes`，在对应的controller中添加Strong Parameters的方法
[http://www.railsexperiments.com/using-strong-parameters-with-nested-forms/](http://www.railsexperiments.com/using-strong-parameters-with-nested-forms/)


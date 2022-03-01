---
title: Rails小记录：devise登入注册
date: 2013-05-11 14:00:06
categories:
    - tech
tags:
    - rails
---
.添加gem
```
gem 'devise'
```
2.bundle install
3.rails g devise:install #将看到如下的 提示，满足这些条件才能正常使用devise
```
Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here 
     is an example of default_url_options appropriate for a development environment 
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { :host => 'localhost:3000' }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root :to => "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying Rails 3.1+ on Heroku, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

       rails g devise:views
```
4.rails g devise User #生成相应的model和migration
model：
```
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :token_authenticatable, :confirmable,
  # :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :confirmable #添加该项，注册后要邮箱验证

  # Setup accessible (or protected) attributes for your model
  attr_accessible :email, :password, :password_confirmation, :remember_me
  # attr_accessible :title, :body
end
```
migration：
```
#默认一下部分是注释的，因为我们在user的model中添加了confirmable，所以去掉这里的注释
 ## Confirmable
       t.string   :confirmation_token
       t.datetime :confirmed_at
       t.datetime :confirmation_sent_at
       t.string   :unconfirmed_email # Only if using reconfirmable

add_index :users, :confirmation_token,   :unique => true
```
5.rake db:migrate #数据迁移
6.接下来我们按照提示去配置mail，暂时只配置development环境
config/initializer/devise.rb
```
config.mailer_sender = "xxx@126.com" #设置网站邮箱,我暂时使用网易邮箱测试
config.password_length = 6..128 #这里顺便把默认密码长度设置一下，默认值是8-128，我将它设置为6-128
```
上面我们install devise的时候已经有提示要我们配置下面这个文件（在development环境，如果是production环境就编辑prodaction.rb）
config/environment/development.rb
```
# Don't care if the mailer can't send
  config.action_mailer.raise_delivery_errors = true #改为true，并添加如下内容
  config.action_mailer.default_url_options = { :host => "localhost:3000" }
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = {
    :address => "smtp.126.com",
    :port => 25,
    :domain => "126.com",
    :authentication => :login,
    :user_name => "xxx@126.com", #你的邮箱
    :password => "xxxxxx" #你的密码
  }
```
7.接下来在app/views/layouts/application.html.erb添加自己的flash messages
```
<% flash.each do |key,value| %>
	<div class="alert alert-<%= key %>">
	  <button type="button" class="close" data-dismiss="alert">×</button>
		<%= value %>
	</div>
    <% end %>
```
8.确保网站有root页面
9.rails g devise:views #生成登录，注册等views

#####注意下面一些参量方法
```
new_user_session_path #登录路径
new_user_registration_path #注册路径
destroy_user_session_path#账户退出路径
edit_registration_path#修改密码

user_signed_in? #用户是否登录
current_user #当前用户
before_filter :authenticate_user! 
# To set up a controller with user authentication, just add this before_filter:
```
#####devise views一些文件注释
```
views/devise/confirmations/new.html.erb #重发注册后的验证邮件，记得打开confirmation模块
views/devise/mailer/confirmation_instructions.html.erb#注册后验证邮箱发送内容
views/devise/mailer/reset_password_instructions.html.erb#重设密码所发邮件内容
views/devise/mailer/unlock_instructions.html.erb#登录次数过多帐号被锁定,所发邮件内容
views/devise/passwords/edit.html.erb#忘记密码后，通过邮件链接来到的修改密码的页面
views/devise/passwords/new.html.erb#你点击忘记密码，来到的页面，在这你可以填写你的邮箱,然后会向你发送一个邮件
views/devise/registrations/edit.html.erb#修改密码的页面
views/devise/registrations/new.html.erb#注册页面
views/devise/sessions/new.html.erb#登入页面
```

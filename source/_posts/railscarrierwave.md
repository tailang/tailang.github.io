---
title: Rails小记录：carrierwave图片上传
date: 2013-02-14 14:00:06
categories:
    - tech
tags:
    - rails
---
1.添加gem
```
gem 'carrierwave', '0.6.2'
gem 'mini_magick' #不使用rmagick，占内存
```
2.bundle install
3.为users表添加一个avatar字段,也可以为其他名称，注意相应的代码修改
```
rails g migration add_avatar_to_users avatar:string
rake db:migrate
```
4.生成Avatar，跟你添加的字段相同
```
rails generate uploader Avatar #将会生成文件app/uploaders/avatar_uploader.rb
```
5.为user的model user.rb添加如下代码，使表之间关联
```
mount_uploader :avatar, AvatarUploader
```
6.接下来进行修改app/uploaders/avatar_uploader.rb，下面是一个例子
```
# encoding: utf-8

class AvatarUploader < CarrierWave::Uploader::Base
  include CarrierWave::MiniMagick #使用minimagick处理压缩图片，确保安装magickimage这个东东，ubuntu可以sudo apt-get install magickimage

  # Choose what kind of storage to use for this uploader: 
  storage :file
  # storage :fog

  # Override the directory where uploaded files will be stored.
  # This is a sensible default for uploaders that are meant to be mounted:
  def store_dir  #定义上传到哪个文件夹下
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  def default_url  #可以定义默认图片，如过用户没有上传图片，则可以使用默认的图片
    "avatar/#{version_name}.png"
  end

  
#图片的处理，有不同版本大小，网站可以在不同的地方调用不同的图片大小
 version :normal do
    process :resize_to_fill => [48, 48]
  end

  version :small do
    process :resize_to_fill => [16, 16]
  end

  version :large do
    process :resize_to_fill => [64, 64]
  end

  version :big do
    process :resize_to_fill => [120, 120]
  end
  # Add a white list of extensions which are allowed to be uploaded.
  # For images you might use something like this:
#指定上传文件的格式
  def extension_white_list
    %w(jpg jpeg gif png)
   end

  # Override the filename of the uploaded files:
  # Avoid using model.id or version_name here, see uploader/store.rb for details.
  # def filename
  #   "something.jpg" if original_filename
  # end

end
```
7.如何在表单中上传
```
<%= form_for(@user) do |f| %>
  <div class="field">
    <%= f.file_field :avatar %>
    <%= f.hidden_field :avatar_cache %>
  </div>
...
<%end%>
```
8.如何显示图片
```
    <%= image_tag(@user.avatar_url(:large)) if @user.avatar %>#这里的:large就是指定图片的版本为large 64x64大小
```
##### 详细查看carrierwave的[reademe](https://github.com/carrierwaveuploader/carrierwave/blob/master/README.md)

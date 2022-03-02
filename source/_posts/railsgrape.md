---
title: 使用Rails Task查看Grape API路由
date: 2015-12-10 15:29:46
categories:
    - tech
tags:
    - rails
    - grape
---
年后到现在，公司工作量突增，996的工作时长搞的我们各个身心疲惫（吐吐槽，工作还是要认真做的，哈哈）~~ 但平时几个好友还是会聚聚利用业余时间做一些有趣的东东。在最近的一个项目中，我们准备使用Rails+Grape的方案作为API服务，在开发的过程中，没有现成的task去获得已开发的API 路由（包括自定的路由显示格式）。如果只是使用Rails，我们可以轻松的使用`rake routes`罗列所有的路由，那如果搭载了Grape该咋办？其实我们也可以编写自己的task来实现...
首先创建一个task任务(假设项目名是atom)：atom/lib/tasks/routes.rake
```ruby
namespace :api do
  desc 'API Routes'
  task routes: :environment do
    Atom::API.routes.each do |route|  #这里的迭代后得到的route是Grape的路由对象，可以查看Grape源码
    #因为Grape本来就提供了route对象，我们本可以取出对应值直接打印就可以，但它提供的格式我不太喜欢，所以进行了重新拼接
      if route.route_path && route.route_method && route.route_version && route.route_path.include?(":version")
        http_method = route.route_method
        api_path = route.route_path.gsub ":version", route.route_version
        puts "#{http_method}:#{api_path}"
      end
    end
  end
end
```
然后你就可以像使用`rake routes`一样使用`rake api:routes`就可以获得对应的API路由了，其样子如下，你也可以自己拼接数据显示成其它格式
```
GET:/api/v1/user/name
```
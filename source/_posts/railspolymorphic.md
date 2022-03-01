---
title: Rails小记录：多态关联
date: 2013-06-20 14:00:06
categories:
    - tech
tags:
    - rails
---
刚开始看Rails Guide的时候对多态的表关联真的是一头雾水。后来自己写了一个博客应用的时候用到了acts_as_commentable这个gem，它就是用到了多态表的关联，然后我又看了Terry在railscasts china上的[视频](http://railscasts-china.com/episodes/file-uploading-by-carrierwave?autoplay=true)，对多态的理解就深了很多。
###### 理解什么是多态
一般表的关联有一对一，一对多，多对多，这些都是非常好理解的，然后对于多态的表关联可能稍微有点不好理解。其实多态关键就是一个表关联到多个表上。就如Comment（评论）表吧，一个Topic应该有Comment（一个帖子应该有许多的评论），除此之外Micropost（微博）也可能有很多的Comment。然后一个网站中既有Topic的论坛功能，又有Micropost的功能，我们怎么处理Comment表呢？当然我们可以建两个独立的表比如TopicComment和MicropostComment，再分别关联到Topic和Micropost上，但这不是一种好的选择，我们可以只建一个表，然后去关联这两个表，甚至多个表。这也就实现了多态的能力。
######一个例子
1.首先我们先生成一个Comment 的model，假设已经有Topic和Micropost这两个model了
```
rails g model comment content:text commentable_id:integer comment_type:string
```
2.然后我们 会得到一个migration
```
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :content
      t.integer :commentable_id
      t.string  :commentable_type
      t.timestamps
    end
  end
end
```
也可以通过t.references来简化上面的
```
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.text :content
      t.references :commentable, :polymorphic => true #这里指明了多态，这样会生成comment_id和comment_type这两个字段的，如上
      t.timestamps
    end
  end
end
```
多态魔法就在这里，commentable_typle字段用于指明comment所关联的表的类型，如topic或micropost等，而comment_id用于指定那个关联表的类型对象的id。如：可以把一个comment关联到第一篇topic上，那么comment_type字段为topic，而comment_id为对应topic对象的id  1,同理这样就可以关联到不同表了，从而实现多态的关联。

3,数据迁移 ```rake db:migrate```就能生成我们要的表了

4,对model进行操作从而现实表的关联
```
####comment model
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true  
end
```
看到没有，这里的comment belongs_to没有写topic，micropost等，而写了commentable,因为commentable中有type和id两个字段，可以指定任何其他model对象的，从而才能实现多态，如果这里写belongs_to topic的话就没办法实现多态了。
然后我们看看topic和mocropost的model该如何写。

```
class Topic < ActiveRecord::Base
  has_many :comments, :as => :commentable
end
```
```
class Micropost < ActiveRecord::Base
  has_many :comments, :as => :commentable
end
```

看到这里的as了吗？as在这我们可以解释为：作为（我的理解，可能这种理解补科学，哈哈），也就是说Topic有许多的comments，但是它是通过将自己作为commentable，实现的。Micropost同理。

然后就是controller和views中（如form表单）的设计了，这也是我刚学的时候，最头疼这个了，因为对params参数通过表单到controller的传递没掌握好。

在写这些之前，我们先看看如何写路由吧，因为一个topic有多个comments，Micropost同理。所以我们可以这样写
```
resources :topics do
  resources :comments
end

resources :microposts do
  resources :comments
end
```

然后我们通过命令```rake routes```就可以得到相应的路由了如：
```
          topic_comments GET    /topics/:topic_id/comments(.:format)          comments#index
                                    POST   /topics/:topic_id/comments(.:format)          comments#create
       new_topic_comment GET    /topics/:topic_id/comments/new(.:format)      comments#new
      edit_topic_comment GET    /topics/:topic_id/comments/:id/edit(.:format) comments#edit
           topic_comment GET    /topics/:topic_id/comments/:id(.:format)      comments#show
                                    PUT    /topics/:topic_id/comments/:id(.:format)      comments#update
                                   DELETE /topics/:topic_id/comments/:id(.:format)      comments#destroy
```

这些待会我们会用到。

然后我们再来分析controller和views之间的参数传递。我们通过完整的创建comment的过程进行说明

(1)首先页面上肯定有一个创建comment的连接或按钮（假设创建comment的表单和topic show页面不在统一页面上），代码应该是这样的：
```
<%= link_to "发表评论", new_topic_comment_path%>
```
(2)点击这个链接后，通过路由来到controller中的new方法(同时会将对应的topic相关的参数传给controller)
```
def new
  @topic = Topic.find(parmas[:id]) #找到comment属于的topic
  @comment = @topic.comments.build #建立这个关系
end
```
(3)经过这个方法（action）后，页面来到了comments/new.html.erb,在这个页面中有一个评论的表单，大概是这样的
```
 <%= form_for([@comment.commentable, @comment]) do |f| %>
  ......
<%end%>
```
这个表的参数是一个数组，分别是@comment.commentable和@comment，如果没有关联的化，一个@comment就ok了。但这里是关联的，待会传给create方法时要两个参数一个是comment的实体，还有一个就是commentable，这里也就是topic。

还记得new中的```@comment = @topic.comments.build```的吗，这里就暂时将对应的topic对象写入commentable（注意：只是暂时建立关系，还没有写入数据库），所以在form表单中的@comment.commentable就是指@topic。

(4)然后你填完表单后，按提交按钮后，表单中的参数（包括commentable，@post的id等信息），一起来到controller的create方法中
```
def create
  Topic.find(parmas[:topic_id]).comments.create(parmas[:comment])
  ......
end

```
这样就真正创建了一个新的comment。micropost同理。

其实多态讲的也差不多了，但在提一个地方
假设一个comment已经建立了，它的commentable_type是:topic.comment_id是1。如果我们得到了这个id为1的topic，@topic，那么我们怎么得到它的comments呢？是的很简单，直接 ```@topic.comments```就ok了。但是反过来呢，我们得到了这个comment，@comment，我们如何得到对应的topic的信息呢？我以前刚学的时候，就用了```@comment.topic```，呵呵，没错，得到的是一串错误，正确一概是```@comment.commentable```

关于多态我们已经讲的差不多了。

补充：
上面的例子comment的表单是独立在comments/new.html.erb中的，但是一般的应用comment的表单是在topics/show.html.erb中，也就是上面一个topic，topic下有一个comment表单。这样的话在controller中我们就不需要new这个方法了，那么我们在哪建立关系呢？
```
@comment = @topic.comments.build #建立这个关系
```
我们就在表单的```<%= form_for ...%>```之前写```<@comment = @topic.comments.build>```

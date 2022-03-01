---
title: Rails小记录：html_safe、raw、sanitize
date: 2013-07-17 14:00:06
categories:
    - tech
tags:
    - rails
---
Rails3后默认设置为溢出html，从而防止因为疏忽而造成跨站脚本攻击XSS。
那么什么是跨站脚本攻击呢？比如我们最常见的评论系统，假设某个恶意用户在评论中输入这样一个脚本并提交
```
<stript>...</stript>
```
其中...为一段恶意的脚本。如果不做溢出的话，这个评论不会作为普通的字符串显示在浏览器上，而是会去执行```<stript>...</stript>```这个脚本，从而造成损失。Rails默认做了溢出，所以不会去执行这段代码，它会将`<stript>...</stript>`处理成`&lt;script&gt;...;&lt;/script&gt;`,从而在浏览器上只是显示为一段普通的字符`<stript>...</stript>`.

但是有的时候，我们就是要用到不溢出的html，怎么办呢？那就要用到html_safe、raw、sanitize了。
举个例子：
```
<%= link_to "<span>链接</span>" url_path %>
```
这里我们的“链接”两个字用到`<span>`（我们对span定义了某种样式）这个标签。因为Rails默认是溢出的，所以这个将显示为`<span>链接</span>`，这不是我们想要的。所以我们可以将它做不溢出的处理
```
<%= link_to raw("<span>链接</span>") url_path %>
```
这样的话，就不会溢出了，我们会得到“链接”这两个字和它所对应的<span>样式了。

注：这个例子举的不好，其是对于link_to，我们完全可以用块来处理这种问题，如
```
<%= link_to  url_path do %>
  <span>链接</span>
<% end %>
```
但这我只是想说明不溢出的问题，所以用了link_to的例子


明白了什么是溢出和不溢出，然后我们看看这几个不溢出处理的方法之间的区别（html_safe、raw、sanitize）
1. html_safe 作用就是对字符串做处理，使他不溢出，它可以用在models和helpers
2. raw 作用和html_safe一样，它只是一个helper方法，其内部调用了html_safe。作为一个helper方法它只能用在controller和view中使用

然后说说 sanitize 这个方法，它是一个很灵活的方法，它一般可以溢出<stript>这些危险的标签，但一些相对安全的标签可以不溢出，还可以自己定义哪些标签不溢出，看一下它的API
>sanitize(html, options = {}) Link
This sanitize helper will html encode all tags and strip all attributes that aren’t specifically allowed.

>It also strips href/src tags with invalid protocols, like javascript: especially. It does its best to counter any tricks that hackers may use, like throwing in unicode/ascii/hex values to get past the javascript: filters. Check out the extensive test suite.

><%= sanitize @article.body %>
You can add or remove tags/attributes if you want to customize it a bit. See ActionView::Base for full docs on the available options. You can add tags/attributes for single uses of sanitize by passing either the :attributes or :tags options:

>Normal Use

><%= sanitize @article.body %>
Custom Use (only the mentioned tags and attributes are allowed, nothing else)

><%= sanitize @article.body, tags: %w(table tr td), attributes: %w(id class style) %>
Add table tags to the default allowed tags

>class Application < Rails::Application
  config.action_view.sanitized_allowed_tags = 'table', 'tr', 'td'
end
Remove tags to the default allowed tags

>class Application < Rails::Application
  config.after_initialize do
    ActionView::Base.sanitized_allowed_tags.delete 'div'
  end
end
Change allowed default attributes

>class Application < Rails::Application
  config.action_view.sanitized_allowed_attributes = 'id', 'class', 'style'
end
Please note that sanitizing user-provided text does not guarantee that the resulting markup is valid (conforming to a document type) or even well-formed. The output may still contain e.g. unescaped ‘<’, ‘>’, ‘&’ characters and confuse browsers.

Rails为了安全默认做了溢出，那我们有的时候又要不溢出，考虑到安全该怎么选择？
大多数的建议是： 内部人员输入的内容，需要不溢出的时候可以做不溢出处理；如果是外部人员输入的内容，最好做全部的溢出，防止被攻击，即使迫不得已做不溢出处理，也要谨慎使用这些方法。


说了这么多，好像还没提h()方法，其实它的作用和raw，html_safe相反，就是把不溢出的内容，转成溢出。因为Rails3以前默认是不溢出的，所以要手动的使用h()方法做溢出。

以上内容可能有些地方，描述错误，愿指正

参考：[http://stackoverflow.com/questions/4251284/raw-vs-html-safe-vs-h-to-unescape-html](http://stackoverflow.com/questions/4251284/raw-vs-html-safe-vs-h-to-unescape-html)

[http://ihower.tw/rails3/security.html](http://ihower.tw/rails3/security.html)

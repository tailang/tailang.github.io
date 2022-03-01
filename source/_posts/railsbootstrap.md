---
title: Rails小记录：bootstrap
date: 2013-01-10 14:00:06
categories:
    - tech
tags:
    - rails
---
接触rails也有很长一段时间了，但是进步一直很慢，前端时间练手写了一个blog，因为不怎么会前端所以选择了bootstrap这个框架，用的gem是[bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass)。现在准备自己写一个社区论坛，前端还是使用bootstrapt，但是最近bootstrap3的RC版已经出来了，但bootstrap-sass暂时只支持到2。所以准备使用anjlab-bootstrap-rails这个gem，现将使用的方法做一下小记。   
1.当然是使用bundle安装这个gem
2.rename这个文件app/assets/stylesheets/application.css为app/assets/stylesheets/application.css.scss
3.在app/assets/stylesheets/application.css文件中引入bootstrapt  
```sass
 *= require_self
 *= require_tree .
 */

@import "twitter/bootstrap";
```
4.在app/assets/javascripts/application.js引入bootstrapt
```javascript
//= require jquery
//= require jquery_ujs
//= require twitter/bootstrap
//= require_tree .
```
5.如果你对默认的样式不满意，如size，color，[详细的参量在这](http://getbootstrap.com/customize)。 可以自己定制
方法一：直接在application.css.scss文件中修改，例：
```sass
*= require_self
 *= require_tree .
 */

$navbar-height: 5px;

$container-tablet: 945px; //container sizes 
$container-desktop: 945px;
$container-large-desktop: 945px;
//在@import "twitter/bootstrap";之前

@import "twitter/bootstrap";
```
方法二(推介)：重新创建一个文件，在该文件中修改，再在application.css.scss文件中引入，例：创建新文件app/assets/stylesheets/plugins/bootstrapt.css.scss,并编辑
```sass
$navbar-height: 5px;

$container-tablet: 945px; //container sizes 
$container-desktop: 945px;
$container-large-desktop: 945px;
```
然后在application.css.scss中引入
```sass
*= require_self
 *= require_tree .
 */


@import "plugins/bootstrap.css.scss";
@import "twitter/bootstrap";
```
最后就可以在项目中快乐使用了。

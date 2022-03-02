---
title: Rails部署全记录
date: 2015-09-11 15:00:56
categories:
    - tech
tags:
    - rails
---
一路过来也部署过3、4个Rails App了，其中也使用过mina等远程部署项目，但每次去部署新的App还是会遇到一些大大小小问题。最近和几个朋友正在做一个应用，在部署的过程中又被自己坑了~所以今天准备总结一下，方便自己未来的部署之路，也方便Rails新手学习使用。

### 环境说明 
因为Linux的发型版太多，所以不能一一举例，经过自己的实践体验，发现Ubuntu可以更轻松的部署您的Rails App（个人看法，曾经在Centos下部署，遇到了好多坑）。所以该博文将基于以下的部署环境。

操作系统：Ubuntu14.04
Rails：4.2.0
Ruby：2.2.1
Mysql: 5.6.22
Nginx: 1.8.0
Puma: 2.11.0
Redis: 3.0.0
Grape: 0.11.0 #因为这次我们主要是将Grape挂载在Rails做API服务使用，所以会牵扯到一些Grape，但不影响其它普通Rails的部署
注：假设主机IP为：139.162.29.24 #乖！不要乱试了，这不是我们的服务器地址，是我乱写的~

### 前期准备 
#### 1.为主机创建新的非root用户
从各方面考虑，在部署项目的时候最好使用普通用户部署而不是Root用户。所以我将先创建一个新的用户tailang。

adduser tailang #新建用户tailang
passwd tailang #为tailang用户添加密码
这样就创建了一个名为tailang的新用户，当你切换到tailang这个用户后，会发现tailang无法使用sudo的权限。所以接下来我们将tailang添加到sudo用户列表。
首先切换到root，然后编辑/etc/sudoers,添加如下行

```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL
tailang    ALL=(ALL:ALL) ALL  #这是需要添加的行，将tailang加入到sudo
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```
#### 2.ssh免密码登入服务器
每次通过SSH登入服务器都需要输入密码Oh,shit不论你是否厌烦，反正我是厌烦了。那我该怎么整，才能免密码登入服务器呢？如果你使用过github、gitlab或其它一些服务，还记得要为账户添加ssh key吗？没错，在这我们也将通过使用ssh的公钥私钥实现免密码登入服务器。
本机生成ssh key： 您也可以参考这里
```
#填写您的邮箱，该命令的作用是生成ssh key，在生成的过程中需要你按几次enter键
ssh-keygen -t rsa -C "your_email@example.com"
然后参看~/.ssh文件夹，你会发现生成了对应的密钥

id_rsa #私钥文件
id_rsa.pub #公钥文件，该公钥待会将会用到
然后将对应的公钥文件上传到服务器，并做对应的权限控制

scp ~/.ssh/id_rsa.pub tailang@139.162.29.24:~/  #将本地的公钥文件上传到服务器tailang用户的主目录
服务器对应操作(tailang用户)

mkdir ~/.ssh
chmod 700 -R .ssh

touch ~/.ssh/authorized_keys
chmod 600 authorized_keys

cat id_rsa.pub >> ~/.ssh/authorized_keys  #将刚刚的公钥写入authorized_keys
```
最后测试验证免密码登入，返回本机输入ssh tailang@139.162.29.24，你将发现不输入密码也可以登入服务器

### 安装必要东东
 终于要步入正轨了，接下来我们将安装各种东东了，如Nginx、MySQL等。和在本地安装一样，你可以选择Ubuntu默认的源安装这些软件，但一般情况下默认源的软件版本都比较低，所以我会选择添加ppa来安装，如果你还不知道什么是ppa，可以打开Google搜索“Ubuntu ppa”，相信爱折腾你已经用过这东东了~
```
sudo add-apt-repository ppa:git-core/ppa  #添加git的ppa
sudo add-apt-repository ppa:nginx/stable  #添加nginx的ppa
sudo add-apt-repository ppa:ondrej/mysql-5.6 #添加mysql的ppa
sudo add-apt-repository ppa:chris-lea/redis-server #添加redis的ppa
sudo apt-get update #添加完ppa后，一定要记得更新一下
完成了这些步骤后就可以安装必要的软件了。

sudo apt-get install curl  #安装curl
sudo apt-get -y install git #安装git
```
#### 安装MySQL
```
sudo apt-get -y install libxml2-dev libxslt1-dev libmysqlclient-dev #添加必要的MySQL开发库，不然mysql2这个gem将无法工作
sudo apt-get -y install mysql-server #安装MySql
```

#### 安装Nginx
```
sudo apt-get install python-software-properties
sudo apt-get -y install nginx
```
#### 安装Redis
```
sudo apt-get -y install redis-server
```
#### 安装RVM，我们将通过RVM安装管理Ruby，当然你也可以选择RBENV
```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
source ~/.bashrc
source ~/.profile
```
#### 安装Ruby
```
#RVM具体操作请查看其文档
rvm install 2.2.1 #安装你要Ruby版本
rvm use 2.2.1 default #切换Ruby到2.2.1版本并将其设置成默认Ruby版本
```

### 配置

#### MySQL
 Ubuntu下按以上的方式安装MySQL，在安装的时候已经创建了一个root用户及你设置的root用户密码。没错，我以前的做法就是用该root用户作为我的Rails项目的链接用户，但我觉得这样并不安全，所以现在我的做法是新建一个项目用户，比如我的项目的名字为Atom，那么我会创建一个新的用户叫atom，并重新授予适量的权限，而不是像root用户那样可以为所欲为。

```
mysql -uroot -p #以root用户登入

mysql>CREATE USER atom IDENTIFIED BY '111111'; #创建一个叫atom的用户 密码为111111
GRANT ALL ON atom_production.* TO atom; #授予atom用户可以完全操作atom_production.*数据库
flush privileges; #刷新权限
```
注：也许当我们创建了atom这个新用户后，却无法通过该用户登入，而是遇到了如下的错误提示：

ERROR 1045 (28000): Access denied for user 'atom'@'localhost' (using password: YES)
引起该问题可能是因为，mysql数据库的user表中的一条记录先于atom这条记录匹配造成的，我暂时的解决方法是删除该记录（该记录的特征是：host字段值为localhost，user、password字段为空）

#### 用环境变量来设置一些数据 
一些比较敏感的设置数据，我们该怎样处理。目前我知道的比较流行的方法有
1.文件软连接 
2.设置环境变量。以前我一直采用软连接的方式，但实践告诉我这不是一个好方法，比如像database.yml文件，我首先需要把它放到.gitignore中，不让git提交。当将项目部署到服务器后要在其它地方创建database.yml等文件，然后软连接到项目中......这样很容易造成项目文件的不完整（有时会创建database.yml.exmple作为临时代替）。到了Rails4以后，我突然发现Rails默认采用了环境变量。如在config/secrets.yml中是这样写的(database.yml同样采用了环境变量)：
```
production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```
所以我一般会在tailang用户的~/.bashrc文件添加必要的环境变量，如：
```
export ATABASE_USER_NAME="atom"   #请填写您对应数据库用户名
export DATABASE_USER_PASSWORD="111111" #数据库用户密码
export SECRET_KEY_BASE="xxx" #对应的secret值   可以通过 bundle exec rake screte 获得
#你可以添加更多的你需要的环境变量 如redies的配置等
```
注：填写完记的执行source ~/.baserd

#### 做必须做的事 
完成以上的步骤，我一般不会急着去配置puma、Nginx之类的，而是先使用rails s测试
```
bundle install #安装必要的gem
rake db:create RAILS_ENV=production  #创建数据库
rake db:migrate RAILS_ENV=production #迁移数据库
rake assets:precompile #管道预编译
rails s -e production -b 0.0.0.0 #注意哦：记的绑定到0.0.0.0上，不然你通过浏览器访问139.162.29.24:3000是访问不到的
```
如果一切正常，应用也跑起来了，也可以访问了，那我们可以先吸口气了，放松放松了，毕竟我们的应用可以跑了嘛~~~

#### Puma配置 
对于Puma的配置我会在对应的Rails项目中创建两个文件，/config/puma.rb (作为puma的配置文件) /puma.sh (作为puma的启动关闭脚本，记的该文件的权限是可执行哦，记的赋值权限) 下面的配置及脚本仅供参考
```
#########/config/puma.rb

#!/usr/bin/env puma

# The directory to operate out of.
#
# The default is the current directory.
#
# directory '/u/apps/lolcat'

# Use a object or block as the rack application. This allows the
# config file to be the application itself.
#
# app do |env|
#   puts env
#
#   body = 'Hello, World!'
#
#   [200, { 'Content-Type' => 'text/plain', 'Content-Length' => body.length.to_s }, [body]]
# end

# Load “path” as a rackup file.
#
# The default is “config.ru”.
#
# rackup '/u/apps/lolcat/config.ru'

# Set the environment in which the rack's app will run. The value must be a string.
#
# The default is “development”.
#
environment 'production' #设置为生产环境

preload_app! 

# Daemonize the server into the background. Highly suggest that
# this be combined with “pidfile” and “stdout_redirect”.
#
# The default is “false”.
#
# daemonize
daemonize true

#设置pidfile（进程id）、state_path、stdout_redirect等文件的路径，这里都将文件输出到项目目录下方便查看
wd          = File.expand_path('../../', __FILE__)
tmp_path    = File.join(wd, 'log')
Dir.mkdir(tmp_path) unless File.exist?(tmp_path)

pidfile          File.join(tmp_path, 'puma.pid')
state_path       File.join(tmp_path, 'puma.state')
stdout_redirect  File.join(tmp_path, 'puma.out.log'), File.join(tmp_path, 'puma.err.log'), true

# Store the pid of the server in the file at “path”.
#
# pidfile '/u/apps/lolcat/tmp/pids/puma.pid'

# Use “path” as the file to store the server info state. This is
# used by “pumactl” to query and control the server.
#
# state_path '/u/apps/lolcat/tmp/pids/puma.state'

# Redirect STDOUT and STDERR to files specified. The 3rd parameter
# (“append”) specifies whether the output is appended, the default is
# “false”.
#
# stdout_redirect '/u/apps/lolcat/log/stdout', '/u/apps/lolcat/log/stderr'
# stdout_redirect '/u/apps/lolcat/log/stdout', '/u/apps/lolcat/log/stderr', true

# Disable request logging.
#
# The default is “false”.
#
# quiet

# Configure “min” to be the minimum number of threads to use to answer
# requests and “max” the maximum.
#
# The default is “0, 16”.
#
threads 0, 16  #配置最大最小线程数，根据实际情况配置

#配置绑定Nginx作为反向代理需要，默认使用“tcp://0.0.0.0:9292”，但文档上说unix://作为sock速度会提高10%，请根据实际情况配置
# Bind the server to “url”. “tcp://”, “unix://” and “ssl://” are the only
# accepted protocols.
#
# The default is “tcp://0.0.0.0:9292”.

#bind 'unix:///var/run/puma.sock'
#bind 'unix:///var/run/puma.sock?umask=0777'
# bind 'ssl://127.0.0.1:9292?key=path_to_key&cert=path_to_cert'

# Instead of “bind 'ssl://127.0.0.1:9292?key=path_to_key&cert=path_to_cert'” you
# can also use the “ssl_bind” option.
#
# ssl_bind '127.0.0.1', '9292', { key: path_to_key, cert: path_to_cert }

# Code to run before doing a restart. This code should
# close log files, database connections, etc.
#
# This can be called multiple times to add code each time.
#
# on_restart do
#   puts 'On restart...'
# end

# Command to use to restart puma. This should be just how to
# load puma itself (ie. 'ruby -Ilib bin/puma'), not the arguments
# to puma, as those are the same as the original process.
#
# restart_command '/u/app/lolcat/bin/restart_puma'

# === Cluster mode ===

# How many worker processes to run.
#
# The default is “0”.
#
#设置进程数
workers 0

# Code to run when a worker boots to setup the process before booting
# the app.
#
# This can be called multiple times to add hooks.
#
# on_worker_boot do
#   puts 'On worker boot...'
# end

# === Puma control rack application ===

# Start the puma control rack application on “url”. This application can
# be communicated with to control the main server. Additionally, you can
# provide an authentication token, so all requests to the control server
# will need to include that token as a query parameter. This allows for
# simple authentication.
#
# Check out https://github.com/puma/puma/blob/master/lib/puma/app/status.rb
# to see what the app has available.
#
# activate_control_app 'unix:///var/run/pumactl.sock'
# activate_control_app 'unix:///var/run/pumactl.sock', { auth_token: '12345' }
# activate_control_app 'unix:///var/run/pumactl.sock', { no_token: true }
########/puma.sh
#!/bin/sh

# set ruby GC parameters
RUBY_GC_HEAP_INIT_SLOTS=600000
RUBY_GC_HEAP_FREE_SLOTS=200000
RUBY_GC_MALLOC_LIMIT=60000000
export RUBY_GC_HEAP_INIT_SLOTS RUBY_GC_HEAP_FREE_SLOTS RUBY_GC_MALLOC_LIMIT

state_file="log/puma.state"

case "$1" in
  start)
    bundle exec puma -C config/puma.rb
    ;;
  stop)
    bundle exec pumactl -S $state_file stop
    ;;
  restart)
    bundle exec pumactl -S $state_file restart
    ;;
  status)
    bundle exec pumactl -S $state_file status
    ;;
  force-stop)
    bundle exec pumactl -S $state_file halt
    ;;
  *)
    echo $"Usage: $0 {start|stop|force-stop|restart|status}"
    ;;
esac
```
#### Nginx配置
修改/etc/nginx/nginx.conf
```
user root; #将user该成root，当user为nginx时，会出现权限不够，索性我就将user改成了root
当然你要根据你的服务器配置等因素修改其它参数，使性能最佳。
```
修改完后进入/etc/nginx/sites-enabled文件夹，将其中的default文件删除，然后进入/etc/nginx/sites-available/文件夹
```
sudo touch atom #创建对应你应用的文件，这个文件将作为我这个应用的Nginx配置，文件名随便取，也可以是aaa
sudo ln -s /etc/nginx/sites-available/atom /etc/nginx/sites-enabled/atom #将创建的文件软连接到sites-enabled目录
```
然后开始编辑/etc/nginx/sites-available/atom作为Nginx配置，一下配置作为参考
```
upstream puma {   #上游取名为puma
  server 127.0.0.1:9292;  #指定上游服务器，通过127.0.0.1:9292访问，这个和你的puma配置中的绑定有关
}

server {
  listen 80; #设置访问端口
  server_name exmple.com;  #你的域名 也可以是Ip地址，如139.162.29.24
  root /home/atom/atom/public; #资源根目录，对应rails项目中的public目录

  access_log /home/atom/atom-api/log/nginx.access.log; #设置nginx日志路径
  error_log /home/atom/atom-api/log/nginx.error.log; #设置nginx错误日志路径

  #下面的404和500是因为整个应用作为api服务，如果作为web应用你应该指定对应的html文件，如：
  # error_page 500 502 503 504 /500.html;
  
  error_page 404 @404;
  location @404 { echo '{"code": "404", "message": "您请求的资源不存在哦~"}'; }

  error_page 500 @500;
  location @500 { echo '{"code": "500", "message": "您的请求导致服务器出现异常~"}'; }

#静态缓存
  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

#反向代理
  location / {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://puma;  #注意哦，这里对应你的上游别名
    proxy_intercept_errors on;
  }

  client_max_body_size 4G;
  keepalive_timeout 10;
}
```
最后有点激动，迎来的可能是失败，也有可能是成功 完成了配置，接下来就是怀着忐忑的心情打开服务器。
```
./puma.sh  #记的该文件是有可执行权限的
ps aux | grep puma #查看Puma进程是否存在，如果不存在，则表明启动失败了，但不要灰心，信心爱折腾的你可以解决的
sudo /etc/init.d/nginx start #启动Nginx 注意查看状态，如果无法启动，那么可能你的nginx配置出现问题了；
```
如果访问出现50*错误不一定是Nginx的问题，也有可能是puma的问题，比如puma没有起来
最后让我们快乐的在浏览器中访问我们的应用吧~

### 最后想说的
上面介绍的是纯手动部署Rails应用，如果你是首次部署Rails应用，我建议采用该方法，这样可以让你了解部署的每一个流程。以后为了效率等，可以考虑使用mina 或 capistrano等远程部署工具
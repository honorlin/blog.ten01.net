---
layout: post
title: "安裝 Rails Nginx Passenger 上線環境 在CentOS上"
date: 2013-11-06 03:05:40 +0800
comments: true
categories: 
---


最近要在CentOS上面安裝Rails 4.0 的上線環境,遇到一些問題,將一些問題解決後,整理了一些步驟,可以在CentOS完成安裝Rilas的上線環境


1 做yum更新

  yum update

2 安裝 Ruby on Rails 所需要的一些套件

  yum groupinstall "Development Tools"
  yum install zlib-devel wget openssl-devel pcre pcre-devel make gcc gcc-c++ curl-devel

3 安裝 Ruby

先到Ruby官網看目前的最新版本是多少
[Ruby language site](http://www.ruby-lang.org/en/downloads/)
再將以下的安裝碼換成最新的版本

  cd /opt
  wget ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p247.tar.gz
  tar -zxvf ruby-2.0.0-p247.tar.gz
  cd ruby-2.0.0-p247
  ./configure --bindir=/usr/bin --sbindir=/usr/sbin/
  make -j3
  make install

4 安裝 Ruby gems

到 [RubyForge files page](http://rubyforge.org/frs/?group_id=126)
將以下的安裝碼,換成要安裝的版本

  cd /opt
  wget http://production.cf.rubygems.org/rubygems/rubygems-1.8.25.tgz
  tar -zxvf rubygems-1.8.25.tgz
  cd /opt/rubygems-1.8.25/
  ruby setup.rb
  RubyForge files page

5 更新 rubygems:

  gem update --system

6 用 gems 安裝 rake, rack 和 fastthread

  gem install rake rack
  gem install fastthread --no-rdoc --no-ri

7 安裝 Rails

way 1 指定版本

  gem install rails --version 3.0.4 --no-ri --no-rdoc

way 2 安裝最新版本

  gem install rails --no-ri --no-rdoc
x:  --no-ri --no-rdoc  參數是不安裝文件,加快安裝速度

8 安裝 nginx

Download the repo:

  wget http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm

  rpm -ivh nginx-release-rhel-6-0.el6.ngx.noarch.rpm

  yum install nginx

9 安裝 passenger

  sudo gem install passenger

10 安裝 nginx and nginx-passenger-module

  sudo passenger-install-nginx-module

11 將yum安裝的nginx和passenger-install-nginx-module安裝的nginx做結合

說明一下 passenger-install-nginx-module 所安裝的是和passenger一起編譯後的nginx
但是這樣安裝完之後,並沒有自動產生一個nginx的script,所以這裡的做法是在第8步驟,先用yum安裝nginx然後順便產生基本的script,再來再將yum安裝的nginx置換成和passenger一起編譯後的nginx,這樣就跟yum幫你安裝的設定檔無縫接軌了

方法如下：

  rm /usr/sbin/nginx

  ln -s /opt/nginx/sbin/nginx  /usr/sbin/nginx

12 設定ngnix.conf

找出

/opt/nginx/conf/nginx.conf

裡面的

passenger_root /usr/local/lib/ruby/gems/2.0.0/gems/passenger-4.0.20;
passenger_ruby /usr/bin/ruby;

再到

vi /etc/nginx/nginx.conf

在http裡面加上,變成如下

  http{
  .....
  passenger_root /usr/local/lib/ruby/gems/2.0.0/gems/passenger-4.0.20;
  passenger_ruby /usr/bin/ruby;
  ......
  }

13 新增conf檔在 /etc/nginx/conf.d/xxxx.conf

  server {
  listen 80;
  server_name eliving.co;
  root /home/eliving.co/public;
  passenger_enabled on;
  charset utf-8;
  rails_spawn_method smart;
  rails_env production;
  access_log /var/log/nginx/eliving.co_access.log;
  error_log /var/log/nginx/eliving.co_error.log;
  }

14 安裝nginx php環境 參考 ngnix php phpmyadmin 環境

15 安裝phpmyadmin 參考 ngnix php phpmyadmin 環境

16 安裝nginx mysql 參考 安裝CenOS Nginx,MySQL,PHP

17 因為rails app 要執行bundle 需要 compile 的東西都要安裝 xxx-devel 才能用

記得執行下面的指令,不然遇到bundle mysql2時會錯誤 

  yum install mysql-devel 

18 安裝rails  app

(1)上傳rails  app 到  /home/eliving/

(2)ssh登入後在此路徑/home/eliving/ 下執行 bundle

(3)設定public的權限和/home/eliving/的權限

(4)設定/config/database.yml的資料填上正確的資料庫使用資訊

14.登入phpmyadmin

新增

{rails_app_project_name}_development,  {rails_app_project_name}_production,  {rails_app_project_name}_test 三個資料庫,並設定為unicode

15.部署資料庫

  rake db:migrate RAILS_ENV="production"

16.啓動nginx

  /etc/init.d/nginx start


參考資料：

https://www.phusionpassenger.com/download/#open_source

https://library.linode.com/frameworks/ruby-on-rails-nginx/centos-5

https://github.com/useruby/rails-nginx-passenger-centos

http://ihower.tw/rails3/deployment.html

https://github.com/xdite/rails-nginx-passenger-ubuntu
---
layout: post
title:  "nginx unicorn mina部署rails项目"
date:   2015-03-26 11:41:00
categories: rails部署
comments: true
---
1. 安装nginx
sudo apt-get install nginx
命令会生成/etc/nginx/nginx.conf配置文件, /etc/nginx/sites-available和/etc/nginx/sites-enabled目录

nginx.conf配置文件:
{% highlight ruby %}
worker_processes 5;
user www-data; # for systems with a "nogroup"

pid /var/run/nginx.pid;
#error_log /path/to/nginx.error.log;

events {
  worker_connections 1024; # increase if you have lots of clients
  accept_mutex off; # "on" if nginx worker_processes > 1
}

http {
  include mime.types;

  default_type application/octet-stream;

  sendfile on;

  tcp_nopush on; # off may be better for *some* Comet/long-poll stuff
  tcp_nodelay off; # on may be better for some Comet/long-poll stuff

  gzip on;
  gzip_types application/json;

   include /etc/nginx/conf.d/*.conf;
   include /etc/nginx/sites-enabled/*;

}
{% endhighlight %}

sites-available/xiaocui文件配置:

{% highlight ruby %}
upstream app_server {
  server unix:/tmp/unicorn.xcuiblog.sock fail_timeout=0;
  #server 127.0.0.1:4201;
}

server {
  listen 80 default_server;
  server_name xcuiblog.com;
  #rewrite ^/(.*)$ http://xcuiblog.com permanent;
  try_files $uri/index.html $uri.html $uri @app;

  location @app {
    root /home/cuijunwei/PrivateCodes/xcuiblog/public;
    proxy_redirect     off;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header Host $host:83;
    proxy_set_header   X-Forwarded-Host $host;

    proxy_pass http://app_server;
    proxy_set_header Via    "nginx";
  }
}
{% endhighlight %}
OK, nginx配置完毕.

配置unicorn:
gem install 'unicorn'
rails项目中Gemfile添加gem 'unicorn'
rails项目中创建文件config/unicorn.rb:
{% highlight ruby %}
worker_processes 1

app_root = File.expand_path("../..", __FILE__)
working_directory app_root

# Listen on fs socket for better performance
listen "/tmp/unicorn.xcuiblog.sock", :backlog => 64
#listen 4201, :tcp_nopush => false

# Nuke workers after 30 seconds instead of 60 seconds (the default)
timeout 30

# App PID
pid "#{app_root}/tmp/pids/unicorn.pid"

# By default, the Unicorn logger will write to stderr.
# Additionally, some applications/frameworks log to stderr or stdout,
# so prevent them from going to /dev/null when daemonized here:
stderr_path "#{app_root}/log/unicorn.stderr.log"
stdout_path "#{app_root}/log/unicorn.stdout.log"

# To save some memory and improve performance
preload_app true
GC.respond_to?(:copy_on_write_friendly=) and
  GC.copy_on_write_friendly = true

# Force the bundler gemfile environment variable to
# reference the Сapistrano "current" symlink
before_exec do |_|
  ENV["BUNDLE_GEMFILE"] = File.join(app_root, 'Gemfile')
end

before_fork do |server, worker|
  # 参考 http://unicorn.bogomips.org/SIGNALS.html
  # 使用USR2信号，以及在进程完成后用QUIT信号来实现无缝重启
  old_pid = app_root + '/tmp/pids/unicorn.pid.oldbin'
  if File.exists?(old_pid) && server.pid != old_pid
    begin
      Process.kill("QUIT", File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
      # someone else did our job for us
    end
  end

  # the following is highly recomended for Rails + "preload_app true"
  # as there's no need for the master process to hold a connection
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end
{% endhighlight %}
然后执行: RAILS_ENV=production rake assets:precompile手动进行静态文件的编译

启动unicorn:
bundle exec unicorn -E production -c config/unicorn.rb

这样访问xcuiblog.com即可

UPDATE:2015-03-26 22:20

配置到unicorn手动编译静态文件和运行启动命令是可以使项目运行到服务器的，但是每次更新都需要手动操作，未免有点麻烦，所以就出现了Capistrano和mina两种常用的工具，我这里选择了mina，下面发布下部署记录:
Gemfile:
{% highlight ruby %}
group :development do
  gem 'mina'
end
{% endhighlight %}
运行mina init, 生成config/deploy.rb文件
配置deploy.rb:
{% highlight ruby %}
require 'mina/bundler'
require 'mina/rails'
require 'mina/git'
require 'mina/rvm'    # for rvm support. (http://rvm.io)

set :term_mode, nil

set :domain, 'xcuiblog.com'
set :deploy_to, '/home/lytx/cuijunwei/PrivateCodes/xcuiblog'
set :repository, 'git@github.com:xiaocuixt/xcuiblog.git'
set :branch, 'master'

# For system-wide RVM install.
set :rvm_path, '/home/lytx/.rvm/scripts/rvm'

# Manually create these paths in shared/ (eg: shared/config/database.yml) in your server.
# They will be linked in the 'deploy:link_shared_paths' step.
set :shared_paths, ['config/database.yml', 'log', 'tmp']

# This task is the environment that is loaded for most commands, such as
# `mina deploy` or `mina rake`.
task :environment do
  # If you're using rbenv, use this to load the rbenv environment.
  # Be sure to commit your .ruby-version or .rbenv-version to your repository.
  # invoke :'rbenv:load'

  # For those using RVM, use this to load an RVM version@gemset.
  invoke :'rvm:use[ruby 2.1.1]'
end

# Put any custom mkdir's in here for when `mina setup` is ran.
# For Rails apps, we'll make some of the shared paths that are shared between
# all releases.
task :setup => :environment do
  queue! %[mkdir -p "#{deploy_to}/#{shared_path}/log"]
  queue! %[chmod g+rx,u+rwx "#{deploy_to}/#{shared_path}/log"]

  queue! %[mkdir -p "#{deploy_to}/#{shared_path}/config"]
  queue! %[chmod g+rx,u+rwx "#{deploy_to}/#{shared_path}/config"]

  queue! %[touch "#{deploy_to}/#{shared_path}/config/database.yml"]
  queue  %[echo "-----> Be sure to edit '#{deploy_to}/#{shared_path}/config/database.yml'."]
end

desc "Deploys the current version to the server."
task :deploy => :environment do
  deploy do
    # Put things that will set up an empty directory into a fully set-up
    # instance of your project.
    invoke :'git:clone'
    invoke :'bundle:install'
    invoke :'rails:db_migrate'
    invoke :'rails:assets_precompile'
    invoke :'deploy:link_shared_paths'

    to :launch do
      invoke :'unicorn:restart'
    end
  end
end

namespace :unicorn do
  set :unicorn_pid, "#{deploy_to}/tmp/pids/unicorn.pid"
  set :start_unicorn, %{
    cd #{deploy_to} && unicorn -c config/unicorn.rb -E production -D
  }
 
#                                                                    Start task
# ------------------------------------------------------------------------------
  desc "Start unicorn"
  task :start => :environment do
    queue 'echo "-----> Start Unicorn"'
    queue! start_unicorn
  end
 
#                                                                     Stop task
# ------------------------------------------------------------------------------
  desc "Stop unicorn"
  task :stop do
    queue 'echo "-----> Stop Unicorn"'
    queue! %{
      test -s "#{unicorn_pid}" && kill -QUIT `cat "#{unicorn_pid}"` && rm -rf "#{unicorn_pid}" && echo "Stop Ok" && exit 0
      echo >&2 "Not running"
    }
  end
 
#                                                                  Restart task
# ------------------------------------------------------------------------------
  desc "Restart unicorn using 'upgrade'"
  task :restart => :environment do
    invoke 'unicorn:stop'
    invoke 'unicorn:start'
  end
end
{% endhighlight %}
然后运行 mina setup 设置密码等配置.

最后运行mina deploy 项目会自动进行部署.

---
layout: post
title: "網頁效能調校"
description: "工具及用法筆記"
categories: [rails]
tags: [performance]
redirect_from:
  - /2016/12/08/
---

# traceroute
可列出沒用到的routes和actions
~~~
gem ’traceroute’
rake traceroute
~~~

# bullet
Check n+1 query and not used eager load
~~~
gem ’bullet’
在config/environments/development.rb中設定bullet。
config.after_initialize do
  Bullet.enable = true
  Bullet.alert = true
  Bullet.bullet_logger = true
  Bullet.rails_logger = true
end
~~~

# brakeman
Security check
~~~
gem ’brakeman’, require: false
brakeman -o result.html
~~~

# rack_mini_profiler
檢測每一頁所花的時間
可看出在sql上花多少時間，在sql以外花多少時間
~~~
gem ’rack-mini-profiler’, require: false
~~~
在gemfile中，必須放在資料庫的gem之後，否則sql不會被包含進來

可產生flamegraphs：
~~~
gem ’flamegraph’
visit a page in your app with ?pp=flamegraph
~~~

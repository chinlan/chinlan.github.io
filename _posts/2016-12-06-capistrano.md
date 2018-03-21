---
layout: post
title: "capistrano"
description: ""
categories: [rails]
tags: [deploy]
redirect_from:
  - /2016/12/06/
---
# env / stage
是以deploy資料夾下面的檔案作為指令的依據：
- 使用deploy/staging.rb中的設定進行deploy
~~~
cap staging deploy
~~~

environments資料夾下面的檔案設定的是環境，會放在deploy.rb裡面做設定：
~~~
set :rails_env, 'production'
~~~
這樣就是使用environments/production.rb的設定進行deploy

以上兩個設定加起來就是使用production.rb的環境（env）進行staging階段（stage）的deploy

# role filter
deploy.rb
~~~~~~~ ruby
task :admin_linked_files do
  on roles(:backend) do
    set :linked_files, fetch(:linked_files, []).push(’config/database.yml’, ’config/secrets.yml’, ’config/sunspot.yml’)
  end
end
before "deploy:check:linked_files", "admin_linked_files"

task :be_linked_files do
  on roles(:frontend) do
    set :linked_files, fetch(:linked_files, []).push(’config/database.yml’, ’config/secrets.yml’)
  end
end
~~~~~~~~
參考官方文件，看自訂任務要放在哪個動作之前或之後

http://capistranorb.com/documentation/getting-started/flow/#toc_2

# assets:pipeline precompile
precompile的時候會做ugilify，將所有的空白都消去，變數名也會改成簡短的，將整段程式碼做壓縮

產生finger-point亂數（client browser偵測到這個亂數，也就是檔名的後綴，有改過，才會去抓新的，產生新的cache，如果沒有改，就會使用以前的cache，這樣會比較快）

rails會做記錄，assets檔案沒改的話，就不做precompile，這些紀錄的依據或說機制，存放在/tmp/cache資料夾，在production環境通常會將這個資料夾放在gitignore，所以repo是不會有這些檔案的，因此production機器第一次的precompile要自己手動跑，cap deploy並不會有效果，因為剛開始沒有那些作為機制/紀錄的檔案，第一次跑過後，以後的cap deploy就會自動做precompile了

遠端server機器上跑：
~~~
RAILS_ENV=production bundle exec rake assets:precompile
~~~

根據官方文件，似乎也可以讓capistrano在初次佈署時就涵蓋precompile，

在capfile加上
~~~ ruby
load ’deploy/assets’  
~~~       
edit: capistrano-3起不需要
ref: http://guides.rubyonrails.org/asset_pipeline.html

# 專案有更新時需要restart
如果有用gem: capistrano-passenger的話，可直接在capfile裡
~~~ ruby
require ’capistrano/passenger’
~~~
這樣就會自動在publishing之後deploy:restart，

edit: 要加 `after 'deploy:publishing', 'deploy:restart'`

或者在current下面執行`touch tmp/restart.txt`

如果tmp下面沒有這個檔案要自己加，內容是空白的

# 在複數個地方都有cache檔案
(current/shared)/tmp/cache/assets下的資料是用來記錄要不要重新產生新的finger-print。

但真正產生完的assets是放在 shared/public/assets 下面。

# 需要加上load 'deploy/assets'在capfile嗎？
這是capistrano 2的語法，升級到capistrano 3的話，只需要放

`require  ’capistrano/rails’ `就行了

# 需要加上config.action_dispatch.x_sendfile_header = 'X-Accel-Redirect’ 設定嗎？

只有需要傳送非public的（需要權限管理的）大型靜態文件時才需要

[ref](http://thedataasylum.com/articles/how-rails-nginx-x-accel-redirect-work-together.html)

[ref](http://copytime.org/posts/194)

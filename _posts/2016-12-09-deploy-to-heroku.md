---
layout: post
title: "Deploy the project to heroku"
description: "Deploy the project to heroku (free)"
categories: [rails]
tags: [deploy, heroku]
redirect_from:
  - /2016/12/09/
---
- 因為heroku 使用的是pg，所以production環境要安裝pg
Gemfile:
~~~~~~~
group :production do
  gem ’pg’
end
~~~~~~~

- 新增一個remote
~~~~~~~
heroku create
~~~~~~~

- 或者指定連到已存在的remote(通常是參加到既存的專案時)
~~~~~~~
heroku git:remote -a falling-wind-1624
~~~~~~~
這個remote預設的名字就叫做heroku

- 可以改名(把heroku改名為production)
~~~~~~~
git rename heroku production
~~~~~~~

- 刪除remote heroku
~~~~~~~
git remote rm heroku
~~~~~~~

- 看看有哪些remote repo
~~~~~
git remote -v
~~~~~

- 連到該連的repo (remote name can be ‘staging’ or ‘production’)
~~~~~
remotname git:remote -a reponame
~~~~~

- 把要push的分支push到該repo
~~~~~
git push remotename branchname
~~~~~

- 有需要的話跑資料庫遷移
~~~~~
heroku run rake db:migrate
~~~~~

- 跑完遷移重啟
~~~~~
heroku restart --app app-name
~~~~~

- 跑資料庫遷移可指定哪個application
~~~~~
heroku run rake db:migrate --app app-name
~~~~~

- 如果要部署的不是master這個分支，要加上:master
~~~~~
git push app-name develop:master
~~~~~

- 檢視app的線上log
~~~~~
heroku logs -t --app app-name
~~~~~

- 進入線上的環境
~~~~~
heroku run bash -a app-name
~~~~~

- 出錯的時候可以進console看看為什麼倒站
~~~~~~~
heroku run rails console --app app-name
~~~~~~~

- 或者重啟
~~~~~~
heroku restart
~~~~~~

- 和key相關的隱密資料可以用環境變數處理
- 用figaro gem 設定環境變數
~~~~~~~
figaro heroku:set -e production
~~~~~~~

- 也可以另建分支deploy，在這個分支上用`git add -f filename`來強行加入被ignore的檔案，

- 然後用這個分支推到heroku
~~~~~~~
git push heroku deploy:master
~~~~~~~
- 然後記得每次master上開發完後要切換到deploy來merge master
- master push 到github

- deploy push到heroku

- 第一次要堆到heroku時，precompile失敗的話，在本地先跑`rake assets:precompile`
然後將更新push到repository
接下來就可以成功推到heroku了



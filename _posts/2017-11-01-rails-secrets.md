---
layout: post
title: "Rails 5: Secrets"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/11/01/
---

## Use of secret_key_base
用來產生encrypted session_cookie

修改secret_key_base會使已經存放在使用者瀏覽器上的cookie session和signed cookie失效

可用來強制讓使用者重新登入

[ref](https://qiita.com/kikunantoka/items/dd22cf51071b93ba5b62)

以往都是使用放在secrets.yml裡面然後gitignore掉，

還有些專案會使用環境變數，就可以不gitignore，

Rails 5.1開始追加了encrypted secrets功能，環境變數可以編碼後用git管理

[ref](https://www.engineyard.com/blog/encrypted-rails-secrets-on-rails-5.1)
[ref](https://github.com/rails/rails/issues/31109)

## vim在docker環境下不知為何無法正常編輯secrets，
在docker外使用下面指令進行編輯

編輯：
~~~
EDITOR=vim bin/rails secrets:edit
~~~

唯讀：
~~~
bin/rails runner "puts Rails::Secrets.read"
~~~

## 當secrets.enc.yml 在git merge時出現conflict的解決法：

有衝突時，先用上面唯讀那行把master的版本read出來
然後切到head把head的檔案更新，加上master的內容

merge的時候出現衝突就`git checkout head_branch config/secrets.yml.enc`

表示使用新編輯的head_branch這邊的內容
然後`git add .` `git commit` 就完成了


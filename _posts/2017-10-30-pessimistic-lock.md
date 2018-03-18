---
layout: post
title: "Rails: Pessimistic Lock"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/10/30/
---

防止兩個人同時更新一件record造成混亂，
資料庫內for_update的功能（暫時鎖住record，只有目前程序可以更新record）。

~~~ ruby
record.lock!

Record.lock.where(name: xxx)
~~~

[ref](http://api.rubyonrails.org/classes/ActiveRecord/Locking/Pessimistic.html)

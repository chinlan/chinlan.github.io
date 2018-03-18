---
layout: post
title: "Rails: callback"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/12/
---

# Get the backtrace
在找callback是被哪邊呼叫的時候很好用，binding.pry之後:

~~~ ruby
Thread.current.backtrace.join("\n")
~~~

# 注意after_create是比after_save先執行

# before_save: 在create和update的時候觸發

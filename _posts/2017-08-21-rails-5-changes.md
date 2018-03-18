---
layout: post
title: "Rails 5: changes"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/21/
---

整理在工作專案上碰到的更新（和Rails 4不同的地方）

# belongs_to :resource, optional: true
在Rails 5，resource如果不是一定會有的關聯，要加上optional: true

# params裡的空陣列
rails 5裡面不會直接把params裡的空陣列轉換成nil，不過在test的時候空陣列的項目會直接被拿掉，詳看下面issue
[ref](https://github.com/rails/rails/issues/27215)
[ref](http://blog.bigbinary.com/2016/07/06/rails-5-does-not-convert-blank-array-to-nil-in-deep-munging.html)

# redirect_back
rails 5 的redirect_to :back 改成 redirect_back fallback_location: root_path

fallback_locaation可以指定如果沒能成功redirect_back 時要去的頁面

# form_with
form_for和form_tag 將會逐漸淘汰
[ref](https://m.patrikonrails.com/rails-5-1s-form-with-vs-old-form-helpers-3a5f72a8c78a)

# bulk update
rails 5 bulk update with update, with validation and callback
[ref](https://blog.bigbinary.com/2016/06/10/rails-5-allows-updating-relation-objects-along-with-callbacks-and-validations.html)

# attribute_changed? deprecated
before_save:
attribute_changed? => will_save_change_to_attribute?
attribute_was => attribute_in_database

after_save:
attribute_changed? => saved_change_to_attribute?
attribute_was => attribute_before_last_save

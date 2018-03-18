---
layout: post
title: "Rails: 設定自定義model複數名稱"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/18/
---

config/initializers/inflections.rb
~~~ ruby
ActiveSupport::Inflector.inflections do |inflect|

  inflect.irregular 'have', 'haves'

end
~~~

f

---
layout: post
title: "Rails: DateTime to cron format"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/12/07/
---
A convenient way to set a custom datetime format to use in the application, for example, cron format.

/initializers/time_formats.rb

~~~ruby
Time::DATE_FORMATS[:cron] = '%Y%m%d%H%M'

Time.current.to_s(:cron)
~~~

[ref](https://qiita.com/Vit-Symty/items/399c77d1fd681b77d593)

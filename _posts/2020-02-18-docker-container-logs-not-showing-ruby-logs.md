---
layout: post
title: "Docker container logs sometimes do not show ruby logs"
description: ""
categories: [docker rails]
tags: []
redirect_from:
  - /2020/02/18/
---

因為docker container的log有時候不會顯示ruby的log，

[Reference](https://blog.eq8.eu/til/ruby-logs-and-puts-not-shown-in-docker-container-logs.html)

參考上面連結之後，在`config/environments/development.rb`加上了
~~~ruby
  config.logger = Logger.new('/proc/1/fd/1')
~~~

就一直都會顯示了，
只是有時候好像會有同樣的log連續顯示兩次的情況，
需要再觀察。

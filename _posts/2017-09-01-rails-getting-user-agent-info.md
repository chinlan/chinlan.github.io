---
layout: post
title: "Rails: getting user agent info"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/09/01/
---

[ref](http://qiita.com/reeenapi/items/5c088561363cd8b00332)
[rack-user_agent](https://github.com/k0kubun/rack-user_agent)
[rack-smartphone_detector](https://github.com/ihara2525/rack-smartphone_detector)

~~~ ruby
request.user_agent

request.headers["HTTP_USER_AGENT"]
~~~

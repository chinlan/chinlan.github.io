---
layout: post
title: "Ruby: heredoc"
description: ""
categories: [ruby]
tags: []
redirect_from:
  - /2018/12/05/
---
To write multiple line string in heredoc:
~~~Ruby

<<-URL.gsub(/^[\s\t]*|[\s\t]*\n/, '')
  www.example.com/#{example.foo}/
  #{example.bar}
URL

~~~

will output ->

`"www.example.com/foo/bar"`

if without the gsub part, will output ->

`" www.example.com/foo/bar\n"`


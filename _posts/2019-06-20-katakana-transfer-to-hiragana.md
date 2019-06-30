---
layout: post
title: "Ruby: Transfer Katakana to Hiragana"
description: ""
categories: [ruby]
tags: []
redirect_from:
  - /2019/06/20/
---

[reference](https://qiita.com/y_minowa/items/c204992e4665a8687d4a)

Use `NKF` module.

~~~ruby
require 'nkf'
NKF.nkf('-w --hiragana', カタカナ)
~~~

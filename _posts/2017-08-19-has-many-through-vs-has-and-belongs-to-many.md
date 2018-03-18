---
layout: post
title: "Rails -- has_many :through vs has_and_belongs_to_many"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/19/
---

當不需要針對關聯model做些什麼的時候，用has_and_belongs_to_many就可以了，只是之後要記得幫資料庫加上中間table。
但是當需要對關聯model使用validations, callbacks等時必須使用has_many :through

[ref](http://blog.flatironschool.com/why-you-dont-need-has-and-belongs-to-many/)

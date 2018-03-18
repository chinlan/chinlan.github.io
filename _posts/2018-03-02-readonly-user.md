---
layout: post
title: "DB: Read-only user"
description: ""
categories: [devops]
tags: []
redirect_from:
  - /2018/03/02/
---

[ref](https://www.ruby-forum.com/topic/213137)
Rails專案，
內部/beta測試時擔心production db被動到的話，
在database.yml裡先設定個readonly_user，
進入production db建立這個readonly_user。

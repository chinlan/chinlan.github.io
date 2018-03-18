---
layout: post
title: "docker-compose"
description: ""
categories: [docker]
tags: [container]
redirect_from:
  - /2017/08/09/
---

使用docker-compose的作用在於彙整複數的container，一旦使用了docker-compose，就會像是開了一個私有獨立空間出來，要連到電腦上其他container會需要建立bridge network，如果沒有使用docker-compose直接建立一個一個containers，就會像是在公用空間

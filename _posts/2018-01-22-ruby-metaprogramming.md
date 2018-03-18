---
layout: post
title: "Ruby: Metaprogramming"
description: ""
categories: [ruby]
tags: []
redirect_from:
  - /2018/01/22/
---

- Instance屬性存在instance之中，instance方法存在class之中
- 可以通过send方法调用任何方法，即使是私有方法; 但是respond_to?（私有方法）會回傳false
- 當作用於class時，class_eval會定義instance方法，而instance_eval定義class方法


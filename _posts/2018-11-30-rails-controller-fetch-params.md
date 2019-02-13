---
layout: post
title: "Rails: fetch controller params"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/11/30/
---
When need to make sure some params (not related to any model, just flags for controller action condition judgement) exist, using `params.fetch(:foo)` rather than `params[:foo]`, this way if this param is not being passed, will cause `KeyError:Key not found: :foo`

[ref](https://andycroll.com/ruby/use-hash-fetch-when-using-params-in-controllers/)

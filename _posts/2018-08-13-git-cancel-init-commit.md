---
layout: post
title: "Git: Cancel init commit"
description: ""
categories: [git]
tags: []
redirect_from:
  - /2018/08/13/
---

To revert init commit,
`git reset --soft HEAD~`
would fail, use below:
`git update-ref -d HEAD`

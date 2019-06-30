---
layout: post
title: "Git: ignore files which are already in the repo"
description: ""
categories: [git]
tags: []
redirect_from:
  - /2019/02/26/
---


By these 3 steps, it can be done.

`git rm -r --cached .`

`git add .`

`git commit -m "Clean up ignored files"`

[ref](https://www.git-tower.com/learn/git/faq/ignore-tracked-files-in-git
)

---
layout: post
title: "Git: Resolve git cherry-pick conflict"
description: ""
categories: [git]
tags: []
redirect_from:
  - /2018/07/12/
---

從feature-01分支cherry pick一個commit到master

`git co master`

`git cherry-pick <commit-sha>`

如果有conflict

`git st`

查看conflict的狀況

有衝突的檔案git會把它放在untracked (red)的部分

修改完這些衝突後`git cherry-pick --continue`

如果要取消cherry-pick

`git cherry-pick --abort`

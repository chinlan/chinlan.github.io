---
layout: post
title: "Git: 常用"
description: ""
categories: [git]
tags: []
redirect_from:
  - /2017/11/10/
---

[ref](https://zlargon.gitbooks.io/git-tutorial/content/file/recover.html)
# 還原檔案

使用 `git checkout -- <file>` 來還原 "指定的檔案的內容"

使用 `git reset HEAD <file>` 來還原 "指定的檔案的狀態"

`git reset --soft HEAD~` 取消上一個commit

使用 git reset --hard HEAD 來清空 Changes not staged for commit 和 Changes to be committed 區塊

# Squash commits
~~~
git log => list commits

git reset --soft HEAD~  => cancel last commit

git commit -m "msg" => combine new revise and last commit cotent into one commit

git push --force => revise remote origin
~~~

~~~
git reset --soft HEAD~3  => cancel last 3 commit
~~~

# Make empty commit
~~~
git commit --allow-empty -m "first commit"
~~~

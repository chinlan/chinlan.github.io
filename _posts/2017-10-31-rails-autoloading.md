---
layout: post
title: "Rails: Autoloading"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/10/31/
---

### A copy of xxx has been removed from the module tree but is still active

當把reloadable的module include到not reloadable的東西（例：ActionRecord::Base / ActionMailer::Base)中時，會出現的錯誤訊息

解決辦法:

1.改成include到可以reloadable的class，像是models, controllers, 或是建立一個abstract base class，再include到這個class

2.讓這個要include的module變得不能reload，也就是改放到不能auto-loading的路徑，這樣會需要另外require它

3.在module前加上::，也就是::Module，讓他變成頂層

[ref](https://stackoverflow.com/questions/29636334/a-copy-of-xxx-has-been-removed-from-the-module-tree-but-is-still-active
)
[ref](http://neethack.com/2015/04/rails-circular-dependency/)

---
layout: post
title: "CSS- page overflow"
description: ""
categories: [css]
tags: []
redirect_from:
  - /2016/12/02/
---

讓頁面滿版
~~~ css
body{
  overflow: hidden;
}
header{
  height: 101%;
  position: absolute;
  top:-5px;
  left:0;
}
div.wrapper{
  padding-top:0;
}
~~~

---
layout: post
title: "Linux: screen"
description: ""
categories: [linux]
tags: []
redirect_from:
  - /2019/06/14/
---

有時候會需要在遠端主機上執行一些運行時間比較久的task，一直維持ssh連在那邊監看不太實際，這時就可以使用sceen指令

`screen` 進入screen模式
運行長時間task
`ctrl + a ctrl + d` 切換回原本模式（screen仍在背景執行長時間task）
`screen -r` 切換到screen模式檢視task運行狀況

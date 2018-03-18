---
layout: post
title: "Rails 5: adding webpack"
description: ""
categories: [rails]
tags: [webpack]
redirect_from:
  - /2017/06/05/
---

從來沒有使用過webpack做開發的話，

第一次使用時需要先`brew install yarn`（注意：如果沒有安裝yarn，雖然在安裝webpacker時不會有錯誤訊息，但是後面會沒辦法跑`rake webpacker:install`，一定要先安裝yarn)

`brew link yarn`

然後才可以在rails專案裡
`rails new myapp --webpack`    
或是
`rails new myapp` 然後在gemfile加`gem 'webpacker'`

`rake webpacker:install`  (產生需要的檔案和資料夾）

跑webpack的command時，記得前頭要加`bin/`

跑yarn的指令加新的package時，記得前頭要加`bin/`

沒有加的話，不會被當作webpack/yarn的指令，會沒辦法成功install在webpack要安裝的資料夾裡

ref:

http://qiita.com/yoshiokaCB/items/564ed0440f0428c0009a

http://pixelatedworks.com/articles/embracing-change-rails51-adopts-yarn-webpack-and-the-js-ecosystem/


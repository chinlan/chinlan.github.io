---
layout: post
title: "Rails: 建立api文件的gem比較"
description: ""
categories: [rails]
tags: [api]
redirect_from:
  - /2017/08/08/
---
分別試用了三種方法：

### 1. Grape + Swagger

需要的gem:

~~~ ruby
gem 'grape'

gem 'grape-swagger'

gem 'grape-swagger-rails'
~~~

grape支援版本化api，搭配swagger可以產生出網頁介面api文件，而且還可以直接測試api是否產生正確結果

[Github Source](https://github.com/chinlan/grape-demo)

### 2. Swagger

需要的gem:

~~~ ruby
gem 'swagger-docs'
~~~

這邊使用了rails 5 的API mode來實作，

沒有使用grape也還是可以使用swagger來產生json文件檔案

如果再搭配swagger-ui就可以在網頁介面瀏覽文件了

[Github Source](https://github.com/chinlan/swagger-demo)

### 3. Autodoc

需要的gem:

~~~ ruby
gem 'autodoc'

gem 'rspec-rails', '~> 3.5'
~~~

使用autodoc的話，需要搭配寫rspec測試，

跑完測試後markdown格式的api文件也自動產生出來了

本來就使用rspec寫測試的專案使用這個gem應該也算方便

[Github Source](https://github.com/chinlan/try-autodoc)

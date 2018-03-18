---
layout: post
title: "Rails 5: using elasticsearch"
description: ""
categories: [rails, elasticsearch]
tags: []
redirect_from:
  - /2017/12/30/
---
[ref](https://www.sitepoint.com/full-text-search-rails-elasticsearch/)
~~~ ruby
gem 'elasticsearch-model'
gem 'elasticsearch-rails'
~~~

When import not working at first time: using `force: true`.
[ref](https://rubyplus.com/articles/3961-Full-Text-Search-using-ElasticSearch-in-Rails-5)

---------------------
Elasticsearch 6 is needed to set content-type on your own.
[ref](https://github.com/mobz/elasticsearch-head/issues/361)

----------------------
# Eager loading associations with kaminari

[ref](https://qiita.com/hanzochang/items/99557cabc418c5b6aba3)
[ref](https://github.com/elastic/elasticsearch-rails/pull/472)

Using `records(includes: [:association])`

~~~ ruby
Post.search(query).page(params[:page]).per(params[:per]).records(includes: [:product_images])
~~~

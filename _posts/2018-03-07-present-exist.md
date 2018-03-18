---
layout: post
title: "Rails: present? vs. exist?"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/07/
---

[ref](https://stackoverflow.com/questions/13186722/what-is-the-difference-between-using-exists-and-present-in-ruby)

[presence](https://apidock.com/rails/Object/presence)

~~~ ruby
class Product
  after_initialize -> { puts 'Be initialized.' }
end

Product.where(name: 'Jelly').present?
~~~

~~~
Product Load (8.1ms) SELECT "products".* FROM "products" WHERE "products"."name" = $1 ORDER BY products.id ASC  [["name", 'Jelly']]
Be initialized.
Be initialized.
~~~

~~~ ruby
Product.exists?(name: 'Jelly')
~~~

~~~
Product Load (8.1ms) SELECT "products".* FROM "products" WHERE "products"."name" = $1 ORDER BY products.id ASC  [["name", 'Jelly']]
~~~


---
layout: post
title: "Rails: 用custom_name設定關聯"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/17/
---
# has_many :through要用source指定原本的model名稱
~~~ ruby
has_many :members, -> { where(enabled: "enabled" }, through: :plans_users, source: :user
~~~

# 指定scope的話，要放在第二個argument，如上

# has_many要用class_name指定原本的model名稱
~~~ ruby
has_many :allow_users, class_name: 'User'
~~~

# self referential
[ref](https://stackoverflow.com/questions/11100091/rails-has-many-self-referential)

~~~ ruby
class Item

    belongs_to :parent, -> { where(deleted: false) }, class_name: 'Item, 'foreign_key: 'parent_id'

    has_many :children, -> { where(deleted: false) }, class_name: 'Item', foreign_key: 'parent_id'

end
~~~

# Setting foreign key which doesn't point to the default primar key (id)

[ref](https://stackoverflow.com/questions/40281067/foreign-key-that-doesnt-point-to-a-primary-key-in-rails-and-postgres)

~~~ ruby
belongs_to :amazon_item, foreign_key: 'amazon_code', primary_key: 'code'
~~~

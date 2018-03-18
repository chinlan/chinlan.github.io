---
layout: post
title: "factory_girl_rails: 建立關聯資料"
description: ""
categories: [rails]
tags: [test]
redirect_from:
  - /2017/08/11/
---

- 設定belongs_to關聯
~~~ ruby
factory :post do
  # ...
  association :author, factory: :user, last_name: "Twain"
end
~~~

- 還可以像這樣設定只能build
~~~ ruby
factory :post do
  # ...
  association :author, factory: :user, strategy: :build
end
~~~

- has_many的設定：
~~~ ruby
# user factory without associated posts
  factory :user do
    name "Ernest Hemingway"

    # user_with_posts will create post data after the user has been created
    factory :user_with_posts do
      # posts_count is declared as a transient attribute and available in
      # attributes on the factory, as well as the callback via the evaluator
      transient do
        posts_count 5
      end

      # the after(:create) yields two values; the user instance itself and the
      # evaluator, which stores all values from the factory, including transient
      # attributes; `create_list`'s second argument is the number of records
      # to create and we make sure the user is associated properly to the post
      after(:create) do |user, evaluator|
        create_list(:post, evaluator.posts_count, user: user)
      end
    end
  end
~~~
這樣的話，就可以如下使用：
~~~ ruby
create(:user).posts.length # 0
create(:user_with_posts).posts.length # 5
create(:user_with_posts, posts_count: 15).posts.length # 15
~~~
[ref](http://www.rubydoc.info/gems/factory_girl/file/GETTING_STARTED.md#Associations)




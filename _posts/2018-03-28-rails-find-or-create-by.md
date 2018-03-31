---
layout: post
title: "Rails: find_or_create_by"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/28/
---

[ref](https://qiita.com/kyrieleison/items/59d293594fbe3a139129)
這個method不會保證atomic operation!

當兩個requests同時到達時，用這個method可能會產生兩筆重複的資料，如果在model有設定validate uniqueness的話，
就會引發`ActiveRecord::RecordNotUniqueerror`.

解決方法:

1. 對parent使用lock

~~~ ruby
Like.transaction do
  article.lock! 
  article.likes.find_or_create_by(article_id: article.id)
end
~~~

2. 使用rescue

~~~ ruby
begin
  article.likes.find_or_create_by(article_id: article.id)
rescue ActiveRecord::RecordNotUnique
  retry
end
~~~

第二種方法在文件中有介紹：
[ref](https://apidock.com/rails/v4.0.2/ActiveRecord/Relation/find_or_create_by)

--------------------------------------
### query to database:
[ref](http://macotasu.hatenablog.jp/entry/2016/01/09/205003)

[ref](https://qiita.com/kyrieleison/items/59d293594fbe3a139129)

---------------------------------------
使用情境：

When user likes a post, post's like_count will increment one.
（當user對文章按讚，文章的按讚數會加一。）

~~~ ruby
post.likes.find_or_create_by(user_id: current_user.id)
post.increment!(:like_count)
~~~

First, to put these two actions in one transaction:
（首先要把這兩個動作放到同一個transaction：）

~~~ ruby
post.likes.find_or_create_by(user_id: current_user.id) do |_like|
  post.increment!(:like_count)
end
~~~

or
~~~ ruby
Like.transaction do
  post.likes.find_or_create_by!(user_id: current_user.id)
  post.increment!(:like_count)
end
~~~

Because find_or_create_by is not atomic method, duplication can be caused under some consideration, to prevent:
（防止重複建立likes:）

From document (preferable):
（文件上提到的方法：）
~~~ ruby
post.likes.find_or_create_by(user_id: current_user.id) do |_like|
  post.increment!(:like_count)
end
rescue ActiveRecord::RecordNotUnique
  retry
end
~~~

or lock the parent:
（或是對parent使用lock：）
~~~ ruby
post.lock!
post.likes.find_or_create_by(user_id: current_user.id) do |_like|
  post.increment!(:like_count)
end
~~~

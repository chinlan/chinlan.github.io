---
layout: post
title: "Rails: includes, joins, eager_load"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/25/
---
rails version: 5.1.4

ruby version: 2.4.2

## includes：

- eager loading(one query) or preloading(two queries)
~~~ ruby
User.includes(:posts)
~~~
~~~
User Load (5.9ms) SELECT "users".* FROM "users"
Post Load (2.2ms) SELECT "posts".* FROM "posts" WHERE "posts"."user_id" IN (3, 6, 2, 7, 16, 1, 4, 17, 23, 25, 24, 18, 27, 28, 26, 61, 22, 62, 19, 20, 21)
~~~

## joins:

- lazy loading

~~~ ruby
User.joins(:posts)
~~~

~~~
User Load (2.6ms) SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id"
~~~

## eager_load:
~~~ ruby
User.eager_load(:posts)
~~~
equals to
~~~ ruby
User.includes(:posts).references(:posts)
~~~
~~~
SQL (3.8ms) SELECT  "users"."id" AS t0_r0,
                    "users"."created_at" AS t0_r1,
                    "users"."updated_at" AS t0_r2,
                    "users"."email" AS t0_r3,
                    "users"."nickname" AS t0_r4,
                    "posts"."id" AS t1_r0,
                    "posts"."user_id" AS t1_r1,
                    "posts"."public" AS t1_r2,
                    "posts"."title" AS t1_r3,
                    "posts"."content" AS t1_r4,
                    "posts"."created_at" AS t1_r5,
                    "posts"."updated_at" AS t1_r6,
            FROM "users" LEFT OUTER JOIN "posts" ON  "posts"."user_id" = "users"."id"
~~~~~~

# 'where' query using association column(用關聯table的欄位來做where query)

## joins:
~~~ ruby
User.joins(:posts).where(posts: {public: true})
~~~
~~~
User Load (2.8ms) SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id" WHERE "posts"."public" = $1 [["public", "t"]]
~~~
~~~ ruby
User.joins(:posts).where('posts.created_at > ?', 3.month.ago)
~~~
~~~
User Load (2.1ms) SELECT "users".* FROM "users" INNER JOIN "posts" ON "posts"."user_id" = "users"."id" WHERE (posts.created_at > '2017-12-30 12:28:10.875053')
~~~
## includes:
~~~ ruby
User.includes(:posts).where(posts: {public: true})
~~~
~~~
SQL (3.8ms) SELECT  "users"."id" AS t0_r0,
                    "users"."created_at" AS t0_r1,
                    "users"."updated_at" AS t0_r2,
                    "users"."email" AS t0_r3,
                    "users"."nickname" AS t0_r4,
                    "posts"."id" AS t1_r0,
                    "posts"."user_id" AS t1_r1,
                    "posts"."public" AS t1_r2,
                    "posts"."title" AS t1_r3,
                    "posts"."content" AS t1_r4,
                    "posts"."created_at" AS t1_r5,
                    "posts"."updated_at" AS t1_r6,
            FROM "users" LEFT OUTER JOIN "posts" ON "posts"."user_id" = "users"."id"
            WHERE "posts"."deleted" = $1 [["public", "t"]]
~~~

自動採取eager_load了
~~~ ruby
User.includes(:posts).where('posts.created_at > ?', 3.month.ago)
~~~
~~~
#<User::ActiveRecord_Relation:0x2aac49a3382c>
~~~
加上.count或是.first之類的就會出下面錯誤
~~~
ActiveRecord::StatementInvalid: PG::UndefinedTable: ERROR: missing FROM-clause entry for table "posts"
~~~
當使用第二種寫法時，無法自動偵測到要eager_load，所以要自己寫成：
~~~ ruby
User.includes(:posts).where('posts.created_at > ?', 3.month.ago).references(:posts)
~~~

這樣效果就等同於
~~~ ruby
User.eager_load(:posts).where('posts.created_at > ?', 3.month.ago)
~~~

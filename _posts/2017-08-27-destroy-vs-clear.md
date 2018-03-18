---
layout: post
title: "Rails: destroy vs. clear"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/27/
---

[ref](https://stackoverflow.com/questions/15752516/rails-how-does-clear-differ-from-destroy)

clear不會跑callback

model:
~~~ ruby
class Plan < ActiveRecord::Base
  has_many :tags, through: :tags_plans
end
~~~

~~~
p.tags.clear

SQL (0.4ms)  DELETE FROM "tags_plans" WHERE "tags_plans"."plan_id" = $1 AND 1=0  [["plan_id", 18]]
~~~

~~~
p.tags_plans.clear

SQL (2.6ms)  UPDATE "tags_plans" SET "plan_id" = NULL WHERE "tags_plans"."id" IN (SELECT "tags_plans"."id" FROM "tags_plans" WHERE "tags_plans"."plan_id" = $1 ORDER BY id)  [["plan_id", 18]]
~~~


------------------------

# destroy_all vs. delete_all

destroy_all會呼叫call_back, validation等, delete_all不會


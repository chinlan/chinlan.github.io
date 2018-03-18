---
layout: post
title: "Rails: query"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/09/04/
---

### GroupingError: ERROR: column must appear in the GROUP BY clause or be used in an aggregate function

[ref](https://stackoverflow.com/questions/20942477/groupingerror-error-column-must-appear-in-the-group-by-clause-or-be-used-in-an)

### 1 = 0

[ref](https://stackoverflow.com/questions/20942477/groupingerror-error-column-must-appear-in-the-group-by-clause-or-be-used-in-an)
[ref](https://stackoverflow.com/questions/25816792/activerecord-appends-and-1-0-to-end-of-queries)

~~~ ruby
Post.where(author_id: [])
~~~
~~~
SELECT * FROM posts WHERE 1=0;
~~~

### Using raw sql in rails
~~~ ruby
sql = "Select * from ... your sql query here"
records_array = ActiveRecord::Base.connection.execute(sql)
~~~

[ref](https://qiita.com/yut_h1979/items/4cb3d9a3b3fc87ca0435)

~~~ ruby
Author.find_by_sql("
  SELECT * FROM authors
  INNER JOIN posts ON authors.id = posts.author_id
  ORDER BY authors.created_at desc
")
# => [<Author id: 1, name: "Nulla">, <Author id: 2, name: "Marcello">...]
~~~

[ref](ref: https://stackoverflow.com/questions/14824453/rails-raw-sql-example)

----------------------------------

### count(0)=count(1)=count(*) -- 不忽略null值和空值
### count(列名) -- 忽略null值


----------------------------------
### Using Arel to construct complicated query

~~~ ruby
activities = Activity.arel_table
maps = Map.arel_table
a = activities.project(activities[:id], activities[:map_id], activities[:id].count.as('count'))
              .where(activities[:start_at].gt(Time.current - 30.days)
              .and(activities[:public].eq(true))
              .and(activities[:deleted].eq(false)))
              .group(activities[:map_id], activities[:id])
              .order('count desc').take(10).as('a')
join_maps = maps.join(a, Arel::Nodes::InnerJoin).on(maps[:id].eq(a[:id])).join_sources
@maps = Map.joins(join_maps)
~~~

-----------------------------------
### Combine 2 ActiveRecord::Relation objects
[ref](https://stackoverflow.com/questions/9540801/combine-two-activerecordrelation-objects)
AND
~~~ ruby
obj1.merge(obj2)
~~~
OR
~~~ ruby
obj1.or(obj2)
~~~

------------------------------------
### Order with null
[ref](https://stackoverflow.com/questions/5826210/rails-order-with-nulls-last)
If using PostgreSQL:
~~~ ruby
User.order('coupon_id DESC NULLS LAST')
User.order('coupon_id DESC NULLS FIRST')
~~~


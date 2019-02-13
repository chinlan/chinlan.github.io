---
layout: post
title: "Rails: kaminari pagination with joins relations"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/09/20/
---
query before:

~~~ruby
current_user.conversations.joins(:messages).with_relations.order('messages.created_at desc')
~~~

We are doing joins(:messages) to avoid displaying conversation with no messages in conversations list. (To do inner join, if it's includes + references, would be left outer join.)

with_relations contains includes(:messages), it is set in the model file including other associations which will be used in conversation serializer, (we use ActiveModelSerializer for API response.)

~~~ruby
RELATIONS = [:messages, users: User::AVATAR_RELATIONS].freeze
scope :with_relations, -> { includes(RELATIONS) }
~~~

When trying to paginate this query result, its total_count (kaminari method) will contain duplicate records, though query above tried in the console should select distinct records.

If we monkey patch total_count to use `count`, it works fine. Also fine with `length` and `size`, but these options would increase the query numbers, we would have to give up on these considering about performance.

[total_count](https://github.com/kaminari/kaminari/blob/9e9ac3b7516b409a4aed1ce1bc6c5e90108c9511/kaminari-activerecord/lib/kaminari/activerecord/active_record_relation_methods.rb)
[references_eager_loaded_tables?](https://github.com/rails/rails/blob/d681adbbb5e945994580a4e3b5103081888491b9/activerecord/lib/active_record/relation.rb)

After reading code from above links, we decided to remove joins from the query.

query after:

solution 1:

~~~ruby
conversations =
current_user.conversations.with_relations.references(:messages)
.group('images.id, users.id, messages.id, conversations.id')
.having('COUNT (messages.id) > 0')
.order('messages.id desc')
~~~

solution 2:

~~~ruby

conversation_ids = current_user.conversations.joins(:messages).pluck(:id)

conversations = current_user.conversations.with_relations.where(id: conversation_ids).order('messages.id desc')

~~~

[counting or sorting by an association](https://tomkadwill.com/2017/12/18/counting-or-sorting-by-an-association-in-a-rails-sql-query.html)


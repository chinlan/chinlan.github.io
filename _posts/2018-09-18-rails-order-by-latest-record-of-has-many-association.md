---
layout: post
title: "Rails: order by latest record of a has_many association"
description: ""
categories: [rails]
tags: [postgresql]
redirect_from:
  - /2018/09/14/
---
Using case:

Conversations ordered by latest updated messages

[ref](https://stackoverflow.com/questions/35562511/rails-order-a-model-by-the-last-element-of-a-has-many-association)

Better to have a updated_at column on Conversation table, and touch it when message saved.

If not,

~~~ruby
def index
  conversations =
    current_user.conversations.preload(:users)
    .joins(:messages)
    .order('messages.updated_at desc')
    render json: conversations, paginate: true
end
~~~

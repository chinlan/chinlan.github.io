---
layout: post
title: "Rails: controller actions"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/09/03/
---

[ref](http://jeromedalbert.com/how-dhh-organizes-his-rails-controllers/)

According to the article from above link,
it seems better to only use default 7 actions (index, show, new, create, edit, update, destroy) in the controller.

For example, when wanting to add a function which can edit the order of posts, instead of writing a edit_order action in PostsController, new a PostOrderController, Post::OrderController or things like this (depending on the design of the project) and write its own edit action.

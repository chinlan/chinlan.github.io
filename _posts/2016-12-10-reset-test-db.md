---
layout: post
title: "Reset test db"
description: ""
categories: [rails]
tags: [test]
redirect_from:
  - /2016/12/10/
---
# Reset test database:
~~~~~~~~~~
bundle exec rake db:drop db:create db:migrate RAILS_ENV=test
~~~~~~~~~~

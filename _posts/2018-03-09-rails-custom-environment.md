---
layout: post
title: "Rails: Add a custom environment"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/09/
---

[ref](https://stackoverflow.com/questions/2369511/how-to-create-a-new-environment-in-ruby-on-rails)

[ref](https://richonrails.com/articles/creating-a-custom-rails-environment)

- 建立相應的檔案

`config/environments/staging.rb`

- 在以下檔案中增加相應的部分

`config/cable.yml`
`config/database.yml`
`config/secrets.yml` (`secrets.yml.enc`)
`Gemfile` (如果有staging專用的gem)

- 使用這個環境：
~~~
rails server -e staging rails console staging

rails console staging
Rails.env.staging?
~~~


 

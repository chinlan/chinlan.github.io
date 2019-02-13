---
layout: post
title: "Rails: send mail in local environment"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/12/25/
---
Production用`AWS SES`，local開發使用gmail測試：

Setting:

[ref](https://stackoverflow.com/questions/18541909/can-a-rails-application-send-emails-from-the-local-host)


[Register to get password](https://support.google.com/mail/answer/185833?hl=en)

In development.rb
~~~ruby
if %i[user_name password].all? {|method| Settings.gmail.send(method) }
  config.action_mailer.smtp_settings = {
  address: 'smtp.gmail.com',
  port: 587,
  domain: 'yamap.co.jp',
  authentication: :plain,
  user_name: Settings.gmail.user_name,
  password: Settings.gmail.password,
  enable_starttls_auto: true
  }
end
~~~

settings.local.yml
~~~
gmail:
user_name: (xxx@gmail.com)
password: (password)
~~~

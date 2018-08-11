---
layout: post
title: "Devise: authenticate user with AJAX"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/07/06/
---
Add below in /initializers/devise.rb
~~~~~~~~
config.http_authenticatable_on_xhr = false
config.navigational_formats = [:"*/*", "*/*", :html, :js]
~~~~~~~~~

And create sign_in.js in /public/users
~~~~~~~~
alert('Please sign in.');
window.location = '/users/sign_in';
~~~~~~~~~

[ref](https://stackoverflow.com/questions/9901781/how-to-handle-devises-authenticate-user-with-ajax-call)

[ref2](https://blog.andrewray.me/how-to-set-up-devise-ajax-authentication-with-rails-4-0/)

[ref3](https://hackhands.com/sign-users-ajax-using-devise-rails/)

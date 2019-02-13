---
layout: post
title: "Rails: remove html tags"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/12/06/
---
[ref](https://stackoverflow.com/questions/2576394/remove-all-html-tags-from-attributes-in-rails)

~~~ruby
plain_text = strip_tags(html_input)​
~~~

or

~~~ruby
include ActionView::Helpers::SanitizeHelper

def foo
  sanitized_output = sanitize(html_input, tags: [])
end
~~~

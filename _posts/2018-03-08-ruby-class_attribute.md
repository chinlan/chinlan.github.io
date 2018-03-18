---
layout: post
title: "Ruby: class_attribute"
description: ""
categories: [ruby]
tags: []
redirect_from:
  - /2018/03/08/
---

[ref](http://tbpgr.hatenablog.com/entry/20140128/1390914894)
[doc](https://apidock.com/rails/Class/class_attribute)
[doc](http://railsdoc.eiel.info/active_support/core_ext/class/)

class_attribute用來添加class的attribute，可以透過instance_writer, instance_reader, instance_accessor等options來控制instance對這些attributes的讀寫。
可以被繼承。

~~~ ruby
class A
  class_attribute :star, instance_accessor: false
  self.star = 5
end

class B < A
end

class C < A
  self.star = 3
end

B.star  # => 5
C.star  # => 3
A.new.star # => NoMethodError
~~~

---
layout: post
title: "Ruby: refinement"
description: ""
categories: [ruby]
tags: []
redirect_from:
  - /2017/12/27/
---

只有using的scope才會有refinement的效果

~~~ ruby
class MyClass
  def numbers
    123
  end
end

module NumbersRefinement
  refine MyClass
    def numbers
      456
    end
  end
end

c = MyClass.new
c.numbers # => 123

using NumbersRefinement

c.numbers # => 456
~~~

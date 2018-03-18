---
layout: post
title: "Rails: 中文短網址"
description: ""
categories: [rails]
tags: [slug, url]
redirect_from:
  - /2016/11/01/
---

可以使用friendly_id

1. 在migration中加入一個slug的col。

~~~ ruby
class CreateBlogPosts < ActiveRecord::Migration
  def change
    create_table :blog_posts do |t|
      # ...
      t.string :title
      t.string :slug, null: false
      # ...
    end
    add_index :blog_posts, :slug, unique: true
  end
 end
 ~~~

2. 在model中設定friendly_id。friendly_id會在model被建立的時候將一個指定的欄位(以這個例子而言就是title)轉成網址用的slug並存在slug這個欄位中。

~~~ ruby
class BlogPost < ActiveRecord::Base
extend FriendlyId
friendly_id :title, use: :slugged
# ...
end
~~~
3. 在controller中如果要用params[:id]找對應的record，要記得加上friendly。

~~~ ruby
class BlogPostsController < ApplicationController
  before_action :set_blog_post, only: [:show, :edit, :update, :destroy]
  # ...
  private
  def set_blog_post
    @blog_post = BlogPost.friendly.find(params[:id])
  end
end
~~~

目前friendly_id只能支援英文的slug，如果有中文字就會被強迫變成-，這時候就要用babosa了。

4. 在model中overwrite normalize_friendly_id。

~~~ ruby
class BlogPost < ActiveRecord::Base
  extend FriendlyId
  friendly_id :title, use: :slugged
  # ...
  def normalize_friendly_id(input)
    input.to_s.to_slug.normalize.to_s
  end
  # ...
  private
  def gen_ubid
    self.ubid = RandomToken.gen(64)
  end
end
~~~

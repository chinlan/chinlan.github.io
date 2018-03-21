---
layout: post
title: "Polymorphic exercise: followable"
description: ""
categories: [rails]
tags: [polymorphic]
redirect_from:
  - /2016/11/01/
---

~~~ ruby
class Blog < ActiveRecord::Base
  has_many :follows, as: :followable, dependent: :destroy
  include Followable

  def follow_target
    self.name
  end

  def follow_description
    self.summary
  end

  def follow_link
    Rails.application.routes.url_helpers.blog_path(id: self.slug)
  end

  def follow_mail
    render ’course_news’
  end
end
~~~

~~~ ruby
# == Schema Information
#
# Table name: follows
#
#  id               :integer          not null, primary key
#  followable_id    :integer          not null
#  followable_type  :string(255)      not null
#  follower_id      :integer
#  is_noticed       :boolean          default(true)
#  created_at       :date_time        not null

class Follow < ActiveRecord::Base
  belongs_to :followable, polymorphic: true
  belongs_to :follower, class_name: "User"
end
~~~

~~~ ruby
module Followable
  extend ActiveSupport::Concern

  included do
    has_many :follows, :as => :followable
    has_many :valid_follows, ->　{where(:is_noticed => true)}, :as => :followable, :class_name => "Follow"
    has_many :followers, through: :valid_follows
  end

  def follow_target
    raise "You should implemented \"follow_target\" in your Followable-based model"
  end

  def follow_description
    raise "You should implemented \"follow_description\" in your Followable-based model"
  end

  def follow_link
    raise "You should implemented \"follow_link\" in your Followable-based model"
  end

  def follow_mail
    raise "You should implemented \"follow_mail\" in your Followable-based model"
  end
~~~

~~~ ruby
class CreateFollows < ActiveRecord::Migration
  def change
    create_table :follows do |t|
      t.references :followable, polymorphic: true, index: true, null: false
      t.references :follower
      t.boolean :is_noticed, default: true
      t.datetime :created_at, null: false
    end
    add_foreign_key :follows, :users, column: :follower_id
  end
end
~~~

~~~ ruby
class AuthorProfile < ActiveRecord::Base
  has_many :follows, as: :followable, dependent: :destroy
  include Followable

  def follow_target
    self.nickname
  end

  def follow_description
    self.author_description
  end

  def follow_link
    Rails.application.routes.url_helpers.my_author_profile_path(id: self.uid)
  end

  def follow_mail
    render ’author_news’
  end
end
~~~

~~~ ruby
class User < ActiveRecord::Base
  has_many :follows, foreign_key: "follower_id"
end
~~~

# Polymorphic mailer: using service object (interactor)
- mailer interactor
~~~ ruby
def send_follow_news_mail
  NotifyFollowers.new(@blog).call
end
~~~

~~~ ruby
def send_follow_news_mail
  @blog.author_profiles.each do |t|
    NotifyFollowers.new(t).call
  end
end
~~~

~~~ ruby
class NotifyFollowers
  def initialize followable
    @followable = followable
  end

  def call
    followable.followers.each do |follower|
      UserMailer.follow_news_mail(follower: follower, followable: @followable).deliver_later
    end
  end
end
~~~

~~~ ruby
def follow_news_mail(follower:, followable:)
  mail({to: follower.email,
           subject: "Your follow has been updated." ) do |format|
        format.html { render ’follow_news_mail’, :followable => @followable}
    end})
rescue => e
  handle_unknown_error(e)
end
~~~

~~~ html
<h2>Update Notif</h2>
<%= followable.follow_mail %>
#views/user_mailer/_author_news.html.erb
<p>There is some update about author</p>
#views/user_mailer/_blog_news.html.erb
<p>There is some update about blog</p>
~~~

# Mailer: not using service object, using array to deliver to multiple followers at one time

tips: using bcc: not to:
So that receivers will not see others’ email addresses.

~~~ ruby
def send_follow_news_mail
  @blog.author_profiles.each do |t|
    UserMailer.follow_news_mail(followable_id: t.id, followable_type: "AuthorProfile").deliver_later
  end
end
~~~

~~~ ruby
def follow_news_mail(followable_id:, followable_type:)
    return unless Object.const_defined?(followable_type)
    @followable = followable_type.constantize.find(followable_id)
    if @followable.follows
      @follower_email = []
      @followable.follows.each do |f|
        @follower_email << f.follower.email
      end
      mail({bcc: @follower_email,
             subject: "Your follow has been updated." ) do |format|
          format.html { render ’follow_news_mail’, :followable => @followable}
      end})
    end
  rescue => e
    handle_unknown_error(e)
  end
~~~

# Interactor Design Pattern
[Ref](https://github.com/collectiveidea/interactor)


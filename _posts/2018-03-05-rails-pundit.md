---
layout: post
title: "Rails: Pundit"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/05/
---

# Using multiple scopes per models:

[ref](https://github.com/varvet/pundit/issues/234)

原本只有一個scope的時候只需要：
xxx_controller.rb
~~~ ruby
users = policy_scope(User)
~~~

user_policy.rb
~~~ ruby
class Scope < Scope
  def resolve
    if user
      scope.where.not(id: ng_user_ids)
    else
      scope
    end
  end
end
~~~

想多定義一個scope:
user_policy.rb
~~~ ruby
class Scope < Scope
  def resolve
    # the same
  end

  def includes_limited_allowed
    if user
      scope.where(query)
    else
      scope
    end
  end
end
~~~

xxx_controller.rb
~~~ ruby
users = UserPolicy::Scope.new(current_user, User).includes_limited_allowed
~~~

----------------------------------
# 在rails console的時候policy_scope不能用
~~~ ruby
user = User.find 1
users = User.where(enabled: true)
Pundit.policy_scope(user, users)
~~~

-----------------------------------
# Using class or ActiveRecord::Relation
~~~ ruby
policy_scope(User)
~~~

~~~ ruby
@users = User.enabled
policy_scope(@users)
~~~

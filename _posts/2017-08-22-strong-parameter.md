---
layout: post
title: "Rails: strong parameter"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/22/
---

# Insert attributes into params
[ref](https://stackoverflow.com/questions/18369592/modify-ruby-hash-in-place-rails-strong-params)

~~~ ruby
modified_plan_params = plan_params
modified_plan_params["checkpoints_attributes"].each_with_index do |c, index|
  c["position"] = index
  c["pass_at"] = Time.at(c[:pass_at].to_i)
end
@plan = current_user.myplans.create(modified_plan_params)
~~~

[ref](https://stackoverflow.com/questions/16530532/rails-4-insert-attribute-into-params)

~~~ ruby
current_user.posts.create(post_params)

def create
  @post = Post.new(post_params.merge(user_id: current_user.id))
  # save the post, etc
end

private

def post_params
  params.require(:post).permit(:some_attribute)
end
~~~

~~~
private

def post_params
  params.require(:post).permit(:some_attribute).merge(user_id: current_user.id)
end
~~~

# Using custom key for rails params
~~~ ruby
Havecomment.create(comment_params)
~~~

~~~
params: {
    comment: {
        body: 'comment 1'
     }
}
~~~

~~~ ruby
def comment_params
    params.fetch(:comment).permit(:body)
end
~~~

[ref](https://apidock.com/rails/v4.0.2/ActionController/Parameters/fetch)

 
# 或者wrap_parameters
[ActionController::ParamsWrapper](http://edgeapi.rubyonrails.org/classes/ActionController/ParamsWrapper.html)
 
# Strong Parameters
[ref](http://blog.trackets.com/2013/08/17/strong-parameters-by-example.html)

如果要customize的部分實在太多，直接寫個class XxxParameter可能比較清楚好整理

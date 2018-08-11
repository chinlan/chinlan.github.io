---
layout: post
title: "Rails: Pundit"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/08/02/
---

Sometimes the authorization will depend on a parent object,

such as creating a comment to a community:
~~~~~~~
class CommentsController < ApplicationController
   def create
        authorize @community, :commentable?
        @community.comments.create(comment_params)
        # ...........
   end
end

class CommunityPolicy < ApplicationPolicy
   def commentable?
      #.......
    end
end
~~~~~~~~

# Passing parameter from controller to policy:
[ref](https://github.com/varvet/pundit/issues/140)

# Additional context with pundit:
[ref](https://stackoverflow.com/questions/28216678/pundit-policies-with-two-input-parameters)

[doc](https://github.com/varvet/pundit#additional-context)

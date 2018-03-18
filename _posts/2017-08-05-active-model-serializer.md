---
layout: post
title: "Rails: Active Model Serializer"
description: ""
categories: [rails]
tags: [serializer, api]
redirect_from:
  - /2017/08/05/
---
# Serializer RSpec
~~~~~ ruby
require 'rails_helper'

RSpec.describe Plans::RelationsSerializer do
  let(:plan) do
    create(:plan_with_relations)
  end

  subject do
    Plans::RelationsSerializer.new(plan).serializable_hash
  end

  it {
    is_expected.to have_key(:tags)
    is_expected.to have_key(:haves)
    is_expected.to have_key(:members)
    is_expected.to have_key(:checkpoints)
  }
end
~~~~~

# Customize serializer
~~~~~ ruby
class Plans::RelationsSerializer < ActiveModel::Serializer

  has_many :tags
  has_many :haves
  has_many :members, serializer: MemberSerializer
  # A member is an instance of User class
  attributes :checkpoints

  def checkpoints

    ActiveModelSerializers::SerializableResource.new(object.landmarks, each_serializer: CheckpointSerializer, plan_id: object.id)

  end
end
~~~~~

~~~~~ ruby
class Plans::RelationsController < ApplicationController

  def index
    plan = Plan.eager_load(:tags, :members, :haves, landmarks: :landmarks_plans).where(id: params[:plan_id]).first
    render json: plan, serializer: Plans::RelationsSerializer
  end
end
~~~~~

~~~~~ ruby
class CheckpointSerializer < ActiveModel::Serializer
  attributes :landmark, :pass_at
  def landmark
    LandmarkSerializer.new(object.landmark)
  end
  def pass_at
    object.pass_at.to_i
  end
end
~~~~~

# Merge other items to result hash

[ref](https://stackoverflow.com/questions/20947266/active-model-serializers-adding-extra-information-outside-root-in-arrayserializ)

# Using custom key for hash root
~~~ ruby
render json: @users, serializer: MemberSerializer, root: 'members'
~~~

# Access current_user in Active Model Serializer
[ref](https://github.com/rails-api/active_model_serializers/issues/315)
[ref2](https://github.com/rails-api/active_model_serializers/tree/0-9-stable#customizing-scope)

application_controller.rb
~~~ ruby
serialization_scope :current_user
~~~
Default scope is current_user, so above may not be necessary.

post_serializer.rb
~~~ ruby
def is_liked
  scope.like_post?(object) if scope
end
~~~

# Print serializer result in pretty JSON format in Rails console

~~~
puts JSON.pretty_generate(ProfileSerializer.new(p).serializable_hash)
~~~

[ref](https://stackoverflow.com/questions/22001912/prettify-json-output-of-active-model-serializer-in-rails-console)

# Serialize a plain old ruby object(PORO)
Inherit ActiveModelSerializers::Model
~~~ ruby
class MyModel < ActiveModelSerializers::Model
  attr_accessor :name, :description
end
~~~
Under this situation, MyModelSerializer would be the default serializer.

[ref](https://stackoverflow.com/questions/36485337/how-to-use-active-model-serializer-without-an-active-record-model)

# Setting: nested associations
[ref](https://qiita.com/qsona/items/f9d58976c561b8331922)
Can be set in each controller using 'include', or if the nested number of each serializer is the same, it can be set in the initializer.

Default setting: 1 layer
~~~ ruby
ActiveModelSerializers.config.default_includes = '*'
~~~

Add a layer
~~~ ruby
ActiveModelSerializers.config.default_includes = '*.*'
~~~

All layers
~~~ ruby
ActiveModelSerializers.config.default_includes = '**'
~~~


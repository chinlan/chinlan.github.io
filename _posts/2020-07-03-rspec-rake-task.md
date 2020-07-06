---
layout: post
title: "RSpec: rake tasks"
description: ""
categories: [rails]
tags: [rake RSpec]
redirect_from:
  - /2020/07/03/
---

To avoid sporadic failure of rake task spec when executing all specs at a time,
loading tasks before running each rake task spec, and clear them after running each rake task spec.

spec_helper.rb
~~~ ruby
config.before(:each, rake: true) do
  Rails.application.load_tasks
end

config.after(:each, rake: true) do
  Rake::Task.clear
end
~~~

The flag `rake: true` makes sure only rake tasks spec will load and clear the tasks.

spec/lib/tasks/example_task_spec.rb
~~~ ruby
require 'rake'

RSpec.describe "example_task", rake: true do
  ...
  it 'does something' do
    result = Rake::Task['example_task'].invoke
    expect(result).to eq(something)
  end
end
~~~

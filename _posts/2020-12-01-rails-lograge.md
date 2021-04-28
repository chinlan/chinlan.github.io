---
layout: post
title: "Rails: lograge"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2020/12/01/
---
/config/environments/production.rb
~~~ruby
  config.log_level = :info
  config.logger = Logger.new(STDOUT)
  config.lograge.enabled = true
  config.lograge.formatter = Lograge::Formatters::Json.new
  config.lograge.ignore_actions = ['HealthcheckController#index']
  config.lograge.custom_payload do |controller|
    {
      host: controller.request.host,
      remote_ip: controller.request.remote_ip
    }
  end

  config.lograge.custom_options = lambda do |event|
    exceptions = %w(controller action format)
    options = {
      time: event.time,
      host: event.payload[:host],
      remote_ip: event.payload[:remote_ip],
      params: event.payload[:params].except(*exceptions)
    }
    options[:current_user_id] = event.payload[:current_user_id] if event.payload[:current_user_id]
    options[:current_admin_user_id] = event.payload[:current_admin_user_id] if event.payload[:current_admin_user_id]
    options[:exception_object] = event.payload[:exception_object] if event.payload[:exception_object]
    options[:exception] = event.payload[:exception] if event.payload[:exception]
    options[:backtrace] = event.payload[:exception_object].try(:backtrace) if event.payload[:exception_object]
    options
  end
~~~

lograge currently not log exceptions occured during rake tasks execution:
https://github.com/roidrage/lograge/issues/48
https://stackoverflow.com/questions/62501459/rails-how-to-log-all-exceptions-thrown-in-rake-tasks

`ActionController::RoutingError` is also not loged with lograge formatter.
https://github.com/roidrage/lograge/issues/146

Add monkey_patches in bin/rake file:
~~~ruby
module Rake
  class Task
    alias_method :invoke_without_loggable, :invoke

    def invoke(*args)
      begin
        invoke_without_loggable(*args)
      rescue StandardError => e
        data = {
          task: self.name,
          error: "#{e.class.name}: #{e.message}",
          backtrace: e.backtrace.join("\n")
        }
        formatted_message = Lograge.formatter.(data)
        Rails.logger.error formatted_message
        raise e
      end
    end
  end
end
~~~
And revise to call `bin/rake xxx:yyy` instead of `bundle exec rake xxx:yyy` in docker entry-point shell script and CloudWatch Event rules.

[rails db:migrate vs rake db:migrate](https://stackoverflow.com/questions/38403533/rails-dbmigrate-vs-rake-dbmigrate)

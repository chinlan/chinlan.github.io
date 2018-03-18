---
layout: post
title: "Cron job: whenever gem"
description: "Using whenever gem to implement cron job"
categories: [rails]
tags: [deploy, cron-job, scheduler]
redirect_from:
  - /2016/12/01/
---

# Add gem
~~~
gem ’whenever’, :require => false
~~~
run in terminal
~~~
wheneverize .
~~~
to generate schedule.rb.
The setting of cron job can be written in this file.
ex:
~~~ ruby
set :output, ’log/whenever.log’     ＃可以印出結果，偵錯
set :environment, ’production’
# Learn more: http://github.com/javan/whenever
#job_type :rake,    "cd :path && :environment_variable=:environment bundle exec rake :task --silent :output"
#job_type :runner,  "cd :path && bin/rails runner -e :environment ’:task’ :output"
job_type :runner, "cd :path && /opt/rbenv/shims/bundle exec bin/rails runner -e :environment ’:task’ :output"   ＃可以另外設定執行路徑，如果在production有設定rbenv之類的

every 5.minutes do
  runner "Blacklist.check_expire"
  runner "PointHistory.check_enable"
  runner "PointHistory.check_expire"
  runner "CourseSection.check_start"
  runner "CourseSection.check_end"
  runner "CourseSection.check_sale_start"
  runner "CourseSection.check_sale_end"
  runner "CourseTicket.check_sale_start"
  runner "CourseTicket.check_sale_end"
  runner "Promotion.check_start"
  runner "Promotion.check_end"
end
every 1.minute do
  runner "Promotion.print_test"
end
~~~

設定local的cron job
~~~
whenever --update-crontab --set environment=’development’
~~~

# 與capistrano的連用
capfile
~~~
require "whenever/capistrano"
~~~

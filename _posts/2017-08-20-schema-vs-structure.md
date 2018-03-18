---
layout: post
title: "Rails: schema.rb vs structure.sql"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2017/08/20/
---

schema.rb => ruby
~~~
rake db:schema:load
~~~
structure.sql => sql
~~~
rake db:structure:load
~~~

或者兩個都可以使用的
~~~
rake db:setup
~~~

如果有用到比較特別的資料庫功能像是view, trigger之類的才需要用到structure.sql。

要改成用structure.sql，在application.rb內做以下設定：
~~~ ruby
config.active_record.schema_format = :sql # default is :ruby
~~~


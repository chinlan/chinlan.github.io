---
layout: post
title: "PostgreSQL: update the value of enum type"
description: ""
categories: [rails]
tags: [postgresql]
redirect_from:
  - /2018/09/14/
---
~~~ruby
class AlterReportContentTypeEnum < ActiveRecord::Migration[5.2]
  disable_ddl_transaction!

  def up
  execute "ALTER TYPE report_content_type_enum ADD VALUE IF NOT EXISTS 'message';"
  end

  def down
  execute <<-SQL
  ALTER TABLE reports
  ALTER COLUMN content_type TYPE text
  USING content_type::text;

  DELETE FROM reports WHERE content_type = 'message';

  DROP TYPE report_content_type_enum;

  CREATE TYPE report_content_type_enum AS ENUM (
  'activity', 'community', 'topic', 'event', 'have'
  );

  ALTER TABLE reports
  ALTER COLUMN content_type TYPE report_content_type_enum
  USING content_type::report_content_type_enum;
  SQL
  end
end
~~~

注意update enum無法使用ddl transaction，要disable掉

還有可以添加卻無法remove某個value，如果要去掉某個value的話，需要迂迴的重建一個新的enum type，就像上面那樣，先把欄位指到別的型別（text之類），然後把enum整個drop掉，再重建，最後再把欄位指到重建的這個enum type。

[ref](https://dev.to/amplifr/postgres-enums-with-rails-4ld0)

[ref](https://medium.com/pretto/update-postgresql-enums-with-rails-migrations-ceb8a86d8cee)

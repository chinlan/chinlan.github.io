---
layout: post
title: "Rails: return rows with 0 count when doing group by query with PostgreSQL"
description: ""
categories: [rails]
tags: [postgresql]
redirect_from:
  - /2018/12/18/
---

使用情境：

如果group by Post的created_at，然後照月份排序

~~~ruby
posts =
  Post.order(:created_at).group_by{|t| t.created_at.beginning_of_month}.map do |date, monthly_activities|
      # 處理成想要的資料格式
    end
~~~

那沒有post的那個月份就不會被return，這在前端應用上比較不好處理，因此希望能夠將沒有post的月份也return，但是各項資料的值為0。

預想了幾種處理方式：

Solution 1:

找到起始月和結束月 （Find start month and end month）
~~~Ruby
t1 = posts.first.date
t2 = posts.last.date
~~~
建立從起始月至結束月的一個陣列（crate an array start from start month end at end month）
~~~ruby
arr = (t1.to_date..t2.to_date).select{|d| d.day == 1}
~~~
然後迴圈比較（looping compare）
~~~ruby
posts.map do |p|
  #something
end
~~~

Solution 2:
建立一個新的Hash，各個月的default value是0，然後再merge query的結果

因為query的最終結果是array所以這個解法看來也不適用

Solution 3:

看有沒有辦法在group_by的時候處理：在group_by之前用LEFT OUTER JOIN而不是INNER JOIN
（但因為此query完全不使用join，不是要把完全沒有子關聯的record也return，而是自身table的欄位值的問題，因此此解法看來不適用）

[ref1](https://stackoverflow.com/questions/346132/postgres-how-to-return-rows-with-0-count-for-missing-data)

[ref2](https://stackoverflow.com/questions/19093487/ruby-create-range-of-dates)

[ref3](https://stackoverflow.com/questions/1724639/iterate-every-month-with-date-objects)

[ref4](https://stackoverflow.com/questions/5631141/rails-activerecord-group-by-time-but-pad-dates-with-nil-values)

[ref5](https://stackoverflow.com/questions/882070/sorting-an-array-of-objects-in-ruby-by-object-attribute)

最終使用解法：
~~~ruby
def assign_0_to_none_records_month
  total_months_range.each do |date|
    next if exist_months_range.include?(date)
    @posts.push(
    MonthlyStatistics.new(Time.zone.at(date.to_datetime).beginning_of_month, [])
    )
  end
  @posts.sort_by!(&:date)
end


def total_months_range
  from = @posts.first.date
  to = @posts.last.date
  (from.to_date..to.to_date).select {|d| d.day == 1 }
end

def exist_months_range
  @posts.map { |p| p.date.to_date }
end
~~~

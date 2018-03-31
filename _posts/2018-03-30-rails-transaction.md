---
layout: post
title: "Rails: Transaction"
description: ""
categories: [rails]
tags: []
redirect_from:
  - /2018/03/30/
---

在rails中，transaction可以用在class和instance層級：

~~~ ruby
Post.transaction do
  @post.likes.create!
  @post.increment!(:like_count)
  NotificationJob.new(........).perform
end

@post.transaction do
  @post.likes.create!
  @post.increment!(:like_count)
  NotificationJob.new(.......).perform
end
~~~

一個transaction內可以牽涉到很多model classes的action。

------------------------------
### Transaction & Callbacks

Rails的`save`和`destroy`本來就有包transaction，所以update單一record的時候是不需要再特別寫transaction的。
`after_save`會和save被包在同一個transaction裡，所以如果想要在這個transaction確認完成後才去觸發某些行為，要使用`after_commit`或`after_detroy`這些callbacks。
還有`after_rollback`，他會在save的transaction失敗時被回呼。

--------------
### Exception & Rollback

需注意：transaction的rollback會將records全部倒回到transaction開始前的狀態，但是rollback只能經由exception來觸發，所以如果transaction內的程式碼根本不會丟例外（exception)，這個transaction內有東西出錯時就不會rollback。
因此確保transaction內的各個邏輯環節有處理好例外是很重要的，否則就算寫了trnsaction也無法確保資料的完整性。

當例外發生時，transaction進行rollback，然後例外會在transaction外面引發，所以還要準備處理這些例外。

如果不使用例外又想讓transaction rollback，可以使用[ActiveRecord::Rollback](http://api.rubyonrails.org/classes/ActiveRecord/Rollback.html)，不會在外面引發，不用在transaction外面處理例外。

[with_transaction_returning_status](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/transactions.rb)

負責引發rollback的method。
[ref](http://vaidehijoshi.github.io/blog/2015/08/18/safer-sql-using-activerecord-transactions/)

~~~ ruby
User.transaction do
   # do something
end
rescue ActiveRecord::RecordInvalid => e
  e.record
end
~~~

------------------------------------

### nested transaction
~~~ ruby
Post.transaction do
  Post.create(post_params)
  Post.transaction do
    Post.create(post_params)
    raise ActiveRecord::Rollback
  end
end
~~~
須注意：即使在裡面那層transaction裡引發ActiveRecord::Rollback，外面那層的transaction並不會接收到這個例外，結果就是裡外兩則posts都成功建立。
如果要讓nested transaction可以確實rollback，要在裡面那層設定`requires_new: true`，這樣的話就會rollback裡面那一層，外面那一層不會rollback。
~~~ ruby
Post.transaction do
  Post.create(post_params)
  Post.transaction(requires_new: true) do
    Post.create(post_params)
    raise ActiveRecord::Rollback
  end
end
~~~
這樣的話就是外面那一層的post 建立成功，裡面的rollback ，結果成功建立外面那一篇post。
[doc](http://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html)

-----------------------------
### model or controller?

把transaction放在model較符合MVC。

當transaction中的create/update的params光靠default的strong_parameter不足以處理，還必須另外對controller的params去做轉換時，
為避免把controller的params直接丟到model去處理，
目前公司專案採取的做法是：
另外寫了PostParameter這樣的class去處理controller params，
transaction則放在model層。



